# Riscos, Ética, Viés, Segurança e LGPD - EngMan Copilot

**Versão:** Sprint 01 · **Recorte:** A - Diagnóstico e Confiabilidade · **Persona:** engenheiro de manutenção

O EngMan Copilot é uma proposta acadêmica de apoio à análise de falhas em CFTV, PMV e Callbox. Como suas recomendações podem influenciar a avaliação de intervenções em equipamentos rodoviários, o projeto precisa deixar claros os limites do assistente, as fontes utilizadas e as situações que exigem validação humana.

Nesta Sprint, a arquitetura, as ferramentas e os controles são especificações de projeto: ainda não estão implementados nem foram testados em operação. Os dados de ativos, TAGs, históricos, POPs e ordens de serviço são simulados, salvo quando o documento indicar que a informação vem do briefing ou de uma referência externa.

## 1. Critérios usados na matriz

**Tipos de impacto:**

- **S — Segurança:** risco para pessoas, equipes de campo ou usuários da rodovia.
- **F — Financeiro:** custo de deslocamento, reparo, indisponibilidade ou decisão de investimento.
- **L — Legal/regulatório:** privacidade, responsabilidade técnica ou descumprimento de requisito aplicável.
- **R — Reputacional:** perda de confiança no assistente ou no projeto.

A classificação de gravidade (alta, média ou baixa) é uma avaliação qualitativa do grupo para esta proposta. Não representa uma análise oficial de risco da Motiva.

## 2. Matriz de riscos

