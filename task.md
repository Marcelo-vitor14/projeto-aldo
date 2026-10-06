# Tasks.md — Checklist de Tarefas por Fases

## Fase 1: Alinhamento de Especificação e Configuração Inicial
- [ ] **[T1.1]** Validar e consolidar o arquivo `Spec.md` com a equipe e o Product Owner.
- [ ] **[T1.2]** Inicializar a estrutura do repositório Git e definir fluxo de branches e Pull Requests.
- [ ] **[T1.3]** Criar arquivo de dependências (`requirements.txt` ou `pyproject.toml`) e configurar ambiente virtual.
- [ ] **[T1.4]** Adicionar os arquivos base (Regimento/Manual em PDF ou Markdown) no diretório `data/raw/`.

## Fase 2: Ingestão de Dados e Construção do RAG
- [ ] **[T2.1]** Desenvolver o script de extração e segmentação de texto (chunking) dos documentos (`src/rag/ingest.py`). *Depende de: T1.4*
- [ ] **[T2.2]** Configurar e persistir o banco vetorial ChromaDB com embeddings (`src/rag/vectorstore.py`). *Depende de: T2.1*
- [ ] **[T2.3]** Criar módulo de busca vetorial que retorne o contexto acompanhado de metadados de citação (`src/rag/retriever.py`). *Depende de: T2.2*
- [ ] **[T2.4]** Criar testes unitários para o pipeline de busca vetorial. *Depende de: T2.3*

## Fase 3: Desenvolvimento da Tool e Agente Inteligente
- [ ] **[T3.1]** Implementar a função determinística Python `calcular_nota_final(p1, p2)` em `src/tools/calculator.py`.
- [ ] **[T3.2]** Criar testes unitários Pytest para a calculadora cobrindo aprovação, prova final e entradas inválidas. *Depende de: T3.1*
- [ ] **[T3.3]** Implementar a lógica do Agente associando a busca RAG e o acionamento da Tool de cálculo (`src/agent/core.py`). *Depende de: T2.3, T3.1*

## Fase 4: API Backend e Interface do Usuário
- [ ] **[T4.1]** Desenvolver a API FastAPI com as rotas `/chat` e `/health` em `src/api/main.py`. *Depende de: T3.3*
- [ ] **[T4.2]** Criar a interface de usuário (Streamlit ou HTML/JS) conectada ao backend (`src/frontend/`). *Depende de: T4.1*
- [ ] **[T4.3]** Formatar a interface para exibir visualmente as fontes/citações do RAG e os resultados das Tools. *Depende de: T4.2*

## Fase 5: Evals, Containerização e Publicação
- [ ] **[T5.1]** Criar o dataset com os 15 casos de teste estruturados em `data/evals_dataset.json`.
- [ ] **[T5.2]** Desenvolver o script de execução de Evals (`src/evals/run_evals.py`) para gerar métricas de acurácia. *Depende de: T4.1, T5.1*
- [ ] **[T5.3]** Escrever o `Dockerfile` e o `docker-compose.yml` para empacotamento da aplicação. *Depende de: T4.2*
- [ ] **[T5.4]** Realizar o deploy do contêiner em ambiente de nuvem acessível via HTTPS. *Depende de: T5.3*
- [ ] **[T5.5]** Finalizar a documentação do projeto e instruções de uso no `README.md`. *Depende de: T5.4*

---

## Pontos de Paralelismo
- As atividades da **Fase 2 (Processamento RAG)** e da **Fase 3 (Ferramenta de Cálculo)** podem ser executadas simultaneamente por pessoas diferentes da equipe:
  - **Membro A:** Foca nas tarefas **T2.1 a T2.4** (Ingestão do regimento e busca vetorial).
  - **Membro B:** Foca nas tarefas **T3.1 e T3.2** (Calculadora em Python e testes) e depois apoia na **T4.2** (Interface Web).
- A montagem da base de avaliação **T5.1** pode ser iniciada