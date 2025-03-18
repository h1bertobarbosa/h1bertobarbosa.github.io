---
layout: post
author: humberto_barbosa
title: "O que acontece quando não tem testes?"
date: 2025-03-17 21:19:08 -0300
categories: [desenvolvimento de software, metodologias ágeis, engenharia de software, práticas de TI, gestão de projetos]
tags: [testes, desenvolvimento ágil, metodologia XP, práticas de programação, software colaborativo]
---

Quem nunca ficou na dúvida se o código vai rodar depois de uma mudança? E ai, como vocês lidam com isso?

## O que acontece quando não tem testes?

Cada deploy é uma torcida. Você muda uma linha e fica esperando pra ver se algo quebrou. Feedback lento, retrabalho.

Já passou um tempão preenchendo um formulário só pra testar um fluxo? Já quebrou algo numa parte do sistema que _aparentemente_ não tinha nada a ver? (eu já, várias vezes).

Isso gera medo de mexer no código. E medo de mexer no código significa evolução mais lenta. Agora, quando você tem testes rodando pra vários cenários, qual o motivo pra temer?

Testes não garantem que nunca vai ter um bug. Mas garantem que você tem mais controle sobre o código quando precisar mexer.

## O que realmente é um teste automatizado?

É basicamente definir entradas e verificar se a saída tá batendo com a expectativa.

- **Given:** preparo do cenário
- **When:** execução do comportamento
- **Then:** validação do resultado

E se os testes forem lentos e dependentes entre si? Aí já começou errado. Eles devem rodar rápido e de forma independente.

E o famoso _"não tenho tempo pra testar"_? Mas tem tempo pra corrigir bug em produção, responder cliente puto, testar na mão, lidar com código legado que ninguém quer mexer, treinar dev novo… Pois é.
