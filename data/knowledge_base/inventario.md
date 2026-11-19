# Inventário da Base de Conhecimento - EngMan Copilot

> **Escopo desta entrega:** este arquivo descreve quais documentos farão parte da base de conhecimento, quais informações cada um deverá conter, sua origem e a forma de consulta. Na Sprint 01, o grupo entrega o inventário; isso não significa que todos os documentos ou registros já estejam produzidos.
>
> O cenário operacional criado para o projeto é fictício. TAGs, ativos cadastrados, históricos, fichas técnicas, POPs, matriz de criticidade, relatos e ordens de serviço serão **simulados** e não representam registros da Motiva. O briefing do Challenge e as normas oficiais são referências documentais; não devem ser confundidos com dados operacionais reais da concessionária.

## 1. Organização e acesso aos documentos

A base será dividida conforme o tipo de informação. Dados estruturados que precisam ser consultados por TAG serão retornados por tools. Documentos de orientação e referência serão recuperados pelo RAG. O mesmo dado não deverá ser mantido em duas fontes diferentes sem controle de versão.

| ID | Documento | Conteúdo previsto | Origem | Quantidade prevista | Forma de consulta |
|---|---|---|---|---:|---|
| D01 | Catálogo de ativos | TAG, tipo de ativo, fabricante, modelo, km, praça, data de instalação, vida útil e criticidade | Simulado | 36 ativos: 16 CFTV, 10 PMV e 10 Callbox | TOOL `consultar_ativo` |
| D02 | Plano de manutenção preventiva | Ativo, tarefa, periodicidade, responsável por função, tempo estimado, recursos e condições para execução | Simulado | 1 documento, com aproximadamente 20 tarefas | RAG |
| D03 | Histórico de falhas do FSM | TAG, data da ocorrência, sintoma, tipo de falha, causa registrada, tempo de reparo, peça substituída e indicação de reincidência | Simulado | 60 registros | TOOL `consultar_historico_falhas`; será também a fonte de dados da tool de métricas, quando disponível |
| D04 | Procedimentos operacionais (POPs) | Objetivo, ativos aplicáveis, pré-requisitos, condições de segurança, responsáveis e etapas de inspeção ou manutenção | Simulado | 4 POPs | TOOL `consultar_procedimento` |
| D05 | Fichas técnicas dos equipamentos | Características gerais, componentes, função, condições de operação e falhas típicas de CFTV PTZ, PMV e Callbox | Simulado, elaborado pelo grupo a partir do escopo do Challenge | 3 fichas, uma por tipo de ativo | RAG |
| D06 | Matriz de criticidade e prioridade | Criticidade do ativo e do trecho, impacto operacional, segurança viária, arrecadação, redundância e critérios de classificação P1 a P4 | Simulado | 1 documento | RAG |
| D07 | Glossário de siglas | Termos usados no projeto, como CFTV, PMV, CCO, CBM, RCM, FSM, MTBF, MTTR, WIM, OCR, PTZ, POP, OS e TAG | Elaborado pelo grupo a partir do briefing e dos materiais da disciplina | 1 documento | RAG |
| D08 | Referências de segurança | Síntese informativa dos pontos relevantes da NR-10, NR-35 e cuidados de trabalho em via | Resumo elaborado pelo grupo com base nas normas e referências oficiais | 1 documento | RAG; a execução deve seguir a norma vigente e os procedimentos oficiais aplicáveis |
| D09 | Briefing do Challenge | Contexto da Motiva apresentado no Challenge, descrição dos temas e necessidades apontadas para a Engenharia de Manutenção | Briefing da disciplina | 1 conjunto de referência | RAG |
| D10 | Relatos de atendimento | Descrição do sintoma observado, condições ambientais, verificações realizadas e resultado do atendimento, sem dados pessoais | Simulado; relatos derivados de ocorrências do D03 | 12 relatos | RAG |

**Total previsto: 10 documentos.** A quantidade indicada é o planejamento do inventário, não uma declaração de que os arquivos completos já foram produzidos nesta Sprint.

### 1.1 Regras de acesso e fonte principal

- **D01 — Catálogo:** a identificação, os dados cadastrais e a criticidade de um ativo devem vir de `consultar_ativo`.
- **D03 — Histórico:** ocorrências e reincidência devem vir de `consultar_historico_falhas`. A tool de métricas, se habilitada, deverá utilizar os mesmos registros de D03, sem manter uma segunda cópia do histórico.
- **D04 — POPs:** o procedimento será consultado por `consultar_procedimento`. O conteúdo integral dos POPs não será duplicado no índice RAG nesta proposta.
- **D06 — Matriz:** a classificação de prioridade deverá seguir os critérios documentados neste arquivo e detalhados no documento D06. Se os dados necessários não estiverem disponíveis, o assistente não deverá forçar uma classificação.
- **D08 — Segurança:** o resumo serve para localizar e contextualizar requisitos. Não substitui a norma vigente, o procedimento oficial da concessionária ou a avaliação de profissional habilitado.
- **D10 — Relatos:** servem como contexto narrativo complementar. Não substituem o registro estruturado do histórico D03.

