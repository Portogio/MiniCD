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

## Próximos Passos
- Modelagem Lógica - Transformação do DER em tabelas com PK e FK
- Modelagem Física - Criação das tabelas no Oracle
