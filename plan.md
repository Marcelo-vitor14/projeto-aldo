# Plan.md — Plano de Arquitetura e Engenharia Técnica

## 1. Resumo Técnico
Aplicação Web em Python composta por backend em FastAPI e frontend em Streamlit (ou interface HTML/JS), empacotada em um contêiner Docker. O sistema utiliza um banco vetorial local para o pipeline RAG e integra um agente inteligente capaz de delegar cálculos matemáticos para funções determinísticas em Python.

## 2. Contexto Técnico
- **Linguagem e Versão:** Python 3.11+
- **Frameworks Web:** FastAPI (backend) / Streamlit ou HTML+JS (frontend)
- **Framework de IA:** LangChain ou Google GenAI SDK
- **Banco Vetorial:** ChromaDB ou FAISS (persistido em disco)
- **Modelo de IA:** Gemini 1.5 Flash / Pro
- **Testes:** Pytest (testes unitários e de integração)
- **Plataforma/Infraestrutura:** Docker, Docker Compose, deploy em nuvem (Render / Railway / Fly.io / K8s)
- **Desempenho & Escala:** Execução leve com consumo mínimo de memória e requisições assíncronas no backend.

## 3. Tabela de Verificação da Constituição

| Princípio | Como o Plano Atende | Evidência Prevista |
| :--- | :--- | :--- |
| **Fundamentação das Respostas (RAG)** | As consultas ao banco vetorial trazem os metadados de artigo e página junto com o texto. | Citação explícita exibida no contrato da API e na interface. |
| **Determinismo em Cálculos** | O agente aciona a função `calcular_nota_final` sem delegar lógica matemática ao LLM. | Testes unitários com Pytest aprovados para o módulo da Tool. |
| **Segurança e Gestão de Segredos** | As credenciais de API são lidas exclusivamente via variáveis de ambiente. | Arquivo `.env.example` no repositório e chaves injetadas no contêiner. |
| **Empacotamento e Operação** | Toda a aplicação roda isolada dentro de um contêiner reproduzível. | `Dockerfile` e `docker-compose.yml` funcionais no projeto. |
| **Avaliação do Sistema (Evals)** | Suíte de testes com 15 cenários de referência avaliando RAG e acionamento de Tools. | Script `run_evals.py` gerando relatório detalhado de acertos. |

## 4. Arquitetura e Fluxo de Dados
[ Usuário ] ---> [ Interface Web ]

v
[ Backend FastAPI ]
|
+--------------+--------------+
|                             |
v                             v
[ RAG Engine ]             [ Agent Tool Executor ]

ChromaDB Vetorial        - calcular_nota_final()

Regimento em PDF         - Validação em Python
|                             |
+--------------+--------------+
|
v
[ Provedor de LLM ] (Gemini API)
## 5. Estratégias de Teste e Avaliação (Evals)

1. **Testes Unitários (Pytest):**
   - Regra de negócio de cálculo: $MP = \frac{P1 + P2}{2}$; Se $MP \ge 7.0 \Rightarrow \text{Aprovado}$; Se $4.0 \le MP < 7.0 \Rightarrow NF = 12 - MP$.
   - Validação de schemas Pydantic para entradas e saídas de rotas.
   - Pré-processamento de PDFs e extratores de texto.

2. **Testes de Integração:**
   - Endpoint POST `/api/v1/chat` validando retornos HTTP 200, 422 (payload inválido) e 503 (indisponibilidade da API de LLM).
   - Teste de persistência e consulta no ChromaDB com documentos sintéticos.

3. **Matriz de Avaliação de IA (15 Evals Estritos):**
   - **RAG - Regimento (6 casos):** Perguntas sobre trancamento, limite de faltas (25%), colação de grau e jubilamento. *Métrica: Precision de citação do artigo e presença da palavra-chave exata.*
   - **Agente / Tool Execution (4 casos):** Pedidos explícitos de cálculo com variações de frases. *Métrica: Invocação correta do nome da Tool e acerto de 100% no valor numérico.*
   - **Fora do Escopo / Safety (3 casos):** Perguntas sobre esportes, receitas e opiniões. *Métrica: Resposta de recusa padronizada sem inventar fatos.*
   - **Casos de Borda e Erro (2 casos):** Notas informadas fora da escala ($P1 = 15.0$, $P2 = -3.0$). *Métrica: Mensagem amigável solicitando correção dos valores sem crashar o serviço.*

## 6. Observabilidade, Saúde e Operação

- **Structured Logging:** Formato JSON via biblioteca `structlog` contendo `request_id`, `latency_ms`, `tool_called`, `rag_chunks_retrieved` e `model_name`.
- **Health Checks & Probes (Kubernetes Ready):**
  - `/health/liveness`: Retorna HTTP 200 se a aplicação FastAPI estiver no ar.
  - `/health/readiness`: Retorna HTTP 200 apenas se o banco vetorial estiver acessível e montado.
- **Circuit Breaker & Fallback:** Se a API do Gemini expirar (timeout de 5s), o sistema retorna uma mensagem amigável de erro temporário, evitando *hangs* na interface.
- **Runbook Simplificado:**
  - Build local: `docker build -t app-tira-duvidas:v1 .`
  - Ingestão inicial: `docker run --rm -v $(pwd)/data:/app/data app-tira-duvidas:v1 python -m src.rag.ingest`
  - Rodar suíte Evals: `python -m pytest tests/evals/`

## 7. Tabela de Riscos Técnicos e Mitigações

| Risco Técnico | Causa Raiz | Impacto | Mitigação Arquitetural |
| :--- | :--- | :--- | :--- |
| **Alucinação em normas acadêmicas** | Contexto insuficiente ou temperatura alta no LLM. | **Alto** (fornecer regras/prazos falsos ao aluno). | Fixar `temperature=0.0`. Injetar prompt estrito: *"Responda APENAS com base nos trechos fornecidos. Se não houver evidência, responda que não encontrou no regimento."* |
| **Falha na chamada da Tool (Function Calling)** | Ambiguidade no prompt do usuário ou erro no parsing do JSON. | **Médio** (LLM tenta calcular internamente no texto e erra a conta). | Utilizar esquema formal via Pydantic/OpenAPI schema na definição da Tool. Adicionar validação de tipo estrita antes da execução. |
| **Estouro de Rate Limit da API de IA** | Múltiplos testes da suíte de Evals rodando em paralelo. | **Médio** (Bloqueio temporário por erro HTTP 429). | Implementar decorador de *Retry com Exponential Backoff* (`tenacity`) e realizar cache local de embeddings. |
| **Perda dos dados do Banco Vetorial** | Reinicialização do contêiner sem persistência configurada. | **Médio** (Necessidade de reprocessar todos os PDFs). | Mapear volume persistente (`docker volume` ou Kubernetes `PersistentVolumeClaim`) apontando para a pasta do ChromaDB. |
| **Injeção de Prompt / Entradas Adversariais** | Usuários tentando burlar as instruções do sistema no chat. | **Baixo/Médio** (Sistema responder conteúdos não permitidos). | Sanitização das entradas no backend e separação clara entre *System Prompt* e *User Input* nas chamadas da API. |