# Cenário de Negócio e Concepção do Sistema — SaborBrasileiro

## 1. Contexto

Em muitos bairros e comunidades existem cozinheiros caseiros — donas de casa, aposentados, cozinheiros informais — que preparam marmitas, doces e pratos artesanais de forma independente, vendendo principalmente por indicação boca a boca ou grupos de WhatsApp. Esse modelo tem baixo alcance, dificulta o controle de pedidos e pagamentos, e não oferece uma forma organizada de entrega.

Ao mesmo tempo, clientes que buscam esse tipo de comida (mais caseira, mais barata e muitas vezes mais saudável que fast-food) não têm um canal único para descobrir esses cozinheiros, ver cardápios, fazer pedidos e acompanhar a entrega.

## 2. Problema identificado

- Cozinheiros caseiros não têm visibilidade além do círculo de contatos próximos.
- Não existe controle organizado de pedidos, cardápio e status de entrega.
- Clientes não têm um lugar único para comparar opções de comida caseira na região.
- A entrega, quando existe, depende de arranjos informais (o próprio cozinheiro entrega, ou pede para terceiros).

## 3. Envolvidos (stakeholders)

| Envolvido | Papel no negócio |
|---|---|
| Cliente | Busca, pede e paga por comida caseira |
| Cozinheiro/Estabelecimento parceiro | Cadastra cardápio e prepara os pedidos |
| Entregador parceiro | Realiza a entrega dos pedidos |
| Administrador da plataforma | Gerencia cadastros, garante qualidade e mediação de conflitos |

## 4. Necessidades identificadas

- O cliente precisa encontrar facilmente opções de comida caseira perto dele.
- O cozinheiro precisa de uma vitrine simples para divulgar e vender seus pratos.
- O entregador precisa de uma forma organizada de receber e cumprir corridas de entrega.
- Todos precisam de confiabilidade quanto a pagamento, status do pedido e prazos.

## 5. Processos atuais com dificuldades/oportunidades de melhoria

- **Divulgação**: hoje é feita de forma informal (grupos de WhatsApp), o que limita o crescimento do cozinheiro → oportunidade de criar uma vitrine digital única.
- **Pedidos**: muitas vezes feitos por mensagem de texto, sem padronização → oportunidade de digitalizar o processo de pedido.
- **Entrega**: normalmente informal ou inexistente → oportunidade de conectar entregadores parceiros à demanda.
- **Pagamento**: geralmente combinado na entrega (dinheiro) → oportunidade de oferecer pagamento digital dentro da plataforma.

---

## 6. Declaração de Escopo

### 6.1 Justificativa

A plataforma SaborBrasileiro resolve um problema real de baixa visibilidade e falta de organização no comércio informal de comida caseira, ao mesmo tempo em que oferece aos clientes um canal único e confiável para descobrir, pedir e receber esse tipo de comida em casa.

### 6.2 Objetivos

**Objetivo geral:**
Desenvolver uma plataforma de marketplace que conecte cozinheiros caseiros, clientes e entregadores em um único fluxo de pedido e entrega.

**Objetivos específicos:**
- Permitir que cozinheiros cadastrem e gerenciem seus cardápios;
- Permitir que clientes busquem, pesquisem e façam pedidos de forma simples;
- Permitir que entregadores aceitem e acompanhem entregas;
- Viabilizar pagamento digital dentro do próprio sistema;
- Oferecer acompanhamento de status do pedido em tempo real (do pedido à entrega).

### 6.3 Critérios de aceitação

- O sistema deve permitir cadastro de pelo menos 3 tipos de usuário: cliente, cozinheiro e entregador;
- O cliente deve conseguir concluir um pedido do início ao fim (busca → pedido → pagamento → acompanhamento → avaliação) sem sair do sistema;
- O cozinheiro deve conseguir cadastrar, editar e remover itens do cardápio;
- O entregador deve conseguir visualizar e aceitar entregas disponíveis;
- Todo pedido deve ter um status rastreável (recebido, em preparo, saiu para entrega, entregue).

### 6.4 Restrições

- O projeto é acadêmico, com prazo e equipe limitados — o escopo cobre apenas a região de um bairro/cidade piloto;
- Não haverá, nesta primeira versão, integração com meios de pagamento reais (gateway será simulado);
- Pagamento é sempre feito dentro do aplicativo (cartão ou Pix); não há opção de pagamento em dinheiro na entrega, já que o pedido só é confirmado após o pagamento (ver RN01 na modelagem de negócio);
- O sistema não cobrirá logística própria de entrega (frota); depende de entregadores parceiros autônomos;
- Sem suporte multilíngue nesta versão.

### 6.5 Premissas

- Cozinheiros e entregadores possuem smartphone e conexão à internet;
- Os cozinheiros são responsáveis por seguir normas básicas de higiene alimentar (fora do escopo técnico do sistema);
- Haverá adesão inicial de um pequeno grupo de cozinheiros e entregadores para viabilizar o piloto.

### 6.6 Organização inicial

| Papel | Responsabilidade |
|---|---|
| Líder do projeto | Coordenação geral e contato com o professor/orientador |
| Analista de requisitos | Levantamento e documentação do cenário e casos de uso |
| Desenvolvedor(es) | Implementação do sistema |
| Responsável por testes | Validação dos critérios de aceitação |

_(ajuste os papéis conforme o número de integrantes do seu grupo)_

### 6.7 Estimativa de custos

Por se tratar de um projeto acadêmico, os custos são estimados em esforço (horas), não em valores monetários:

| Etapa | Estimativa de esforço |
|---|---|
| Cenário de negócio e escopo | 8h |
| Modelo de caso de uso | 12h |
| Especificação detalhada dos casos de uso | 16h |
| Modelagem de dados | 10h |
| Implementação (MVP) | 40h |
| Testes | 12h |

### 6.8 Cronograma estimado

| Semana | Atividade |
|---|---|
| 1 | Cenário de negócio, concepção e declaração de escopo |
| 2 | Modelo de caso de uso (atores, requisitos, diagrama) |
| 3–4 | Especificação detalhada dos casos de uso |
| 5–6 | Projeto de software (classes, dados) |
| 7–9 | Implementação do MVP |
| 10 | Testes e ajustes finais |

### 6.9 Outros itens relevantes

- Público-alvo inicial: bairro/cidade piloto com forte presença de cozinheiros informais;
- Sucesso do piloto será medido pelo número de pedidos concluídos com sucesso na plataforma.
