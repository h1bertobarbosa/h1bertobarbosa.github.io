---
layout: post
author: humberto_barbosa
title: "Como Desacoplar um Serviço Externo do Seu Código (Sem Sofrer no Futuro)"
date: 2025-03-14 20:52:08 -0300
categories: ["Desenvolvimento de Software", "Arquitetura de Software", "Boas Práticas"]
tags: ["Desacoplamento", "Arquitetura Hexagonal", "Ports and Adapters", "SOLID", "DIP", "Domain-Driven Design", "Anti-Corruption Layer", "Event-Driven", "Resiliência", "Microservices"]
---

Manter um sistema desacoplado de serviços externos não é frescura, **é necessidade**. Se você quer um código mais flexível, testável e resiliente, precisa seguir algumas boas práticas. Vamos direto ao ponto:

## **1. Use um Gateway para Intermediar o Acesso**

Um **Gateway** é basicamente um intermediário entre o seu código e o serviço externo. Ou seja, em vez de chamar diretamente a API do serviço, seu código conversa com um **Gateway**, que gerencia essa comunicação.

🔹 O que isso resolve?
✅ Você pode **trocar o serviço externo sem quebrar seu código**.
✅ Facilita a implementação de **cache, logs e validações**.
✅ Reduz a dependência direta da aplicação no serviço externo.

No livro _Patterns of Enterprise Application Architecture_ reforça que eles são essenciais para encapsular o acesso a sistemas externos.

## **2. Aplique o Princípio da Inversão de Dependência (DIP)**

O DIP, um dos princípios SOLID, diz que **o código deve depender de abstrações, não de implementações concretas**.

🔹 Como aplicar?
✅ Crie **interfaces** para os serviços externos e injete implementações via dependência.
✅ Evite acoplar sua aplicação diretamente a APIs externas.
✅ Facilita testes unitários, já que você pode **mockar a interface em vez de bater na API real**.

