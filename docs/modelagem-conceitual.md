# miniCD - Modelagem Conceitual

## Visão Geral
O miniCD é um ERP de Centro de Distribuição que atende 5 filiais de uma rede de supermercados.
O escopo é somente a operação do CD: recebimento e expedição. Não há controle de venda da loja, financeiro ou contábil.

## Decisão de Arquitetura
O projeto foi iniciado pela modelagem conceitual, antes de qualquer criação de tabela no 【entity-Oracle¦canonical_name=Oracle】.
A modelagem conceitual foi utilizada para definir as entidades, os relacionamentos e as cardinalidades do CD, e para validar o fluxo operacional antes da modelagem lógica e física.

## Fluxo Operacional Definido
1. Pedido de Mercadoria - Criado manualmente quando o estoque atinge o mínimo
2. Recebimento - Entrada da mercadoria, com número igual ao do pedido, funcionando como NF
3. Conferência - Automática, com quantidade sempre igual à do pedido, sem divergência na simulação
4. Armazenagem - Automática, por palete, em endereço livre da mesma rua
5. Expedição - Pedido da filial + Saída do CD, liberando o endereço
6. Loja - Campo escuro, apenas vende e gera novo pedido, reiniciando o ciclo

## Regras de Negócio Travadas na Modelagem
- 1 endereço = 1 palete = 1 produto
- Rua imutável após o cadastro do produto
- Overflow pula para o próximo endereço livre da mesma rua
- Sistema cíclico: entrada ocupa endereço, saída libera endereço
- Cadastro aberto para novo fornecedor e novo produto, com 20 produtos de carga inicial

## Entidades Definidas
- Fornecedor
- Produto
- Filial
- Endereço
- Pedido
- Recebimento
- Expedição
- Inventário

## Atributos por Entidade

### 1. FORNECEDOR
- id_fornecedor - PK, único, obrigatório
- nome_fornecedor - obrigatório, único

### 2. PRODUTO
- id_produto - PK, único, obrigatório
- nome_produto - obrigatório
- id_fornecedor - FK, obrigatório - liga com FORNECEDOR
- rua - obrigatório, imutável após cadastro
- qtd_por_caixa - informação, só consulta
- qtd_por_fardo - informação, só consulta
- qtd_por_palete - obrigatório - define quantos cabem em 1 endereço
- estoque_minimo - obrigatório - quando atingir, gera Pedido

### 3. FILIAL
- id_filial - PK, único, obrigatório
- nome_filial - obrigatório, único

### 4. ENDEREÇO
- id_endereco - PK, único, obrigatório
- codigo_endereco - único, obrigatório - Ex: R01-E01
- rua - obrigatório - Ex: Rua 1 - Cereais
- status - obrigatório - LIVRE ou OCUPADO
- id_produto - FK - qual produto está lá, se estiver OCUPADO

### 5. PEDIDO
- id_pedido - PK, único, obrigatório - funciona como NF
- id_fornecedor - FK, obrigatório
- data_pedido - obrigatório
- status - obrigatório - ABERTO / RECEBIDO

### 6. RECEBIMENTO
- id_recebimento - PK, único, obrigatório - mesmo número do pedido
- id_pedido - FK, obrigatório
- data_recebimento - obrigatório
- quantidade_recebida - obrigatório - sempre igual à do pedido

### 7. EXPEDIÇÃO
- id_expedicao - PK, único, obrigatório
- id_filial - FK, obrigatório
- data_expedicao - obrigatório
- status - obrigatório - PEDIDO DA LOJA / SAÍDA

### 8. INVENTÁRIO
- id_inventario - PK, único, obrigatório
- rua - obrigatório - qual rua vai contar
- data_inventario - obrigatório
- status - obrigatório - ABERTO / FECHADO

## Cardinalidades Decididas

- FORNECEDOR - PRODUTO = 1 - 1
    - 1 fornecedor tem só 1 produto, 1 produto vem de só 1 fornecedor

- PRODUTO - ENDEREÇO = 1 - N
    - 1 produto pode estar em vários endereços, 1 endereço só tem 1 produto

- PEDIDO - RECEBIMENTO = 1 - 1
    - 1 pedido gera 1 recebimento, 1 recebimento vem de 1 pedido

- FILIAL - EXPEDIÇÃO = 1 - N
    - 1 filial pode receber várias expedições, 1 expedição vai pra 1 filial só
    - FILIAL é lado escuro, só como destino, sem controle de venda ou estoque da filial

- RECEBIMENTO - ENDEREÇO = 1 - N
    - 1 recebimento pode gerar vários endereços, 1 endereço vem de 1 recebimento só

- ENDEREÇO - INVENTÁRIO = N - N
    - 1 inventário conta vários endereços, 1 endereço pode ser contado em vários inventários
