+++
title = "CloudNativePG e a Evolução dos Bancos de Dados em Containers"
description = ""
date = 2025-03-10T18:31:15-03:00
weight = 20
draft = false
+++


# CloudNativePG e a Evolução dos Bancos de Dados em Containers

Há alguns anos, eu era cético quanto à ideia de rodar bancos de dados dentro de containers. Acreditava que esse ambiente não ofereceria a robustez e a confiabilidade necessárias para aplicações críticas. Hoje, com a maturidade das tecnologias e o avanço dos operadores modernos, minha visão mudou consideravelmente.

{{< figure src="/img/db-k8s.webp" alt="banco de dados em kubernetes" width="600" class="center" >}}


## Uma Visão que Evoluiu com o Tempo

Inicialmente, minha principal preocupação era a complexidade e os riscos envolvidos em containerizar bancos de dados. A instabilidade, a dificuldade de persistência dos dados e a complexidade na manutenção me levavam a evitar essa abordagem. No entanto, ao acompanhar de perto o desenvolvimento de soluções como o [CloudNative-pg](https://cloudnative-pg.io){:target="_blank"}, percebi que muitas dessas barreiras foram derrubadas.

{{< figure src="/img/cloudnativepg-logo.svg" alt="cloudnativepg-logo" width="200" class="center" >}}

A experiência ao longo dos últimos meses e anos mostrou que, com o uso de operadores inteligentes, é possível garantir alta disponibilidade, replicação robusta e backups eficientes – inclusive com a possibilidade de point-in-time recovery. Essa evolução não só facilita o gerenciamento dos bancos de dados, mas também torna o ambiente muito mais resiliente e escalável.

## Benefícios dos Operadores Modernos

O CloudNativePG representa uma mudança de paradigma ao permitir a gestão declarativa dos bancos de dados em clusters Kubernetes. Alguns dos principais benefícios que os operadores modernos entregam são:

- **Alta Disponibilidade e Replicação:** Através de mecanismos automáticos de failover e replicação, garante-se que a aplicação continue operando mesmo diante de falhas.
- **Backup e Point-in-time Recovery:** Recursos que, historicamente, eram complicados de se implementar, agora vêm integrados de forma simplificada, proporcionando segurança e confiabilidade.
- **Infraestrutura Declarativa:** Ao adotar a prática de definir a infraestrutura como código, combinada com estratégias GitOps, ganhamos clareza no processo de deploy e na rastreabilidade de mudanças.

Essas melhorias demonstram que as dificuldades tradicionais da containerização de bancos de dados podem ser mitigadas, transformando a operação em um processo mais ágil e confiável.

## Exemplo: Banco de Dados PostgreSQL com CloudNativePG

Para demonstrar a simplicidade e o poder dos operadores modernos, vamos criar um banco de dados PostgreSQL utilizando o CloudNativePG com duas réplicas.

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: meu-banco
spec:
  instances: 3  # Um líder e duas réplicas
  primaryUpdateStrategy: unsupervised
  storage:
    size: 10Gi
```

Esse manifest declara um cluster PostgreSQL com três instâncias: uma principal e duas réplicas. O operador gerencia automaticamente a replicação e o failover.

## Evoluindo o Setup: Backup para S3

Agora, vamos adicionar uma configuração de backup armazenando os dados em um bucket S3. Isso garante que possamos recuperar os dados mesmo em caso de perda total do cluster.

```yaml
backup:
  target: "s3://meu-bucket-backup/postgresql/"
  retentionPolicy: "30d"
  s3:
    endpoint: "s3.amazonaws.com"
    region: "us-east-1"
    accessKey: "MEU_ACCESS_KEY"
    secretKey: "MEU_SECRET_KEY"
```

Com essa configuração, os backups serão armazenados no S3 e mantidos por 30 dias.

## Nem Sempre a Solução Ideal para Todas as Organizações

Apesar dos avanços, é fundamental reconhecer que essa abordagem não é para qualquer organização. Cada empresa deve avaliar seu contexto, complexidade e necessidades específicas. Organizações com estruturas menos maduras ou que não possuam equipes especializadas podem enfrentar desafios na transição para um ambiente 100% containerizado.

Portanto, a adoção do CloudNativePG e de operadores modernos deve ser feita de forma consciente, considerando tanto os benefícios quanto as limitações, para que se evite sobrecarregar times e infraestruturas que ainda não estão preparados para essa transformação.

Uma boa abordagem para começar essa jornada é implantar primeiro os bancos de dados de ferramentas de apoio às aplicações, como sistemas de monitoramento, rastreamento de logs ou serviços internos. Isso permite que os times adquiram maturidade e experiência com bancos de dados rodando em Kubernetes antes de migrar cargas mais críticas.

## A Evolução dos Times de Banco de Dados

Outro ponto importante é a necessidade de evolução dos times de banco de dados. Para aproveitar plenamente os benefícios dos operadores modernos, os profissionais precisam ampliar seus conhecimentos, adotando práticas e ferramentas que acompanhem essa transformação digital.

A integração entre equipes de desenvolvimento, operações e banco de dados é essencial para criar ambientes mais dinâmicos e resilientes, onde a inovação tecnológica se traduza em ganhos reais de performance e segurança.

Nos últimos anos, vimos o surgimento de novos papéis, como o **Database Reliability Engineer (DBRE)**, que traz para o mundo dos bancos de dados práticas inspiradas no SRE. Mas será que podemos ir além? Uma abordagem **DevDBAOps**, unindo as melhores práticas do DevOps, SRE e administração de bancos de dados, pode ser o próximo passo na evolução dos times. Essa cultura visa quebrar ainda mais os silos organizacionais, incentivando colaboração contínua entre DBAs, engenheiros de infraestrutura e desenvolvedores.

Se os times de banco de dados não acompanharem essa mudança e insistirem em processos manuais e isolados, correm o risco de se tornarem um gargalo na organização. A questão que fica é: **o seu time de banco de dados está preparado para essa transformação?**

## Conclusão

Ao olhar para trás, a mudança de postura em relação à containerização de bancos de dados reflete o quanto o mercado e a tecnologia evoluíram. Hoje, com soluções como o CloudNativePG, podemos aproveitar uma série de vantagens que antes pareciam impossíveis ou muito complicadas de implementar.

No entanto, a transição deve ser feita com cautela, considerando as particularidades de cada organização e a necessidade de um time preparado para abraçar essa nova era. Assim, a modernização dos bancos de dados não é apenas uma tendência, mas uma jornada de evolução contínua que demanda clareza, simplicidade e, acima de tudo, coragem para inovar.
