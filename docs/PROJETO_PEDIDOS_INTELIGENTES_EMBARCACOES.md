qul# 🚢 PROJETO — PEDIDOS INTELIGENTES PARA EMBARCAÇÕES

Documentação oficial da arquitetura, decisões, segurança, implementação, testes e evolução do sistema inteligente de leitura e tratamento de pedidos recebidos de embarcações.

> Este documento é a referência técnica principal do projeto.
> Toda decisão arquitetural relevante, mudança de tecnologia, regra de negócio, etapa concluída e limitação conhecida deverá ser registrada aqui.

**Status do documento:** versão inicial da arquitetura — v0.1.0  
**Última atualização:** 11/09/2026  
**Nome comercial do produto:** ainda não definido  
**Nome técnico provisório do repositório:** `pedidos-inteligentes-embarcacoes`

=====================================================================

# 📌 STATUS GERAL DO PROJETO

```text
✅ P0 — Definição inicial do problema, arquitetura e regras centrais
    ✅ fluxo principal do pedido definido
    ✅ PostgreSQL oficial do cliente identificado
    ✅ banco oficial definido como SOMENTE LEITURA
    ✅ dados mínimos do catálogo oficial definidos
    ✅ banco próprio do sistema definido
    ✅ estratégia de memória por cliente/embarcação definida
    ✅ histórico de substituições aceitas/rejeitadas definido conceitualmente
    ✅ princípio human-in-the-loop definido
    ✅ cronograma macro definido

🟡 P1 — Banco próprio e modelo de dados
    ⏳ P1-A — definir entidades canônicas
    ⏳ P1-B — definir relacionamentos
    ⏳ P1-C — definir campos e tipos
    ⏳ P1-D — definir índices e constraints
    ⏳ P1-E — definir auditoria e rastreabilidade
    ⏳ P1-F — gerar primeira migration

⏳ P2 — Entrada e leitura de documentos
⏳ P3 — Extração e normalização inteligente
⏳ P4 — Integração somente leitura com o PostgreSQL oficial
⏳ P5 — Motor de matching de produtos
⏳ P6 — Memória por cliente/embarcação e substituições
⏳ P7 — Interface de revisão e decisão humana
⏳ P8 — Testes com pedidos reais, métricas e calibração
⏳ P9 — Automações e integrações posteriores
```

=====================================================================

# 🎯 OBJETIVO DO PROJETO

Criar um sistema capaz de receber pedidos de abastecimento enviados por marinheiros ou responsáveis por embarcações em formatos não padronizados e transformá-los em uma estrutura confiável para consulta do catálogo oficial do cliente.

O sistema deverá:

```text
PEDIDO RECEBIDO
PDF / PDF escaneado / Excel / Word / imagem
        ↓
LEITURA DO DOCUMENTO
        ↓
EXTRAÇÃO DOS ITENS
        ↓
NORMALIZAÇÃO
        ↓
MEMÓRIA DO CLIENTE / EMBARCAÇÃO
        ↓
BUSCA NO CATÁLOGO OFICIAL
        ↓
MATCHING INTELIGENTE
        ↓
ENCONTRADO / PROVÁVEL / SIMILAR / NÃO ENCONTRADO
        ↓
REVISÃO HUMANA QUANDO NECESSÁRIO
        ↓
DECISÃO REGISTRADA
        ↓
MEMÓRIA ATUALIZADA
```

O objetivo não é permitir que a IA substitua a base oficial de produtos.

O objetivo é utilizar IA e mecanismos tradicionais de busca para interpretar o pedido, localizar candidatos e ajudar o operador a tomar uma decisão rastreável.

=====================================================================

# 🧭 PRINCÍPIOS ARQUITETURAIS

## 1. Banco oficial do cliente é fonte operacional de verdade

O PostgreSQL já existente no cliente continuará sendo a autoridade sobre os produtos cadastrados.

Nosso sistema deverá acessá-lo exclusivamente para leitura.

