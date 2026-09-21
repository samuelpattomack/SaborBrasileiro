# Especificação de Requisitos — SaborBrasileiro

Levantamento e especificação dos requisitos do sistema, separados em **Requisitos Funcionais (RF)** — o que o sistema deve fazer — e **Requisitos Não Funcionais (RNF)** — como o sistema deve se comportar, classificados no modelo **FURPS+**. Este documento formaliza e detalha o que já havia sido introduzido na seção de FURPS+ do [Modelo de Caso de Uso](02-modelo-caso-uso.md).

## 1. Requisitos Funcionais (RF)

| Código | Descrição | Prioridade | Caso de uso relacionado |
|---|---|---|---|
| RF01 | O sistema deve permitir que Cliente, Cozinheiro e Entregador se cadastrem e façam login na plataforma | Essencial | UC01 |
| RF02 | O sistema deve permitir que o Cozinheiro cadastre, edite e remova pratos do seu cardápio, incluindo marcar disponibilidade | Essencial | UC02 |
| RF03 | O sistema deve permitir que o Cliente busque pratos e cozinheiros por região | Essencial | UC03 |
| RF04 | O sistema deve permitir que o Cliente monte e confirme um pedido a partir do cardápio de um cozinheiro | Essencial | UC04 |
| RF05 | O sistema deve permitir que o Cliente efetue o pagamento do pedido dentro do aplicativo (cartão ou Pix) | Essencial | UC05 |
| RF06 | O sistema deve permitir que o Cliente aplique um cupom de desconto válido a um pedido | Desejável | UC04 (extend) |
| RF07 | O sistema deve permitir que o Cozinheiro visualize, aceite ou recuse pedidos recebidos e atualize seu status de preparo | Essencial | UC06 |
| RF08 | O sistema deve permitir que o Entregador visualize entregas disponíveis, aceite uma corrida e atualize o status até a entrega | Essencial | UC07 |
| RF09 | O sistema deve permitir que o Cliente acompanhe o status do pedido em tempo real (recebido, em preparo, a caminho, entregue) | Importante | UC09 |
| RF10 | O sistema deve permitir que o Cliente avalie o prato e a entrega separadamente, após o pedido ser concluído | Importante | UC08 |

**Legenda de prioridade:** *Essencial* (indispensável para o funcionamento básico do marketplace), *Importante* (agrega valor relevante, mas o sistema funciona sem ela no piloto), *Desejável* (pode ficar para uma iteração futura sem comprometer o objetivo do projeto).

## 2. Requisitos Não Funcionais (RNF) — modelo FURPS+

### F — Functionality (funcionalidades e regras de negócio)
As funcionalidades já estão cobertas pelos RF acima. As regras de negócio associadas (RN01 a RN08) estão detalhadas na [Modelagem de Negócio](03-modelagem-negocio.md) e são o que garante que os RF sejam implementados de forma consistente (ex: RF05 só é considerado concluído se respeitar a RN01).

| Código | Requisito | Critério de aceite |
|---|---|---|
| RNF-F01 | O sistema deve impedir a confirmação de um pedido sem pagamento aprovado | Nenhum pedido deve existir com status diferente de "pendente" sem um registro de pagamento aprovado associado (RN01) |
| RNF-F02 | O sistema deve impedir que um cozinheiro visualize pedidos de outro cozinheiro | Requisições de listagem de pedidos devem ser filtradas pelo id do cozinheiro autenticado (RN03) |

### U — Usability (usabilidade)

| Código | Requisito | Critério de aceite |
|---|---|---|
| RNF-U01 | O cadastro de um novo prato pelo cozinheiro deve ser possível em no máximo 2 telas | Validado no protótipo: tela "Gerenciar cardápio" → formulário único de prato |
| RNF-U02 | Todos os elementos tocáveis (botões, cards) devem ter área mínima de toque de 44x44px | Aplicado ao design final das telas, a partir dos wireframes |
| RNF-U03 | O sistema deve funcionar com uso predominante em uma única mão (mobile-first) | Elementos de ação principal posicionados na metade inferior das telas |

