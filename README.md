# 🍽️ Documentação do Sistema - RU UESPI

Este documento centraliza a modelagem de dados e processos do aplicativo móvel para o **Restaurante Universitário da Universidade Estadual do Piauí (UESPI)**. A estrutura utiliza diagramas em formato **Mermaid** (renderizados nativamente pelo GitHub/Notion) acompanhados de tabelas descritivas para facilitar o entendimento do projeto.

---

## 🎯 1. Diagrama de Casos de Uso

O diagrama abaixo ilustra as interações dos atores principais com as funcionalidades do aplicativo móvel.

```mermaid
graph LR
    %% Atores
    Estudante((Estudante))
    Admin((Admin do RU))
    SisPag((API Pix))

    %% Casos de Uso Estudante
    Estudante --> UC1(Visualizar Cardápio)
    Estudante --> UC2(Comprar Fichas/Tickets)
    Estudante --> UC3(Visualizar QR Code do Ticket)

    %% Casos de Uso Admin
    Admin --> UC4(Gerenciar Cardápio Semanal)
    Admin --> UC5(Validar Ticket no Acesso)

    %% Relacionamentos com o Sistema de Pagamento
    UC2 --> SisPag
    SisPag --> UC6(Confirmar Pagamento)
    UC6 -.-> |<< include >>| UC2
```

### 📋 Detalhamento dos Casos de Uso

| Ator | Caso de Uso | Descrição |
| :--- | :--- | :--- |
| **Estudante** | `UC1` Visualizar Cardápio | Consulta os pratos e refeições planejadas para a semana corrente. |
| **Estudante** | `UC2` Comprar Fichas/Tickets | Inicia o fluxo de compra de créditos para refeição via PIX. |
| **Estudante** | `UC3` Visualizar QR Code | Exibe a ficha digital na tela para ser escaneada na entrada do RU. |
| **Admin do RU** | `UC4` Gerenciar Cardápio | Permite o cadastro, edição e exclusão dos menus semanais. |
| **Admin do RU** | `UC5` Validar Ticket | Realiza a leitura e validação do QR Code do estudante na catraca/entrada. |
| **API Pix** | `UC6` Confirmar Pagamento | Sistema externo que valida a transação financeira e notifica o app. |

---

## 🗄️ 2. Dicionário de Dados & DER

Abaixo está o modelo lógico do banco de dados (DER) acompanhado das tabelas que especificam cada entidade do sistema.

```mermaid
erDiagram
    ESTUDANTE ||--o{ TICKET : "compra"
    ESTUDANTE ||--o{ PAGAMENTO : "efetua"
    PAGAMENTO ||--|| TICKET : "gera"
    ADMINISTRADOR ||--o{ CARDAPIO : "cadastra"

    ESTUDANTE {
        int id PK
        string nome
        string matricula UK
        string email
        string senha
    }

    TICKET {
        int id PK
        int estudante_id FK "Relaciona com ESTUDANTE"
        int pagamento_id FK "Relaciona com PAGAMENTO"
        string codigo_qr UK
        string status
        date data_geracao
        date data_uso
    }

    PAGAMENTO {
        int id PK
        int estudante_id FK "Relaciona com ESTUDANTE"
        float valor
        string chave_pix
        string status_pagamento
        timestamp data_hora
    }

    CARDAPIO {
        int id PK
        int administrador_id FK "Relaciona com ADMINISTRADOR"
        date data_dia
        string dia_semana
        string tipo_refeicao
        string descricao_pratos
    }

    ADMINISTRADOR {
        int id PK
        string nome
        string email
        string credencial
    }
```

### 📊 Especificação das Tabelas (Dicionário de Dados)

#### Entidade: `ESTUDANTE`
| Atributo | Tipo | Restrição | Descrição |
| :--- | :--- | :--- | :--- |
| `id` | `int` | **PK** (Primary Key) | Identificador único do estudante. |
| `nome` | `string` | Not Null | Nome completo do usuário. |
| `matricula` | `string` | **UK** (Unique Key) | Matrícula institucional UESPI. |
| `email` | `string` | Not Null / UK | Email acadêmico ou pessoal para login. |
| `senha` | `string` | Not Null | Hash da senha de acesso. |

#### Entidade: `TICKET` (Ficha Digital)
| Atributo | Tipo | Restrição | Descrição |
| :--- | :--- | :--- | :--- |
| `id` | `int` | **PK** | Identificador único do ticket. |
| `codigo_qr` | `string` | **UK** | Token criptografado contido no QR Code. |
| `status` | `string` | Not Null | Estado atual (Disponível, Utilizado, Expirado). |
| `data_geracao`| `date` | Not Null | Data em que a compra foi confirmada. |
| `data_uso` | `date` | Nullable | Registro de quando o aluno consumiu a refeição. |