```text
POSTGRESQL OFICIAL DO CLIENTE
        ↓
      SELECT
        ↓
NOSSA APLICAÇÃO
```

Nenhum fluxo do nosso sistema poderá executar no banco oficial:

```text
❌ INSERT
❌ UPDATE
❌ DELETE
❌ ALTER
❌ DROP
❌ CREATE de objetos operacionais sem autorização explícita do cliente
```

Preferência de segurança:

```text
usuário PostgreSQL exclusivo
        +
permissão somente SELECT
        +
VIEW específica quando possível
```

=====================================================================

## 2. Dados mínimos lidos do catálogo oficial

Inicialmente, nosso sistema precisa apenas de:

```text
codigo
produto / descrição
quantidade disponível
```

O nome exato das colunas será confirmado na etapa P4.

A aplicação deverá normalizar internamente os nomes para algo equivalente a:

```text
external_product_code
external_product_name
available_quantity
```

IMPORTANTE:

```text
quantidade disponível
≠
quantidade solicitada pelo marinheiro
```

São conceitos diferentes e deverão permanecer separados no nosso modelo.

=====================================================================

## 3. Nosso sistema possui seu próprio banco

Todo dado criado pela aplicação ficará no nosso PostgreSQL.

Isso inclui:

```text
✅ clientes
✅ embarcações
✅ pedidos
✅ arquivos recebidos e seus metadados
✅ itens extraídos
✅ texto original
✅ texto normalizado
✅ interpretação da IA
✅ candidatos encontrados
✅ scores de matching
✅ decisões do operador
✅ aliases
✅ memória por cliente
✅ memória por embarcação
✅ substituições aceitas
✅ substituições rejeitadas
✅ histórico de processamento
✅ auditoria
✅ versões de prompts/modelos quando necessário
```

Regra:

> Nenhum conhecimento gerado pela IA será gravado no banco oficial do cliente.

=====================================================================

## 4. A IA não é a autoridade sobre a existência do produto

A IA pode afirmar:

```text
"Há forte indicação de que o item solicitado corresponde ao código 108035."
```

Mas a existência do código 108035 deve ser confirmada pela consulta ao catálogo oficial.

```text
IA
→ interpreta e recomenda

CATÁLOGO OFICIAL
→ confirma a existência do produto e sua disponibilidade atual
```

=====================================================================

## 5. Dúvida relevante exige revisão humana

A aplicação não deverá transformar automaticamente um match incerto em produto confirmado.

Estados iniciais planejados:

```text
🟢 EXACT / ENCONTRADO
🟡 LIKELY / PROVÁVEL
🟠 SIMILAR
🔴 NOT_FOUND / NÃO ENCONTRADO
⚪ REVIEW_REQUIRED / REVISÃO NECESSÁRIA
```

Os limites de confiança não serão definidos arbitrariamente agora.

Eles serão calibrados na fase P8 com pedidos reais.

=====================================================================

# 🧠 MEMÓRIA DO CLIENTE E DA EMBARCAÇÃO

Uma das capacidades centrais do sistema será reaproveitar conhecimento confirmado anteriormente.

Hierarquia planejada:

```text
CONHECIMENTO GLOBAL
        ↓
CLIENTE
        ↓
EMBARCAÇÃO
```

A memória mais específica deverá ter prioridade sobre a mais genérica.

Exemplo:

```text
Cliente A
Embarcação X

texto recebido:
"SAMOSA CURRY VEGETAL"

produto confirmado:
88451 — SAMOSA VEGETABLE CURRY 1KG
```

Em um pedido futuro da mesma embarcação, a aplicação poderá consultar essa memória antes de executar uma pesquisa completa.

=====================================================================

# 🔁 MEMÓRIA DE SUBSTITUIÇÕES

Substituições deverão ser armazenadas separadamente dos aliases de identificação.

Exemplo:

