+++
title = " Cloud + DevOps pro blog"
description = "Criação do blog em arquivos estáticos e hospedagem de graça na Amazon"
date = 2021-03-22T16:24:27-03:00
weight = 20
draft = true
+++


Esse artigo tem como objetivo descrever os passos para a publicação deste blog utilizando diversos serviços da AWS e algumas técnicas de entrega continua para a publicação. Uma experiencia que vai além de manter o blog, se tornou um exercício prático de conceitos que estudamos e que por vezes não temos chances de utilizar.

O framework utilizado foi o Hugo, que tem como característica ser um gerador de páginas estáticas para publicação de sites. Diferente de Wordpress e outros que estamos habituados, com o Hugo, desenvolvemos o site, ele gera todo o conteúdo e esse deve ser publicado em um servidor de páginas estáticas, não há a necessidade de um Banco de Dados. A cada atualização, esse site deve ser gerado novamente e publicado.


Hugo + Amazon CloudFront + Bucket S3 + Github Actions