Exemplo prático: imagine que sua aplicação depende de uma api externa para fazer pagamentos. Em vez de chamar uma classe especifica que encapsula a lib do serviço de pagamento por exemplo: pagarmeService.pay()`, você pode criar uma interface `PaymentService` e injetar diferentes implementações (`PaymentServicePagarme`, `PaymentServicePagseguro`, etc.).

## **3. Arquitetura Hexagonal (Ports and Adapters) ou clean arch**

A arquitetura **Ports and Adapters** resolve um problema clássico: **separar a lógica de negócio da infraestrutura**.

🔹 Como funciona?
✅ **Portas** (Ports) → Interfaces que sua aplicação expõe ou consome.
✅ **Adaptadores** (Adapters) → Traduções entre sua aplicação e serviços externos.

Exemplo prático:

- Sua aplicação precisa enviar um email? Você define uma interface `EmailService`.
- Depois, pode implementar um `SendGridEmailService`, desacoplando sua aplicação do serviço especifico.

Isso significa que, no futuro, **se precisar trocar o sendgrid pelo SES, por exemplo, o impacto no código será mínimo**.

## **4. Utilize Eventos de Domínio (E faça as coisas de forma assíncrona)**

Se você ainda acha que precisa chamar serviços externos **de forma síncrona o tempo todo**, está perdendo uma grande oportunidade de melhorar a escalabilidade e resiliência do seu sistema.

🔹 O que fazer?
✅ **Publicar eventos em vez de chamar APIs diretamente.**
✅ Deixe outro serviço consumir o evento e interagir com a API externa.
✅ Se o serviço externo estiver fora do ar, o evento pode ser processado depois.

Isso reduz o **acoplamento temporal**, ou seja, **sua aplicação não precisa esperar uma resposta imediata do serviço externo para continuar funcionando**.

## **5. Mantenha o Domínio Isolado**

🔹 O que isso significa na prática?
✅ Seu **modelo de domínio** não deve ter **nada** relacionado a serviços externos.
✅ Qualquer integração deve estar nas **camadas externas da arquitetura**.

Se seu domínio está cheio de chamadas para APIs de terceiros, **você está cometendo um erro grave de design**. Uma boa prática é garantir que **a camada de domínio não tenha imports de outras camadas**, mantendo-a **pura e independente**.

---

## **Anti-Corruption Layer (ACL): Blindando Seu Domínio Contra "Poluição Externa"**

Quando dois sistemas se comunicam, existe um risco de que **um modelo de dados acabe "contaminando" o outro**. Para evitar isso, usamos a **Anti-Corruption Layer (ACL)**.

🔹 O que a ACL faz?
✅ **Traduz e adapta os dados** entre sistemas diferentes.
✅ Evita que regras de negócio de um serviço externo afetem seu domínio.
✅ Permite agregar dados de múltiplas fontes sem bagunçar sua aplicação.

Alistair Cockburn até critica o nome "Anti-Corruption Layer" (acha que parece negativo), mas defende a ideia: **é uma camada intermediária que traduz a comunicação entre seu sistema e um serviço externo para evitar que as regras externas "corrompam" seu domínio.**

**Exemplo prático:**
Se sua aplicação precisa se integrar a um ERP antigo, você não deve fazer chamadas diretas a ele. Em vez disso, cria-se uma **ACL** que traduz os dados do ERP para um formato que sua aplicação entende, na pratica você vai usar o pattern adapter.

---

## **Contextos Delimitados: Entenda Onde Cada Conceito Faz Sentido**

No **Domain-Driven Design (DDD)**, um **Contexto Delimitado** define **onde um conceito faz sentido dentro do seu sistema**.

🔹 O que isso resolve?
✅ Evita que um termo tenha significados diferentes dentro do mesmo sistema.
✅ Ajuda na organização modular do código.
✅ Facilita a modelagem de serviços independentes.

**Exemplo prático:**
O que significa "cliente"? No setor financeiro de um hospital, pode significar um paciente pagante. Já no setor de internação, "cliente" pode ser **qualquer paciente**. **O contexto muda completamente!**

Cada **contexto delimitado** pode ter seu próprio modelo de dados, regras e até sua própria equipe de desenvolvimento.

**Dica:**
Contextos bem definidos **não significam automaticamente microserviços**, mas podem ajudar na **modularização e escalabilidade** do sistema.

---

## **As 4 Perguntas que Todo Arquiteto de Software Deve Fazer**

Ao avaliar uma proposta de design, um arquiteto de software precisa responder 4 perguntas essenciais:

1️⃣ **Essa solução atende aos objetivos do negócio?**
- Se não atende, não faz sentido implementá-la.
2️⃣ **A solução respeita as restrições do projeto?**
- Não adianta propor algo que não cabe dentro das limitações do sistema.
3️⃣ **Ela cumpre os atributos de qualidade?**
- Performance, escalabilidade, segurança, etc.
4️⃣ **Existe uma alternativa mais barata ou menos arriscada?**
- Se existir, **volte para a prancheta** e refaça o planejamento.

💡 Se a resposta para a quarta pergunta for "sim", é melhor reconsiderar a solução. Um bom arquiteto não busca apenas a "melhor" solução, mas sim a **solução mais eficiente dentro do contexto do negócio**.

---

## **Conclusão**

Se você quer um sistema desacoplado de verdade, precisa aplicar boas práticas de arquitetura.

🔹 **Use Gateways** para encapsular integrações externas.
🔹 **Aplique DIP** para evitar dependências diretas.
🔹 **Adote Ports and Adapters** para separar seu domínio da infraestrutura.
🔹 **Use eventos assíncronos** para melhorar escalabilidade e resiliência.
🔹 **Proteja seu domínio com ACL** e **defina contextos delimitados no DDD**.

O objetivo final é simples: **um código flexível, fácil de manter e preparado para mudanças**. Se seu sistema ainda está acoplado diretamente a serviços externos, **você está deixando um problema enorme para o futuro**.

Agora, bora refatorar esse código! 🚀


## Links
[Hexagonal Architecture (Ports and Adapters) with Alistair Cockburn // Live #98](https://www.youtube.com/watch?v=AOIWUPjal60&list=TLGGMaacBIriC9UxNDAzMjAyNQ)
[SOLID DIP Dependency Inversion Principle + Design Patterns Adapter](https://www.youtube.com/watch?list=TLGGm5-xFN3ig2wxNDAzMjAyNQ&v=9JBU-Fh3bz4)
[Como criar e desacoplar camadas usando SOLID](https://www.youtube.com/watch?list=TLGGcj37oEUQjXkxNDAzMjAyNQ&v=tetVY6jmlnM)
[10 perguntas sobre Domain-Driven Design](https://www.youtube.com/watch?list=TLGG9YslJm1WW2UxNDAzMjAyNQ&v=6jHgd1LY6IE)
[Domain Driven Design do Jeito Certo](https://www.youtube.com/watch?list=TLGGr7bjDWPIC1MxNDAzMjAyNQ&v=cz6EU7Z_BhE)
[DDD, Domain-Driven Design com Elemar Jr. // Live #52](https://www.youtube.com/watch?list=TLGGpFdV-xSBKOExNDAzMjAyNQ&v=NsQnmmIykoE)
[Event-Driven Architecture, Microservices e CQRS: A combinação perfeita](https://www.youtube.com/watch?list=TLGG5KI3m6k5uYoxNDAzMjAyNQ&v=2LQkKU3dPGo)