```text
produto solicitado:
HEINZ KETCHUP 5KG

produto oferecido:
HELLMANN'S KETCHUP 5KG

cliente aceitou:
3 vezes

cliente recusou:
0 vezes
```

Uma aceitação anterior NÃO significa automaticamente autorização permanente.

O sistema deverá inicialmente tratar o histórico como evidência de preferência:

```text
1 aceitação
→ histórico existente

várias aceitações sem rejeição
→ preferência forte

regra automática de substituição
→ somente se existir autorização/configuração comercial explícita
```

Essa regra evita transformar uma decisão antiga em uma obrigação futura incorreta.

=====================================================================

# 🔎 ORDEM PRELIMINAR DO MATCHING

A pesquisa de um item deverá priorizar métodos mais determinísticos e baratos antes de usar IA para casos ambíguos.

Ordem inicial:

```text
1. código explícito válido no pedido
        ↓
2. memória específica da embarcação
        ↓
3. memória do cliente
        ↓
4. alias global confirmado
        ↓
5. correspondência textual exata/normalizada
        ↓
6. busca fuzzy
        ↓
7. busca semântica por embeddings
        ↓
8. IA compara apenas os melhores candidatos
        ↓
9. revisão humana quando necessário
```

Objetivo:

> Quanto mais o sistema for usado e confirmado, menos processamento caro será necessário para pedidos repetidos.

=====================================================================

# 📦 ESCOPO DO MVP

O MVP deverá permitir:

```text
✅ cadastrar/identificar cliente
✅ cadastrar/identificar embarcação quando aplicável
✅ receber arquivo de pedido
✅ aceitar PDF
✅ aceitar PDF escaneado
✅ aceitar Excel
✅ aceitar Word
✅ aceitar imagem quando necessário
✅ extrair itens do pedido
✅ preservar o texto original
✅ identificar código quando existir
✅ identificar descrição
✅ identificar quantidade solicitada
✅ identificar unidade
✅ normalizar descrição sem apagar o original
✅ consultar catálogo oficial em modo somente leitura
✅ encontrar produto por código
✅ buscar produto por descrição
✅ buscar por similaridade textual
✅ buscar por similaridade semântica
✅ classificar resultado
✅ apresentar candidatos ao operador
✅ registrar confirmação/correção
✅ guardar memória do cliente/embarcação
✅ guardar substituições aceitas/rejeitadas
✅ manter rastreabilidade da decisão
```

=====================================================================

# 🚫 FORA DO ESCOPO INICIAL

Não faz parte do primeiro MVP:

```text
❌ gravar no PostgreSQL oficial do cliente
❌ alterar cadastro oficial de produto
❌ reservar estoque automaticamente
❌ emitir compra automaticamente para fornecedor
❌ responder automaticamente ao marinheiro sem regra validada
❌ aceitar substituição automaticamente sem política explícita
❌ pesquisar fornecedor de forma autônoma
❌ efetuar pagamento
❌ controlar financeiro completo
❌ integrar automaticamente e-mail/WhatsApp antes do núcleo estar validado
❌ treinar um modelo próprio de IA no primeiro momento
```

Esses itens podem entrar no backlog e ser promovidos para uma fase futura.

=====================================================================

# 🏗️ ARQUITETURA BASE

```text
                    ┌─────────────────────────┐
                    │ DOCUMENTO DO MARINHEIRO │
                    │ PDF / XLSX / DOCX / IMG │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ INGESTÃO DE DOCUMENTOS  │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ EXTRAÇÃO E NORMALIZAÇÃO │
                    │ código / produto / qtd  │
                    │ unidade / atributos     │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ NOSSO POSTGRESQL        │
                    │ histórico + memória     │
                    └────────────┬────────────┘
                                 │
              ┌──────────────────┴──────────────────┐
              │                                     │
              ▼                                     ▼
     ┌────────────────────┐              ┌──────────────────────┐
     │ MEMÓRIA HISTÓRICA  │              │ MOTOR DE MATCHING    │
     │ cliente/embarcação │              │ texto/fuzzy/vector   │
     └──────────┬─────────┘              └──────────┬───────────┘
                │                                   │
                └────────────────┬──────────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ POSTGRESQL DO CLIENTE   │
                    │ SOMENTE LEITURA         │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ CANDIDATOS DO CATÁLOGO  │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ IA / RERANK QUANDO      │
                    │ NECESSÁRIO              │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ REVISÃO DO OPERADOR     │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │ DECISÃO + NOVA MEMÓRIA  │
                    └─────────────────────────┘
```