### R — Reliability (confiabilidade)

| Código | Requisito | Critério de aceite |
|---|---|---|
| RNF-R01 | O status do pedido não pode ser perdido em caso de falha momentânea de conexão do entregador | Último status confirmado deve persistir no servidor e ser reenviado ao cliente ao reconectar |
| RNF-R02 | O sistema deve manter disponibilidade mínima de 99% durante o horário de funcionamento do piloto | Monitorado via uptime do serviço em produção (fora do escopo desta entrega acadêmica, mas registrado como meta) |

### P — Performance (desempenho)

| Código | Requisito | Critério de aceite |
|---|---|---|
| RNF-P01 | A busca de pratos por região deve retornar resultados em até 2 segundos | Testado com até 500 cozinheiros cadastrados na base |
| RNF-P02 | O sistema deve suportar ao menos 100 pedidos simultâneos na versão piloto | Testado por carga simulada na fase de implementação |

### S — Supportability (suporte/manutenção)

| Código | Requisito | Critério de aceite |
|---|---|---|
| RNF-S01 | O sistema deve permitir incluir uma nova região/bairro sem alteração estrutural do banco de dados ou do código | Região tratada como dado (tabela), não como valor fixo no código |
| RNF-S02 | O código deve seguir separação em camadas (apresentação, regras de negócio, dados) para facilitar manutenção futura | Validado na arquitetura da implementação |

### + — Restrições adicionais (Plus)

| Código | Requisito |
|---|---|
| RNF-PL01 | Aplicativo mobile-first, com prioridade de uso via smartphone |
| RNF-PL02 | Pagamento simulado nesta versão — sem integração com gateway de pagamento real |
| RNF-PL03 | Sem suporte multilíngue nesta versão (apenas português) |
| RNF-PL04 | Sem logística própria de entrega — depende de entregadores parceiros autônomos |

## 3. Matriz de rastreabilidade (RF ↔ Caso de Uso ↔ Regra de Negócio ↔ Tela)

| RF | Caso de uso | Regra de negócio | Tela do protótipo |
|---|---|---|---|
| RF01 | UC01 | — | [01-login](../prototipos/01-login.svg), [02-cadastro](../prototipos/02-cadastro.svg) |
| RF02 | UC02 | RN07 | [09-gerenciar-cardapio-cozinheiro](../prototipos/09-gerenciar-cardapio-cozinheiro.svg) |
| RF03 | UC03 | — | [03-home-busca-cliente](../prototipos/03-home-busca-cliente.svg) |
| RF04 | UC04 | RN08 | [04-cardapio-pedido-cliente](../prototipos/04-cardapio-pedido-cliente.svg) |
| RF05 | UC05 | RN01 | [05-confirmar-pagamento-cliente](../prototipos/05-confirmar-pagamento-cliente.svg) |
| RF06 | UC04 (extend) | RN02 | [05-confirmar-pagamento-cliente](../prototipos/05-confirmar-pagamento-cliente.svg) |
| RF07 | UC06 | RN03, RN04 | [08-painel-cozinheiro-pedidos](../prototipos/08-painel-cozinheiro-pedidos.svg) |
| RF08 | UC07 | RN05 | [10-painel-entregador-entregas](../prototipos/10-painel-entregador-entregas.svg) |
| RF09 | UC09 | — | [06-acompanhamento-pedido-cliente](../prototipos/06-acompanhamento-pedido-cliente.svg) |
| RF10 | UC08 | RN06 | [07-avaliar-pedido-cliente](../prototipos/07-avaliar-pedido-cliente.svg) |

Essa matriz mostra a consistência entre o problema de negócio (regras de negócio), a especificação (requisitos funcionais), o comportamento esperado (casos de uso) e a interface (telas do protótipo) — nenhum requisito fica "solto" sem uma tela ou regra correspondente, e vice-versa.

---

Ver também: [Cenário de Negócio](01-cenario-negocio-escopo.md), [Modelo de Caso de Uso](02-modelo-caso-uso.md), [Modelagem de Negócio](03-modelagem-negocio.md) e [Prototipação](04-prototipacao.md).
