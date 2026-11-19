# Decisões Técnicas - EngMan Copilot

**Etapa:** Sprint 01 · **Recorte:** A - Diagnóstico e Confiabilidade  
**Status:** proposta técnica; nenhuma das opções foi implementada ou comparada em execução nesta Sprint.

O projeto precisa ter um caminho de execução gratuito. Por isso, a proposta prioriza ferramentas que possam rodar localmente, sem depender de uma API paga. “Gratuito” aqui significa ausência de cobrança por chamada: ainda serão necessários computador, armazenamento, energia e tempo para baixar e executar os modelos.

As escolhas abaixo são preliminares. A qualidade em português técnico, o tempo de resposta e o consumo de memória deverão ser comparados na etapa de implementação, usando o eval set do projeto. Não estamos afirmando que os modelos já foram testados pelo grupo.

## 1. Modelos de linguagem (LLM)

Foram selecionados dois modelos locais para comparação. Ambos podem ser executados pelo Ollama, o que permite testar os mesmos casos sem enviar os dados simulados a um serviço externo.

| Critério | Qwen 2.5 7B Instruct | Llama 3.1 8B Instruct |
|---|---|---|
| Custo de inferência | Sem cobrança por chamada ao rodar localmente; depende do hardware disponível. | Sem cobrança por chamada ao rodar localmente; depende do hardware disponível. |
| Licença | Apache 2.0 para a versão 7B. | Llama 3.1 Community License e política de uso da Meta; não é Apache 2.0. |
| Português brasileiro | Modelo multilíngue. A qualidade em perguntas de manutenção em pt-BR ainda precisa ser medida pelo eval set. | Modelo multilíngue. A qualidade em perguntas de manutenção em pt-BR ainda precisa ser medida pelo eval set. |
| Contexto máximo informado pelo modelo | Até 128 mil tokens. | Até 128 mil tokens. |
| Privacidade | Pode rodar localmente; os dados enviados ao modelo permanecem na máquina, desde que não sejam encaminhados a serviços externos pela aplicação. | Pode rodar localmente; os dados enviados ao modelo permanecem na máquina, desde que não sejam encaminhados a serviços externos pela aplicação. |
| Integração com Python | Ollama API e biblioteca Python `ollama`. | Ollama API e biblioteca Python `ollama`. |
| Documentação e suporte | Documentação do projeto Qwen e integração disponível no ecossistema Ollama. | Documentação da Meta e integração disponível no ecossistema Ollama. |

**Modelo principal proposto: Qwen 2.5 7B Instruct.** A escolha é preliminar e considera a licença Apache 2.0 da versão 7B, a possibilidade de execução local e o tamanho um pouco menor em relação ao Llama 3.1 8B. O Llama 3.1 8B será a alternativa para comparação no eval set.

**Trade-off aceito:** os modelos locais evitam o envio de dados para APIs externas e não geram custo por chamada, mas exigem memória e processamento do computador. O tempo de resposta e a qualidade em português técnico podem ser inferiores aos de modelos maiores. Se o hardware dos integrantes não suportar a execução, o grupo deverá testar uma quantização menor ou rever a configuração; não será presumido que qualquer computador consiga executar os modelos com o mesmo desempenho.

Não foi mantida uma API gratuita em nuvem como contingência principal. Embora existam serviços com faixas gratuitas, seus limites e termos podem mudar e os dados seriam enviados a terceiros. Para esta proposta, manter o fluxo local é mais coerente com a preocupação de privacidade e confidencialidade.

## 2. Armazenamento vetorial (vector store)

| Critério | ChromaDB | FAISS |
|---|---|---|
| Custo e licença | Gratuito, código aberto, licença Apache 2.0. | Gratuito, código aberto, licença MIT. |
| Função principal | Armazenar embeddings, documentos e metadados e permitir consultas por similaridade. | Biblioteca voltada à busca por similaridade e agrupamento de vetores. |
| Persistência | Possui opção de armazenamento persistente local. | O índice pode ser salvo e carregado; documentos e metadados precisam ser organizados separadamente pela aplicação. |
| Filtro por metadados | Possui filtros de metadados nas consultas (`where`). | Não oferece o mesmo fluxo de consulta documental com metadados do Chroma; exige estrutura externa para relacionar IDs, documentos e atributos, além de filtragem complementar. |
| Integração com Python | API direta e integração simples para um projeto pequeno. | API Python disponível; exige mais código de apoio para manter documentos e metadados associados aos vetores. |
| Adequação ao projeto | Boa para uma base pequena que precisa filtrar por TAG, tipo de ativo e documento. | Pode ser útil em cenários focados em busca vetorial e controle mais manual da estrutura. |