=====================================================================

# 🧰 TECNOLOGIAS ADOTADAS / PLANEJADAS

## Backend

```text
Python
FastAPI
Pydantic
SQLAlchemy 2
Alembic
Uvicorn
```

Responsabilidades:

```text
API interna
upload e validação de documentos
orquestração da extração
integração com IA
consulta ao banco oficial
matching
memória
regras de negócio
auditoria
```

=====================================================================

## Banco de dados próprio

```text
PostgreSQL
```

Extensões planejadas:

```text
pg_trgm
→ similaridade textual / fuzzy

pgvector
→ busca semântica por embeddings
```

O schema definitivo será desenhado na P1.

=====================================================================

## Banco oficial do cliente

```text
PostgreSQL
modo: READ ONLY
```

Recomendação operacional:

```text
usuário exclusivo
+ SELECT apenas
+ VIEW específica com os campos necessários, quando viável
```

=====================================================================

## Inteligência Artificial

```text
OpenAI API
Responses API
Structured Outputs com JSON Schema
entrada multimodal para documentos/imagens quando necessário
Embeddings para busca semântica
```

Regra de manutenção:

> O projeto não deverá espalhar IDs de modelos diretamente pela regra de negócio.

Os modelos deverão ser configuráveis por ambiente, por exemplo:

```text
OPENAI_EXTRACTION_MODEL=
OPENAI_MATCHING_MODEL=
OPENAI_EMBEDDING_MODEL=
```

Isso permitirá atualizar modelos sem alterar o núcleo da aplicação.

=====================================================================

## Leitura de documentos

### Excel

```text
openpyxl
```

`pandas` poderá ser usado quando facilitar a manipulação de tabelas, mas não será requisito obrigatório para todo arquivo.

### Word

```text
python-docx
```

### PDF

```text
PyMuPDF
```

Fluxo planejado:

```text
PDF com texto utilizável
→ extração tradicional primeiro

PDF escaneado / tabela visual difícil
→ renderização das páginas necessárias
→ análise multimodal
```

Não adotaremos OCR tradicional como dependência obrigatória antes de comprovar necessidade.

=====================================================================

## Busca e matching

```text
PostgreSQL SQL
pg_trgm
pgvector
OpenAI embeddings
IA como reranker/analista dos melhores candidatos
```

Não será utilizada uma estratégia de "enviar o catálogo inteiro para a IA".

=====================================================================

## Testes e qualidade

Planejado:

```text
pytest
pytest-asyncio quando necessário
Ruff
mypy — adoção progressiva
coverage
```

Princípio:

```text
parser
matching
memória
integração read-only
regras de substituição

→ deverão possuir testes independentes
```

=====================================================================

## Infraestrutura

Planejado:

```text
Docker
Docker Compose para desenvolvimento local
variáveis de ambiente
secret manager no ambiente de produção
```

O provedor de cloud/deploy ainda NÃO está definido.

Essa decisão será tomada somente quando conhecermos:

```text
volume de pedidos
volume de arquivos
necessidade de alta disponibilidade
restrições do cliente
custos aceitáveis
```

=====================================================================

## Frontend

A tecnologia da interface ainda NÃO está congelada.

Requisitos já definidos:

```text
interface web interna
upload de arquivos
tabela de itens extraídos
comparação entre solicitado e encontrado
visualização de confiança
confirmação/correção humana
histórico do cliente
histórico da embarcação
substituições anteriores
```

