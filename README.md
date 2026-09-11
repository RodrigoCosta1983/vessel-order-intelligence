# Vessel Order Intelligence — Pedidos Inteligentes para Embarcações

[Read in English](#english-version)

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=for-the-badge&logo=openai&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

**Vessel Order Intelligence** é um sistema inteligente para leitura, interpretação, normalização e tratamento de pedidos de suprimentos recebidos de embarcações.

O projeto foi concebido para receber pedidos em formatos variados — como **PDF, PDF escaneado, Excel, Word e imagem** — identificar automaticamente os itens solicitados e procurar correspondências no catálogo oficial de produtos do cliente.

O catálogo oficial permanece em um **PostgreSQL externo com acesso somente leitura**. Todo o histórico operacional, decisões, memória por cliente/embarcação, aliases, substituições e informações geradas pela IA são armazenados exclusivamente no banco PostgreSQL próprio do sistema.

> O nome comercial definitivo do produto ainda não foi definido.  
> `Vessel Order Intelligence` é o nome técnico atual do projeto e do repositório.

---

## 📌 Status Atual

```text
✅ P0 — Arquitetura inicial e regras centrais
    ✅ fluxo principal do pedido definido
    ✅ PostgreSQL oficial do cliente identificado
    ✅ acesso ao banco oficial definido como SOMENTE LEITURA
    ✅ campos mínimos do catálogo oficial definidos
    ✅ banco PostgreSQL próprio do sistema definido
    ✅ memória por cliente/embarcação definida conceitualmente
    ✅ histórico de substituições definido conceitualmente
    ✅ princípio human-in-the-loop definido
    ✅ cronograma macro definido
    ✅ documentação técnica inicial criada
    ✅ estratégia de versionamento Git/GitHub definida

🟡 P1 — Banco próprio e modelo de dados
    ⏳ P1-A — definir entidades canônicas
    ⏳ P1-B — definir relacionamentos
    ⏳ P1-C — definir campos e tipos
    ⏳ P1-D — definir índices e constraints
    ⏳ P1-E — definir auditoria e rastreabilidade
    ⏳ P1-F — criar primeira migration Alembic

⏳ P2 — Entrada e leitura de documentos
⏳ P3 — Extração e normalização inteligente
⏳ P4 — Integração read-only com o catálogo oficial
⏳ P5 — Motor de matching
⏳ P6 — Memória por cliente e embarcação
⏳ P7 — Interface de revisão e decisão humana
⏳ P8 — Testes reais, métricas e calibração
⏳ P9 — Automações e integrações posteriores
```

A documentação técnica detalhada e viva do projeto está em:

```text
docs/PROJETO_PEDIDOS_INTELIGENTES_EMBARCACOES.md
```

---

## 🎯 Objetivo do Projeto

Automatizar e apoiar o processo de interpretação de pedidos enviados por marinheiros ou responsáveis por embarcações antes da atracação.

Fluxo principal:

```text
MARINHEIRO / EMBARCAÇÃO
        ↓
envia pedido
PDF / Excel / Word / imagem
        ↓
LEITOR DE DOCUMENTOS
        ↓
EXTRAÇÃO E NORMALIZAÇÃO
        ↓
produto / código / quantidade / unidade
        ↓
MEMÓRIA DO CLIENTE / EMBARCAÇÃO
        ↓
MOTOR DE MATCHING
        ↓
CATÁLOGO OFICIAL DO CLIENTE
PostgreSQL — SOMENTE LEITURA
        ↓
ENCONTRADO / PROVÁVEL / SIMILAR / NÃO ENCONTRADO
        ↓
REVISÃO HUMANA
        ↓
DECISÃO
        ↓
HISTÓRICO + MEMÓRIA DO SISTEMA
```

A IA deverá auxiliar principalmente nos casos **novos, incompletos ou ambíguos**.

Quanto mais um cliente utilizar o sistema, mais decisões anteriores poderão ser reaproveitadas sem necessidade de uma nova pesquisa completa.

---

## ✨ Funcionalidades Planejadas

O projeto está em desenvolvimento. Os itens abaixo representam o escopo planejado do MVP e das próximas fases.

### 📄 Leitura de Pedidos

- **PDF com texto:** extração estruturada de conteúdo.
- **PDF escaneado:** interpretação visual por modelo multimodal.
- **Excel:** leitura de planilhas e tabelas.
- **Word:** leitura de tabelas e conteúdo textual.
- **Imagens:** suporte planejado para pedidos enviados como foto ou captura.
- **Estruturas inconsistentes:** o sistema não dependerá de um modelo único de planilha.

### 🧠 Extração Inteligente

Para cada linha do pedido, o sistema buscará identificar:

- descrição original;
- descrição normalizada;
- código informado, quando houver;
- quantidade solicitada;
- unidade;
- marca, quando identificável;
- embalagem;
- peso ou volume;
- observações relevantes;
- nível de confiança da extração.

Exemplo conceitual:

```json
{
  "descricao_original": "SAMOSA CURRY VEGETAL",
  "descricao_normalizada": "samosa vegetal com curry",
  "codigo": null,
  "quantidade": 120,
  "unidade": "KG",
  "marca": null,
  "embalagem": null,
  "confianca_extracao": 0.97
}
```

### 🔎 Busca e Matching de Produtos

A ordem de pesquisa prevista é:

```text
1. código exato
2. memória específica da embarcação
3. memória do cliente
4. aliases / conhecimento já confirmado
5. busca textual
6. busca fuzzy
7. busca semântica com embeddings
8. IA compara os melhores candidatos
```

O objetivo é evitar o uso desnecessário de IA quando uma resposta já é conhecida pelo sistema.

### 🟢 Classificação de Resultado

Cada item poderá receber um dos seguintes estados:

```text
🟢 ENCONTRADO
→ correspondência clara ou código exato

🟡 PROVÁVEL
→ forte candidato, exige confirmação

🟠 SIMILAR
→ item exato não localizado, mas existem alternativas

🔴 NÃO ENCONTRADO
→ nenhuma correspondência confiável no catálogo
```

### 🧠 Memória por Cliente e Embarcação

O sistema terá memória persistida no próprio banco.

Exemplo:

```text
Cliente A
"SAMOSA CURRY VEGETAL"
        ↓
produto interno 88451
        ↓
confirmado 5 vezes
rejeitado 0 vezes
```

Em pedidos futuros do mesmo cliente, o sistema poderá reutilizar o conhecimento anterior antes de executar uma busca completa.

### 🔄 Histórico de Substituições

O sistema também registrará substituições aceitas e rejeitadas.

Exemplo:

```text
Produto solicitado:
HEINZ KETCHUP 5 KG

Produto substituto:
HELLMANN'S KETCHUP 5 KG

Aceito:
4 vezes

Rejeitado:
0 vezes
```

Em uma nova solicitação, o operador poderá visualizar que aquele cliente já aceitou anteriormente a troca.

Uma aceitação anterior **não significa autorização automática permanente**. As regras de automação serão configuráveis e auditáveis.

### 👤 Revisão Humana

O sistema seguirá o princípio **human-in-the-loop**.

Correspondências ambíguas não serão tratadas como verdade apenas porque a IA sugeriu um produto.

O operador poderá:

- confirmar;
- rejeitar;
- corrigir;
- escolher outro produto;
- marcar como não disponível;
- registrar substituição;
- adicionar informação ao histórico.

---

## 🏗️ Arquitetura Base

### Fonte Oficial de Produtos

O catálogo existente do cliente continuará sendo a fonte operacional de verdade.

```text
POSTGRESQL OFICIAL DO CLIENTE
        ↓
SOMENTE LEITURA
        ↓
código
produto
quantidade disponível
```

Regra arquitetural:

```text
BANCO DO CLIENTE
→ fonte oficial do catálogo

NOSSO BANCO
→ operação + inteligência + memória + histórico
```

### Banco Oficial — Nunca Escrever

A aplicação deverá utilizar uma credencial com permissão restrita a leitura.

```text
✅ SELECT

❌ INSERT
❌ UPDATE
❌ DELETE
❌ ALTER
❌ DROP
```

Sempre que possível, será preferível que o cliente exponha uma `VIEW` dedicada contendo apenas os campos necessários.

### Nosso PostgreSQL

O banco próprio do sistema armazenará, entre outras informações:

```text
clientes
embarcacoes
pedidos
pedido_itens
catalogo_indexado
memoria_cliente
memoria_embarcacao
aliases_produtos
substituicoes
decisoes
processamentos_ia
auditoria
```

O modelo definitivo será definido durante a P1.

---

## 🔒 Segurança e Isolamento de Dados

Princípios já definidos:

- O PostgreSQL oficial do cliente é **somente leitura**.
- A IA não poderá alterar o catálogo oficial.
- Credenciais nunca serão versionadas no Git.
- O arquivo `.env` real permanecerá fora do repositório.
- Apenas `.env.example` será versionado.
- Chaves da OpenAI não serão registradas em logs ou documentação.
- Dados gerados pela IA não serão tratados como dados oficiais sem validação.
- Toda decisão relevante deverá possuir rastreabilidade.
- Matching incerto deverá exigir revisão humana.
- O sistema deverá falhar de forma segura quando não houver confiança suficiente.

---

## 🤖 Estratégia de IA

A arquitetura prevê o uso da OpenAI API para tarefas específicas.

### Extração

```text
documento
    ↓
modelo multimodal / texto
    ↓
Structured Output
    ↓
JSON validado
```

### Matching

A IA não deverá receber todo o catálogo indiscriminadamente.

Fluxo planejado:

```text
item normalizado
    ↓
busca no PostgreSQL
    ↓
Top N candidatos
    ↓
IA compara somente os melhores candidatos
    ↓
resultado + confiança + justificativa
```

### Configuração por Ambiente

Os modelos não serão fixados dentro da regra de negócio.

Exemplo:

```env
OPENAI_EXTRACTION_MODEL=
OPENAI_MATCHING_MODEL=
OPENAI_EMBEDDING_MODEL=
```

Isso permite trocar modelos futuramente sem reconstruir a aplicação.

---

## 🧩 Tecnologias Planejadas

### Backend

- **Linguagem:** Python
- **Framework:** FastAPI
- **Validação e schemas:** Pydantic
- **ORM:** SQLAlchemy 2
- **Migrations:** Alembic
- **Servidor ASGI:** Uvicorn

### Banco de Dados

- **Banco próprio:** PostgreSQL
- **Banco oficial do cliente:** PostgreSQL — read-only
- **Busca fuzzy:** `pg_trgm`
- **Busca vetorial:** `pgvector`

### Inteligência Artificial

- **OpenAI API**
- **Responses API**
- **Structured Outputs / JSON Schema**
- **Modelos multimodais**
- **Embeddings**

### Leitura de Documentos

- **Excel:** `openpyxl`
- **Word:** `python-docx`
- **PDF:** `PyMuPDF`
- **Arquivos escaneados:** visão multimodal quando necessário

### Qualidade e Testes

- **Testes:** `pytest`
- **Lint / formatter:** `Ruff`
- **Type checking:** `mypy` de forma progressiva
- **Coverage:** `coverage.py` / integração com pytest

### Infraestrutura e Desenvolvimento

- **IDE principal:** Visual Studio Code
- **Banco / inspeção:** DBeaver
- **Containers:** Docker
- **Orquestração local:** Docker Compose
- **Controle de versão:** Git
- **Repositório:** GitHub
- **CI/CD planejado:** GitHub Actions

---

## 📦 Escopo do MVP

O primeiro MVP deverá permitir:

```text
✅ receber um pedido real
✅ identificar o formato do arquivo
✅ extrair os itens solicitados
✅ obter código quando informado
✅ obter quantidade
✅ obter unidade
✅ preservar a descrição original
✅ normalizar a descrição
✅ consultar o catálogo oficial somente leitura
✅ localizar correspondências por código
✅ localizar correspondências por texto
✅ localizar candidatos fuzzy
✅ localizar candidatos semanticamente
✅ utilizar IA quando necessário
✅ classificar resultado
✅ permitir revisão humana
✅ registrar decisão
✅ armazenar memória por cliente/embarcação
✅ registrar substituições aceitas/rejeitadas
```

Não faz parte inicialmente:

```text
❌ alterar produtos no banco oficial do cliente
❌ compra automática em fornecedor
❌ envio automático irrestrito de respostas ao marinheiro
❌ aprovação automática de matching ambíguo
❌ cadastro completo de fornecedores
❌ automação integral sem supervisão humana
```

Esses itens poderão ser avaliados em fases posteriores.

---

## 🔮 Roadmap

### P0 — Arquitetura e Regras

Definir:

- problema;
- fluxo;
- fronteiras dos bancos;
- segurança;
- cronograma;
- documentação;
- versionamento.

**Status:** ✅ concluído inicialmente.

### P1 — Banco Próprio e Modelo de Dados

Definir:

- entidades;
- relacionamentos;
- campos;
- tipos;
- índices;
- constraints;
- auditoria;
- migrations.

### P2 — Entrada e Leitura de Documentos

Implementar leitores para:

- PDF;
- PDF escaneado;
- Excel;
- Word;
- imagem.

### P3 — Extração e Normalização

Transformar arquivos em estruturas canônicas:

```text
produto
código
quantidade
unidade
marca
embalagem
peso/volume
observações
confiança
```

### P4 — Integração com Catálogo Oficial

Conectar ao PostgreSQL do cliente com credencial **read-only**.

### P5 — Motor de Matching

Implementar:

- código exato;
- aliases;
- texto;
- fuzzy;
- embeddings;
- ranking;
- comparação por IA.

### P6 — Memória

Adicionar memória por:

- cliente;
- embarcação;
- expressão utilizada;
- produto confirmado;
- histórico de substituições;
- confirmações;
- rejeições.

### P7 — Interface de Revisão

Criar a interface operacional para revisão e decisão humana.

### P8 — Testes, Métricas e Calibração

Validar com pedidos reais e medir:

- precisão da extração;
- precisão do matching;
- falsos positivos;
- itens resolvidos sem IA;
- itens resolvidos pela memória;
- necessidade de intervenção humana;
- tempo médio de processamento.

### P9 — Automação

Avaliar posteriormente:

- recebimento automático de anexos;
- integração com e-mail;
- notificações;
- respostas assistidas;
- pesquisa de fornecedores;
- fluxos de compra;
- outras integrações operacionais.

---

## 📁 Estrutura Planejada do Repositório

```text
vessel-order-intelligence/
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── ai/
│   │   ├── core/
│   │   ├── db/
│   │   ├── document_readers/
│   │   ├── integrations/
│   │   │   └── client_catalog/
│   │   ├── models/
│   │   ├── schemas/
│   │   └── services/
│   │
│   ├── alembic/
│   ├── tests/
│   └── pyproject.toml
│
├── docs/
│   └── PROJETO_PEDIDOS_INTELIGENTES_EMBARCACOES.md
│
├── frontend/
│
├── .github/
│   └── workflows/
│
├── .vscode/
│   ├── extensions.json
│   └── settings.json
│
├── .env.example
├── .gitignore
├── docker-compose.yml
├── README.md
└── CHANGELOG.md
```

A estrutura poderá evoluir conforme as fases forem implementadas.

---

## 🌿 Estratégia Git

Branches iniciais:

```text
main
feature/*
fix/*
docs/*
```

A branch `main` deverá permanecer estável.

Exemplos de commits:

```text
chore: initialize project structure
docs: define initial architecture
feat: add customer data model
feat: implement pdf reader
feat: add client catalog read-only integration
feat: implement product matching
test: validate extraction pipeline
fix: prevent ambiguous match auto-approval
```

---

## 🏁 Como Executar o Projeto

> O projeto ainda está na fase inicial. Os comandos abaixo representam a estrutura prevista e serão atualizados conforme a implementação avançar.

### 1. Pré-requisitos

- Python 3.12+ recomendado
- Git
- Docker Desktop
- PostgreSQL ou Docker para o banco local
- Visual Studio Code
- acesso ao repositório privado

### 2. Clone o Repositório

```bash
git clone https://github.com/RodrigoCosta1983/vessel-order-intelligence.git

cd vessel-order-intelligence
```

Se o repositório estiver privado, será necessário estar autenticado no GitHub.

### 3. Crie o Ambiente Virtual

Windows / PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Linux / macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 4. Variáveis de Ambiente

Copie:

```text
.env.example
```

para:

```text
.env
```

Exemplo de configuração:

```env
APP_ENV=development

DATABASE_URL=

CLIENT_DATABASE_URL=

OPENAI_API_KEY=
OPENAI_EXTRACTION_MODEL=
OPENAI_MATCHING_MODEL=
OPENAI_EMBEDDING_MODEL=
```

Nunca envie o `.env` real ao GitHub.

### 5. Dependências

A instalação oficial será documentada assim que o `pyproject.toml` for criado durante a preparação do backend.

---

## 📚 Documentação

O projeto utiliza documentação viva.

Documento principal:

```text
docs/PROJETO_PEDIDOS_INTELIGENTES_EMBARCACOES.md
```

Regra:

> Toda decisão arquitetural relevante, alteração de tecnologia, regra de negócio, limitação conhecida e etapa concluída deverá ser registrada na documentação.

O README apresenta a visão pública/técnica resumida.

A documentação em `docs/` mantém o histórico arquitetural detalhado.

---

## 👨‍💻 Autor

**RodrigoCostaDEV**

GitHub: [@RodrigoCosta1983](https://github.com/RodrigoCosta1983)

LinkedIn: [RodrigoCostaDEV](https://www.linkedin.com/in/dev-rodrigo-costa/)

Website: [storeconnect.com.br](https://www.storeconnect.com.br)

---

# <a name="english-version"></a> English Version

[Leia em Português](#vessel-order-intelligence--pedidos-inteligentes-para-embarcações)

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-412991?style=for-the-badge&logo=openai&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)

**Vessel Order Intelligence** is an intelligent system for reading, interpreting, normalizing, and processing supply orders received from vessels.

The project is designed to accept orders in multiple formats — including **PDF, scanned PDF, Excel, Word, and images** — automatically identify the requested items, and search for matches in the customer's official product catalog.

The official catalog remains in an external **PostgreSQL database with read-only access**. All operational history, decisions, customer/vessel memory, aliases, substitutions, and AI-generated information are stored exclusively in the system's own PostgreSQL database.

> The final commercial product name has not yet been defined.  
> `Vessel Order Intelligence` is currently the technical project and repository name.

---

## 📌 Current Status

```text
✅ P0 — Initial architecture and core rules
    ✅ main order flow defined
    ✅ customer's official PostgreSQL identified
    ✅ official database defined as READ ONLY
    ✅ minimum official catalog fields defined
    ✅ system-owned PostgreSQL defined
    ✅ customer/vessel memory concept defined
    ✅ substitution history concept defined
    ✅ human-in-the-loop principle defined
    ✅ macro roadmap defined
    ✅ initial technical documentation created
    ✅ Git/GitHub version control strategy defined

🟡 P1 — System database and data model
    ⏳ P1-A — define canonical entities
    ⏳ P1-B — define relationships
    ⏳ P1-C — define fields and types
    ⏳ P1-D — define indexes and constraints
    ⏳ P1-E — define auditing and traceability
    ⏳ P1-F — create first Alembic migration

⏳ P2 — Document input and reading
⏳ P3 — Intelligent extraction and normalization
⏳ P4 — Read-only official catalog integration
⏳ P5 — Matching engine
⏳ P6 — Customer and vessel memory
⏳ P7 — Human review and decision interface
⏳ P8 — Real-world testing, metrics, and calibration
⏳ P9 — Future automations and integrations
```

Detailed living documentation is available at:

```text
docs/PROJETO_PEDIDOS_INTELIGENTES_EMBARCACOES.md
```

---

## 🎯 Project Goal

Automate and assist the interpretation of supply orders sent by sailors or vessel representatives before docking.

Main flow:

```text
SAILOR / VESSEL
        ↓
sends order
PDF / Excel / Word / image
        ↓
DOCUMENT READER
        ↓
EXTRACTION AND NORMALIZATION
        ↓
product / code / quantity / unit
        ↓
CUSTOMER / VESSEL MEMORY
        ↓
MATCHING ENGINE
        ↓
CUSTOMER OFFICIAL CATALOG
PostgreSQL — READ ONLY
        ↓
FOUND / PROBABLE / SIMILAR / NOT FOUND
        ↓
HUMAN REVIEW
        ↓
DECISION
        ↓
SYSTEM HISTORY + MEMORY
```

AI should mainly assist with **new, incomplete, or ambiguous cases**.

As a customer uses the system repeatedly, previous confirmed decisions can be reused before running a full search again.

---

## ✨ Planned Features

The project is under development. The items below describe the planned MVP and future phases.

### 📄 Order Reading

- **Text-based PDF:** structured text extraction.
- **Scanned PDF:** visual interpretation using a multimodal model.
- **Excel:** spreadsheet and table parsing.
- **Word:** table and text parsing.
- **Images:** planned support for photographed or captured orders.
- **Inconsistent structures:** the system will not depend on a single spreadsheet template.

### 🧠 Intelligent Extraction

For each order line, the system will try to identify:

- original description;
- normalized description;
- provided code, when available;
- requested quantity;
- unit;
- brand, when identifiable;
- package type;
- weight or volume;
- relevant notes;
- extraction confidence.

Conceptual example:

```json
{
  "original_description": "SAMOSA CURRY VEGETAL",
  "normalized_description": "vegetable curry samosa",
  "code": null,
  "quantity": 120,
  "unit": "KG",
  "brand": null,
  "package": null,
  "extraction_confidence": 0.97
}
```

### 🔎 Product Search and Matching

Planned search priority:

```text
1. exact code
2. vessel-specific memory
3. customer memory
4. aliases / previously confirmed knowledge
5. textual search
6. fuzzy search
7. semantic search with embeddings
8. AI compares the best candidates
```

The goal is to avoid unnecessary AI usage when the system already knows the answer.

### 🟢 Result Classification

Each item may receive one of the following statuses:

```text
🟢 FOUND
→ clear match or exact code

🟡 PROBABLE
→ strong candidate, requires confirmation

🟠 SIMILAR
→ exact item not found, but alternatives exist

🔴 NOT FOUND
→ no reliable match in the catalog
```

### 🧠 Customer and Vessel Memory

The system will maintain persistent memory in its own database.

Example:

```text
Customer A
"SAMOSA CURRY VEGETAL"
        ↓
internal product 88451
        ↓
confirmed 5 times
rejected 0 times
```

For future orders from the same customer, previous knowledge can be reused before performing a full search.

### 🔄 Substitution History

Accepted and rejected substitutions will also be recorded.

Example:

```text
Requested product:
HEINZ KETCHUP 5 KG

Substitute:
HELLMANN'S KETCHUP 5 KG

Accepted:
4 times

Rejected:
0 times
```

A previous acceptance **does not automatically become permanent authorization**. Automation rules will remain configurable and auditable.

### 👤 Human Review

The system follows a **human-in-the-loop** principle.

Ambiguous matches will not be treated as authoritative merely because an AI suggested them.

An operator will be able to:

- confirm;
- reject;
- correct;
- select another product;
- mark as unavailable;
- register a substitution;
- add information to the history.

---

## 🏗️ Base Architecture

### Official Product Source

The customer's existing catalog remains the operational source of truth.

```text
CUSTOMER OFFICIAL POSTGRESQL
        ↓
READ ONLY
        ↓
code
product
available quantity
```

Architectural rule:

```text
CUSTOMER DATABASE
→ official catalog source

OUR DATABASE
→ operations + intelligence + memory + history
```

### Official Database — Never Write

The application must use a credential restricted to read operations.

```text
✅ SELECT

❌ INSERT
❌ UPDATE
❌ DELETE
❌ ALTER
❌ DROP
```

Whenever possible, the preferred integration is a dedicated database `VIEW` exposing only the required fields.

### System PostgreSQL

The system-owned database will store information such as:

```text
customers
vessels
orders
order_items
indexed_catalog
customer_memory
vessel_memory
product_aliases
substitutions
decisions
ai_processes
audit
```

The definitive model will be defined during P1.

---

## 🔒 Security and Data Isolation

Core principles:

- The customer's official PostgreSQL is **read only**.
- AI cannot modify the official catalog.
- Credentials are never committed to Git.
- The real `.env` file stays outside the repository.
- Only `.env.example` is versioned.
- OpenAI keys are never written to logs or documentation.
- AI-generated data is not treated as official data without validation.
- Relevant decisions must remain traceable.
- Uncertain matches require human review.
- The system should fail safely when confidence is insufficient.

---

## 🤖 AI Strategy

The architecture plans to use the OpenAI API for specific tasks.

### Extraction

```text
document
    ↓
multimodal / text model
    ↓
Structured Output
    ↓
validated JSON
```

### Matching

AI will not receive the entire catalog indiscriminately.

Planned flow:

```text
normalized item
    ↓
PostgreSQL search
    ↓
Top N candidates
    ↓
AI compares only the best candidates
    ↓
result + confidence + explanation
```

### Environment Configuration

Models will not be hard-coded into the business logic.

Example:

```env
OPENAI_EXTRACTION_MODEL=
OPENAI_MATCHING_MODEL=
OPENAI_EMBEDDING_MODEL=
```

This allows model upgrades without rebuilding the application architecture.

---

## 🧩 Planned Technology Stack

### Backend

- **Language:** Python
- **Framework:** FastAPI
- **Validation and schemas:** Pydantic
- **ORM:** SQLAlchemy 2
- **Migrations:** Alembic
- **ASGI Server:** Uvicorn

### Database

- **System database:** PostgreSQL
- **Customer official database:** PostgreSQL — read-only
- **Fuzzy search:** `pg_trgm`
- **Vector search:** `pgvector`

### Artificial Intelligence

- **OpenAI API**
- **Responses API**
- **Structured Outputs / JSON Schema**
- **Multimodal models**
- **Embeddings**

### Document Processing

- **Excel:** `openpyxl`
- **Word:** `python-docx`
- **PDF:** `PyMuPDF`
- **Scanned files:** multimodal vision when required

### Quality and Testing

- **Tests:** `pytest`
- **Lint / formatter:** `Ruff`
- **Type checking:** progressively with `mypy`
- **Coverage:** `coverage.py` / pytest integration

### Infrastructure and Development

- **Primary IDE:** Visual Studio Code
- **Database inspection:** DBeaver
- **Containers:** Docker
- **Local orchestration:** Docker Compose
- **Version control:** Git
- **Repository:** GitHub
- **Planned CI/CD:** GitHub Actions

---

## 📦 MVP Scope

The first MVP should support:

```text
✅ receive a real-world order
✅ identify its file format
✅ extract requested items
✅ obtain code when provided
✅ obtain quantity
✅ obtain unit
✅ preserve original description
✅ normalize description
✅ query the official catalog read-only
✅ find exact code matches
✅ find text matches
✅ find fuzzy candidates
✅ find semantic candidates
✅ use AI when necessary
✅ classify result
✅ allow human review
✅ store the decision
✅ maintain customer/vessel memory
✅ record accepted/rejected substitutions
```

Initially out of scope:

```text
❌ modifying products in the customer's official database
❌ automatic supplier purchasing
❌ unrestricted automatic replies to vessels
❌ automatic approval of ambiguous matches
❌ full supplier management
❌ fully autonomous operation without human supervision
```

These items may be evaluated in later phases.

---

## 🔮 Roadmap

### P0 — Architecture and Rules

Define:

- problem;
- workflow;
- database boundaries;
- security;
- roadmap;
- documentation;
- version control.

**Status:** ✅ initially completed.

### P1 — System Database and Data Model

Define:

- entities;
- relationships;
- fields;
- types;
- indexes;
- constraints;
- audit model;
- migrations.

### P2 — Document Input and Reading

Implement readers for:

- PDF;
- scanned PDF;
- Excel;
- Word;
- images.

### P3 — Extraction and Normalization

Transform files into canonical structures:

```text
product
code
quantity
unit
brand
package
weight/volume
notes
confidence
```

### P4 — Official Catalog Integration

Connect to the customer's PostgreSQL using a **read-only** credential.

### P5 — Matching Engine

Implement:

- exact code;
- aliases;
- text;
- fuzzy matching;
- embeddings;
- ranking;
- AI candidate comparison.

### P6 — Memory

Add persistent knowledge by:

- customer;
- vessel;
- received expression;
- confirmed product;
- substitution history;
- confirmations;
- rejections.

### P7 — Review Interface

Build the operational interface for human review and decision-making.

### P8 — Testing, Metrics and Calibration

Validate using real orders and measure:

- extraction accuracy;
- matching accuracy;
- false positives;
- items solved without AI;
- items solved through memory;
- human intervention rate;
- average processing time.

### P9 — Automation

Evaluate later:

- automatic attachment intake;
- email integration;
- notifications;
- assisted replies;
- supplier search;
- purchasing workflows;
- other operational integrations.

---

## 📁 Planned Repository Structure

```text
vessel-order-intelligence/
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── ai/
│   │   ├── core/
│   │   ├── db/
│   │   ├── document_readers/
│   │   ├── integrations/
│   │   │   └── client_catalog/
│   │   ├── models/
│   │   ├── schemas/
│   │   └── services/
│   │
│   ├── alembic/
│   ├── tests/
│   └── pyproject.toml
│
├── docs/
│   └── PROJETO_PEDIDOS_INTELIGENTES_EMBARCACOES.md
│
├── frontend/
│
├── .github/
│   └── workflows/
│
├── .vscode/
│   ├── extensions.json
│   └── settings.json
│
├── .env.example
├── .gitignore
├── docker-compose.yml
├── README.md
└── CHANGELOG.md
```

The repository structure may evolve as implementation progresses.

---

## 🌿 Git Strategy

Initial branches:

```text
main
feature/*
fix/*
docs/*
```

The `main` branch should remain stable.

Commit examples:

```text
chore: initialize project structure
docs: define initial architecture
feat: add customer data model
feat: implement pdf reader
feat: add client catalog read-only integration
feat: implement product matching
test: validate extraction pipeline
fix: prevent ambiguous match auto-approval
```

---

## 🏁 Getting Started

> The project is still in its initial phase. The commands below represent the planned setup and will be updated as implementation progresses.

### 1. Prerequisites

- Python 3.12+ recommended
- Git
- Docker Desktop
- PostgreSQL or Docker for the local database
- Visual Studio Code
- access to the private repository

### 2. Clone the Repository

```bash
git clone https://github.com/RodrigoCosta1983/vessel-order-intelligence.git

cd vessel-order-intelligence
```

If the repository is private, GitHub authentication is required.

### 3. Create the Virtual Environment

Windows / PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Linux / macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 4. Environment Variables

Copy:

```text
.env.example
```

to:

```text
.env
```

Example:

```env
APP_ENV=development

DATABASE_URL=

CLIENT_DATABASE_URL=

OPENAI_API_KEY=
OPENAI_EXTRACTION_MODEL=
OPENAI_MATCHING_MODEL=
OPENAI_EMBEDDING_MODEL=
```

Never commit the real `.env` file.

### 5. Dependencies

The official installation command will be added once `pyproject.toml` is created during backend setup.

---

## 📚 Documentation

This project follows a living-documentation approach.

Main document:

```text
docs/PROJETO_PEDIDOS_INTELIGENTES_EMBARCACOES.md
```

Rule:

> Every relevant architectural decision, technology change, business rule, known limitation, and completed phase must be recorded in the documentation.

The README provides the summarized public/technical overview.

The `docs/` directory contains detailed architectural history.

---

## 👨‍💻 Developer

**RodrigoCostaDEV**

GitHub: [@RodrigoCosta1983](https://github.com/RodrigoCosta1983)

LinkedIn: [RodrigoCostaDEV](https://www.linkedin.com/in/dev-rodrigo-costa/)

Website: [storeconnect.com.br](https://www.storeconnect.com.br)