**Escolha proposta: ChromaDB.** O projeto precisa recuperar trechos de documentos e filtrar resultados por metadados, como `tipo_ativo`, `tag`, `doc_id` e `simulado`. O Chroma oferece esse filtro diretamente. Para o tamanho previsto da base acadêmica, não há necessidade identificada de priorizar a otimização de busca em grande escala oferecida pelo FAISS.

**Trade-off aceito:** ChromaDB acrescenta uma camada de armazenamento própria, enquanto FAISS permite maior controle sobre a estrutura de índices. A escolha será reavaliada caso a implementação mostre dificuldade de instalação, persistência ou desempenho.

## 3. Modelo de embeddings e RAG

A proposta é utilizar **`intfloat/multilingual-e5-base`**, executado localmente por meio da biblioteca Sentence Transformers.

A escolha considera três características documentadas do modelo: suporte multilíngue a aproximadamente 100 idiomas, limite de entrada de até 512 tokens e licença MIT. O desempenho específico em português técnico de manutenção ainda precisa ser avaliado; suporte multilíngue não significa que a recuperação será automaticamente precisa para as siglas e os termos do projeto.

O modelo E5 utiliza prefixos diferentes para consulta e conteúdo indexado: `query:` para a pergunta do usuário e `passage:` para os trechos dos documentos. A aplicação deverá manter esse padrão para evitar uma utilização diferente daquela prevista no cartão do modelo.

**Parâmetros iniciais do RAG:**

- Trechos de aproximadamente **350 a 450 tokens**, com sobreposição inicial de cerca de **50 tokens**. O tamanho deverá ser contado com o tokenizador do modelo de embeddings.
- O limite de 512 tokens do multilingual-e5-base deve ser respeitado. Trechos maiores podem ser truncados e perder parte do conteúdo.
- Recuperar inicialmente até **4 trechos** por consulta, priorizando filtros por `tipo_ativo`, `tag` e `doc_id` quando esses dados estiverem disponíveis.
- Guardar metadados mínimos: `doc_id`, `tipo_documento`, `tipo_ativo`, `tag` (quando aplicável), `data_referencia` e `simulado`.
- Não indexar dados pessoais, imagens de ocupantes, placas identificáveis ou informações operacionais reais e confidenciais.

A divisão em trechos é uma configuração inicial, não um resultado de teste. Será necessário verificar no eval set se os trechos recuperados contêm contexto suficiente para responder sem misturar procedimentos ou ativos.

## 4. Orquestração, ambiente e interface

| Camada | Escolha proposta | Motivo |
|---|---|---|
| Orquestração | Chamadas diretas à API do Ollama, organizadas em funções próprias. | Menos abstrações para a primeira implementação e maior clareza sobre montagem do contexto, histórico e chamadas de ferramentas. |
| Recuperação | ChromaDB + multilingual-e5-base local. | Permite busca semântica com filtros por metadados. |
| Ferramentas | Funções Python com contratos descritos em `tools/tools_spec.json`. | Mantém consultas ao catálogo, histórico, POPs, métricas e abertura de OS separadas da geração de texto. |
| Ambiente | Python 3.10 ou superior, VS Code e Ollama local. | Ambiente previsto para a etapa de implementação. A compatibilidade das versões das bibliotecas deverá ser conferida nessa etapa. |
| Interface | Terminal interativo como primeira opção; interface gráfica pode ser considerada depois. | A Sprint 01 exige documentação, não uma interface ou protótipo funcional. |
| Versionamento | Git e GitHub público, sem dados reais ou credenciais. | Permite registrar a contribuição dos integrantes e manter a documentação auditável. |

A opção por chamadas diretas, em vez de LangChain, é uma decisão de simplicidade para o escopo atual. Não significa que LangChain seja inadequado; apenas não foi identificada uma necessidade que justifique adicionar essa camada nesta primeira implementação.

## 5. Parâmetros iniciais do Ollama

Os valores abaixo são pontos de partida para os testes, não parâmetros já validados.

| Parâmetro | Valor proposto | Justificativa |
|---|---:|---|
| `temperature` | `0.2` | Reduz a variação da geração em respostas factuais. Não garante respostas idênticas entre execuções. |
| `top_p` | `0.9` | Limita a amostragem às opções com maior probabilidade acumulada. Será mantido como valor inicial de comparação. |
| `num_predict` | `700` | Limita a quantidade máxima de tokens gerados na resposta. |
| `num_ctx` | `12288` | Define o tamanho do contexto usado na execução local. É menor que o máximo anunciado pelos modelos e foi escolhido como ponto inicial para limitar o consumo de memória. |

