+++
title = "Configurações iniciais e de segurança em um servidor Ubuntu 18.04"
description = "Configurações iniciais e de segurança em um servidor Ubuntu 18.04"
date = 2020-02-28T15:43:28-03:00
weight = 20
draft = false
+++


Instalar um novo servidor requer alguns cuidados iniciais que devem ser tomados na questão de segurança e configurações de localização. Nesse tutorial vou passar alguns passos que realizo quando instalo um novo servidor com Ubuntu (a versão utilizada era a mais atual 18.04).

## Locales e timezone
O primeiro passo é configurar os Locales e timezone para o novo servidor:

{{< highlight bash >}}
dpkg-reconfigure locales
timedatectl set-timezone America/Sao_Paulo
{{< / highlight >}}

O primeiro comando você tem a opção de instalar novos locales, e setar o padrão que deseja utilizar. O segundo define o fuso horário que no meu caso é America/Sao_Paulo.



## Hardening
Já de cara alterar senhas do usuário criado na instalação e senha do root do Ubuntu. Uso senhas geradas automaticamente pelo 1Password e guardo por lá mesmo.

O próximo passo é adicionar minha chave pública, como chave autorizada para o usuário root do servidor. Deve ser inserido no arquivo:

{{< highlight bash >}}
nano ~/.ssh/authorized_keys
{{< /highlight >}}

Algumas alterações que realizo na configuração do servidor SSH. **/etc/ssh/sshd_config**

**PermitRootLogin prohibit-password** – Acesso do root não pode ser feito por senha

**PasswordAuthentication No** – Não permitir o acesso por senha

**MaxAuthTries 6** – Limitar tentativas de autenticação

## Firewall
O serviço que utilizo para firewall é o ufw, que utiliza uma sintaxe simples como interface para automaticamente criar regras do IPTABLES. Alguns exemplos de regras criadas:

{{< highlight bash >}}
ufw allow ssh
ufw allow http
ufw allow https
ufw allow 1234/tcp
ufw enable
{{< /highlight >}}

## Fail2ban
O Fail2ban é um serviço que bane tentativas de autenticação repetidas por IP. Esse banimento é feito lendo logs de tentativa de autenticação e criando regras no firewall quanto se ultrapassa as tentativas sem sucesso configurada.

{{< highlight bash >}}
apt install fail2ban
nano /etc/fail2ban/jail.d/sshd.conf
{{< /highlight >}}

O primeiro comando instala o fail2ban e o segundo criamos um arquivo de texto para cuidar das tentativas de autenticação por SSH no servidor. Inserimos o seguinte conteúdo:

{{< highlight bash >}}
[sshd]
enable = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
{{< /highlight >}}

Em seguida reiniciamos o serviço e alguns comandos para verificarmos o status do serviço:

{{< highlight bash >}}
systemctl restart fail2ban
fail2ban-client status
fail2ban-client status sshd
{{< /highlight >}}

## Conclusões
Depois desses passos você tem um servidor pronto pra começar a ser configurado para rodar algum serviço. Já tem acesso por SSH com autenticação por chaves, um firewall configurado e o fail2ban banindo tentativas de ataques.
