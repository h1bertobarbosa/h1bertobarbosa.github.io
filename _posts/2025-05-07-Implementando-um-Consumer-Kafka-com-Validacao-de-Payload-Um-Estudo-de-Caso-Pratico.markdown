---
layout: post
author: humberto_barbosa
title: "Implementando um Consumer Kafka com Validação de Payload - Um Estudo de Caso Prático"
date: 2025-05-07 09:24:00 -0300
categories: [Arquitetura de Software, Engenharia de Dados, Desenvolvimento Backend, Mensageria e Eventos, Boas Práticas de Programação, Testes de Software, Integrações com Sistemas Externos, Node.js, JavaScript]
tags: [Kafka, eventos assíncronos, sistemas distribuídos, integração de sistemas, Node.js, consumer Kafka, testes unitários, validação de payload, persistência de dados, arquitetura desacoplada, mensageria, microserviços, Jest, design de software, repositório, subentidades]
---

No universo dos sistemas distribuídos, a comunicação assíncrona via eventos é uma estratégia cada vez mais adotada para garantir escalabilidade e desacoplamento entre serviços. Recentemente, enfrentei o desafio de implementar um consumer Kafka responsável por processar eventos de atualização de entidades de negócio, com a necessidade de validar e persistir dados de integrações específicas. Neste artigo, compartilho a abordagem adotada, os desafios encontrados e exemplos de testes, utilizando nomes e estruturas genéricas para ilustrar o processo.

#### Contexto do Problema

Imagine um sistema que utiliza eventos Kafka para propagar alterações em entidades, como contas de clientes ou configurações de parceiros. Um dos tópicos, que chamaremos de `entity.update.event`, carrega informações sobre essas entidades, incluindo integrações com serviços externos. O objetivo era garantir que, ao receber um evento, o sistema validasse a presença e a estrutura de um objeto de integração (por exemplo, `externalService`) antes de salvar ou atualizar as configurações no banco de dados.

#### Desafios Enfrentados

- **Validação de Estrutura Dinâmica:** O payload pode ou não conter o objeto de integração, e sua ausência ou estrutura incorreta pode causar inconsistências.
- **Persistência Condicional:** Era necessário criar uma nova configuração caso a entidade não existisse, ou atualizar a existente, sempre preservando dados já salvos.
- **Manutenção de Subentidades:** As entidades podem ter múltiplas subentidades (como localizações), que também precisam ser atualizadas ou adicionadas sem sobrescrever informações antigas.
- **Testabilidade:** Garantir que todos os cenários fossem cobertos por testes unitários robustos.

#### Implementação

A solução foi implementada em um framework Node.js, utilizando o padrão Repository para persistência e Jest para testes. O serviço principal ficou responsável por toda a lógica de validação e persistência.

##### Estrutura do Payload (Exemplo Fictício)

O payload relevante para a integração segue a estrutura:
```typescript
{
  data: {
    settings: {
      externalService: {
        featureA: {
          enabled: boolean
        },
        featureB: {
          enabled: boolean
        }
      }
    }
  }
}
```

##### Lógica de Processamento

1. **Recebimento do Evento:** O consumer escuta o tópico `entity.update.event` e aciona o método de processamento.
2. **Busca de Configuração Existente:** O serviço tenta localizar uma configuração existente para a entidade.
3. **Validação dos Atributos de Integração:** Antes de salvar, verifica se o objeto `externalService` está presente e se possui os campos necessários.
4. **Persistência:** Se a entidade não existir, cria uma nova configuração; caso contrário, atualiza a existente, sempre preservando dados antigos e atualizando apenas o necessário.
5. **Atualização de Subentidades:** Ao receber informações de subentidades, o serviço adiciona ou atualiza a subentidade correspondente, sem sobrescrever as demais.

##### Exemplo de Teste Unitário

Os testes cobrem cenários como criação de nova configuração, atualização de configuração existente, manipulação de subentidades e tratamento de payloads incompletos. Veja um exemplo de teste para atualização de subentidade:
```typescript
it('deve atualizar a configuração com a nova subentidade quando a configuração existir', async() = >{
  repository.findOneBy.mockResolvedValue(mockExistingConfiguration);
  const input = {
    entityId: 'entity-123',
    subentityId: 'new-subentity-123',
    payload: mockPayload,
  };
  await service.saveSubentity(input);
  expect(repository.save).toHaveBeenCalledWith(expect.objectContaining({
    id: mockExistingConfiguration.id,
    entity: expect.objectContaining({
      subentities: expect.arrayContaining([expect.objectContaining({
        id: 'new-subentity-123',
        name: 'Test Subentity',
        active: true,
        customField: 'value',
      }), ]),
    }),
  }), );
});
```
##### Tratamento de Erros e Logs

Quando a configuração não é encontrada, o serviço registra um log informativo, facilitando o diagnóstico de problemas em produção.

#### Boas Práticas Adotadas

- **Validação Estrita de Payload:** Antes de qualquer persistência, o payload é validado para garantir integridade dos dados.
- **Testes Abrangentes:** Todos os fluxos principais e alternativos são cobertos por testes unitários.
- **Logs Informativos:** Mensagens de log claras ajudam na manutenção e monitoramento do sistema.
- **Preservação de Dados:** Atualizações nunca sobrescrevem dados antigos sem necessidade.

#### Conclusão

A implementação de um consumer robusto para eventos Kafka exige atenção especial à validação de dados e à integridade das operações de persistência. No caso apresentado, a validação dos atributos de integração foi fundamental para evitar inconsistências e garantir a confiabilidade do sistema. Com testes bem estruturados e logs informativos, o resultado foi uma solução confiável e fácil de manter.

Se você também trabalha com eventos e integrações, recomendo sempre investir tempo em validação de payloads e cobertura de testes. Isso faz toda a diferença na qualidade e estabilidade do seu sistema!