## 2. Detalhamento dos documentos e decisões de modelagem

### D01 — Catálogo de ativos

O catálogo terá 36 ativos fictícios distribuídos entre os três tipos cobertos pelo assistente: 16 câmeras CFTV, 10 painéis PMV e 10 Callbox. Cada registro deverá conter, no mínimo, TAG, tipo, fabricante e modelo simulados, rodovia, km, praça ou trecho, data de instalação, vida útil estimada e criticidade.

Os tipos de ativo serão registrados como `CFTV`, `PMV` e `CALLBOX`. Para Callbox, o prefixo usado nas TAGs será `CBX`, por exemplo `CBX-R01-K420-03`. Assim, o nome do tipo de ativo e o prefixo da TAG não serão confundidos.

### D02 — Plano de manutenção preventiva

O documento reunirá aproximadamente 20 tarefas relacionadas a CFTV, PMV e Callbox. Cada tarefa deverá indicar o ativo aplicável, o que deve ser verificado, a periodicidade proposta, o responsável por função, o tempo estimado e as condições necessárias para execução.

As periodicidades e os tempos serão simulados para fins acadêmicos. Não devem ser apresentados como plano oficial da Motiva ou recomendação de fabricante. Qualquer atividade com risco elétrico, trabalho em altura ou atuação em via deverá remeter ao POP correspondente e aos requisitos oficiais de segurança.

### D03 — Histórico de falhas do FSM

O conjunto terá **60 registros simulados**, associados aos 36 ativos do catálogo. Cada ocorrência deverá conter TAG, data, sintoma, tipo de falha, causa registrada quando houver confirmação, tempo de reparo, peça substituída quando aplicável e informação sobre reincidência.

Os cinco ativos âncora da seção 3 somam **14 ocorrências resumidas**. Os outros **46 registros previstos** serão associados aos demais ativos do catálogo. Essa divisão deixa explícito que os exemplos dos ativos âncora não representam, sozinhos, os 60 registros do histórico.

A ausência de causa registrada deverá ser representada como dado ausente, e não preenchida por uma hipótese. O assistente poderá formular uma hipótese com base nos sintomas e no histórico, mas deverá identificá-la como hipótese, sem convertê-la em causa confirmada.

A tool de métricas poderá utilizar o histórico D03 para calcular λ, MTBF e R(t), caso esteja disponível e receba os dados de operação necessários. A metodologia, o período, o horizonte de R(t), as premissas e eventuais limitações da amostra deverão ser informados. **Não será apresentada tendência crescente, estável ou decrescente enquanto o método para essa classificação não estiver definido e documentado na especificação da tool.**

**Período de referência do conjunto simulado:** 01/11/2025 a 31/10/2026. As janelas de consulta deverão ser interpretadas em relação a essa data de referência do dataset, e não à data em que alguém executar futuramente o protótipo. A especificação da tool deverá refletir essa regra.

### D04 — Procedimentos operacionais (POPs)

Serão elaborados quatro POPs simulados:

- **POP-01:** inspeção e limpeza externa da lente de câmera PTZ;
- **POP-02:** verificação de foco e comunicação de CFTV;
- **POP-03:** inspeção de módulos de LED e comunicação de PMV;
- **POP-04:** verificação funcional de Callbox, incluindo botão, áudio, bateria e geolocalização.

Cada POP deverá indicar finalidade, ativo aplicável, pré-requisitos, riscos, condições que exigem interrupção e encaminhamento, responsável habilitado e etapas permitidas. O inventário não autoriza atividade em altura, intervenção elétrica ou trabalho em via. Os procedimentos serão apenas especificações acadêmicas e deverão ser validados por profissional competente antes de qualquer uso real.

Os POPs serão consultados pela tool `consultar_procedimento`, que deverá retornar o documento aplicável. Para evitar versões divergentes, o mesmo conteúdo não será indexado em paralelo no RAG nesta proposta.

### D05 — Fichas técnicas dos equipamentos

Serão criadas três fichas gerais: uma para câmera PTZ, uma para PMV e uma para Callbox. Como ainda não foram definidos fabricantes e modelos reais para o cenário, as características específicas dessas fichas serão simuladas. O grupo não atribuirá especificações inventadas a fabricantes reais.

Se, em uma etapa futura, for incorporado um manual ou datasheet de fabricante, a ficha deverá identificar fabricante, modelo, título do documento, versão ou data e endereço da fonte. Informações genéricas do briefing não serão apresentadas como especificações oficiais de equipamento.

