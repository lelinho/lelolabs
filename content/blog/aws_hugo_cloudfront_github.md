+++
title = " Cloud + DevOps para o blog"
description = "Criação do Blog em Arquivos Estáticos e Hospedagem Gratuita (ou quase) na Amazon"
date = 2021-03-22T16:24:27-03:00
weight = 20
draft = false
+++


Este artigo tem como objetivo descrever os passos para a publicação deste blog utilizando diversos serviços da AWS e algumas técnicas de entrega contínua. Mais do que apenas manter o blog, este projeto se tornou um exercício prático dos conceitos que estudo e, muitas vezes, não tenho (tinha) a chance de aplicar no dia a dia.

O framework escolhido foi o Hugo, um gerador de páginas estáticas que facilita a criação e publicação de sites. Diferente de plataformas como o WordPress, onde há a necessidade de um banco de dados e um servidor dinâmico, o Hugo gera todo o conteúdo do site de forma estática, que pode ser facilmente hospedado em um servidor de páginas estáticas. Isso significa que, a cada atualização no conteúdo, o site deve ser gerado novamente e publicado.

A estrutura final utiliza o Hugo para criar as páginas e a AWS para hospedar o conteúdo, garantindo desempenho e custo zero com os serviços certos. A configuração é simples, e a cada alteração no código, o site é automaticamente atualizado e distribuído através da AWS usando uma pipeline de entrega contínua.

**Stack utilizada:** Hugo + Amazon CloudFront + S3 Bucket + GitHub Actions

## Passo a Passo
- **Criação do site com Hugo:** O primeiro passo foi configurar o Hugo localmente para desenvolver o conteúdo do blog. Com sua estrutura simples, é fácil criar posts e definir temas.
- **Publicação no Amazon S3:** Após gerar o conteúdo estático com o Hugo, ele é armazenado em um bucket S3, um serviço de armazenamento de objetos da Amazon. Esse bucket serve como o repositório de todas as páginas e arquivos estáticos que compõem o site.
- **Distribuição pelo Amazon CloudFront:** Para garantir que o conteúdo do blog seja carregado de forma rápida, utilizo o Amazon CloudFront, um serviço de CDN (Content Delivery Network). Ele garante que o conteúdo seja distribuído globalmente, oferecendo baixas latências e maior desempenho, independente de onde o visitante esteja acessando.
- **Automação com GitHub Actions:** Para facilitar o processo de publicação, criei uma pipeline com GitHub Actions. A cada mudança no repositório, a pipeline automaticamente dispara a regeneração do site e publica os arquivos atualizados no S3. Com isso, o processo de publicação se torna transparente e eficiente, seguindo os princípios de entrega contínua.

## Conclusão
Utilizar a combinação de Hugo e AWS para criar e manter o blog foi uma experiência gratificante. Não só permitiu a construção de um blog leve, rápido e sem custos, mas também foi uma excelente oportunidade para praticar conceitos de DevOps no mundo real. A automação do fluxo de publicação com GitHub Actions simplificou a manutenção do site, eliminando o trabalho manual e permitindo que o foco ficasse na criação de conteúdo. Além disso, a utilização do Amazon S3 e CloudFront mostrou-se uma solução eficaz para hospedagem de páginas estáticas, garantindo performance e escalabilidade sem complicações.

Essa experiência reforça a ideia de que, com as ferramentas certas, é possível desenvolver projetos pessoais de forma prática e profissional, aplicando boas práticas de DevOps e utilizando a nuvem para obter ótimos resultados.

Se você também está interessado em experimentar um projeto prático de DevOps, sugiro explorar essa combinação de ferramentas e serviços da AWS. O investimento em conhecimento é sempre recompensador, e começar com um projeto pessoal é uma ótima maneira de aprender.

