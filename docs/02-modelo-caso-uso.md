# Modelo de Caso de Uso — SaborBrasileiro

## 1. Requisitos funcionais (visão geral)

Os requisitos completos (funcionais e não funcionais, classificados no modelo **FURPS+**, com critérios de aceite e rastreabilidade) estão detalhados no documento dedicado: [**05 — Especificação de Requisitos**](05-especificacao-requisitos.md). Abaixo, um resumo dos requisitos funcionais que dão origem diretamente aos casos de uso deste documento:

| Código | Requisito funcional | Ator principal |
|---|---|---|
| RF01 | Cadastrar-se na plataforma | Cliente / Cozinheiro / Entregador |
| RF02 | Gerenciar cardápio (cadastrar, editar, remover pratos, marcar disponibilidade) | Cozinheiro |
| RF03 | Buscar pratos e cozinheiros por região | Cliente |
| RF04 | Realizar pedido | Cliente |
| RF05 | Efetuar pagamento do pedido | Cliente |
| RF06 | Aplicar cupom de desconto | Cliente |
| RF07 | Gerenciar pedidos recebidos (aceitar/recusar, atualizar status de preparo) | Cozinheiro |
| RF08 | Aceitar e realizar entrega | Entregador |
| RF09 | Acompanhar status do pedido em tempo real | Cliente |
| RF10 | Avaliar pedido e entrega | Cliente |

---

## 2. Atores

| Ator | Descrição |
|---|---|
| **Cliente** | Usuário que busca, pede e paga por comida caseira |
| **Cozinheiro** | Usuário que cadastra cardápio e prepara os pedidos |
| **Entregador** | Usuário que aceita e realiza a entrega dos pedidos |


## 3. Casos de uso

| Código | Caso de uso | Ator(es) | Descrição resumida |
|---|---|---|---|
| UC01 | Cadastrar-se | Cliente, Cozinheiro, Entregador | Criar conta na plataforma informando dados básicos |
| UC02 | Gerenciar cardápio | Cozinheiro | Cadastrar, editar ou remover pratos oferecidos |
| UC03 | Buscar pratos | Cliente | Pesquisar pratos/cozinheiros disponíveis na região |
| UC04 | Realizar pedido | Cliente | Selecionar pratos e confirmar um pedido |
| UC05 | Efetuar pagamento | Cliente | Pagar pelo pedido dentro da plataforma |
| UC06 | Gerenciar pedidos recebidos | Cozinheiro | Aceitar/recusar pedido e atualizar status de preparo |
| UC07 | Realizar entrega | Entregador | Aceitar uma entrega disponível e atualizar seu status até a conclusão |
| UC08 | Avaliar pedido | Cliente | Avaliar o prato e a entrega após concluído |
| UC09 | Acompanhar pedido | Cliente | Visualizar em tempo real o status do pedido (recebido, em preparo, a caminho, entregue) |

### 3.1 Relacionamentos entre casos de uso

- **UC04 (Realizar pedido)** `<<include>>` **UC05 (Efetuar pagamento)** — todo pedido exige pagamento para ser confirmado;
- **UC04 (Realizar pedido)** pode ser estendido por **Aplicar cupom de desconto** (`<<extend>>`), quando o cliente possuir um cupom válido.


## 4. Diagrama de caso de uso (PlantUML)

O código-fonte do diagrama está em [`diagramas/diagrama-caso-uso.puml`](../diagramas/diagrama-caso-uso.puml). A imagem já renderizada está em [`diagramas/diagrama-caso-uso.png`](../diagramas/diagrama-caso-uso.png):

![Diagrama de Caso de Uso - SaborBrasileiro](../diagramas/diagrama-caso-uso.png)

Código-fonte:

```plantuml
@startuml SaborBrasileiro - Diagrama de Caso de Uso

left to right direction
skinparam packageStyle rectangle
skinparam nodesep 30
skinparam ranksep 60
scale 1.3

actor Cliente
actor Cozinheiro
actor Entregador

rectangle "SaborBrasileiro" {

  rectangle "Área do Cliente" {
    usecase "Cadastrar-se" as UC01c
    usecase "Buscar pratos" as UC03
    usecase "Realizar pedido" as UC04
    usecase "Efetuar pagamento" as UC05
    usecase "Aplicar cupom de desconto" as UC05b
    usecase "Acompanhar pedido" as UC09
    usecase "Avaliar pedido" as UC08
  }

  rectangle "Área do Cozinheiro" {
    usecase "Cadastrar-se " as UC01k
    usecase "Gerenciar cardápio" as UC02
    usecase "Gerenciar pedidos recebidos" as UC06
  }

  rectangle "Área do Entregador" {
    usecase "Cadastrar-se  " as UC01e
    usecase "Realizar entrega" as UC07
  }
}

Cliente --> UC01c
Cliente --> UC03
Cliente --> UC04
Cliente --> UC09
Cliente --> UC08
UC04 .> UC05 : <<include>>
UC05b .> UC04 : <<extend>>

Cozinheiro --> UC01k
Cozinheiro --> UC02
Cozinheiro --> UC06

Entregador --> UC01e
Entregador --> UC07

@enduml
```

## 5. Observações finais

Este modelo oferece a base funcional do SaborBrasileiro e servirá de ponto de partida para a etapa seguinte do projeto: a especificação detalhada de cada caso de uso (fluxo principal, fluxos alternativos e de exceção), que dará suporte ao projeto de software e à implementação.