### D06 — Matriz de criticidade e prioridade

A matriz relacionará a criticidade do ativo e do trecho ao impacto sobre segurança viária, arrecadação e disponibilidade. A classificação da prioridade também considerará gravidade da degradação ou perda de função, persistência, reincidência e existência de cobertura alternativa ou redundância.

A proposta de classificação a documentar no D06 é:

| Prioridade | Situação de referência | Prazo simulado para atendimento |
|---|---|---|
| P1 | Perda total de função ou condição de impacto imediato, associada a ativo/trecho crítico e sem cobertura alternativa adequada | Até 4 horas |
| P2 | Falha relevante ou degradação importante, com impacto operacional elevado, criticidade alta ou reincidência que exige atendimento prioritário, ainda que não haja perda total de função | Até 24 horas |
| P3 | Degradação parcial, com função principal ainda disponível ou cobertura alternativa, sem evidência de impacto imediato elevado | Até 72 horas |
| P4 | Anomalia de baixo impacto, sem perda relevante de função e que pode ser programada para a próxima rotina preventiva | Na próxima preventiva |

Esses critérios e prazos são **propostas simuladas do grupo**, criadas para permitir que os casos do projeto sejam avaliados de maneira consistente. Não representam SLA, regra ou matriz oficial da Motiva. O documento D06 deverá explicar como os critérios são combinados e quais informações mínimas são necessárias para classificar um caso. Se houver conflito entre critérios ou falta de informação essencial, a prioridade deverá ficar pendente de validação do engenheiro.

### D07 — Glossário de siglas

O glossário reunirá as siglas e os termos usados no projeto, com definições curtas e fonte de referência. A lista inicial inclui CFTV, PMV, CALLBOX, CCO, CBM, RCM, WCM, FSM, MTBF, MTTR, λ, WIM, OCR, PTZ, POP, OS e TAG.

As siglas **RGA** e **LDA** aparecem no briefing do Challenge, mas o material considerado neste inventário não fornece definição suficiente para explicar sua aplicação. Elas poderão constar como termos a esclarecer, mas o assistente não deverá inventar seus significados nem afirmar que executa essas análises sem uma definição validada em material da disciplina ou referência indicada pelo professor.

### D08 — Referências de segurança

O documento será uma síntese acadêmica das referências oficiais relacionadas à NR-10, à NR-35 e ao trabalho em via. Na elaboração, o grupo deverá registrar o título da fonte consultada, a versão ou data de atualização disponível e a data de acesso.

O resumo não será tratado como texto normativo completo, autorização de serviço, laudo ou substituto dos procedimentos oficiais. Em caso de dúvida, ausência de orientação ou conflito, o assistente deverá encaminhar o usuário à documentação oficial e ao profissional responsável.

### D09 — Briefing do Challenge

Reunirá o material fornecido na disciplina sobre o contexto da Motiva, o parque de ativos, as dores apresentadas e os temas de inovação. As informações institucionais serão atribuídas ao briefing. Elas não devem ser misturadas aos dados operacionais simulados pelo grupo.

### D10 — Relatos de atendimento

Serão elaborados 12 relatos narrativos simulados, derivados de 12 ocorrências selecionadas do histórico D03. Cada relato poderá incluir sintoma observado, horário ou condição ambiental relevante, verificações realizadas e resultado do atendimento.

Os relatos não serão chamados de laudos técnicos. Não incluirão nomes, CPF, matrícula, telefone, placas legíveis ou imagens de pessoas identificáveis. Como derivam de ocorrências D03, não serão contabilizados como 12 falhas adicionais.

## 3. Ativos âncora do conjunto simulado

Os cinco ativos abaixo serão usados como exemplos recorrentes no prompt, nas demonstrações e no eval set. A rodovia R01 e todos os registros associados são fictícios. O resumo apresenta **14 ocorrências** dentro do conjunto de 60 registros de D03.

| TAG | Ativo | Criticidade simulada | Ocorrências resumidas no período de referência |
|---|---|---|---|
| `CFTV-R01-K412-01` | Câmera PTZ, km 412 | Alta | 5 ocorrências: 3 de imagem degradada à noite, associadas nos registros a teia de aranha refletindo o infravermelho e resolvidas com limpeza; 1 de desfoque associado à vibração do poste; 1 de perda de link associada a conector. |
| `CFTV-R01-K398-02` | Câmera PTZ, km 398 | Média | 2 ocorrências: fuligem na lente após queimada na faixa de domínio e perda de link associada a switch. |
| `PMV-R01-K405-01` | Painel de mensagens variáveis, km 405 | Alta | 3 ocorrências: 2 de módulo de LED queimado e 1 de falha de comunicação com o CCO. |
| `CBX-R01-K420-03` | Callbox, km 420 | Alta | 3 ocorrências: áudio unidirecional, falha de bateria de backup e falha de geolocalização. |
| `PMV-R01-K388-02` | Painel de mensagens variáveis, km 388 | Média | 1 ocorrência: mensagem travada, com reinício remoto registrado. |

