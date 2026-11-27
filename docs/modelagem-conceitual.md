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

 
    - # miniCD - Modelagem Lógica
    - 
## Antes de começar - Modificações feitas

Durante a revisão da Modelagem Conceitual, foram feitas modificações para deixar o projeto mais fiel ao funcionamento real do CD.

A Modelagem Lógica abaixo já está com todas as modificações aplicadas. Ela é a versão atualizada e válida para criar as tabelas no Oracle.

Modificações:
- ENDERECO ganhou quantidade de fardos e validade
- RECEBIMENTO ganhou validade e quantidade por fardo
- PEDIDO, RECEBIMENTO, EXPEDICAO e INVENTARIO ganharam tabela de detalhe
- EXPEDICAO ganhou fluxo completo com FEFO e baixa no endereço ao faturar
- INVENTARIO passou a ser contagem no papel, digitação no sistema e divergências no final

## Visão Geral
Transformação do DER da Modelagem Conceitual em tabelas, com PK, FK e tipos de dados, antes de criar no Oracle.

## Regras Travadas
- 1 endereço = 1 palete = até QTD_POR_PALETE
- QTD_POR_PALETE no cadastro do produto é o limite do endereço
- Armazenamento por palete, com quantidade por fardo e validade no endereço
- FILIAL é lado escuro, só como destino, sem controle de venda ou estoque da filial
- Carga inicial com 20 fornecedores e 20 produtos, cadastro aberto para adicionar mais

## Tabelas

### 1. TB_FORNECEDOR
- ID_FORNECEDOR NUMBER - PK, único, obrigatório
- NOME_FORNECEDOR VARCHAR2(100) - obrigatório, único

### 2. TB_PRODUTO
- ID_PRODUTO NUMBER - PK, único, obrigatório
- NOME_PRODUTO VARCHAR2(100) - obrigatório
- ID_FORNECEDOR NUMBER - FK, obrigatório - liga com TB_FORNECEDOR
- RUA VARCHAR2(50) - obrigatório, imutável após cadastro
- QTD_POR_CAIXA NUMBER - informação, só consulta
- QTD_POR_FARDO NUMBER - informação, só consulta
- QTD_POR_PALETE NUMBER - obrigatório - limite do endereço
- ESTOQUE_MINIMO NUMBER - obrigatório

### 3. TB_FILIAL
- ID_FILIAL NUMBER - PK, único, obrigatório
- NOME_FILIAL VARCHAR2(50) - obrigatório, único

### 4. TB_ENDERECO
- ID_ENDERECO NUMBER - PK, único, obrigatório
- CODIGO_ENDERECO VARCHAR2(20) - único, obrigatório - Ex: R01-E01
- RUA VARCHAR2(50) - obrigatório
- STATUS VARCHAR2(20) - obrigatório - LIVRE ou OCUPADO
- ID_PRODUTO NUMBER - FK - qual produto está lá
- QUANTIDADE_FARDOS NUMBER - quantos fardos tem no endereço
- DATA_VALIDADE DATE - validade do lote no endereço

### 5. TB_PEDIDO
- ID_PEDIDO NUMBER - PK, único, obrigatório - funciona como NF
- ID_FORNECEDOR NUMBER - FK, obrigatório
- DATA_PEDIDO DATE - obrigatório
- STATUS VARCHAR2(20) - obrigatório - ABERTO / RECEBIDO

### 6. TB_PEDIDO_DETALHE
- ID_DETALHE NUMBER - PK
- ID_PEDIDO NUMBER - FK
- ID_PRODUTO NUMBER - FK
- QUANTIDADE_PEDIDA_FARDOS NUMBER - quantos fardos pediu

### 7. TB_RECEBIMENTO
- ID_RECEBIMENTO NUMBER - PK, único, obrigatório - mesmo número do pedido
- ID_PEDIDO NUMBER - FK, obrigatório
- DATA_RECEBIMENTO DATE - obrigatório

### 8. TB_RECEBIMENTO_DETALHE
- ID_DETALHE NUMBER - PK
- ID_RECEBIMENTO NUMBER - FK
- ID_PRODUTO NUMBER - FK
- QUANTIDADE_RECEBIDA_FARDOS NUMBER - quantos fardos chegaram
- DATA_VALIDADE DATE - validade do lote
- CODIGO_ENDERECO VARCHAR2(20) - em qual endereço foi parar

### 9. TB_EXPEDICAO
- ID_EXPEDICAO NUMBER - PK, único, obrigatório
- ID_FILIAL NUMBER - FK, obrigatório - só como destino
- DATA_PEDIDO_FILIAL DATE - obrigatório
- STATUS VARCHAR2(30) - PEDIDO / ANALISADO / IMPRESSO / SEPARADO / APROVADO / NF GERADA

### 10. TB_EXPEDICAO_DETALHE
- ID_DETALHE NUMBER - PK
- ID_EXPEDICAO NUMBER - FK
- ID_PRODUTO NUMBER - FK
- QUANTIDADE_PEDIDA_FARDOS NUMBER - quanto a filial pediu
- QUANTIDADE_SEPARADA_FARDOS NUMBER - quanto separaram de verdade
- DATA_VALIDADE DATE - validade do que saiu
- CODIGO_ENDERECO_ORIGEM VARCHAR2(20) - de qual endereço saiu

### 11. TB_INVENTARIO
- ID_INVENTARIO NUMBER - PK, único, obrigatório
- RUA VARCHAR2(50) - obrigatório - qual rua vai contar
- DATA_INVENTARIO DATE - obrigatório
- STATUS VARCHAR2(20) - obrigatório - ABERTO / FECHADO

### 12. TB_INVENTARIO_DETALHE
- ID_DETALHE NUMBER - PK
- ID_INVENTARIO NUMBER - FK
- CODIGO_ENDERECO VARCHAR2(20) - obrigatório
- ID_PRODUTO NUMBER - FK
- QUANTIDADE_FARDOS_CONTADOS NUMBER - quantos fardos contou
- DATA_VALIDADE_CONTADA DATE - qual validade contou
- STATUS_DIVERGENCIA VARCHAR2(20) - OK / DIVERGENTE

## Fluxos Definidos

### Inventário
Contagem no papel na rua, digitação no sistema em tela de contagem por rua, e no final o sistema mostra as divergências entre o sistema e a contagem. Conta nome, quantidade de fardos e validade.

### Expedição
Filial pede, aparece no sistema para analisar, sistema sugere endereço com validade mais perto de vencer primeiro - FEFO, imprime papel já com endereço marcado, separador separa, folha volta, digita quantidade separada e aprova, quando faturar a NF já dá baixa naquele endereço que saiu a separação. Se zerou, fica LIVRE. Se sobrou, fica OCUPADO com o que sobrou.

### Recebimento
Continua automático, com quantidade por fardo e validade. Conferência igual ao pedido.

- RECEBIMENTO - ENDEREÇO = 1 - N
    - 1 recebimento pode gerar vários endereços, 1 endereço vem de 1 recebimento só

- ENDEREÇO - INVENTÁRIO = N - N
    - 1 inventário conta vários endereços, 1 endereço pode ser contado em vários inventários
