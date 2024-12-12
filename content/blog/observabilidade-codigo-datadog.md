+++
title = "Observabilidade como Código: Criando monitores do Datadog por código com Terraform"
description = ""
date = 2024-12-09T18:31:15-03:00
weight = 20
draft = false
+++

No nosso papel como profissionais de DevOps e SRE, estamos constantemente criando soluções para automatizar tarefas e otimizar o trabalho dos outros. Mas e quando chega a hora de olhar para o nosso próprio trabalho? Automação não deveria começar por nós mesmos? Observabilidade como Código surge exatamente para isso: transformar a criação e manutenção de monitores em algo tão eficiente quanto as pipelines que gerenciamos. Se "tudo como código" é o norte, por que ainda criar monitores manualmente no painel do Datadog? A resposta é simples: não faz sentido.

Implementar Observabilidade como Código usando Terraform e Datadog traz uma série de benefícios que vão além da automação óbvia. Primeiro, tem a consistência: toda configuração de monitor passa a ser um arquivo versionado, revisto e auditado, em vez de depender de cliques no console que ninguém lembra quem fez. Segundo, a escalabilidade: adicionou um novo serviço? Clona um monitor padrão e, pronto, você já está rastreando pontos críticos sem precisar reinventar a roda.

E não é só sobre organização — é sobre agilidade e segurança. Com Observabilidade como Código, mudanças são planejadas, testadas e aplicadas como qualquer outro recurso na infraestrutura. Isso reduz drasticamente os famosos "erros humanos" e faz com que até mesmo times menores possam operar com a eficiência de gigantes.

Neste artigo, exploro como essa abordagem funciona na prática, usando Terraform para criar monitores no Datadog. Desde a configuração do ambiente até boas práticas para organização do código, passando por exemplos práticos e dicas para "sofrer" no próximo incidente em produção (ou melhor, preveni-lo).

Porque, no final das contas, observabilidade não é só sobre alertas. É sobre confiança: saber que sua infraestrutura está sob controle.

{{< figure src="/img/observabilidade-codigo.png" alt="observabilidade-codigo" width="800" class="center" >}}

---

## Configurando o Ambiente

Antes de começar, é essencial preparar o ambiente. Criar monitores como código sem uma configuração sólida é como tentar montar um quebra-cabeça sem as peças principais, o resultado nunca será completo.

### Ferramentas Necessárias

Você vai precisar de três coisas principais:

1. **Terraform**: A ferramenta que transforma infraestrutura em código, permitindo que tudo seja configurado, versionado e gerenciado de forma automatizada.
2. **Datadog API Key**: Para que o Terraform possa conversar com o Datadog.
3. **Acesso ao console do Datadog**: Só para conferir se tudo está no lugar (ou para aquele toque final, se precisar).

### Configurando o Terraform

Primeiro, instale o Terraform e adicione o provedor do Datadog no seu arquivo de configuração:

```hcl
terraform {
  required_providers {
    datadog = {
      source = "terraform-providers/datadog"
      version = "~> 3.49.0"
    }
  }
}

# Configurar o provider do Datadog
provider "datadog" {
  api_key = var.datadog_api_key
  app_key = var.datadog_app_key
}
```

### Tratando Segredos com Cuidados

Ninguém quer uma API key exposta por aí, certo? Use variáveis de ambiente ou ferramentas como o Vault para manter tudo seguro. Aqui vai uma dica rápida com variáveis de ambiente:

```bash
export TF_VAR_datadog_api_key="sua-chave-aqui"
export TF_VAR_datadog_app_key="sua-chave-aqui"
```

Agora é só rodar `terraform init` e deixar a mágica começar.

---

## Criando Monitores no Datadog com Terraform

Vamos ao que interessa: criar monitores no Datadog usando Terraform. Aqui, você vai ver como é possível deixar tudo automatizado, escalável e bonito no código.

### Criando um Monitor Básico

Quer começar com algo simples? Aqui vai um exemplo de monitor para rastrear a latência de uma aplicação:

```hcl
resource "datadog_monitor" "latencia_p90_exemplo" {  
  name                  = "SisExemplo com latência elevada"
  message               = "Alerta de latência alta no SisExemplo. @hangouts-Alertas"
  query                 = "percentile(last_10m):p90:trace.servlet.request{env:prod,service:sisExemplo} > 0.02"
  tags                  = ["service:sisExemplo","env:prod"]
  escalation_message    = ""
  evaluation_delay      = "0"
  include_tags          = "true"
  type                  = "query alert"

  monitor_thresholds {
    critical = "0.02"
    warning  = "0.025"
  }  
}
```

## Reutilizando Configurações

Quer ajustar os thresholds ou enviar notificações diferentes? Tudo é parametrizável. 

