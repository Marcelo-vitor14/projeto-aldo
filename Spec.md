# Spec.md — Tira-Dúvidas Acadêmico & Copiloto do Aluno

## 1. Problema e Resultado Esperado
- **Problema:** Os alunos universitários enfrentam dificuldades para encontrar respostas rápidas sobre regras institucionais (prazos, limites de faltas, notas mínimas, trancamentos e normas de TCC) em documentos PDF extensos e burocráticos. Além disso, muitos realizam cálculos errados da nota necessária na Prova Final, gerando ansiedade e sobrecarga no atendimento da secretaria.
- **Resultado Esperado:** Uma aplicação web que responda a dúvidas em linguagem natural com citação exata das fontes (RAG) do Regimento e Manual do Aluno. O sistema deve identificar intenções de cálculo de nota e acionar de forma automática e precisa uma ferramenta (Agent Tool) para calcular a nota mínima necessária na Prova Final.

## 2. Escopo
### O que ESTÁ no escopo (In-Scope)
- Leitura e indexação vetorial do Regimento Acadêmico e Manual do Aluno.
- Respostas RAG com citação explícita de fonte (artigo, capítulo, seção ou página).
- Agente com capacidade de decidir quando invocar a ferramenta determinística de cálculo de notas.
- Ferramenta (Tool) em Python para cálculo de média e nota necessária na Prova Final.
- Tratamento de perguntas fora do escopo institucional (recusa graciosa sem inventar dados).
- Interface Web responsiva containerizada (Docker) publicada com HTTPS.

### O que NÃO ESTÁ no escopo (Out-of-Scope)
- Integração direta com APIs do portal do aluno para busca automática de notas autenticadas.
- Envio automático de requisições ou e-mails formais para a secretaria.
- Upload de novos documentos em tempo de execução pela interface do usuário.

## 3. Cenários de Usuário e Teste (Histórias de Usuário)

### História 1: Consulta sobre limite de faltas e frequência
- **Como** estudante do curso,
- **Quero** consultar a quantidade máxima de faltas permitida em uma disciplina,
- **Para que** eu evite a reprovação por frequência.
- **Cenário de Aceite:**
  - *Dado* que pergunto: "Quantas faltas posso ter numa disciplina de 60 horas?",
  - *Quando* o RAG busca a informação na base,
  - *Então* o sistema deve responder informando o limite exato (ex: 25% / 15 horas) e citar o artigo correspondente do Regimento.

### História 2: Cálculo de Nota para a Prova Final
- **Como** estudante que realizou as avaliações P1 e P2,
- **Quero** informar minhas notas e saber quanto preciso tirar na Prova Final,
- **Para que** eu planeje meus estudos para o exame.
- **Cenário de Aceite:**
  - *Dado* que informo: "Tirei 5.5 na P1 e 6.0 na P2, quanto preciso na Final?",
  - *Quando* o agente identifica a necessidade de cálculo,
  - *Então* ele executa a ferramenta `calcular_nota_final(p1=5.5, p2=6.0)` e exibe o resultado numérico exato com a regra de aprovação.

### História 3: Consulta fora do contexto acadêmico
- **Como** usuário da plataforma,
- **Quero** fazer uma pergunta não relacionada às regras da faculdade,
- **Para que** o sistema responda sem inventar dados.
- **Cenário de Aceite:**
  - *Dado* a pergunta: "Qual o melhor restaurante da cidade?",
  - *Quando* o RAG verifica a ausência de evidência na base documental,
  - *Então* o sistema deve informar educadamente que responde apenas sobre o regimento e manual acadêmico.

## 4. Casos de Borda
- **Entrada de notas inválidas:** Notas negativas ou superiores a 10 (ex: P1 = 12 ou P2 = -1). O sistema deve validar e pedir valores válidos entre 0.0 e 10.0.
- **Média suficiente para aprovação direta:** Se $MP \ge 7.0$, a ferramenta deve informar que o aluno já está aprovado e não precisa realizar a Prova Final.
- **Ausência de norma específica:** Se o aluno perguntar sobre um assunto não coberto no documento, o sistema deve assumir limitação e não alucinar regras inexistentes.

## 5. Requisitos Funcionais
- **RF01:** O sistema deve buscar trechos relevantes na base de vetores e retornar respostas acompanhadas das devidas referências.
- **RF02:** O agente deve extrair parâmetros numéricos do texto e acionar a função de cálculo de média quando identificada a intenção.
- **RF03:** A interface web deve permitir troca de mensagens em formato de chat responsivo.
- **RF04:** O sistema deve apresentar mensagem amigável em caso de erros de integração com a API do modelo de linguagem.

## 6. Entidades Principais
- **Chunk de Documento:** Trecho do texto extraído do regimento contendo conteúdo, metadados de localização (capítulo/artigo) e vetor de embedding.
- **Mensagem / Conversa:** Estrutura contendo o histórico de entradas do usuário, respostas do sistema e lista de fontes citadas.
- **Resultado do Cálculo:** Estrutura de dados contendo `nota_p1`, `nota_p2`, `media_parcial`, `status` e `nota_necessaria_final`.

## 7. Critérios de Sucesso e Requisitos de Qualidade
- **Acurácia RAG:** 100% das respostas baseadas em normas devem indicar a citação da fonte.
- **Determinismo Matemática:** 0% de erro nos cálculos de notas (garantido por execução Python pura via Tool).
- **Desempenho:** Tempo de resposta médio inferior a 3.5 segundos por requisição.
- **Disponibilidade:** Aplicação rodando em contêiner Docker publicado e operando via HTTPS.

## 8. Suposições e Dependências
- **Suposição:** Regimento e Manual estão disponíveis para conversão em formato de texto/Markdown/PDF.
- **Dependência:** Chave de API ativa para modelo de linguagem (ex: Gemini API / OpenAI API) configurada em variável de ambiente.

## 9. Questões em Aberto e Decisões Confirmadas
- **Decisão Confirmada:** A regra de cálculo adota $MP = \frac{P1 + P2}{2}$. Para $4.0 \le MP < 7.0$, a Nota Mínima na Final é $NF = 12 - MP$ (ou fórmula equivalente da instituição).
- **Decisão Confirmada:** O banco vetorial adotado será o ChromaDB / FAISS integrado ao projeto.