Decisão de framework será registrada antes da P7.

=====================================================================

# 📄 CONTRATO CANÔNICO PRELIMINAR DO ITEM EXTRAÍDO

Exemplo conceitual:

```json
{
  "line_number": 1,
  "original_description": "SAMOSA CURRY VEGETAL",
  "normalized_description": "samosa vegetal com curry",
  "provided_code": null,
  "requested_quantity": 120,
  "requested_unit": "KG",
  "brand": null,
  "package_type": null,
  "package_size": null,
  "language": "es",
  "extraction_confidence": 0.97
}
```

Esse contrato será formalizado com Pydantic + JSON Schema na P3.

Regra obrigatória:

> O valor original nunca deverá ser apagado pela interpretação normalizada.

=====================================================================

# 🗃️ ENTIDADES PRELIMINARES DO NOSSO BANCO

O desenho detalhado será feito na P1, mas as entidades conceituais já identificadas são:

```text
clients
vessels
orders
order_files
order_items
catalog_product_index
product_aliases
client_product_memory
vessel_product_memory
product_substitutions
match_candidates
operator_decisions
ai_processing_runs
audit_events
```

Esses nomes ainda podem sofrer ajustes durante a modelagem.

Nenhuma tabela será considerada definitiva até concluirmos P1-C e P1-D.

=====================================================================

# 🔐 SEGURANÇA

## Banco oficial

Obrigatório:

```text
✅ credencial exclusiva para integração
✅ privilégios mínimos
✅ somente leitura
✅ conexão criptografada quando suportada
✅ nenhuma senha no código
✅ nenhum dado gerado pela IA gravado no banco oficial
```

=====================================================================

## Segredos

Nunca deverão ser versionados:

```text
OPENAI_API_KEY
DATABASE_URL
CLIENT_DATABASE_URL
senhas
certificados privados
tokens
segredos de produção
```

O repositório deverá possuir:

```text
.env.example
```

E ignorar:

```text
.env
.env.*.local
```

=====================================================================

## Documentos reais

Pedidos reais usados em testes deverão ser tratados com cuidado.

Antes de colocar qualquer exemplo no GitHub:

```text
✅ anonimizar quando necessário
✅ remover credenciais
✅ remover informações comerciais que não precisem estar no repositório
✅ preferir fixtures sintéticas para testes automatizados
```

O repositório NÃO será utilizado como armazenamento de pedidos de produção.

=====================================================================

# 🧾 RASTREABILIDADE DAS DECISÕES

Para permitir manutenção futura, uma decisão importante deverá poder responder:

```text
O que o marinheiro escreveu?
O que a aplicação extraiu?
Como o texto foi normalizado?
Qual catálogo foi consultado?
Quais candidatos foram retornados?
Qual score cada candidato recebeu?
A IA participou?
Qual configuração/modelo foi usado?
O operador confirmou ou corrigiu?
Qual produto foi finalmente escolhido?
Houve substituição?
O cliente aceitou ou recusou?
Quando ocorreu?
```

Esse histórico será um requisito de arquitetura, e não apenas um log de debug.

=====================================================================

# 📊 MÉTRICAS QUE SERÃO MEDIDAS NA P8

O projeto não considerará "funciona" apenas pela impressão visual.

Métricas previstas:

```text
taxa de itens extraídos corretamente
taxa de códigos extraídos corretamente
taxa de quantidades/unidades corretas
taxa de match exato correto
taxa de falsos matches
taxa de itens que exigiram revisão humana
taxa de reutilização da memória do cliente
taxa de substituições reutilizáveis
tempo médio de processamento por pedido
quantidade de chamadas à IA por pedido
custo estimado de IA por pedido
```

Os thresholds definitivos serão definidos depois dos testes reais.

=====================================================================

# 🗓️ CRONOGRAMA OFICIAL POR FASES

## ✅ P0 — Arquitetura e regras centrais

