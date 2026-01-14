
# Projeto: Esquema de E-commerce – Clientes PF/PJ, Pagamentos Múltiplos e Entrega

## Descrição do Projeto (Conceitual)
Este projeto modela o domínio de um e-commerce com as seguintes premissas:

- **Cliente** pode ser **Pessoa Física (PF)** ou **Pessoa Jurídica (PJ)**, **mas nunca ambos** ao mesmo tempo.
- **Pagamento**: um pedido pode ser liquidado por **múltiplas formas de pagamento** (ex.: parte no cartão e parte via Pix), mantendo os valores, parcelas e status individuais.
- **Entrega**: cada pedido possui **entrega com status** (ex.: `CRIADA`, `EM_TRANSPORTE`, `ENTREGUE`, `PROBLEMA`) e **código de rastreio** opcional/único, quando aplicável.

## Diagrama do Projeto

![iagrama ER do E-commerce

### Entidades Principais
- **Cliente**: dados cadastrais comuns (e-mail, telefone, endereços). A natureza PF/PJ é exclusiva.
- **PessoaFisica**: extensão de Cliente contendo `cpf`, `nome`, `data_nascimento`.
- **PessoaJuridica**: extensão de Cliente contendo `cnpj`, `razao_social`, `inscricao_estadual`.
- **Produto**: catálogo com preço atual e controle de estoque.
- **Pedido**: cabeçalho do pedido (cliente, datas, total, status).
- **ItemPedido**: itens (produto, quantidade, preço no momento).
- **FormaPagamento**: tipos disponíveis (Cartão, Boleto, Pix, etc.).
- **Pagamento**: alocação dos pagamentos ao pedido (um-para-muitos), podendo ter múltiplas formas.
- **Entrega**: informações de envio por pedido (transportadora, status, código de rastreio, prazos).

### Regras de Negócio
1. **Exclusividade PF/PJ**: cada `cliente` deve ter **exatamente uma** relação: PF **ou** PJ.
2. **Pagamentos Múltiplos**: `pedido` pode ter **n** registros em `pagamento`, cada um com sua `forma_pagamento` e `valor`.
3. **Entrega**: cada `pedido` possui um registro em `entrega` com `status` obrigatório e `codigo_rastreio` único quando informado.

---

## Diagrama (ER – Conceitual)

```mermaid
erDiagram
    CLIENTE ||--o| PESSOAFISICA : "é"
    CLIENTE ||--o| PESSOAJURIDICA : "é"
    CLIENTE ||--o{ PEDIDO : "realiza"
    PEDIDO ||--|{ ITEMPEDIDO : "contém"
    PRODUTO ||--o{ ITEMPEDIDO : "é item de"
    PEDIDO ||--o{ PAGAMENTO : "possui"
    FORMAPAGAMENTO ||--o{ PAGAMENTO : "é utilizada por"
    PEDIDO ||--|| ENTREGA : "gera"

    CLIENTE {
      uuid id PK
      text email
      text telefone
      timestamptz criado_em
      timestamptz atualizado_em
    }

    PESSOAFISICA {
      uuid cliente_id PK,FK
      text nome
      date data_nascimento
      text cpf UNIQUE
    }

    PESSOAJURIDICA {
      uuid cliente_id PK,FK
      text razao_social
      text cnpj UNIQUE
      text inscricao_estadual
    }

    PRODUTO {
      uuid id PK
      text nome
      text sku UNIQUE
      numeric preco
      int estoque
      bool ativo
    }

    PEDIDO {
      uuid id PK
      uuid cliente_id FK
      text status  "CRIADO|PAGO|CANCELADO"
      numeric total_bruto
      numeric total_desconto
      numeric total_liquido
      timestamptz criado_em
      timestamptz pago_em
    }

    ITEMPEDIDO {
      uuid id PK
      uuid pedido_id FK
      uuid produto_id FK
      int quantidade
      numeric preco_unitario
      numeric subtotal
    }

    FORMAPAGAMENTO {
      uuid id PK
      text codigo "CARTAO|PIX|BOLETO|CREDITO_LOJA"
      text descricao
      bool ativo
    }

    PAGAMENTO {
      uuid id PK
      uuid pedido_id FK
      uuid forma_pagamento_id FK
      numeric valor
      int parcelas
      text status "PENDENTE|APROVADO|NEGADO|ESTORNADO"
      text referencia_externa
      timestamptz criado_em
    }

    ENTREGA {
      uuid id PK
      uuid pedido_id FK UNIQUE
      text transportadora
      text status "CRIADA|EM_TRANSPORTE|ENTREGUE|PROBLEMA|DEVOLVIDA"
      text codigo_rastreio UNIQUE
      date previsao_entrega
      timestamptz atualizado_em
    }