| ID | Risco | Como pode ocorrer no projeto | Impacto | Mitigação prevista | Rastreabilidade |
|---|---|---|---|---|---|
| R01 | Informação inventada ou sem fonte | O assistente completa TAG, modelo, histórico, norma, número de OS ou métrica que não apareceu nos documentos nem nas tools. | F, S, R · **alta** | Usar somente D01–D10 e retornos de tools; citar a fonte; quando o dado não existir ou a consulta falhar, informar a lacuna e não estimar. O controle de saída da aplicação deverá conferir referências e campos críticos antes de exibir a resposta. | Prompt: regras 1 e 6; bloco BORDA. Tools: retornos de erro e ausência de registro. Arquitetura: blocos 3, 6 e 7. Eval: C01, C08 e C17. |
| R02 | Prompt injection e jailbreak | Uma mensagem ou um texto dentro de relato, OS ou documento tenta alterar as regras, mudar a prioridade ou fazer o assistente revelar instruções. | S, F, R · **alta** | Tratar conteúdo recuperado e material enviado como dado, nunca como instrução. Não obedecer a comandos embutidos. A aplicação deverá separar instruções de sistema dos documentos recuperados e aplicar validações de entrada e saída; o prompt, sozinho, não é uma barreira de segurança suficiente. | Prompt: regra 8 e FRASE F3. Arquitetura: blocos 2, 3 e 7. Eval: C12 e C13. |
| R03 | Orientação operacional insegura | O assistente fornece instruções para abrir equipamento energizado, trabalhar em altura ou atuar em pista sem observar o POP e os requisitos aplicáveis. | S, L · **alta** | Não autorizar nem orientar execução física. Quando a solicitação envolver atividade elétrica, altura ou via, não improvisar etapas: consultar o POP quando disponível, informar apenas os requisitos e limites que constarem da fonte e encaminhar a execução a profissional habilitado. A resposta não substitui o procedimento oficial nem a avaliação local. | Prompt: regra 4 e FRASE F2. Base: D04 e D08. Arquitetura: blocos 2 e 7. Eval: C06, C14 e C15. |
| R04 | Priorização enviesada ou inconsistente | Um ativo recebe prioridade maior apenas por ser de determinado tipo, trecho ou praça, ou apenas pelo percentual de degradação. | F, S · **média** | Aplicar os critérios da Matriz D06: criticidade do ativo e do trecho, gravidade/perda de função, persistência e reincidência, cobertura alternativa e impacto em segurança viária e arrecadação. Explicar os critérios utilizados. Se faltarem informações, não atribuir prioridade definitiva. D06 é uma proposta simulada do grupo, não uma matriz oficial da Motiva. | Prompt: regra 10 e exemplos de classificação. Base: D06. Arquitetura: recuperação de D06 no bloco 3. Eval: C02 e C03. |
| R05 | Exposição de dados pessoais | Relatos enviados ao assistente contêm nomes, CPF, telefone, placas legíveis ou imagens de pessoas. | L, R · **alta** | Não incluir esses dados no inventário, no RAG, nos exemplos ou nos dados simulados. Antes de persistir ou indexar documentos, a aplicação deverá sanitizar o conteúdo. Se um dado pessoal ainda aparecer na entrada, o assistente deverá omiti-lo da resposta e tratar somente a informação técnica; não deve afirmar que apagou o dado da origem. | Prompt: regra 9. Base: seção 4.3 e D10. Arquitetura: sanitização antes da montagem do contexto (bloco 2/3, a detalhar). Eval: C11. |
| R06 | Exposição de informação operacional | Histórico ou dados reais da empresa são enviados a serviços externos, incluídos em logs ou publicados por engano no GitHub. | F, L, R · **alta** | Nesta proposta, usar dados simulados e manter o caminho planejado com modelo e embeddings locais. Não colocar dados reais, credenciais, arquivos `.env` ou chaves no repositório. A implementação deverá revisar também logs, backups, permissões e qualquer serviço externo antes de processar informação operacional. Modelo local, isoladamente, não garante segurança dos dados. | README: limitações e declaração de dados simulados. Decisões técnicas: escolha de modelo e embeddings locais. `.gitignore`. Arquitetura: bloco 4. |
| R07 | Excesso de confiança na hipótese | O engenheiro interpreta uma hipótese de causa como diagnóstico confirmado ou usa uma resposta isolada para justificar investimento. | F, S · **alta** | Separar [FATO] de [HIPÓTESE], citar evidências, indicar o que precisa ser verificado e apresentar nível de confiança com justificativa. A recomendação é apoio à decisão; a avaliação e a decisão final permanecem com o engenheiro. | Prompt: regras 2 e 10; blocos FORMATO e PAPEL. Arquitetura: bloco 7. Eval: C03. |
| R08 | Prioridade atribuída com dados insuficientes | O assistente escolhe P1–P4 sem saber gravidade, criticidade, redundância ou impacto no trecho. | F, S · **alta** | Não inferir os campos ausentes. Solicitar os dados necessários ou deixar a prioridade pendente de validação. A Matriz D06 deve documentar como os critérios são combinados e quais informações mínimas são exigidas para cada classificação. | Prompt: regra 10. Base: D06, especialmente critérios e limitações. Eval: C03. |
| R09 | Mistura de TAGs ou perda de contexto | Em uma conversa longa, o assistente associa sintomas ou histórico de um equipamento a outro, ou assume que uma referência como “o do km 398” identifica um único ativo. | F, S · **média** | Manter uma TAG ativa por análise; confirmar a identificação quando houver ambiguidade ou troca de ativo; resumir o atendimento sem misturar históricos; limitar o histórico enviado ao modelo e preservar a identificação do ativo no resumo. | Prompt: bloco BORDA e FORMATO. Arquitetura: blocos 3 e 8. Eval: C16. |
| R10 | Abertura indevida ou duplicada de OS | A ferramenta é chamada sem confirmação válida, com parâmetros alterados depois da aprovação ou após falha/timeout. | F, L, R · **alta** | A aplicação deve criar um identificador para o rascunho, registrar a confirmação do usuário para aquela versão exata e só então disponibilizar a tool. Qualquer alteração exige novo rascunho e nova confirmação. Informar “OS aberta” somente após retorno de sucesso. A OS desta fase é simulada. | Prompt: regra 12. Tools: `abrir_ordem_servico` e `id_rascunho`. Arquitetura: blocos 5 e 6. Eval: C05, C18 e C19. |
| R11 | Falha ou indisponibilidade de tool | Uma consulta retorna timeout, erro, registro vazio ou dados incompletos, mas o assistente responde como se tivesse consultado com sucesso. | F, S, R · **média** | Informar qual consulta não foi concluída; não substituir o retorno por memória ou estimativa. Para métricas, não apresentar λ, MTBF ou R(t) se a tool diferencial estiver indisponível ou retornar dados insuficientes. | Prompt: regra 6, regra 11 e bloco BORDA. Tools: respostas de erro/ausência de registro. Eval: C08 e C17. |
| R12 | Dado simulado confundido com dado real | Um leitor interpreta TAG, criticidade, histórico, POP, prazo ou número de OS fictício como registro operacional da Motiva. | L, R · **média** | Identificar como simulado o conteúdo criado pelo grupo; separar o que vem do briefing e de referências normativas; não usar nomes, logotipos ou afirmações que sugiram integração real. Ao utilizar dado simulado em uma resposta, incluir a identificação definida no prompt. | Prompt: regra 13. Inventário: identificação de origem por documento e seção de ativos âncora. README: apresentação e limitações. Eval: critérios dos casos que usam dados simulados. |

## 3. Medidas previstas para dados pessoais e LGPD

O projeto adota, como diretrizes de tratamento, a finalidade, a necessidade, a segurança, a prevenção e a não discriminação. Nesta fase, essas diretrizes são requisitos de projeto; a documentação não representa certificação nem comprovação de conformidade jurídica ou operacional.