Precisa monitorar várias aplicações? Use módulos e variáveis para evitar repetir código. Aqui que entra o poder de usar código, um laço percorrendo um dicionário com as aplicações, thresholds personalizados e canais de notificações personalizados carregando tudo isso na criação dos monitores.


### Exemplo Prático: Monitorando a Latência de Aplicações

Vamos consolidar tudo com um caso real: criar monitores para rastrear a latência das aplicações da organização.

1. **Defina o objetivo**: Monitorar a latência p90 das aplicações em produção, com thresholds e canais de notificação personalizados.
2. **Escreva o código**:

```hcl
locals{
  # Dicionário com thresholds e notificações personalizadas
  apps_monitors = {
    "exemplo" = {"latencia" = {"critical" = "","warning" = ""}, "notificacao"= "HangoutsExemplo"},
    "exemplo1" = {"latencia" = {"critical" = "0.05","warning" = "0.08"}, "notificacao"= "HangoutsExemplo1"}
  }
}

resource "datadog_monitor" "latencia_p90" {
  for_each              = local.apps_monitors
  name                  = "${each.key} com latência elevada"
  message               = "Alerta de latência alta. @hangouts-DevOps-Alertas ${local.apps_definition[each.key].notificacao != "" ? local.apps_definition[each.key].notificacao : ""}"
  query                 = "percentile(${each.value.time_window == "" ? "last_10m" : each.value.time_window }):p90:${each.value.query == "" ? "trace.servlet.request" : each.value.query }{env:prod,service:${each.key} ${each.value.exclude!="" ? format(",%s", each.value.exclude) : ""}} > ${each.value.latencia.critical != "" ? each.value.latencia.critical : "0.02"}"
  tags                  = ["service:${each.key}", "env:prod"]
  escalation_message    = ""
  evaluation_delay      = "0"
  include_tags          = "true"
  type                  = "query alert"

  monitor_thresholds {
    critical = "${each.value.latencia.critical != "" ? each.value.latencia.critical : "0.025"}"
    warning  = "${each.value.latencia.warning != "" ? each.value.latencia.warning : "0.02"}"
  }
}
```

3. **Teste e aplique**:
   - Rode `terraform plan` para revisar o que será criado.
   - Depois, aplique com `terraform apply`.

4. **Valide no Datadog**: Confira o monitor criado no console e veja os alertas chegando quando o threshold for atingido.

---

## Boas Práticas para Observabilidade como Código

Agora que você já viu como criar monitores, é hora de ir além e seguir boas práticas que tornam sua infraestrutura mais robusta e fácil de gerenciar.

### Versionando o Código em Git

Manter o código de observabilidade versionado em um repositório Git é essencial para garantir rastreabilidade e colaboração. Aqui estão algumas dicas:

1. **Revisões de Código**: Toda alteração deve passar por Pull Requests (PRs) com revisão do time. Isso ajuda a evitar configurações incorretas ou não testadas.
2. **Tags e Commits Semânticos**: Use mensagens de commit claras e, se possível, adote tags para marcar versões estáveis do código.

### Testes e Validação

Antes de aplicar suas configurações em produção, valide tudo em um ambiente de staging. Evite aquele momento de "Ops, não era isso que eu queria!".

### Automatize os Deploys

Integre o Terraform com seu pipeline CI/CD. Assim, toda alteração nos monitores passa por revisão de código e é aplicada automaticamente (quem sabe outro post sobre isso?).

---

## Desafios e Dicas Extras

Nem tudo são flores. Aqui vão alguns desafios comuns e como enfrentá-los:

- **Excesso de alertas**: Evite configurar thresholds muito baixos. Ajuste com base no comportamento real da aplicação.
- **Limitações no Terraform**: Algumas features novas do Datadog podem levar tempo para serem suportadas no provedor do Terraform. Mantenha a documentação oficial sempre por perto.
- **Custo do Datadog**: Configure apenas monitores essenciais e use tags para focar nos serviços mais críticos.

---

## Conclusão

Observabilidade como Código é uma mudança de jogo. Não só facilita a criação e gestão de monitores, mas também eleva a maturidade do time na operação de sistemas. Com o uso do Terraform e do Datadog, você ganha consistência, controle e escala.

Essa abordagem de automação vai além da criação de monitores. É possível utilizá-la para configurar dashboards, gerenciar equipes, definir SLOs, entre outras tarefas. E o conceito não se limita ao Datadog e Terraform: ferramentas como Ansible, OpenTofu e outras soluções de observabilidade, como Dynatrace e Grafana, podem ser integradas, desde que ofereçam APIs e, no caso do Terraform, providers compatíveis.

Agora é com você: pegue esses exemplos, adapte às suas necessidades e comece a transformar a forma como sua equipe monitora sistemas.
