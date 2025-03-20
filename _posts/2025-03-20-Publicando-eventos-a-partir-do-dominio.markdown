---
layout: post
author: humberto_barbosa
title: "Publicando eventos a partir do domínio - com NestJS"
date: 2025-03-20 11:20:08 -0300
categories: [Arquitetura de Software, NestJS, CQRS]
tags: [Domain Events, Event-Driven, NestJS, Microservices, Clean Architecture]
---

## 📌 Eventos de Domínio: O que são e por que usar?

Eventos de domínio registram fatos importantes dentro do sistema. Algo aconteceu – um usuário se cadastrou, uma compra foi feita, um status mudou. Ao capturar isso como um evento, mantemos um histórico claro, facilitando auditorias e análises. Além disso, deixamos o sistema preparado para reagir a mudanças sem bagunçar tudo.

## 🚀 Implementação na prática

Eventos precisam ser **imutáveis**. Criou? Não mexe mais. Isso garante integridade e rastreabilidade. Outra coisa: desacoplar ao máximo. Se um evento precisa notificar outra parte do sistema, nada de dependência direta. Deixa ele solto e deixa quem precisar ouvir.

Benefícios?

- **Regras de negócio explícitas** – Dá pra entender fácil o que está acontecendo.
- **Código modular e organizado** – Menos acoplamento, mais flexibilidade.
- **Consistência entre agregados** – Se algo muda, tudo que precisa ser atualizado é notificado automaticamente.

## 🎯 Quando usar eventos de domínio?

Sempre que precisar notificar outros contextos sem acoplamento. Exemplo clássico:

- Usuário se cadastra → Dispara evento → Outro serviço manda e-mail de boas-vindas.
- Pedido pago → Dispara evento → Estoque e faturamento são atualizados automaticamente.

Bora pro código.

Para fazer isso, vamos evitar expor o dominio a qualquer tipo de infraestrutura de mensagens, com isso evitamos acoplar nossa entidade explicitamente a um recurso externo.

## 📌 Cenário:

Temos um **Usuário** que se cadastra no sistema. Quando isso acontece, queremos:
✅ Enviar um e-mail de boas-vindas
✅ Registrar um log

### Criando o Evento de Domínio
Usamos **eventos de domínio** para desacoplar esses processos. Bora pro código! 🎯

```typescript
export class UsuarioCadastradoEvent {
  constructor(public readonly usuarioId: string, public readonly email: string) {}
}
```
Um evento simples com id e email do novo usuario
### Criando o Agregado Raiz

```typescript
export class Usuario {
  private domainEvents: any[] = [];

  constructor(public readonly id: string, public readonly nome: string, public readonly email: string) {
    this.adicionarEvento(new UsuarioCadastradoEvent(id, email));
  }

  private adicionarEvento(evento: any) {
    this.domainEvents.push(evento);
  }

  pullEvents(): any[] {
    const events = [...this.domainEvents];
    this.domainEvents = [];
    return events;
  }
}
```
O que rolou aqui?

1. Criamos um **Usuário**.
2. Assim que ele nasce, ele já dispara `UsuarioCadastradoEvent`.
3. `pullEvents()` pega os eventos pendentes e os limpa.
### Criando um Manipulador de Eventos (Event Handler)

```typescript
import { EventsHandler, IEventHandler } from '@nestjs/cqrs';
import { UsuarioCadastradoEvent } from '../events/usuario-cadastrado.event';

@EventsHandler(UsuarioCadastradoEvent)
export class EnviarEmailBoasVindasHandler implements IEventHandler<UsuarioCadastradoEvent> {
  handle(event: UsuarioCadastradoEvent) {
    console.log(`📧 Enviando e-mail de boas-vindas para ${event.email}...`);
  }
}
```
Quando `UsuarioCadastradoEvent` acontece, o **handler** entra em ação. Aqui ele só printa no console, mas poderia enviar e-mail, publicar no Kafka, chamar outro serviço, etc.

### Criando o Serviço de Usuário

O NestJS já fornece um **Event Bus** pronto via `@nestjs/cqrs`, então só precisamos usá-lo.

```typescript
import { Injectable } from '@nestjs/common';
import { EventBus } from '@nestjs/cqrs';
import { Usuario } from '../models/usuario.model';

@Injectable()
export class UsuarioService {
  constructor(private readonly eventBus: EventBus) {}

  criarUsuario(nome: string, email: string): Usuario {
    const usuario = new Usuario(crypto.randomUUID(), nome, email);
    usuario.pullEvents().forEach((event) => this.eventBus.publish(event));
    return usuario;
  }
}
```
1. Criamos o usuário.
2. Pegamos os eventos gerados.
3. Publicamos no **Event Bus do NestJS**.

Agora qualquer serviço que ouvir esse evento pode agir sem precisar mudar código aqui.

Lembre-se de registrar no modulo, algo assim:

```typescript
@Module({
  providers: [UsuarioService, EnviarEmailBoasVindasHandler],
})
```

e claro registrar o cqrs no modulo principal conforme a documentação.
Agora, sempre que um usuário se cadastrar, os eventos certos serão disparados. 🚀

### ✅ Recapitulando
- **Eventos de Domínio** organizam melhor as regras de negócio.
- **Código desacoplado** → Adiciona novos handlers sem mexer no código principal.
- **NestJS + CQRS** → Event Bus já pronto, sem precisar reinventar a roda.


### Links
- [martinfowler.com](https://martinfowler.com/eaaDev/DomainEvent.html)
- [fullcycle.com.br](https://fullcycle.com.br/principais-conceitos-sobre-domain-events/)
- [learn.microsoft.com](https://learn.microsoft.com/pt-br/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation)
- [docs.nestjs.com](https://docs.nestjs.com/recipes/cqrs#events)
