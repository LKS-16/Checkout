# Checkout Service — Plataforma de Delivery

Repositório da **Equipe 3 (Checkout)** para o projeto da disciplina de **Integração de Sistemas**.

O sistema completo é uma plataforma de delivery dividida em 15 microsserviços, distribuídos entre 5 equipes. Esta equipe é responsável pelo domínio **Checkout**, descrito no projeto como *"o motor principal: cruza catálogo, descontos e aciona a logística"*.

## Serviços deste repositório

| Serviço | Responsabilidade | Porta |
|---|---|---|
| `carrinho` | Cálculo de subtotal temporário | `8005` |
| `pedidos` | Orquestração do pedido e status final | `8006` |
| `promocoes` | Validação de cupons e descontos | `8007` |

> Cada serviço é independente: possui seu próprio código, banco de dados e container. Não há compartilhamento de memória ou banco entre eles — toda comunicação, inclusive entre os próprios serviços desta equipe, acontece via HTTP.


- **Carrinho** consulta o serviço de **Cardápio** (Equipe 2) para validar item e preço.
- **Pedidos** consulta **Autenticação** (Equipe 1) para validar o usuário, orquestra Carrinho e Promoções internamente, e aciona **Logística** (Equipe 4) e **Financeiro** (Equipe 5) quando o pedido é confirmado.
- **Promoções** é consumido apenas internamente pelo serviço de Pedidos.

## Requisitos funcionais

**Carrinho**
- Adicionar/remover item
- Atualizar quantidade
- Calcular subtotal
- Limpar carrinho após finalização do pedido

**Pedidos**
- Criar pedido a partir do carrinho
- Consultar status do pedido (recebido, confirmado, em preparo, etc.)
- Consultar histórico de pedidos do cliente

**Promoções**
- Cadastrar cupom
- Validar cupom (vigência, valor mínimo, uso único)
- Aplicar desconto ao pedido

## Requisitos não funcionais

- **Idempotência** na criação de pedidos, para evitar duplicidade em caso de reenvio de requisição.
- **Resiliência**: se um serviço dependente (Catálogo, Identidade, Logística, Financeiro) estiver fora do ar, o Checkout deve tratar a exceção e retornar erro amigável, sem derrubar o processo.
- **Validação atômica** de cupons, evitando uso duplicado em condição de corrida.
- **Cobertura de testes mínima de 75%** (unitários + integração), conforme exigido pela disciplina.
- Expiração/TTL de carrinhos inativos.

## Requisitos de integração (padrões adotados pela turma)

- **Formato de erro** padrão: `{"erro": true, "mensagem": "Detalhes da falha"}` + código HTTP correspondente.
- **Datas**: sempre em ISO 8601 (ex: `2026-08-24T14:30:00Z`).
- **Valores financeiros**: sempre como inteiro em centavos (ex: R$ 25,90 → `2590`).
- **Idioma de rotas e JSON**:
- Todos os contratos documentados em Swagger/OpenAPI antes da implementação da lógica real.

## Fronteiras do serviço

| Serviço | Entra | Sai | Depende de / é consumido por |
|---|---|---|---|
| Carrinho | item_id + quantidade | subtotal calculado | Consulta Cardápio (Eq.2); consumido por Pedidos |
| Pedidos | carrinho fechado + token do usuário | pedido criado com status | Consulta Autenticação (Eq.1), Carrinho e Promoções; aciona Logística (Eq.4) e Financeiro (Eq.5) |
| Promoções | código do cupom + valor do pedido | validação + valor do desconto | Consumido apenas por Pedidos |

**Regra geral**: nenhum serviço externo acessa o banco de dados do Checkout diretamente (nem o inverso). Toda troca de dados entre domínios acontece exclusivamente via chamadas HTTP documentadas em Swagger.