#### Entidade: `PAGAMENTO`
| Atributo | Tipo | Restrição | Descrição |
| :--- | :--- | :--- | :--- |
| `id` | `int` | **PK** | Identificador da transação financeira. |
| `valor` | `float` | Not Null | Valor cobrado pela ficha do RU. |
| `chave_pix` | `string` | Not Null | Código "Copia e Cola" ou ID do Pix gerado. |
| `status_pagamento`| `string` | Not Null | Status do gateway (Pendente, Pago, Cancelado). |
| `data_hora` | `timestamp`| Not Null | Data e hora exata da tentativa de compra. |

---

## 🏛️ 3. Diagrama de Classes

Visão estrutural orientada a objetos (POO) mapeando as classes de negócio do ecossistema do aplicativo.

```mermaid
classDiagram
    class Usuario {
        +int id
        +string nome
        +string email
        +string senha
        +fazerLogin()
    }

    class Estudante {
        +string matricula
        +visualizarCardapio()
        +comprarTicket()
        +exibirQRCode()
    }

    class Administrator {
        +string credencial
        +atualizarCardapio()
        +validarTicket()
    }

    class Ticket {
        +int id
        +string qrCode
        +string status
        +date dataValidade
        +atualizarStatus()
    }

    class Pagamento {
        +int id
        +float valor
        +string status
        +gerarCopiaECola()
        +verificarStatus()
    }

    class Cardapio {
        +int id
        +date data
        +string itens
        +exibirPratos()
    }

    Usuario <|-- Estudante
    Usuario <|-- Administrator
    Estudante "1" --> "*" Ticket : possui
    Estudante "1" --> "*" Pagamento : realiza
    Pagamento "1" --> "1" Ticket : libera
    Administrator "1" --> "*" Cardapio : gerencia
```

### ⚙️ Métodos e Responsabilidades

| Classe | Operação Principal | Objetivo |
| :--- | :--- | :--- |
| **Estudante** | `comprarTicket()` | Dispara o fluxo de pagamento e vincula uma nova ficha à conta do usuário. |
| **Administrator** | `validarTicket()` | Executado no celular/dispositivo do fiscal para alterar o status do ticket para "Utilizado". |
| **Pagamento** | `gerarCopiaECola()`| Comunica-se com a API do Banco Central/Gateway para trazer o Pix dinâmico. |
| **Ticket** | `atualizarStatus()`| Altera internamente o ciclo de vida da ficha digital. |

---

## 🔄 4. Diagrama de Transição de Estados

Mapeamento do ciclo de vida crítico da entidade **Ticket**, vital para evitar fraudes ou duplicidade de acessos.

```mermaid
stateDiagram-v2
    [*] --> Gerado : Estudante solicita compra
    
    Gerado --> AguardandoPagamento : QR Code PIX apresentado
    
    AguardandoPagamento --> Cancelado : Tempo limite esgotado (Timeout)
    AguardandoPagamento --> Disponivel : Pagamento Confirmado via Pix
    
    Disponivel --> Utilizado : QR Code lido na entrada do RU
    Disponivel --> Expirado : Data de validade do ticket venceu
    
    Cancelado --> [*]
    Utilizado --> [*]
    Expirado --> [*]
```

### 🚦 Mapeamento dos Estados do Ticket

| Estado Atual | Gatilho (Transição) | Estado Destino | Regra de Negócio |
| :--- | :--- | :--- | :--- |
| `[*] Inicial` | Solicitação do aluno | **Gerado** | O ticket entra na fila temporária do sistema. |
| `Gerado` | QR Code Pix emitido | **AguardandoPagamento**| Aguarda a notificação do webhook da API de pagamento. |
| `AguardandoPagamento`| Tempo esgotado (ex: 10 min) | **Cancelado** | O Pix expira para não prender requisições no banco. |
| `AguardandoPagamento`| Webhook confirma recebimento| **Disponivel** | O ticket torna-se válido e gera o QR Code definitivo de acesso. |
| `Disponivel` | Leitura ótica na entrada | **Utilizado** | O aluno consome a refeição. O ticket é invalidado imediatamente. |
| `Disponivel` | Validade expirada (fim do dia)| **Expirado** | Fichas diárias que não foram usadas perdem a validade (conforme regras do RU). |
