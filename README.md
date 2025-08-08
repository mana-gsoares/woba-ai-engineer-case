# Woba AI Builder Challenge, Agentes & RAG

**Propósito**: provar, de forma objetiva e mensurável, que você consegue **projetar e implementar** sistemas de IA de produção, **iguais ou melhores** do que os que construímos, usando **frameworks de agentes** e boas práticas de engenharia.

**Formato**: 2–4h de execução (+48–72h de janela para envio) • Debrief 45–60 min.

---

## 0‑A) Requisito obrigatório, Orquestração por **Agentes**

Você deve usar **um framework de agentes** em **pelo menos 1** dos módulos (vale nos 3 se preferir). Opções: **LangGraph**, **CrewAI**, **PydanticAI** ou **Google (Vertex AI Agents / Agent Builder)**.

**Exigências mínimas do fluxo agentic**:

* **3+ nós/funções**: *Router* → (Retrieval/Tool) → *Judge/Coordinator* (pode incluir *Reranker* e *Pricer* como ferramentas).
* **Estado tipado** (Pydantic dataclass/model) e **tool calling**/parâmetros estruturados.
* **Tracing** de nós (latência, sucessos/falhas) e **diagrama** (mermaid/PNG) do grafo.
* **Fallbacks/Timeouts** e **circuit‑breaker** simples para ferramentas externas.

---

## 1) O que você vai construir

Uma **mini‑plataforma de IA** com **módulos independentes**. Entregue **no mínimo 2** dos 3 módulos abaixo (entregar os 3 aumenta a pontuação).

### Módulo A, Busca Híbrida + Reranking + Justificativas

Implemente busca híbrida (semântica + keyword) sobre um catálogo genérico de itens (você escolhe o domínio).

* **Entradas**: `catalog.csv` (dados sintéticos). Campos mínimos: `id, title, description, city, grade, seats, size, price, tags[], features[]`.
* **Requisitos**:

  1. Indexação vetorial + BM25; hiperparâmetro `alpha` (0–1) para mix.
  2. **Reranker** que combine similaridade, *fit* por capacidade (e.g., seats/size) e **preço justo** (score ∈ \[0,1]).
  3. **Justificativa curta** por item (LLM ou heurística) citando evidências de dados.
  4. API: `POST /search {query, limit}` → `[{id, final_score, reason, evidence}]`.
* **Aceitação (automática)**:

  * `eval_search.py` computando **nDCG\@10** e **MRR\@10** contra `labels.json` (relevância 0–3).
  * **Meta**: nDCG\@10 ≥ **0,65** e MRR\@10 ≥ **0,55** em 10+ consultas rotuladas.

### Módulo B, RAG com Guardrails e Avaliação

Construa um assistente que responde com **citações** a um corpus (políticas, FAQs, atas, etc.).

* **Entradas**: `docs/` com 15–30 páginas (sintéticas). Chunking 200–600 tokens com metadados.
* **Requisitos**:

  1. Recuperação + geração com **citations** (IDs/trechos) e **confidence**.
  2. Guardrails: controle de escopo, filtro de PII, timeout e limite de custo.
  3. API: `GET /ask?q=` → `{answer, sources[], confidence}`.
* **Aceitação (automática)**:

  * `eval_rag.py` com **Recall\@5** e **Answer Faithfulness** (LLM‑judge ou `ragas`).
  * **Meta**: Recall\@5 ≥ **0,85** e Faithfulness ≥ **0,70** (10 Q\&As ouro).

### Módulo C, Motor de Decisão/Preço (Multi‑objetivo)

Implemente um *scorer* que recomende **Top‑k** itens balanceando **adequação** e **preço justo**.

* **Requisitos**:

  1. Função de preço justo **genérica** (ex.: base por cidade/grade \* seats \* fator ±15%).
  2. `final_score = w1*similarity + w2*fit + w3*fair_price_score` com pesos calibráveis.
  3. API: `POST /rank {items[], user_prefs}` → top‑k + explicações.
* **Aceitação (automática)**:

  * `eval_price.py` com **MAPE** (price vs. fair\_price) + **Coverage** de constraints.
  * **Meta**: MAPE ≤ **0,20** e Coverage ≥ **0,90**.

### Regras de orquestração por agentes (aplicam‑se a A/B/C)

