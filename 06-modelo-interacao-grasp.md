# Modelos de Interação com GRASP — SaborBrasileiro

Este documento apresenta os **modelos de interação** (diagramas de sequência) do SaborBrasileiro e justifica, mensagem por mensagem, **por que cada responsabilidade foi atribuída a um objeto e não a outro**, usando os padrões **GRASP**. Ele parte dos [casos de uso](02-modelo-caso-uso.md), das [regras de negócio e do modelo de domínio](03-modelagem-negocio.md) e dos [requisitos](05-especificacao-requisitos.md) já definidos, e usa os [wireframes](04-prototipacao.md) como referência do que cada operação precisa entregar.

## 1. Escopo e classes de design

### Casos de uso modelados

| Diagrama | Casos de uso | Por que foi escolhido |
|---|---|---|
| [Diagrama 1 — Realizar pedido](#3-diagrama-1--realizar-pedido-uc04) | UC04 (com o `extend` de cupom) | É o fluxo central do marketplace e o de maior colaboração entre objetos |
| [Diagrama 2 — Efetuar pagamento](#4-diagrama-2--efetuar-pagamento-uc05) | UC05 (incluído em UC04) | Concentra a variação mais importante do sistema: as formas de pagamento |
| [Diagrama 3 — Aceitar pedido e entrega](#5-diagrama-3--aceitar-pedido-e-entrega-uc06-uc07-uc09) | UC06, UC07 e notificações do UC09 | Mostra a colaboração entre os três atores e o acompanhamento do pedido |
| [Diagrama 4 — Avaliar pedido](#6-diagrama-4--avaliar-pedido-uc08) | UC08 | Mostra criação de objetos e atualização de informações derivadas |

UC01 (Cadastrar-se), UC02 (Gerenciar cardápio) e UC03 (Buscar pratos) não foram modelados porque são, essencialmente, operações de cadastro, edição e consulta de um único objeto, sem colaboração relevante a justificar com GRASP.

### Classes de design

O [modelo de domínio](03-modelagem-negocio.md) representa o **negócio** (Cliente, Pedido, Prato...). Ao passar para o projeto de software, surgem classes que **não existem no negócio**, mas são necessárias para a solução. Elas aparecem nos diagramas com o estereótipo `«Controller»`, `«Service»` ou `«Abstract»`:

| Classe | Estereótipo | Papel |
|---|---|---|
| `ControladorPedido` | Controller | Recebe os eventos de UC04 e UC05 |
| `ControladorCozinha` | Controller | Recebe os eventos de UC06 |
| `ControladorEntrega` | Controller | Recebe os eventos de UC07 |
| `ControladorAvaliacao` | Controller | Recebe os eventos de UC08 |
| `ProcessadorPagamento` | Service | Isola a integração de pagamento |
| `MeioDePagamento` (`PagamentoCartao`, `PagamentoPix`) | Abstract | Abstração das formas de pagamento |
| `NotificadorStatus` | Service | Avisa o cliente sobre mudanças de status |

> **Nota de escopo:** a persistência (buscar `Prato` ou `CupomDesconto` no banco a partir de um identificador) foi propositalmente omitida dos diagramas, que recebem os objetos já resolvidos. Na implementação, isso seria feito por repositórios, outra aplicação de **Pure Fabrication**, que poluiriam os diagramas sem acrescentar ao raciocínio de responsabilidades.

## 2. Processo aplicado

O processo de 7 passos foi seguido para cada operação. Abaixo, a aplicação à operação principal, `realizarPedido`:

| Passo | Aplicação no SaborBrasileiro |
|---|---|
| 1. Identificar a operação de sistema | `realizarPedido(cozinheiro, itens, cupom)` — UC04, com UC05 incluído e cupom como `extend` |
| 2. Listar objetos participantes e quem tem a informação | `Pedido` (itens, desconto, total, status), `ItemPedido` (quantidade, preço unitário), `Prato` (preço, disponibilidade), `CupomDesconto` (validade, percentual) |
| 3. Information Expert na primeira mensagem | "Este prato está disponível?" → quem sabe é o `Prato` (mensagem 4); "qual o total?" → quem conhece itens e desconto é o `Pedido` (mensagem 10) |
| 4. Creator para cada instanciação | `Pedido` (pelo controlador), `ItemPedido` (pelo `Pedido`), `Pagamento` (pelo `Pedido`), `Avaliacao` (pelo `Pedido`) |
| 5. Controller como receptor do evento inicial | `ControladorPedido`, um controlador por caso de uso, nunca a tela nem uma entidade |
| 6. Revisar acoplamento e coesão | Ver [seção 7](#7-revisão-de-acoplamento-e-coesão) |
| 7. Polymorphism, Pure Fabrication e Indirection onde há variação | `MeioDePagamento` (formas de pagamento), `ProcessadorPagamento` (integração) e `NotificadorStatus` (avisos) |

---

## 3. Diagrama 1 — Realizar pedido (UC04)

**Operação de sistema:** `realizarPedido(cozinheiro, itens, cupom)`. **Telas de referência:** [04-cardapio-pedido-cliente](../prototipos/04-cardapio-pedido-cliente.svg) e [05-confirmar-pagamento-cliente](../prototipos/05-confirmar-pagamento-cliente.svg). **Regras envolvidas:** RN02, RN07, RN08.

![Diagrama de sequência — Realizar pedido](../diagramas/seq-01-realizar-pedido.png)

Fonte: [`diagramas/seq-01-realizar-pedido.puml`](../diagramas/seq-01-realizar-pedido.puml)

### Tabela de justificativas

| # | Mensagem | Emissor → Receptor | Padrão(ões) | Justificativa |
|---|---|---|---|---|
| 1 | `realizarPedido(cozinheiro, itens, cupom)` | `:Cliente` → `:ControladorPedido` | Controller | O evento de sistema "realizar pedido" deve ser recebido por um objeto que represente o caso de uso, não por um objeto de domínio (`Pedido`, `Cozinheiro`) nem pela interface gráfica. `ControladorPedido` cumpre esse papel e mantém a tela desacoplada do domínio. |
| 2 | `create(cliente, cozinheiro)` para `Pedido` | `:ControladorPedido` → `:Pedido` | Creator | O controlador coordena o fluxo e registra o pedido em andamento, além de possuir os dados de inicialização (cliente autenticado e cozinheiro escolhido). Por Creator, quem registra e tem os dados para inicializar é candidato a criar. |
| 3 | `adicionarItem(prato, qtd)` | `:ControladorPedido` → `:Pedido` | Information Expert + Creator | `Pedido` possui a coleção de itens e sabe mantê-la consistente (por exemplo, somar a quantidade se o prato já está no pedido). Além disso, agrega `ItemPedido`, o que o torna criador natural. Atribuir isso ao controlador o faria conhecer a estrutura interna do pedido. |
| 4 | `estaDisponivel()` | `:Pedido` → `:Prato` | Information Expert | A disponibilidade é definida pelo cozinheiro no próprio prato (RN07, tela [09](../prototipos/09-gerenciar-cardapio-cozinheiro.svg)). Só o `Prato` possui essa informação; consultar o atributo de fora quebraria o encapsulamento. |
| 5 | `create(prato, qtd)` para `ItemPedido` | `:Pedido` → `:ItemPedido` | Creator | No modelo de domínio, `Pedido` contém um ou mais `ItemPedido`. Pela regra de Creator, quem contém e agrega as instâncias é quem deve criá-las, e não o controlador. |
| 6 | `getPreco()` | `:ItemPedido` → `:Prato` | Information Expert | O preço unitário pertence ao `Prato`. `ItemPedido` o consulta uma vez, na criação, e guarda uma cópia em `precoUnitario`, de modo que alterações futuras de preço não mudem pedidos já feitos. |
| 7 | `aplicarCupom(cupom)` | `:ControladorPedido` → `:Pedido` | Information Expert + Low Coupling | `Pedido` conhece o subtotal e garante que há no máximo um cupom por pedido (RN02). O controlador apenas encaminha, sem conhecer como o desconto se relaciona com os itens. |
| 8 | `validar(dataAtual)` | `:Pedido` → `:CupomDesconto` | Information Expert | A data de validade é atributo do cupom, então só ele sabe dizer se está vencido (RN02). |
| 9 | `calcularDesconto(subtotal)` | `:Pedido` → `:CupomDesconto` | Information Expert | O percentual de desconto também é do cupom. `Pedido` não precisa conhecer a regra de cálculo, o que reduz acoplamento. |
| 10 | `calcularTotal()` | `:ControladorPedido` → `:Pedido` | Information Expert | `Pedido` conhece seus itens e o desconto aplicado, sendo o único com toda a informação necessária para o total (RN08). |
| 11 | `getSubtotal()` | `:Pedido` → `:ItemPedido` | Information Expert + High Cohesion | `ItemPedido` conhece quantidade e preço unitário. `Pedido` soma os subtotais em vez de abrir os detalhes de cada item, mantendo cada classe responsável pelo que conhece. |

### Decisões de design relevantes

**Por que o `ControladorPedido` cria o `Pedido` e não o `Cliente`?** O `Cliente` também seria candidato a criar, já que no modelo de domínio ele "realiza" pedidos. Porém, durante o fluxo quem possui os dados de inicialização (o cliente autenticado e o cozinheiro escolhido na tela) e coordena as etapas é o controlador. Deixar a criação no controlador evita que a entidade `Cliente` tenha de conhecer o fluxo de checkout, o que reduziria sua coesão (um cliente deve cuidar dos seus dados, não do processo de compra).

**Por que a pergunta "o prato está disponível?" vai para o `Prato` e não para o controlador ou para o `Cozinheiro`?** Porque a disponibilidade é um atributo do próprio prato, alterado pelo cozinheiro pelo toggle da tela de gerenciar cardápio (RN07). Se a pergunta fosse feita ao `Cozinheiro`, ele teria de manter uma lista de pratos indisponíveis, duplicando informação; se fosse resolvida no controlador, ele teria de ler o estado interno do `Prato`. Perguntar ao `Prato` é a aplicação direta de Information Expert e mantém a regra no lugar onde a informação nasce.

**Por que o `Pedido` calcula o total delegando a cada `ItemPedido`?** O total depende de três informações que estão em lugares diferentes: a quantidade e o preço (no `ItemPedido`), a lista de itens e o desconto (no `Pedido`). Cada objeto responde pelo que sabe: o item calcula seu subtotal (mensagem 11) e o pedido combina os subtotais com o desconto (mensagem 10). Isso evita que o controlador tenha de conhecer a estrutura interna do pedido e mantém alta coesão em `ItemPedido`.

---

## 4. Diagrama 2 — Efetuar pagamento (UC05)

**Operação de sistema:** `confirmarPagamento(pedido, meio)`. **Tela de referência:** [05-confirmar-pagamento-cliente](../prototipos/05-confirmar-pagamento-cliente.svg). **Regra envolvida:** RN01. **Requisitos:** RF05, RNF-PL02.

![Diagrama de sequência — Efetuar pagamento](../diagramas/seq-02-efetuar-pagamento.png)

Fonte: [`diagramas/seq-02-efetuar-pagamento.puml`](../diagramas/seq-02-efetuar-pagamento.puml)

### Tabela de justificativas

| # | Mensagem | Emissor → Receptor | Padrão(ões) | Justificativa |
|---|---|---|---|---|
| 1 | `confirmarPagamento(pedido, meio)` | `:Cliente` → `:ControladorPedido` | Controller | UC05 é incluído em UC04 (`include`), ou seja, faz parte do mesmo fluxo. O mesmo controlador continua coordenando o caso de uso, em vez de a tela falar diretamente com o domínio. |
| 2 | `getTotal()` | `:ControladorPedido` → `:Pedido` | Information Expert | `Pedido` já conhece o total calculado no diagrama 1. O controlador não recalcula nada. |
| 3 | `processar(total, meio)` | `:ControladorPedido` → `:ProcessadorPagamento` | Pure Fabrication + Low Coupling | Nenhum objeto de domínio (`Pedido`, `Cliente`, `Pagamento`) é um bom candidato para conversar com o gateway de pagamento: se fosse o `Pedido`, regras de negócio ficariam misturadas com integração externa (baixa coesão). Cria-se `ProcessadorPagamento` como serviço artificial que isola essa integração. |
| 4 | `pagar(valor)` | `:ProcessadorPagamento` → `:MeioDePagamento` | Polymorphism + Protected Variations | Cartão e Pix se comportam de forma diferente (RF05). Em vez de um `if/else` pela forma de pagamento, a mensagem `pagar()` é enviada à abstração `MeioDePagamento`, e cada subclasse (`PagamentoCartao`, `PagamentoPix`) implementa o seu comportamento. |
| 5 | `registrarPagamento(total, meio)` | `:ControladorPedido` → `:Pedido` | Information Expert | `Pedido` se relaciona com seu `Pagamento` (1 para 1 no modelo de domínio) e controla o próprio status. Só é chamado se o pagamento foi aprovado (RN01). |
| 6 | `create(valor, forma)` para `Pagamento` | `:Pedido` → `:Pagamento` | Creator | No modelo de domínio, o `Pedido` "gera" o `Pagamento`. Por registrar e usar diretamente essa instância, é o criador natural. |
| 7 | `confirmar()` | `:Pedido` → `:Pedido` | Information Expert + High Cohesion | A mudança de status do pedido para "confirmado" é regra do próprio `Pedido`. Mantê-la dentro dele (e não no controlador) preserva a coesão e garante que RN01 seja aplicada sempre do mesmo jeito. |

### Decisões de design relevantes

**Por que existe um `ProcessadorPagamento`, se não há nada parecido com ele no negócio?** Porque nenhum objeto do domínio é um bom candidato a falar com um meio de pagamento. Colocar essa responsabilidade no `Pedido` o faria depender de detalhes de integração (gateway, protocolos), misturando "o que é um pedido" com "como cobrar". Isso é exatamente a situação em que **Pure Fabrication** se aplica: cria-se uma classe que não representa um conceito do negócio, mas que preserva a coesão das classes de domínio. Como o pagamento é simulado nesta versão (RNF-PL02), essa classe também é o único ponto que mudaria quando um gateway real for integrado.

**Por que `pagar()` é enviado para `MeioDePagamento` e não para `PagamentoCartao` ou `PagamentoPix`?** O sistema oferece hoje Cartão e Pix (a opção "Dinheiro" foi retirada justamente por conflitar com RN01) e poderá oferecer outras formas no futuro. Se o processador testasse a forma escolhida com condicionais, cada nova forma de pagamento exigiria alterá-lo. Ao enviar a mensagem para a abstração (**Polymorphism**), o processador não precisa saber qual é a forma concreta, e uma nova forma vira apenas uma nova subclasse. A interface `pagar()` funciona como ponto estável que protege o resto do sistema das mudanças (**Protected Variations**).

**Por que a confirmação do pedido (`confirmar()`) fica dentro do `Pedido`?** RN01 diz que o pedido só é confirmado após o pagamento aprovado. Se essa regra estivesse no controlador, qualquer outro fluxo que precisasse confirmar um pedido teria de reimplementá-la. Dentro do `Pedido`, a regra tem um único dono, o objeto que possui o status.

---

## 5. Diagrama 3 — Aceitar pedido e entrega (UC06, UC07, UC09)

**Operações de sistema:** `aceitarPedido(idPedido)` e `aceitarEntrega(idPedido)`. **Telas de referência:** [08-painel-cozinheiro-pedidos](../prototipos/08-painel-cozinheiro-pedidos.svg), [10-painel-entregador-entregas](../prototipos/10-painel-entregador-entregas.svg) e [06-acompanhamento-pedido-cliente](../prototipos/06-acompanhamento-pedido-cliente.svg). **Regras envolvidas:** RN03, RN04, RN05. **Requisitos:** RF07, RF08, RF09.

![Diagrama de sequência — Aceitar pedido e entrega](../diagramas/seq-03-aceitar-pedido-e-entrega.png)

Fonte: [`diagramas/seq-03-aceitar-pedido-e-entrega.puml`](../diagramas/seq-03-aceitar-pedido-e-entrega.puml)

### Tabela de justificativas

| # | Mensagem | Emissor → Receptor | Padrão(ões) | Justificativa |
|---|---|---|---|---|
| 1 | `aceitarPedido(idPedido)` | `:Cozinheiro` → `:ControladorCozinha` | Controller | Evento de sistema do UC06. Há um controlador próprio para esse caso de uso, separado do `ControladorPedido`, porque atende outro ator e outra responsabilidade. |
| 2 | `pertenceA(cozinheiro)` | `:ControladorCozinha` → `:Pedido` | Information Expert | `Pedido` sabe a qual cozinheiro está destinado. Isso garante a RN03 (um cozinheiro só gerencia pedidos destinados a ele) sem o controlador consultar atributos internos. |
| 3 | `aceitar()` | `:ControladorCozinha` → `:Pedido` | Information Expert + High Cohesion | A mudança do status para "em preparo" (RN04) é regra do `Pedido`, que é dono do próprio estado. O controlador só dispara a ação. |
| 4 | `notificarMudanca(pedido)` | `:Pedido` → `:NotificadorStatus` | Indirection + Low Coupling | `Pedido` não deve saber como nem para quem avisar a mudança de status. O `NotificadorStatus` é o intermediário que desacopla o pedido do mecanismo de aviso. |
| 5 | `enviar(cliente, "em preparo")` | `:NotificadorStatus` → `:Cliente` | Pure Fabrication + Protected Variations | `NotificadorStatus` é uma classe artificial que concentra a entrega do aviso. O canal (notificação no app, por exemplo) pode variar sem afetar o `Pedido`, e o cliente vê o status atualizado no stepper da tela 06. |
| 6 | `aceitarEntrega(idPedido)` | `:Entregador` → `:ControladorEntrega` | Controller | Evento de sistema do UC07, com controlador próprio. Misturá-lo ao `ControladorCozinha` reuniria responsabilidades de atores diferentes. |
| 7 | `podeAceitarEntrega()` | `:ControladorEntrega` → `:Entregador` | Information Expert | Só o `Entregador` sabe se já está em uma entrega (RN05: um entregador por entrega em andamento). |
| 8 | `atribuirEntregador(entregador)` | `:ControladorEntrega` → `:Pedido` | Information Expert | No modelo de domínio, `Pedido` "é entregue por" um `Entregador`. O vínculo e a mudança de status para "a caminho" ficam no `Pedido`, que guarda essa informação. |
| 9 | `notificarMudanca(pedido)` | `:Pedido` → `:NotificadorStatus` | Indirection | Mesmo mecanismo da mensagem 4, reaproveitado. O pedido avisa qualquer mudança de status do mesmo jeito. |

### Decisões de design relevantes

**Por que há controladores separados para cozinheiro, entregador e cliente, em vez de um único "ControladorGeral"?** O padrão Controller pede um objeto que represente o caso de uso ou o sistema, mas um controlador único para tudo acumularia as responsabilidades dos três atores, o que viola **High Cohesion**: ele cresceria a cada funcionalidade e teria motivos demais para mudar. Com um controlador por caso de uso, cada um fica pequeno, com poucas dependências, e uma alteração no fluxo do entregador não afeta o do cozinheiro.

**Por que existe um `NotificadorStatus` entre o `Pedido` e o `Cliente`?** O requisito RF09 pede que o cliente acompanhe o status do pedido em tempo real. Se o `Pedido` avisasse o cliente diretamente, ele passaria a depender de detalhes de comunicação (tela, push, tempo real) que não têm relação com sua responsabilidade. O `NotificadorStatus` é a **Indirection**: um intermediário que recebe o aviso e o encaminha, desacoplando quem muda o status de quem o exibe. Como o mesmo notificador atende as duas mudanças (mensagens 4 e 9), qualquer novo status futuro também passa por ele.

**Por que a pergunta "o entregador pode aceitar?" vai para o `Entregador`?** Porque a informação "estou em uma entrega" pertence a ele (atributo `disponivel` no modelo de domínio). Fazer essa checagem no controlador exigiria que ele soubesse como o estado do entregador é representado, e fazer no `Pedido` o faria depender de dados que não são dele. Aplicado Information Expert, a regra RN05 fica com o objeto que a conhece.

---

## 6. Diagrama 4 — Avaliar pedido (UC08)

**Operação de sistema:** `avaliarPedido(idPedido, notaPrato, notaEntrega, comentario)`. **Telas de referência:** [07-avaliar-pedido-cliente](../prototipos/07-avaliar-pedido-cliente.svg) e [03-home-busca-cliente](../prototipos/03-home-busca-cliente.svg) (onde a nota média aparece). **Regra envolvida:** RN06. **Requisito:** RF10.

![Diagrama de sequência — Avaliar pedido](../diagramas/seq-04-avaliar-pedido.png)

Fonte: [`diagramas/seq-04-avaliar-pedido.puml`](../diagramas/seq-04-avaliar-pedido.puml)

### Tabela de justificativas

| # | Mensagem | Emissor → Receptor | Padrão(ões) | Justificativa |
|---|---|---|---|---|
| 1 | `avaliarPedido(idPedido, notaPrato, notaEntrega, comentario)` | `:Cliente` → `:ControladorAvaliacao` | Controller | Evento de sistema do UC08, recebido por um controlador do caso de uso, e não pela tela nem por `Pedido` ou `Avaliacao`. |
| 2 | `podeSerAvaliado()` | `:ControladorAvaliacao` → `:Pedido` | Information Expert | Só o `Pedido` sabe se já foi entregue. É assim que a RN06 (avaliar apenas pedidos entregues) é garantida pelo objeto que conhece o status. |
| 3 | `registrarAvaliacao(notaPrato, notaEntrega, comentario)` | `:ControladorAvaliacao` → `:Pedido` | Information Expert + Creator | No modelo de domínio, `Pedido` "recebe" a `Avaliacao`. Ele é o ponto de ligação e controla que o pedido tenha uma única avaliação. |
| 4 | `create(notaPrato, notaEntrega, comentario)` para `Avaliacao` | `:Pedido` → `:Avaliacao` | Creator | `Pedido` registra e usa diretamente a `Avaliacao`, portanto é quem a instancia. |
| 5 | `atualizarMedia(notaPrato)` | `:Pedido` → `:Cozinheiro` | Information Expert | `Cozinheiro` conhece suas avaliações anteriores, logo é o único capaz de recalcular sua nota média, a mesma exibida na tela de busca. |
| 6 | `atualizarMedia(notaEntrega)` | `:Pedido` → `:Entregador` | Information Expert | Mesma lógica: cada ator mantém a própria nota média. |

### Decisões de design relevantes

**Por que o `Pedido` cria a `Avaliacao` e não o `Cliente`, que é quem avalia?** Em orientação a objetos, quem *executa* a ação no mundo real nem sempre é quem deve *criar* o objeto. Aqui a `Avaliacao` só existe em função de um pedido específico (e uma só por pedido), e o `Pedido` é quem mantém essa associação e conhece o seu status. Fazer o `Cliente` criar a avaliação o obrigaria a conhecer o status do pedido, acoplando duas classes sem necessidade.

**Por que a média é atualizada pelo `Cozinheiro` e pelo `Entregador`, e não calculada pela `Avaliacao`?** Cada `Avaliacao` conhece apenas a própria nota. A média de um cozinheiro depende de todas as avaliações que ele já recebeu, informação que só ele mantém. As notas de prato e de entrega são enviadas separadamente (mensagens 5 e 6) porque pertencem a atores diferentes, o que evita que uma entrega ruim penalize o cozinheiro (e vice-versa), como já definido na tela de avaliação.

---

## 7. Revisão de acoplamento e coesão

Com os diagramas prontos, cada classe foi revisada quanto a quantas dependências possui (acoplamento) e se suas responsabilidades têm relação entre si (coesão):

| Classe | Responsabilidades | Colabora com | Avaliação |
|---|---|---|---|
| `ControladorPedido` | Coordenar UC04 e UC05 | `Pedido`, `ProcessadorPagamento` | **Baixo acoplamento:** só conhece duas classes. Não depende de `ItemPedido`, `Prato`, `CupomDesconto` nem `Pagamento`. |
| `Pedido` | Itens, desconto, total, status, pagamento, avaliação e entregador do pedido | `ItemPedido`, `Prato`, `CupomDesconto`, `Pagamento`, `Avaliacao`, `NotificadorStatus`, `Cozinheiro`, `Entregador` | Maior acoplamento do modelo, **justificado**: é o objeto central do negócio. Coesão preservada porque tudo gira em torno do ciclo de vida do pedido. O risco de virar uma classe "faz-tudo" é contido delegando cálculos a `ItemPedido` e `CupomDesconto`, cobrança a `ProcessadorPagamento` e avisos a `NotificadorStatus`. |
| `ItemPedido` | Quantidade, preço unitário e subtotal | `Prato` | Alta coesão, acoplamento mínimo. |
| `Prato` | Dados, preço e disponibilidade | nenhuma | Sem dependências de saída. |
| `CupomDesconto` | Validade e cálculo do desconto | nenhuma | Sem dependências de saída. |
| `ProcessadorPagamento` | Orquestrar a cobrança | `MeioDePagamento` | Depende apenas da abstração, não das formas concretas. |
| `NotificadorStatus` | Avisar mudanças de status | `Cliente` (canal) | Concentra a variação do canal de aviso. |
| `ControladorCozinha`, `ControladorEntrega`, `ControladorAvaliacao` | Um caso de uso cada | `Pedido` (mais `Entregador` no caso de `ControladorEntrega`) | Poucos colaboradores e responsabilidade única. |
| `Cozinheiro`, `Entregador` | Nota média e disponibilidade (entregador) | nenhuma | Mantêm a própria informação. |

## 8. Mapa dos padrões GRASP aplicados

| Padrão | Onde foi aplicado | Mensagens |
|---|---|---|
| **Controller** | `ControladorPedido`, `ControladorCozinha`, `ControladorEntrega`, `ControladorAvaliacao` | D1: 1; D2: 1; D3: 1, 6; D4: 1 |
| **Creator** | `Pedido` cria `ItemPedido`, `Pagamento` e `Avaliacao`; o controlador cria `Pedido` | D1: 2, 3, 5; D2: 6; D4: 3, 4 |
| **Information Expert** | `Prato`, `CupomDesconto`, `ItemPedido`, `Pedido`, `Entregador`, `Cozinheiro` respondem pelo que conhecem | D1: 3, 4, 6–11; D2: 2, 5, 7; D3: 2, 3, 7, 8; D4: 2, 3, 5, 6 |
| **Low Coupling** | Controladores só conhecem `Pedido` e serviços; `Pedido` delega cálculo e aviso | D1: 7; D2: 3; D3: 4 |
| **High Cohesion** | Um controlador por caso de uso; regras de status dentro do `Pedido`; cálculo de subtotal no `ItemPedido` | D1: 11; D2: 7; D3: 3 |
| **Polymorphism** | `MeioDePagamento` com `PagamentoCartao` e `PagamentoPix` | D2: 4 |
| **Pure Fabrication** | `ProcessadorPagamento`, `NotificadorStatus` (e os repositórios, omitidos) | D2: 3; D3: 5 |
| **Indirection** | `NotificadorStatus` entre `Pedido` e `Cliente` | D3: 4, 9 |
| **Protected Variations** | Interface `pagar()` protege contra mudança de forma de pagamento ou gateway; notificador protege contra mudança de canal | D2: 4; D3: 5 |

## 9. Rastreabilidade

| Caso de uso | Diagrama | Regras de negócio | Requisitos | Telas |
|---|---|---|---|---|
| UC04 — Realizar pedido (+ cupom) | Diagrama 1 | RN02, RN07, RN08 | RF04, RF06 | [04](../prototipos/04-cardapio-pedido-cliente.svg), [05](../prototipos/05-confirmar-pagamento-cliente.svg) |
| UC05 — Efetuar pagamento | Diagrama 2 | RN01 | RF05, RNF-PL02 | [05](../prototipos/05-confirmar-pagamento-cliente.svg) |
| UC06 — Gerenciar pedidos recebidos | Diagrama 3 (mensagens 1 a 5) | RN03, RN04 | RF07 | [08](../prototipos/08-painel-cozinheiro-pedidos.svg) |
| UC07 — Realizar entrega | Diagrama 3 (mensagens 6 a 9) | RN05 | RF08 | [10](../prototipos/10-painel-entregador-entregas.svg) |
| UC09 — Acompanhar pedido | Diagrama 3 (notificações 4, 5 e 9) | — | RF09, RNF-R01 | [06](../prototipos/06-acompanhamento-pedido-cliente.svg) |
| UC08 — Avaliar pedido | Diagrama 4 | RN06 | RF10 | [07](../prototipos/07-avaliar-pedido-cliente.svg) |

---

Ver também: [Cenário de Negócio](01-cenario-negocio-escopo.md), [Modelo de Caso de Uso](02-modelo-caso-uso.md), [Modelagem de Negócio](03-modelagem-negocio.md), [Prototipação](04-prototipacao.md) e [Especificação de Requisitos](05-especificacao-requisitos.md).
