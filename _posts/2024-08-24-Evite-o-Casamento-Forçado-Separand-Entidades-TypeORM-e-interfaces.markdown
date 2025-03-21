---
layout: post
author: humberto_barbosa
title:  "Evite o Casamento Forçado: Separando Entidades TypeORM de Interfaces"
date:   2024-08-24 13:25:14 -0300
categories: ["Desenvolvimento de Software", "Arquitetura de Software", "Boas Práticas"]
tags: ["TypeORM", "Padrão Repository", "Desacoplamento", "DDD", "Clean Architecture", "Hexagonal Architecture", "Interfaces", "Persistência de Dados", "DAO"]
---

# Evite o Casamento Forçado: Separando Entidades TypeORM e Interfaces

## Resumo

Este artigo explora os desafios e considerações ao remover entidades TypeORM de interfaces, destacando problemas de acoplamento e a violação do princípio Open-Closed. A proposta sugere a adoção do padrão Repository com entidades de domínio para desacoplar a lógica de negócio da persistência, melhorando a flexibilidade e a manutenção do código.

## Motivação

O uso direto de entidades TypeORM em interfaces pode resultar em um acoplamento forte entre a camada de persistência e as abstrações de negócio. Essa abordagem dificulta a manutenção e a evolução do sistema. Portanto, este artigo propõe alternativas para mitigar esses problemas, garantindo que o código permaneça flexível e aderente aos princípios de design de software.

## Problemas Identificados

Uma interface que inclui uma entidade TypeORM cria uma dependência direta com o ORM, limitando a reutilização e a evolução do código. Os métodos de atualização parcial dos repositórios violam a camada de domínio, uma vez que a entidade pode encapsular regras de negócio. Isso pode levar a situações indesejadas, como a violação de regras de preenchimento de dados.
### Entidade do typeOrm
Entidade é uma classe que mapeia uma tabela de banco de dados (ou coleção ao usar o MongoDB). Você pode criar uma entidade definindo uma nova classe e marcando-a com @Entity()
```typescript
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm"

@Entity()
export class User {
    @PrimaryGeneratedColumn()
    id: number

    @Column()
    firstName: string

    @Column()
    lastName: string

    @Column()
    isActive: boolean
    @Column()
    activatedAt?: Date
}
```
### Exemplo de Interface Acoplada
```typescript
interface IUsersRepositoryInterface {
    findBy(where: FindOptionsWhere<User>): Promise<Transaction[]>;
    create(transaction: Partial<User>): Transaction;
    save(user: User): Promise<Transaction>;
    update(id: string, data: Partial<Transaction>): Promise<void>;
    delete(id: string): Promise<void>;
    updateWhere(where: FindOptionsWhere<User>, data: Partial<User>): Promise<void>;
}
```
Essa interface dificulta a extensão do código, pois define que a implementação deve ser com TypeORM, restringindo a flexibilidade.

## Design Proposto

1.  **Interfaces e Abstrações**:

    -   As interfaces devem definir contratos que abstraem a lógica de negócio, sem incluir detalhes específicos de implementação como as entidades do TypeORM.
2.  **Entidades TypeORM**:

    -   As entidades typeorm representam tabelas no banco de dados.
    -   Proposta: Não utilizar entidades TypeORM diretamente em interfaces; em vez disso, criar entidades de domínio.
3.  **Padrão Repository**:

    -   Os repositórios devem ser responsáveis pela persistência e recuperação de entidades, desacoplando a lógica de negócio da implementação do TypeORM.
    -   **Benefícios**: Facilita a testabilidade e a manutenção, permitindo mudanças na implementação de persistência sem afetar a lógica de negócio.