**Regras adicionais do cenário:** no km 412 existe apenas uma câmera PTZ, permitindo identificar o ativo pela referência informal “câmera do km 412”. No km 398 existe apenas uma câmera. Não existe PMV no km 9999; esse exemplo será usado para testar o tratamento de TAG válida ou consulta sem ativo cadastrado, conforme o caso de teste.

## 4. Recuperação de informação (RAG) e metadados

### 4.1 Tamanho dos trechos

Para os documentos textuais indexados no RAG, a proposta inicial é usar trechos de **400 a 600 tokens**, com sobreposição aproximada de **10% a 15%**. O tamanho será ajustado conforme o tipo de documento:

- D02, D05, D07, D09 e D10: divisão por assunto ou seção, preservando o contexto necessário para entender cada trecho;
- D06: divisão por seção e por regra de prioridade, mantendo os critérios relacionados no mesmo trecho;
- D08: divisão por tema ou seção, sem separar uma condição de segurança de sua explicação;
- D01, D03 e D04: não serão indexados no RAG nesta proposta, pois serão consultados por tools.

Como estimativa inicial, recuperar até quatro trechos com cerca de 500 tokens cada poderá adicionar aproximadamente 2.000 tokens ao contexto. Esse valor é uma referência de planejamento, não um limite fixo: a aplicação deverá considerar o tamanho do system prompt, o histórico da conversa, os parâmetros das tools e o limite de contexto do modelo escolhido.

### 4.2 Metadados

Cada trecho indexado deverá manter, quando aplicável, os seguintes metadados:

| Campo | Finalidade |
|---|---|
| `doc_id` | Identifica o documento de origem (D02, D05, D06 etc.). |
| `tipo_documento` | Identifica plano, ficha técnica, matriz, glossário, referência de segurança, briefing ou relato. |
| `tipo_ativo` | Filtra por `CFTV`, `PMV`, `CALLBOX` ou `geral`. |
| `tag` | Relaciona o trecho a um ativo específico quando houver TAG aplicável. |
| `data_referencia` | Indica a data do registro ou a referência temporal do conteúdo, quando aplicável. |
| `versao` | Permite identificar a versão do documento. |
| `simulado` | Booleano que informa se o conteúdo é fictício (`true`) ou se é uma referência documental externa (`false`). |
| `fonte` | Identifica o briefing, norma ou documento de origem. |

### 4.3 Conteúdo que não será armazenado

A base não deverá receber:

- dados pessoais de empregados, como nome, CPF, matrícula, telefone, escala ou endereço;
- imagens de ocupantes de veículos ou imagens que permitam identificar pessoas;
- placas em formato legível; se aparecerem em algum material de exemplo, deverão ser removidas ou mascaradas antes da indexação;
- dados contratuais, comerciais ou financeiros confidenciais da Motiva;
- históricos reais de falhas, dados reais de arrecadação ou informações operacionais não autorizadas;
- credenciais, chaves de API, senhas ou arquivos `.env`.

O projeto desta Sprint utilizará dados sintéticos. A proposta de mascaramento de dados pessoais deverá ocorrer antes de qualquer conteúdo ser enviado ao modelo ou armazenado no RAG; não se presume que o modelo, sozinho, faça essa proteção.

## 5. Limitações conhecidas do inventário

1. O inventário descreve os documentos planejados; os arquivos completos e os 60 registros simulados ainda precisarão ser produzidos e revisados pelo grupo.
2. Os fabricantes, modelos, datas de instalação, vidas úteis e criticidades do catálogo serão fictícios. Não representam cadastro fornecido pela Motiva.
3. Os prazos P1 a P4 são referências acadêmicas simuladas, não compromissos de atendimento da concessionária.
4. As métricas dependem da disponibilidade da tool e de dados de operação coerentes. Sem esses dados, o assistente deverá informar a limitação em vez de estimar valores.
5. Os termos RGA e LDA precisam ter suas definições e métodos confirmados antes de serem usados em uma resposta técnica.
6. Os POPs e o resumo de segurança não estão autorizados para uso operacional real e precisam de validação por profissionais competentes.

## 6. Referências previstas

- Briefing do Challenge Integrador EngMan Copilot — FIAP × Motiva Rodovias, fornecido na disciplina.
- Materiais da disciplina Prompt Engineering & Artificial Intelligence relacionados ao Challenge.
- Textos oficiais vigentes da NR-10 e da NR-35, a serem identificados com versão ou data de atualização na elaboração do D08.

