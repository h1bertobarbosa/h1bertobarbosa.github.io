---
layout: post
author: humberto_barbosa
title: "State Pattern na prática - pare de sofrer com lógicas de estado bagunçadas"
date: 2025-03-18 18:32:08 -0300
categories: [desenvolvimento de software, padrões de design, boas práticas, TypeScript]
tags: [state pattern, design patterns, TypeScript, boas práticas, código limpo]
---

Descobrir o **State Pattern** foi um ponto de virada. Esse padrão não é o mais falado, mas resolve um problema real: código inchado por condicionais monstruosas.

Sabe aquelas máquinas de estado cheias de `if` e `switch`? O código cresce, novos estados surgem e, de repente, qualquer alteração vira uma dor de cabeça. Já viu algo assim?

```typescript
switch (status)
	"active":
		console.log('ja esta ativo...')
		break
	"blocked":
		console.log('user bloqueado com sucesso')
		break
	"canceled":
		console.log('user cancelado com sucesso')
		break
```

No começo, parece tranquilo. Mas conforme o sistema evolui, o inferno começa: um mar de condicionais espalhadas pelo código. Manutenção? Um pesadelo.

## Solução

O **State Pattern** resolve isso. Em vez de espalhar lógica de estado, cada estado vira uma classe separada. O objeto principal apenas mantém uma referência ao estado atual e delega as decisões para ele.

Menos bagunça, mais organização. E quando precisar adicionar um novo estado? Só criar uma nova classe, sem tocar no código existente. Quer código flexível e fácil de manter? State Pattern é o caminho.

## Exemplo

Aqui está um exemplo de implementação do **State Pattern** em **TypeScript** para modelar o status de uma conta (**Account**), que pode estar em um dos seguintes estados:

- `Active`
- `WaitingActivation`
- `Blocked`
- `Cancelled`

Cada estado implementará um comportamento diferente para a conta. Vamos definir as classes de estado e a classe de contexto (`Account`), que gerencia o estado atual.

### 📌 **Passo 1: Definir a Interface do Estado**

Cada estado da conta precisa implementar essa interface para garantir que todos os estados tenham os mesmos métodos.

```typescript
interface AccountState {
  activate(): void;
  block(): void;
  cancel(): void;
  getStatus(): string;
}
```

### 📌 **Passo 2: Criar as Implementações dos Estados**

Cada estado terá uma implementação específica, alterando o comportamento da conta de acordo com a regra de negócio.

```typescript
class ActiveState implements AccountState {
  constructor(private account: Account) {}

  activate(): void {
    console.log("A conta já está ativa.");
  }

  block(): void {
    console.log("Bloqueando a conta...");
    this.account.setState(new BlockedState(this.account));
  }

  cancel(): void {
    console.log("Cancelando a conta...");
    this.account.setState(new CancelledState(this.account));
  }

  getStatus(): string {
    return "Active";
  }
}

class WaitingActivationState implements AccountState {
  constructor(private account: Account) {}

  activate(): void {
    console.log("Ativando a conta...");
    this.account.setState(new ActiveState(this.account));
  }

  block(): void {
    console.log("A conta ainda não está ativada e não pode ser bloqueada.");
  }

  cancel(): void {
    console.log("Cancelando a conta antes da ativação...");
    this.account.setState(new CancelledState(this.account));
  }

  getStatus(): string {
    return "WaitingActivation";
  }
}

class BlockedState implements AccountState {
  constructor(private account: Account) {}

  activate(): void {
    console.log("Não é possível ativar uma conta bloqueada. Contate o suporte.");
  }

  block(): void {
    console.log("A conta já está bloqueada.");
  }

  cancel(): void {
    console.log("Cancelando a conta bloqueada...");
    this.account.setState(new CancelledState(this.account));
  }

  getStatus(): string {
    return "Blocked";
  }
}

class CancelledState implements AccountState {
  constructor(private account: Account) {}

  activate(): void {
    console.log("Uma conta cancelada não pode ser reativada.");
  }

  block(): void {
    console.log("Uma conta cancelada não pode ser bloqueada.");
  }

  cancel(): void {
    console.log("A conta já está cancelada.");
  }

  getStatus(): string {
    return "Cancelled";
  }
}
```

### 📌 **Passo 3: Criar a Classe `Account`**

Essa classe é sua entidade e gerencia o estado atual da conta e delega chamadas para o estado correto.

```typescript
class Account {
  private state: AccountState;

  constructor() {
    // Estado inicial: aguardando ativação
    this.state = new WaitingActivationState(this);
  }

  setState(state: AccountState): void {
    this.state = state;
  }

  activate(): void {
    this.state.activate();
  }

  block(): void {
    this.state.block();
  }

  cancel(): void {
    this.state.cancel();
  }

  getStatus(): string {
    return this.state.getStatus();
  }
}
```

### 📌 **Passo 4: Testar a Implementação**

Agora, vamos criar uma conta e testar as transições de estado, a execução seria algo assim:

```typescript
const myAccount = new Account();
console.log("Status inicial:", myAccount.getStatus());

myAccount.activate(); // Deve ativar a conta
console.log("Status atual:", myAccount.getStatus());

myAccount.block(); // Deve bloquear a conta
console.log("Status atual:", myAccount.getStatus());

myAccount.cancel(); // Deve cancelar a conta
console.log("Status atual:", myAccount.getStatus());

myAccount.activate(); // Tentativa de ativar uma conta cancelada (deve falhar)
console.log("Status final:", myAccount.getStatus());
```

### 📌 **Explicação**

1. **Encapsulamento dos estados:** Cada estado implementa um comportamento específico e sabe para qual estado pode transitar.
2. **Mudança dinâmica de comportamento:** A classe `Account` delega a execução das ações ao estado atual.
3. **Evita `if-else` desnecessários:** O código fica mais organizado e escalável.

Isso garante uma implementação flexível e permite adicionar novos estados sem modificar a lógica central da `Account`. 🚀

### Conclusão

No State Pattern que a gente montou, vários princípios entraram em jogo sem a gente nem precisar pensar muito. Olha só:

Encapsulamento? Sim Cada estado tem sua própria lógica, sem espalhar código bagunçado por aí. O `Account` nem precisa saber como cada estado funciona, só delega e segue o jogo.

Responsabilidade Única? Sim Cada classe faz uma coisa só. O estado lida com as regras do status da conta, e o `Account` só troca de estado quando precisa. Sem mistura de responsabilidades.

Aberto/Fechado? Sim. Se amanhã precisar de um novo estado tipo `SuspendedState`, só criar e encaixar. Não precisa mexer no código velho e correr risco de quebrar tudo.

Substituição de Liskov? Sim. O `Account` nem sabe qual estado tá rodando. Ele só chama os métodos da interface e confia que o estado resolve. Pode trocar um pelo outro sem dor de cabeça.

Inversão de Dependência? Sim. O `Account` depende da abstração (`AccountState`), não de implementações específicas. Zero acoplamento, total flexibilidade.

No fim, isso aqui é código escalável, fácil de manter e sem aquele monte de `if` espalhado. Quer mudar um comportamento? Troca o estado e pronto. Isso é POO do jeito certo. 🚀

### Links
[https://refactoring.guru/pt-br/design-patterns/state](https://refactoring.guru/pt-br/design-patterns/state)
[https://elemarjr.com/clube-de-estudos/artigos/voce-precisa-conhecer-o-state-pattern/](https://elemarjr.com/clube-de-estudos/artigos/voce-precisa-conhecer-o-state-pattern/)