### Exemplo de Interface Desacoplada
```typescript
export interface ICardRepository {
findById(id: string): Promise<User>;
save(account: Card): Promise<User>;
}
```
### Exemplo de Entidade de Domínio
```typescript
export class User {
    constructor(
        private id: number,
        private firstName: CardStatus,
        private lastName: string,
        private isActive: boolean;
        private activatedAt?: Date
    ) {}

    activate() {
	this.isActive  =  true;
	this.activatedAt = new Date()
	}
	deactivate() {
		this.isActive  =  false;
	}
}
```
###  Exemplo de Implementação do Repositório
UserModel: Entidade do typeorm, representa uma tabela
User: Entidade de domínio representa um conceito do negocio, e uma abstração do negocio
```typescript
export class UserRepository implements IUserRepositoryInterface {
    constructor(
        @InjectRepository(UserModel, 'writer')
        private readonly userRepositoryWriter: Repository<CardModel>,
        @InjectRepository(UserModel, 'reader')
        private readonly userRepositoryReader: Repository<UserModel>,
    ) {}

    async save(inputCard: User): Promise<User> {
        // Lógica de persistência
    }

    async findById(id: number): Promise<User> {
        const userData =  await  this.userRepositoryReader.findById(id);
        if  (!userData)  {  throw  new  Error('User not found');  }
        return new User(
        userData.id,
        userData.firstName,
        userData.lastName,
        userData.isActive
        userData?.activatedAt
        );
    }
}
```
## Impacto

A adoção das práticas propostas garantirá que o sistema tenha uma camada de domínio bem definida e explícita. Quando essa camada não existe, as regras de negócio tendem a se espalhar em helpers, utils, acoplamento entre use cases e dificultando a manutenção e a evolução do sistema.

### Vantagens

-   Código mais modular e testável.
-   Maior flexibilidade para evoluções futuras.
-   Centralização de regras de negócio nas entidades.

### Desvantagens

-   Aumento da complexidade inicial pela criação de entidades e interfaces.
-   Investimento inicial que será recompensado a longo prazo.

## Padrão DAO

Para casos onde validações de negócio não são necessárias,por exemplo uma paginação, detalhes dos dados, count,exists, o padrão DAO pode ser utilizado. O DAO encapsula o acesso aos dados em uma camada separada não sendo necessário retornar a entidade de negocio com todos os objetos e regras.

### Exemplo de Implementação DAO
```typescript
export class UserDAO implements IUserDAO {
    constructor(
        @InjectRepository(UserModel, 'reader')
        private readonly userRepository: Repository<UserModel>,
    ) {}

    async exists(where: Partial<IUserConditions>): Promise<boolean> {
        return this.userRepository.exists({ where });
    }
     async findAll(where: Partial<IUserConditions>): Promise<UserDataInterface> {
        return this.userRepository.findAll({ where });
    }
}
```
## Conclusão

A implementação das práticas propostas neste artigo garantirá que o sistema permaneça flexível, modular e aderente aos princípios de DDD, hexagonal arch, clean arch que e a definição de camadas logicas e regra de dependência e entidades representando conceitos de negocio, facilitando a manutenção e evolução do código. A centralização de regras de negócio genéricas nas entidades e regras específicas nos casos de uso é fundamental para um desenvolvimento que visa a manutenibilidade.

## Referências

-   https://www.branas.io/blog/o-que-e-e-quando-devemos-utilizar-o-padrao-repository.html
- https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
-[Interfaces](https://dcm.ffclrp.usp.br/~evandro/ibm1030/topicos/topicos2.html#:~:text=Uma%20interface%20%C3%A9%20uma%20maneira,feita%20atrav%C3%A9s%20de%20seus%20tipos)
- https://medium.com/experiencecode/o-que-%C3%A9-uma-interface-em-programa%C3%A7%C3%A3o-edf40fa5c57d
- https://pt.wikipedia.org/wiki/Objeto_de_acesso_a_dados
- https://medium.com/@felipe.damasceno.b/padr%C3%B5es-de-projeto-e-o-data-access-object-dao-7d7e4818866c
- [https://github.com/rodrigobranas/cccat17_9/tree/master/backend](https://github.com/rodrigobranas/cccat17_9/tree/master/backend)
- [https://www.youtube.com/watch?v=t4o1tOV3q78](https://www.youtube.com/watch?v=t4o1tOV3q78)
- [https://www.youtube.com/live/s3QsigPsXKI?si=uOqFmJBXJlS3unNy](https://www.youtube.com/live/s3QsigPsXKI?si=uOqFmJBXJlS3unNy)
- https://typeorm.biunav.com/en/entities.html#what-is-entity