Objetivo:

```text
entender o problema
definir responsabilidades
definir fronteiras entre bancos
definir princípios de segurança
definir fluxo macro
```

Situação:

```text
✅ concluída para início do desenvolvimento
```

Novas decisões continuam podendo ser adicionadas sem reabrir todo o projeto.

=====================================================================

## 🟡 P1 — Banco próprio e modelo de dados

Entregas:

```text
P1-A entidades
P1-B relacionamentos
P1-C campos e tipos
P1-D índices/constraints
P1-E auditoria
P1-F migrations iniciais
```

Critério para conclusão:

> O fluxo completo do pedido deve poder ser representado no banco sem depender de estruturas improvisadas.

=====================================================================

## ⏳ P2 — Entrada de documentos

Entregas:

```text
upload
validação de extensão/MIME
identificação do tipo de documento
armazenamento de metadados
leitores por formato
tratamento de falha
```

Critério:

> O sistema consegue receber com segurança os principais formatos de pedido.

=====================================================================

## ⏳ P3 — Extração e normalização

Entregas:

```text
extração de linhas de produto
código
produto
quantidade
unidade
atributos relevantes
JSON estruturado
preservação do original
confiança de extração
```

Marco:

```text
MVP 1 — "EU ENTENDO O PEDIDO"
```

=====================================================================

## ⏳ P4 — Integração com catálogo oficial

Entregas:

```text
conexão read-only
mapeamento das colunas reais
consulta por código
consulta por texto
tratamento de indisponibilidade do banco
prova de que nenhuma escrita é permitida
```

Critério:

> A aplicação consulta os produtos atuais sem possuir capacidade de alterá-los.

=====================================================================

## ⏳ P5 — Motor de matching

Entregas:

```text
código exato
normalização textual
pg_trgm
embeddings
pgvector
ranking de candidatos
IA sobre candidatos selecionados
estados de resultado
```

Marco:

```text
MVP 2 — "EU ENCONTRO O PRODUTO"
```

=====================================================================

## ⏳ P6 — Memória e substituições

Entregas:

```text
memória do cliente
memória da embarcação
aliases confirmados
histórico de confirmações
histórico de rejeições
substituições aceitas/rejeitadas
priorização do conhecimento específico
```

Marco:

```text
MVP 3 — "EU CONHEÇO ESSE CLIENTE"
```

=====================================================================

## ⏳ P7 — Interface operacional

Entregas:

```text
upload
resultado da extração
tabela comparativa
candidatos
confiança
confirmar
corrigir
buscar manualmente
marcar não encontrado
visualizar histórico relevante
```

Critério:

> Um operador consegue processar um pedido do início ao fim sem depender de intervenção técnica.

=====================================================================

## ⏳ P8 — Validação, calibração e testes reais

Entregas:

```text
suite de pedidos reais anonimizados
métricas
falsos positivos
falsos negativos
calibração dos thresholds
testes de regressão
medição de custo e tempo
```

Marco:

```text
MVP 4 — "PRONTO PARA OPERAÇÃO CONTROLADA"
```

=====================================================================

## ⏳ P9 — Automação e integrações futuras

Possibilidades:

```text
recebimento automático por e-mail
notificação ao operador
resposta assistida ao marinheiro
busca de fornecedor
workflow de compra
integração com outros sistemas
monitoramento e dashboards
```

Nenhuma automação crítica deverá ser implementada antes de o núcleo estar validado.

=====================================================================

# 💡 BACKLOG DE IDEIAS

Toda ideia nova deverá ser registrada sem interromper automaticamente a fase atual.

Formato sugerido:

```text
IDEIA:

ORIGEM:

BENEFÍCIO:

FASE PROVÁVEL:

DEPENDÊNCIAS:

STATUS:
backlog / análise / aprovada / descartada / implementada
```

Regra:

> Ideia boa não precisa virar tarefa imediatamente.

=====================================================================

