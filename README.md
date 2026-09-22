# Checkout Service: Plataforma de Delivery

Repositório da **Equipe 3 (Checkout)** para o projeto da disciplina de **Integração de Sistemas**.

O sistema completo é uma plataforma de delivery dividida em 15 microsserviços, distribuídos entre 5 equipes. Esta equipe é responsável pelo domínio **Checkout**, descrito no projeto como *"o motor principal: cruza catálogo, descontos e aciona a logística"*.

## Serviços deste repositório

| Serviço | Responsabilidade | Porta |
|---|---|---|
| `carrinho` | Gestão do carrinho, cálculo de subtotal e validação de cupom | `8005` |
| `pedidos` | Orquestração do pedido e status final | `8006` |
| `promocoes` | Cadastro, validação e cálculo de desconto de cupons | `8007` |

> Cada serviço é independente: possui seu próprio código, banco de dados e container. Não há compartilhamento de memória ou banco entre eles — toda comunicação, inclusive entre os próprios serviços desta equipe, acontece via HTTP.

- **Carrinho** consulta o serviço de **Cardápio**  para validar item e preço, e o serviço de **Identidade** para autenticar o usuário via JWT em todas as operações.
- **Carrinho** consulta **Promoções** para validar e aplicar cupons de desconto.
- **Pedidos** consulta **Autenticação** para validar o usuário, orquestra Carrinho e Promoções internamente, e aciona **Logística** e **Financeiro** quando o pedido é confirmado.
- **Promoções** é consumido apenas internamente pelo serviço de Carrinho/Pedidos.

## Requisitos funcionais

**Carrinho**
- Criar, consultar, atualizar e limpar os itens do carrinho de um usuário autenticado
- Calcular o subtotal acumulado dos itens no carrinho
- Validar item e preço via integração HTTP direta com o serviço de Catálogo, a cada adição/atualização de item
- Validar o token JWT do usuário junto ao serviço de Identidade em todas as operações
- Aplicar e validar cupons de desconto informados pelo cliente
- Calcular o valor exato de desconto sobre o subtotal, conforme a regra de negócio do cupom
- Verificar restrições do cupom: validade temporal, valor mínimo do pedido e limite de usos

**Pedidos**
- Criar pedido a partir do carrinho finalizado, orquestrando o processo e controlando o status inicial
- Consultar o status atual de um pedido (recebido, confirmado, em preparo, etc.)
- Consultar o histórico de pedidos anteriores do cliente

**Promoções**
- Cadastrar cupons de desconto
- Validar cupom (vigência, valor mínimo, uso único)
- Calcular e aplicar o desconto sobre o valor do pedido

## Requisitos não funcionais

- **Idempotência** na criação de pedidos, para evitar duplicidade em caso de reenvio de requisição.
- **Resiliência**: se um serviço dependente (Catálogo, Identidade, Logística, Financeiro) estiver fora do ar, o Checkout deve tratar a exceção e retornar erro amigável, sem derrubar o processo.
- **Validação** de cupons, evitando uso duplicado em condição de corrida.
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
| Carrinho | item_id + quantidade + token JWT | subtotal calculado (com desconto, se houver cupom aplicado) | Consulta Catálogo, Identidade e Promoções; consumido por Pedidos |
| Pedidos | carrinho fechado + token do usuário | pedido criado com status | Consulta Autenticação, Carrinho e Promoções; aciona Logística e Financeiro |
| Promoções | código do cupom + valor do pedido | validação + valor do desconto | Consumido apenas por Carrinho/Pedidos |

**Regra geral**: nenhum serviço externo acessa o banco de dados do Checkout diretamente (nem o inverso). Toda troca de dados entre domínios acontece exclusivamente via chamadas HTTP documentadas em Swagger.

**Equipe**: Cesar Augusto, Enriko Matheus, Juliana de Andrade, Lukas de Araújo Assis