**Atenção ao nome dos parâmetros:** `max_tokens` é um nome utilizado por outras APIs e aparece como conceito de limite de saída em materiais genéricos. Na API do Ollama, o parâmetro correspondente é `num_predict`; `num_ctx` define o tamanho do contexto. A implementação deverá usar os nomes aceitos pela API escolhida.

O valor `num_ctx = 12288` não deve ser confundido com a janela máxima que consta no cartão do modelo. A capacidade efetiva depende da configuração de execução e da memória disponível. Aumentar o contexto pode elevar o consumo de memória e tornar a geração mais lenta.

## 6. Estimativa de uso da janela de contexto

A estimativa serve para orientar a configuração inicial. Ainda não foi medida com um tokenizador nem com chamadas reais ao modelo.

| Parte da chamada | Estimativa inicial |
|---|---:|
| System prompt enviado ao modelo | 2.500–2.900 tokens |
| Até 4 trechos RAG de até 450 tokens | até 1.800 tokens |
| Resultado das ferramentas | até 800 tokens, com limite de retorno definido por ferramenta |
| Histórico recente e resumo | até 1.800 tokens |
| Pergunta atual e instruções auxiliares | até 300 tokens |
| Resposta gerada | até 700 tokens |
| **Total estimado** | **aproximadamente 7.900–8.300 tokens** |

O cálculo é aproximado e pode variar com o conteúdo recuperado, o tokenizador e os dados retornados pelas ferramentas. O total estimado fica abaixo de `num_ctx = 12288`, mas a margem precisa ser confirmada com medições na implementação.

A base de conhecimento inteira não será enviada ao modelo em cada chamada. O RAG recuperará apenas os trechos pertinentes; o catálogo de ativos e o histórico estruturado serão consultados pelas ferramentas, que deverão retornar apenas os registros relevantes para a TAG e para a janela solicitada.

O histórico da conversa também não deve crescer indefinidamente. A proposta é manter até 10 mensagens recentes e um resumo do atendimento. A aplicação deverá implementar a contagem, a criação do resumo e a substituição das mensagens antigas; o system prompt, sozinho, não controla o tamanho real do contexto nem garante que o limite de tokens seja respeitado.

## 7. Por que usar um assistente conversacional

O problema não se resume a aplicar uma regra fixa a um alarme. O engenheiro pode descrever sintomas de maneiras diferentes, informar condições ambientais, perguntar sobre reincidência e solicitar a organização de um rascunho de ordem de serviço. Um modelo de linguagem pode interpretar essas solicitações em texto livre e organizar uma resposta usando informações recuperadas.

Isso não significa deixar decisões críticas a cargo do modelo. A consulta de dados deve ser feita por ferramentas; a recuperação de procedimentos, pelo RAG ou pela ferramenta prevista; e regras como formato de TAG, prioridades permitidas, confirmação da OS e validação de parâmetros devem ser controladas pela aplicação. O modelo apresenta uma análise de apoio, com fonte e distinção entre fato e hipótese, mas não confirma sozinho a causa nem autoriza uma intervenção.

## 8. Decisões ainda pendentes para a etapa de implementação

1. Executar o eval set com Qwen 2.5 7B e Llama 3.1 8B, registrando qualidade das respostas, aderência às regras e tempo de resposta.
2. Conferir a memória disponível nos computadores do grupo e escolher a quantização compatível, se necessária.
3. Medir tokens reais do system prompt, do histórico, dos trechos RAG e dos resultados das ferramentas.
4. Verificar se o multilingual-e5-base recupera corretamente siglas, TAGs e descrições técnicas do dataset simulado.
5. Ajustar o tamanho dos chunks e a quantidade de resultados recuperados conforme os testes.
6. Definir limites máximos para os retornos das ferramentas, especialmente para históricos com muitas ocorrências.

## Referências técnicas

- Qwen. [Qwen2.5 — model card e especificações](https://qwen.ai/blog?id=qwen2.5-llm). Licença e contexto do Qwen2.5 7B.
- Meta. [Repositório oficial de modelos Llama](https://github.com/meta-llama/llama-models). Tamanhos, contexto e licenças da família Llama.
- Ollama. [Especificação da API](https://github.com/ollama/ollama/blob/main/docs/openapi.yaml). Opções `temperature`, `top_p`, `num_ctx` e `num_predict`.
- Chroma. [Documentação de filtros](https://cookbook.chromadb.dev/core/filters/) e [repositório oficial](https://github.com/chroma-core/chroma). Filtros por metadados e licença.
- Meta. [Documentação do FAISS](https://faiss.ai/). Busca por similaridade vetorial e índices.
- Hugging Face. [Cartão do multilingual-e5-base](https://huggingface.co/intfloat/multilingual-e5-base). Idiomas, limite de entrada, licença, prefixos e forma de uso.