# 🐙 CONTROLE DE VERSÃO E GITHUB

O código-fonte deverá ser versionado em Git e mantido em um repositório GitHub.

Nome técnico provisório:

```text
pedidos-inteligentes-embarcacoes
```

Recomendação inicial:

```text
repositório privado
```

Motivos:

```text
código comercial
integrações com cliente
regras de negócio
arquitetura proprietária
possível uso de fixtures baseadas em pedidos reais anonimizados
```

=====================================================================

## Estratégia de branches

No início, manter simples:

```text
main
→ versão estável

feature/*
→ novas funcionalidades

fix/*
→ correções

docs/*
→ mudanças relevantes de documentação
```

Não criaremos `develop` apenas por convenção.

Se o time crescer e o fluxo exigir, essa decisão poderá ser revista.

=====================================================================

## Commits

Adotar padrão próximo a Conventional Commits:

```text
feat: nova funcionalidade
fix: correção de defeito
docs: documentação
test: testes
refactor: refatoração sem mudança funcional
perf: melhoria de desempenho
chore: infraestrutura/manutenção
```

Exemplos:

```text
feat: add initial order extraction schema
feat: add read-only product catalog adapter
test: cover client substitution memory
docs: document matching priority rules
```

=====================================================================

## Versionamento

Planejado:

```text
Semantic Versioning
```

Exemplo:

```text
v0.1.0
→ estrutura inicial do projeto

v0.2.0
→ ingestão de documentos

v0.3.0
→ extração inicial

v1.0.0
→ primeiro release de produção validado
```

As versões reais poderão variar conforme a evolução.

=====================================================================

## Pull Requests

Antes de integrar uma funcionalidade relevante na `main`:

```text
✅ código revisado
✅ testes executados
✅ migration revisada quando houver
✅ documentação atualizada quando necessário
✅ nenhum segredo incluído
✅ mudança de arquitetura registrada
```

=====================================================================

## CI planejada

GitHub Actions deverá futuramente executar pelo menos:

```text
lint
format check
testes unitários
testes de integração compatíveis
checagem de migrations
```

Deploy automático NÃO será ativado antes de definirmos o ambiente de produção e suas proteções.

=====================================================================

# 📁 ESTRUTURA INICIAL PLANEJADA DO REPOSITÓRIO

```text
pedidos-inteligentes-embarcacoes/
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── core/
│   │   ├── db/
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── services/
│   │   ├── ai/
│   │   ├── document_readers/
│   │   └── integrations/
│   │       └── client_catalog/
│   ├── alembic/
│   ├── tests/
│   └── pyproject.toml
│
├── frontend/
│   └── definido posteriormente na P7
│
├── docs/
│   ├── PROJETO_PEDIDOS_INTELIGENTES_EMBARCACOES.md
│   ├── ARCHITECTURE.md          # quando houver necessidade de separar
│   ├── DATABASE.md              # gerado/evoluído na P1
│   └── adr/                     # decisões arquiteturais importantes
│
├── .github/
│   └── workflows/
│
├── .env.example
├── .gitignore
├── docker-compose.yml
├── README.md
├── CHANGELOG.md
└── LICENSE                     # decisão futura
```

No início, este documento continuará sendo a referência principal.

Arquivos adicionais serão criados quando a divisão melhorar a manutenção, e não apenas para aumentar a quantidade de documentação.

=====================================================================

# 📝 REGRA DE ATUALIZAÇÃO DA DOCUMENTAÇÃO

Sempre atualizar este documento quando ocorrer:

```text
nova decisão arquitetural relevante
mudança de tecnologia
mudança de responsabilidade entre bancos
nova tabela importante
nova regra de matching
nova regra de memória
nova regra de substituição
mudança de segurança
fase concluída
limitação descoberta
teste importante validado
mudança de escopo
```

Símbolos oficiais:

```text
✅ concluído e validado
🟡 em andamento
⏳ planejado
⚠️ atenção / limitação conhecida
❌ descartado, proibido ou fora de escopo
```