* **Obrigatório** em ≥1 módulo: o *planner/router* decide qual ferramenta/fluxo acionar (retriever, reranker, pricer, summarizer).
* **LangGraph**: forneça `State` tipado, nós e edges condicionais + **checkpoints**.
* **CrewAI/PydanticAI**: defina **roles/tools** com *tool‑calling* e memória curta clara.
* **Google (Vertex AI Agents/Agent Builder)**: descreva *tools* e *handlers* equivalentes (pode mockar chamadas).
* Entregue **artefatos de trace** (JSON/CSV) por nó e **diagrama** do fluxo.

---

## 2) Requisitos transversais

* **Observabilidade**: logging estruturado (JSON), latência p95 por endpoint, contagem de tokens e **tracing por nó de agente**.
* **SLOs**: p95 **< 400ms** sem LLM; **< 1200ms** com LLM; custo máx **US\$0,05**/req (simule se necessário).
* **Reprodutibilidade**: Docker **ou** `make run`; `.env.sample`; dados sintéticos por script determinístico.
* **Segurança**: *input validation* + teste de **prompt‑injection** (3 strings maliciosas) e mitigação.
* **Qualidade**: 5+ testes (unit/integration) e tipagem (mypy/pyright).

---

## 3) Entregáveis (claros e verificáveis)

### Regra geral (obrigatória)

* Você deve **entregar pelo menos 2 dos 3 módulos** (A, B e/ou C).
* Cada módulo escolhido precisa **atingir as metas de aceitação** descritas na sua seção de "Aceitação (automática)".
* Entregar o **3º módulo** é opcional e **aumenta a pontuação** (considerado como diferencial/bônus na rubrica).

### Entregáveis por módulo

**Módulo A, Busca Híbrida + Reranking + Justificativas**

* Código do indexador (vetorial + BM25) e do mixer (`alpha`).
* Endpoint `POST /search` funcional com `query` e `limit`.
* Script **`eval_search.py`** + arquivo **`labels.json`** (relevância 0–3).
* Relatório de avaliação com **nDCG\@10** e **MRR\@10** (meta ≥ 0,65 / ≥ 0,55).
* 5 consultas de exemplo + saída com **reason** e **evidence**.
* (Se aplicar agente) diagrama + **trace por nó** do fluxo agentic.

**Módulo B, RAG com Guardrails e Avaliação**

* Pipeline de ingestão/indexação (chunking + metadados) e **corpus `docs/`**.
* Endpoint `GET /ask?q=` com `{answer, sources[], confidence}`.
* Script **`eval_rag.py`** + **Q\&As ouro** (10 itens) para Recall/Faithfulness.
* Relatório de avaliação com **Recall\@5** e **Faithfulness** (meta ≥ 0,85 / ≥ 0,70).
* Descrição dos **guardrails** implementados (escopo, PII, timeout/custo) e testes de *prompt‑injection*.
* (Se aplicar agente) diagrama + **trace por nó** do fluxo agentic.

**Módulo C, Motor de Decisão/Preço (Multi‑objetivo)**

* Função de **preço justo** documentada + pesos do `final_score`.
* Endpoint `POST /rank` que retorna top‑k + explicações.
* Script **`eval_price.py`** (MAPE + Coverage) + dados sintéticos.
* Relatório de avaliação com **MAPE ≤ 0,20** e **Coverage ≥ 0,90**.
* (Se aplicar agente) diagrama + **trace por nó** do fluxo agentic.

### Pacote de submissão (para todo o projeto)

* Repositório com **README executável** (5 min para rodar) e **Design Doc** (≤ 2 págs).
* **Docker** ou `make run`, arquivo **`.env.sample`**, e scripts de geração de dados sintéticos.
* **Coleção HTTP** (arquivo `.http` ou Postman) para cada endpoint.
* **Logs/metrics**: latência p95 por endpoint, contagem de tokens (se usar LLM) e **tracing** por nó de agente.

### Checklist rápido (antes de enviar)

* ✅ Implementou **2 de 3** módulos?
* ✅ Todos os **scripts de avaliação** rodam e batem as **metas**?
* ✅ Endpoints respondem com exemplos reproduzíveis?
* ✅ Tracing/diagramas do **fluxo agentic** presentes (mín. 1 módulo)?
* ✅ README/Design Doc explicam escolhas, custos e próximos passos?

---

## 4) Critérios de pontuação (100 pts)

* **Arquitetura & Modelagem** (20)
* **Busca/RAG/Ranking** (20)
* **Agentes & Ferramentas** (10)
* **MLOps & Observabilidade** (15)
* **Performance & Custos** (10)
* **Segurança & Guardrails** (10)
* **Qualidade do Código** (10)
* **Documentação & Clareza** (5)