### 3.1 Dados que podem compor a base

- Identificação técnica dos ativos por TAG, tipo, localização operacional simulada, fabricante/modelo fictícios, criticidade e características técnicas.
- Registros simulados de falhas, intervenções, tempos de reparo e condições ambientais, sem identificação de pessoas.
- POPs e referências de segurança usados para contextualizar os limites do atendimento.
- Glossário, plano de manutenção, matriz de criticidade e conteúdo do briefing da disciplina.

### 3.2 Dados que não devem entrar na base, no RAG ou nos exemplos

- Nome, CPF, matrícula, telefone, endereço, escala ou qualquer identificador de empregado ou terceiro.
- Placa legível de veículo ou combinação de dados que permita identificar seu titular.
- Imagens de ocupantes, motoristas, trabalhadores ou outras pessoas identificáveis.
- Dados reais de arrecadação, contratos ou informações operacionais confidenciais que não tenham sido autorizadas para uso acadêmico.
- Credenciais, tokens, chaves de API e conteúdo de arquivos `.env`.

### 3.3 Tratamento quando um usuário enviar dado pessoal

O comportamento previsto é omitir o dado pessoal da resposta e resumir somente o conteúdo técnico que possa ser atendido. O assistente deverá solicitar que o registro original seja sanitizado antes de ser armazenado ou compartilhado. Ele não deve repetir o dado, afirmar que o removeu do arquivo original ou prometer que o sistema o apagou.

A sanitização antes da persistência, indexação, geração de embeddings ou inclusão em logs é responsabilidade da aplicação e precisa ser implementada e testada em uma etapa futura. O texto do system prompt não consegue, sozinho, impedir que um dado enviado seja processado por componentes anteriores ou externos.

## 4. Responsabilidade e limites de uso

- **Apoio, não decisão autônoma:** o EngMan Copilot organiza evidências e recomenda alternativas; o engenheiro avalia e decide.
- **Sem autorização operacional:** o assistente não libera equipamento, não determina retorno à operação e não substitui profissionais habilitados, POPs ou requisitos de segurança.
- **Sem documento com validade legal:** não emite laudo, ART, atestado de conformidade ou parecer de engenharia.
- **OS simulada:** a ferramenta de abertura é apenas uma especificação para a proposta acadêmica. Não representa integração com o FSM nem gera uma ordem real.
- **Fontes delimitadas:** resposta factual deve estar apoiada em D01–D10 ou em retorno de tool. Ausência de dado deve ser tratada como limitação, não como convite a estimar.

## 5. Limitações desta análise de riscos

1. Os controles descritos ainda não foram implementados nem submetidos a testes de segurança. Os casos do eval set são testes planejados, não evidência de que o risco já foi eliminado.
2. A sanitização de dados pessoais, a proteção de logs, o controle de acesso e a validação transacional da confirmação de OS dependem da aplicação que será desenvolvida em etapa posterior.
3. A Matriz D06, os prazos e os critérios P1–P4 são simulados e precisam ser apresentados como proposta acadêmica. Não devem ser atribuídos à Motiva.
4. O caminho local de modelo e embeddings reduz a necessidade de enviar dados a serviços externos, mas não substitui controles de segurança do dispositivo, do armazenamento e do ambiente de execução.
5. A ferramenta de métricas de confiabilidade está classificada como diferencial. Enquanto não estiver disponível e validada, o assistente não poderá apresentar λ, MTBF ou R(t).

## 6. Referências internas e rastreabilidade

- `prompts/system_prompt.md`: regras 1, 2, 3, 4, 5, 6, 8, 9, 10, 11, 12 e 13; blocos PAPEL, FORMATO e BORDA.
- `data/knowledge_base/inventario.md`: D01 a D10, regras de acesso às fontes e seção 4.3 sobre conteúdo que não será armazenado.
- `tools/tools_spec.json`: contratos de `consultar_ativo`, `consultar_historico_falhas`, `consultar_procedimento`, `abrir_ordem_servico` e `calcular_metricas_confiabilidade`.
- `evals/sprint1_eval_set.json`: C01 a C19, com destaque para C06, C08, C11–C15 e C17–C19.
- `docs/arquitetura.png`: deve representar os controles de entrada e saída, a montagem do contexto, a execução das tools e o histórico. O diagrama deverá ser atualizado para deixar explícitos os controles que dependem da aplicação.
- `docs/decisoes_tecnicas.md`: justificativa do modelo e dos componentes locais previstos.

---

**Nota:** as mitigações descritas são decisões e requisitos propostos pelo grupo para o projeto acadêmico. Elas não comprovam que o sistema esteja implementado, seguro para uso operacional ou em conformidade com todos os requisitos internos da Motiva.