=====================================================================

# 🧱 DECISÕES CONFIRMADAS ATÉ 11/09/2026

```text
D001
PostgreSQL oficial do cliente será somente leitura.

D002
Nosso sistema possuirá banco PostgreSQL próprio.

D003
Nenhum dado de IA, memória ou histórico será gravado no banco oficial.

D004
Inicialmente serão consultados do catálogo oficial somente os dados
necessários: código, produto/descrição e quantidade disponível.

D005
O sistema manterá memória separada por cliente e, quando aplicável,
por embarcação.

D006
Substituições aceitas/rejeitadas serão registradas como histórico próprio.

D007
Aceitação anterior de substituição não significa autorização automática
permanente sem regra comercial explícita.

D008
Matches incertos deverão permitir revisão humana.

D009
A pesquisa deverá utilizar métodos determinísticos antes de IA sempre
que possível.

D010
O projeto terá documentação viva e controle de versão em Git/GitHub.

D011
Novas ideias entram primeiro no backlog e não alteram automaticamente
a fase em execução.
```

=====================================================================

# ⚠️ DECISÕES AINDA PENDENTES

```text
⏳ estrutura exata das tabelas do nosso banco
⏳ nomes reais das colunas da tabela/view oficial do cliente
⏳ framework definitivo do frontend
⏳ provedor de hospedagem
⏳ estratégia de armazenamento dos arquivos originais em produção
⏳ política de retenção de documentos e logs
⏳ modelos OpenAI definitivos por função após benchmark
⏳ thresholds de confiança do matching
⏳ regras comerciais que poderão permitir substituição automática
⏳ autenticação e papéis da interface operacional
```

Não antecipar essas decisões sem dados suficientes.

=====================================================================

# 🚀 PRÓXIMA ETAPA OFICIAL

```text
P1 — BANCO PRÓPRIO E MODELO DE DADOS
```

Sequência:

```text
P1-A
→ listar entidades definitivas

P1-B
→ desenhar relacionamentos

P1-C
→ definir campos e tipos

P1-D
→ definir constraints e índices

P1-E
→ definir auditoria/rastreabilidade

P1-F
→ gerar migrations iniciais
```

Primeiro objetivo:

> Criar um modelo de dados capaz de representar o pedido original, sua interpretação, os candidatos encontrados, a decisão humana e o conhecimento reaproveitável do cliente sem tocar no PostgreSQL oficial.

=====================================================================

# 📌 RESUMO DA ARQUITETURA OFICIAL

```text
DOCUMENTO
→ dado externo não confiável

EXTRATOR
→ transforma documento em estrutura

IA
→ interpreta, normaliza e auxilia em ambiguidades

NOSSO POSTGRESQL
→ operação + histórico + memória + auditoria

POSTGRESQL DO CLIENTE
→ fonte oficial de produto / somente leitura

MATCHING
→ código + memória + texto + fuzzy + vetorial + IA

OPERADOR
→ autoridade final quando houver dúvida

DECISÃO CONFIRMADA
→ alimenta memória futura

GITHUB
→ fonte de verdade do código e histórico técnico

DOCUMENTAÇÃO
→ fonte de verdade das decisões arquiteturais e do estado do projeto
```

=====================================================================

# 📚 HISTÓRICO DO DOCUMENTO

## 11/09/2026 — v0.1.0

```text
✅ documentação oficial criada
✅ objetivo e escopo inicial registrados
✅ cronograma P0-P9 registrado
✅ fronteira entre os dois PostgreSQL documentada
✅ arquitetura read-only documentada
✅ memória por cliente/embarcação documentada
✅ substituições documentadas
✅ tecnologias iniciais registradas
✅ estratégia Git/GitHub registrada
✅ estrutura inicial do repositório proposta
✅ decisões confirmadas D001-D011 registradas
✅ próxima etapa P1 definida
```