**Bônus (até +10)**: DSPy, LangGraph avançado (subgrafos/checkpoints), re‑rank LLM/cross‑encoder, HITL, AB‑tests, OpenTelemetry, feature flags.

---

## 5) Starter kit (opcional)

```
repo/
├─ src/
│  ├─ api/
│  ├─ agents/ (graph/crew/pydanticai/google)
│  ├─ search/
│  ├─ rag/
│  ├─ score/
│  ├─ core/
│  └─ eval/ (eval_search.py, eval_rag.py, eval_price.py)
├─ data/
├─ tests/
├─ README.md
└─ docker-compose.yml (qdrant/redis opcional)
```

**Frameworks**: **LangGraph**, **CrewAI**, **PydanticAI** ou **Google Agents**. Pelo menos um módulo deve usar um deles.

---

## 6) Regras finais

* **Linguagem**: Python 3.10+; LLMs e DBs à sua escolha (pode *dry‑run*).
* **Entrega**: repo público ou `.zip`; inclua relatório de métricas e instruções.

> **Entrega forte** = metas batidas + explicações claras de por que sua abordagem funciona e como escalar para 1M+ itens/100 QPS.

---

## 7) Dataset oficial (opcional, recomendado)

Para padronizar e aproximar da realidade, disponibilizamos um **dataset amostral**:

**Arquivo**: `buildings_sample_700.json`
**Conteúdo (campos principais)**: `id, title, address, type, size, rental_price, condo_price, iptu_price, total_price, last_updated, owner, image, furnished, highlights[], coords{lat,lng}, macrozone, region, region_2_0, source_file`.

### 7.1 Mapeamento → `catalog.csv` (para o Módulo A/C)

Crie um script `prepare_dataset.py` que converta o JSON em `catalog.csv` com as colunas:

* `id, title, description, city, grade, seats, size, price, tags[], features[]`.
* **Regras sugeridas**:

  * `city`: extraído de `address`.
  * `grade`: parse do campo `type` (ex.: “Corporate - Classe A” → `A`).
  * `seats`: heurística `floor(size/7)` (ajuste se preferir e documente).
  * `features[]`: de `highlights` + indicadores (ex.: `furnished` → `mobiliado|nao_mobiliado`).
  * `tags[]`: `{macrozone, region, region_2_0}`.
  * `price`: use `total_price` se > 0; senão calcule `size * (rental_price + condo_price + iptu_price)`.
  * Trate valores `0.0` como **missing** e documente a decisão de imputação/descartes.

Saída adicional: `stats.json` com contagens por `grade/region`, `% missing`, p50/p80 de preço por m².

### 7.2 Labels e consultas (para o Módulo A)

Inclua **10+ consultas** em `queries.json` e **relevâncias** em `labels.json (0–3)`.
Sugestões de consultas:

* “escritório classe A na Berrini até 5k”
* “andar alto com 2 vagas perto da Paulista”
* “sala 40–50 m² na Vila Mariana mobiliada”
* “corporate 200–300 m² na Barra Funda com gerador”
* “preço m² até 60 na Mooca com 1 vaga”

### 7.3 Baseline de preço justo (para o Módulo C)

Implemente `fair_price_baseline.py`:

* **Base por m²** por `(region, grade)` usando p20 do preço/m².
* `fair_price = base_m2 * size` e **score**: `1 - min(|price - fair_price|/fair_price, 1)`.
* Calibre limites (±15% default) e documente.

### 7.4 Requisitos de limpeza/qualidade

* Normalize `last_updated` (ISO 8601) e sinalize itens **obsoletos** (ex.: < 2023) para testes de ranking.
* Remova duplicatas por `(title, address, size)`.
* Geo: valide lat/lng e, se desejar, gere features de distância (ex.: até eixos “Faria Lima”, “Paulista”).

### 7.5 Entregáveis adicionais quando usar o dataset oficial

* `prepare_dataset.py` + `catalog.csv` gerado.
* `queries.json`, `labels.json`, `stats.json`.
* Documente **assunções** (unidades R\$/m²/mês, handling de zeros etc.) no README.

> O uso do dataset é **opcional**, mas **recomendado** para comparabilidade entre candidaturas. Você pode combinar com dados próprios, basta manter os scripts de avaliação e o formato de saída.

