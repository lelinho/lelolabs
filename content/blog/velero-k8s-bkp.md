+++
title = "Backup de Clusters Kubernetes com Velero"
description = ""
date = 2025-04-19T11:37:39-03:00
weight = 20
draft = false
+++

Clusters Kubernetes gerenciam aplicações críticas em produção, e perder dados ou configurações pode ter um impacto severo no negócio. Apesar de o Kubernetes ser projetado para ser resiliente e escalável, ele não é imune a falhas humanas, problemas de infraestrutura ou falhas catastróficas. Portanto, um plano de backup confiável é essencial para garantir a continuidade das operações e evitar perdas de dados.

{{< figure src="/img/velero.png" alt="velero" width="600" class="center" >}}

## Apresentação do Velero

O Velero é uma ferramenta open-source desenvolvida pela VMware para backup e recuperação de clusters Kubernetes. Ele permite capturar snapshots do estado do cluster, incluindo volumes persistentes e recursos do Kubernetes, facilitando a restauração em casos de desastre ou migração entre ambientes.

### Recursos do Velero
- Backup e restauração de objetos do Kubernetes
- Snapshots de volumes persistentes
- Migração de clusters
- Retenção de dados baseada em políticas
- Suporte a múltiplos provedores de armazenamento, incluindo S3 e NFS

## Instalando o Velero

A instalação do Velero pode ser feita de diversas formas, mas o método mais comum é utilizando o CLI oficial. Para instalar:

1. Baixe o binário do Velero:
   ```sh
   curl -LO https://github.com/vmware-tanzu/velero/releases/latest/download/velero-linux-amd64.tar.gz
   tar -xvf velero-linux-amd64.tar.gz
   sudo mv velero /usr/local/bin/
   ```
2. Configure o backend de armazenamento (S3, por exemplo).
3. Instale o Velero no cluster:
   ```sh
   velero install --provider aws --bucket <NOME_DO_BUCKET> --backup-location-config region=<REGIAO>,s3ForcePathStyle=true,s3Url=<ENDPOINT_S3>
   ```

Isso instalará os componentes do Velero no namespace `velero` e configurará a conexão com o armazenamento S3.

## Executando o client do Velero em um container Docker

Para interagir com o Velero e criar backups de forma automatizada, podemos utilizar um container Docker com o client. Isso facilita a execução via pipelines de CI/CD. Aqui um exemplo de um Dockerfile com a instalação do Velero e do kubectl para interagir com o cluster Kubernetes:

```dockerfile
FROM velero/velero:latest as velero

FROM alpine:latest as kubectl

RUN apk add curl \
  && curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl" \
  && chmod +x ./kubectl

FROM alpine:latest

#No caso de execução local, para setar credenciais do k8s
#COPY meu_cluster.yaml /root/.kube/config

COPY --from=velero /velero /usr/local/bin/velero
COPY --from=kubectl ./kubectl /usr/local/bin/kubectl

```

## Backup de volumes com NFS (FileSystemBackup)

Para realizar backups de volumes usando o NFS Provisioner, utilizamos o recurso FileSystemBackup (FSB) do Velero.

Documentação: https://velero.io/docs/v1.15/file-system-backup/


### Instalação do Velero usando o FileSystemBackup

O Velero suporta múltiplos backends de armazenamento, incluindo o protocolo S3. Para configurar com o appliance NetApp StorageGRID (uma implementação compatível com S3), usamos um endpoint específico e credenciais configuradas via Secret.

O primeiro passo é criar um Secret com as credenciais de acesso ao StorageGRID. O arquivo `credentials-velero` deve conter as credenciais de acesso ao bucket S3. O formato do arquivo é o seguinte.

Arquivo `credentials-velero`:
```ini
[default]
aws_access_key_id=<CHAVE>
aws_secret_access_key=<SEGREDO>
```

```sh
kubectl create secret generic cloud-credentials --namespace velero --from-file=credentials-velero
```

Obs importante: É necessário manter o `velero-plugin-for-aws` na versão 1.9.0 para funcionar corretamente com os buckets do NetApp. Documentação da issue no GitHub: https://github.com/vmware-tanzu/velero/issues/8013

O comando seguinte instala o Velero com o plugin do AWS e configura o bucket S3 para armazenar os backups.

```sh
velero install \
  --provider aws \
  --use-node-agent  \
  --use-volume-snapshots=false  \
  --plugins velero/velero-plugin-for-aws:v1.9.0 \
  --bucket meu-bucket \
  --secret-file ./credentials-velero 
  --backup-location-config region=whatever,s3ForcePathStyle="true",s3Url=https://s3.meudominio.com
```

## Estratégias para backup de volumes

Existem duas abordagens principais para backup de volumes:

### Opt-in (especificar quais volumes serão incluídos no backup)

Exemplo:

```sh
#Anotar nos pods os volumes a serem incluídos no bkp
kubectl -n YOUR_POD_NAMESPACE annotate pod/YOUR_POD_NAME backup.velero.io/backup-volumes=YOUR_VOLUME_NAME_1,YOUR_VOLUME_NAME_2,...

#Criar o backup:
velero backup create BACKUP_NAME OTHER_OPTIONS
```

### Opt-out (especificar quais volumes serão excluídos do backup)

```sh
#Anotar nos pods os volumes que não serão incluídos no bkp
kubectl -n sample annotate pod/app1 backup.velero.io/backup-volumes-excludes=pvc1-vm

#OPT-OUT
velero backup create BACKUP_NAME --default-volumes-to-fs-backup OTHER_OPTIONS
```

## Agendamento de Backups

O Velero permite a criação de backups manuais e agendados. Para criar um backup manual:
```sh
velero backup create meu-backup --include-namespaces=meu-namespace
```

Para criar backups periódicos, utilizamos `schedules`:
```sh
velero schedule create backup-diario --schedule "0 2 * * *" --include-namespaces=meu-namespace
```

## Conclusão

O Velero é uma ferramenta essencial para qualquer equipe que opera clusters Kubernetes, oferecendo backup e recuperação eficientes. Sua integração com S3, incluindo implementações como o NetApp StorageGRID, proporciona flexibilidade para diferentes infraestruturas. Garantir backups regulares e testá-los periodicamente é uma prática fundamental para evitar surpresas em momentos críticos.

## Referências

- [Documentação oficial](https://velero.io/docs/v1.15/)
- [That DevOps Guy - vídeo cobrindo a instalação](https://www.youtube.com/watch?v=zybLTQER0yY&list=PLAY7vVhT05KhiKUC6OUh2gH94oXJmEyS1&index=18)
- [That DevOps Guy - github com arquivos de instalação](https://github.com/marcel-dempers/docker-development-youtube-series/tree/master/kubernetes/velero)
