# SPEC-2-002 — Entrada pela API e recuperação de falhas

**Fase:** 2 — Central financeira e ingestão multi-fonte  
**Status:** planejada; implementação condicionada às pré-condições abaixo  
**Dono operacional:** Maestro na execução técnica; Fábio Schneider nas respostas, teste e aceite (orientação de 10/11; registros históricos preservados)
**Origem no escopo:** Fase 2; D-003/D-004; RQ-002, RQ-003, RQ-004, RQ-012  
**Degrau da solução:** reuso do sistema existente e recursos nativos autorizados — preservar contrato e decisões da Fase 1.  
**Data:** 2026-10-02

## Contexto e decisões fechadas

- Estado atual: Fase 1 concluída no registro do cliente; handoff em 05_entregas/F1-T012-handoff-fase-2.md. Esta SPEC não declara nova reexecução de testes da Fase 1.
- Estado desejado: Um ciclo de teste chega ao módulo central do próprio sistema pela API autorizada, com relatório de entrada e recuperação sem duplicação. Se inviável, a decisão e o caminho alternativo ficam demonstráveis.
- Decisões: Qualivida é entidade operacional principal; central consumidora é o próprio sistema; catálogo por clínica, guia + record_id, TUSS opcional, retenção permanente do registro, vigência preserva histórico. production-record.v1 e chave sistema:record_id:versão não mudam.
- Bloqueios: Endpoint, autenticação, escopos, esquema, paginação e orçamento de timeout/retry devem existir e ser autorizados em F2-API-CONFIG-001. API inviável é decisão registrada que ativa a rota alternativa; não inventar endpoint ou conceder acesso.

## Resultado observável

Um ciclo de teste chega ao módulo central do próprio sistema pela API autorizada, com relatório de entrada e recuperação sem duplicação. Se inviável, a decisão e o caminho alternativo ficam demonstráveis.

## Limites e dependências

- Inclui: Adaptador do contrato existente; configuração da API; paginação completa; validação, relatório de erro e retry seguro; indicação explícita de fallback disponível.
- Fora de escopo: pagamento/Open Banking, migração MDMED, prontuário clínico, reconciliação/cálculo de repasse, go-live com dados reais e contratação de serviço novo.
- Entradas: handoff aprovado para transição pelo consultor; somente fixtures sintéticas inicialmente. Uso real exige fonte, mapeamento, acesso e política autorizados.
- Saídas: ciclo/relatório com proveniência, versão e estados explícitos; provas abaixo; nenhuma credencial em documentos.
- Atores/permissões: Fábio como administrador/champion; herdar RLS por entidade/unidade/profissional e mínima leitura/escrita necessária. Serviço sem usuário não herda acesso global.
- Superfícies: sistema existente em 07_sistemas; referências do handoff src/lib/operational-record.ts, service-catalog.ts, rls-fixture.ts e demonstração em src/pages/Index.tsx. Antes de alterar, confirmar cópia real e ambiente; ausência não autoriza recriação silenciosa.
- Plano B: configuração ausente fica documentada e etapa técnica bloqueada. API inviável encaminha para PDF autorizado. PDF desconhecido vira exceção.
- Rollback: reversão lógica auditada; preservação do original, versões, chaves e evidências; versão anterior continua consultável.

## Dados e integrações

Contrato production-record.v1 sem alteração: origem/versão, dimensões, service_ref, competency, quantidade e valores, estado, responsável, auditoria e idempotency_key. Metadados do transporte: lot_id, source_ref, request_ref sem segredo, paginação concluída, status e erros sanitizados.

| Origem/destino | Fonte de verdade | Permissão | Falha/recuperação |
|---|---|---|---|
| Produção → central lógica | production-record.v1 do handoff | Escopo mínimo registrado; autenticação do ambiente autorizado | Nunca confirmar parcial; repetição com mesma chave |
| Documento → demonstrativo central | Original autorizado + hash/layout/mapeamento | Pasta em leitura; confirmação por ator autorizado | Exceção por layout/identidade; original não alterado |

| Regra | Condição | Resultado | Fonte |
|---|---|---|---|
| RN-2-006 | Página ausente, erro ou resposta incompleta | Não confirmar lote; indicar estágio de falha | Escopo fase 2 |
| RN-2-007 | 401/403 | Parar e registrar acesso negado sem ampliar escopo | RQ-012 |
| RN-2-008 | Fonte vazia | Mostrar fonte vazia, zero importado; não confundir com sucesso de ciclo | Qualidade de origem |
| RN-2-009 | API indisponível | Oferecer fallback autorizado com origem distinta | D-003/escopo fase 2 |

## Fluxo e regras

1. Confirmar produtor e central lógica no próprio sistema conforme F1-T009; identificar API existente e registrar contrato técnico sem segredos.
2. Registrar endpoint/método, esquema dos itens, cursor/paginação, fonte de verdade, escopos, referência segura da autenticação e limites documentados pelo provedor. Timeout/retry fica explicitamente fechado nesta configuração, nunca arbitrado pelo executor.
3. Se API inviável, registrar motivo, evidência e decisão; não criar integração fictícia. Prosseguir somente com fallback autorizado da SPEC-2-003.
4. Solicitar um ciclo sintético e percorrer todas as páginas no staging. Guardar identificação da resposta e da origem sem armazenar credencial ou payload clínico.
5. Validar os itens contra production-record.v1. Mesmo quando origem e central compartilham sistema, a fronteira de validação e autorização permanece obrigatória.
6. Confirmar via SPEC-2-001 apenas depois de resposta/paginação completa. Relatório registra recebidos, aceitos, duplicados e rejeitados, origem e versão; null não vira zero.
7. No timeout/429/5xx, registrar falha e próximo passo. Repetir somente com idempotência durável e dentro da política aprovada. 401/403 bloqueiam e exigem correção de permissão, sem retry de autenticação infinito.
8. Indisponibilidade mostra alternativa; não varre pasta nem troca a fonte silenciosamente. Depois da recuperação, reprocessar o ciclo mantendo as chaves.

| Cenário | Entrada | Resultado |
|---|---|---|
| Positivo | F2-API-001: um ciclo sintético com duas páginas | Todos os itens visíveis com origem e versão após confirmação |
| Vazio | Fonte sem itens | Relatório fonte vazia; nenhum dado fabricado |
| Acesso | 401/403 ou escopo não autorizado | Sem lote confirmado; orientar responsável |
| Falha | F2-API-ERROR-001: timeout, 429, 5xx, página faltante | Lote em falha; retry controlado; nenhuma duplicação |
| Contrato | Status inválido, payload clínico, identidade ambígua | Rejeitar e registrar categoria, nunca saída parcial |
| Alternativa | API declarada inviável | Decisão registrada, referência à rota PDF autorizada |

## Instruções de execução para o Ethos

1. Ler handoff F1-T012 §§2–10, decisões F1-T005/F1-T009 e esta SPEC.
2. Executar uma task por vez, observando as dependências. Preparação registra decisão/acesso; não conta como integração.
3. Alterar apenas o recorte da task. Preservar contrato, RLS, auditoria, null e originais.
4. Registrar configuração autorizada antes da implementação; se mudar regra de negócio/contrato, devolver ao consultor para emenda. Não inventar arquitetura ou campo.
5. Parar sem acesso, documento, regra, layout, permissão ou confirmação. Não inserir segredos no GitHub.
6. Deixar o sistema anterior funcionando e nenhum lote parcial visível; anexar prova em 05_entregas e solicitar teste humano antes de marcar task.

## Checklist de execução

- [ ] Pré-condições e ambiente conferidos; bloqueios explícitos.
- [ ] Caminho principal demonstrado com fixture sintética.
- [ ] Falhas, permissão, privacidade e repetição demonstradas.
- [ ] Original/versão anterior preservados; rollback demonstrável.
- [ ] Provas e aceite humano registrados, sem segredos ou dados clínicos.

## Critérios de aceite

- [ ] **CA-2-005:** Viabilidade e configuração autorizada da API registradas, ou inviabilidade com motivo. Prova: F2-API-CONFIG-001.
- [ ] **CA-2-006:** Ciclo sintético completo importado pela API com origem/versão por item. Prova: F2-API-001.
- [ ] **CA-2-007:** Falhas e paginação incompleta não confirmam lote; retry não duplica; acesso indevido é negado. Prova: F2-API-ERROR-001.
- [ ] **CA-2-008:** Demonstração e relatório da rota API, ou não aplicabilidade explícita vinculada ao fallback aprovado. Prova: F2-API-HANDOFF-001.

## TDD da SPEC

| Etapa | Ação | Resultado/evidência |
|---|---|---|
| RED | Usar transporte simulado ou sandbox autorizado com duas páginas e falha na segunda; evidenciar que lote parcial precisa permanecer invisível. | Captura/registro da falha esperada antes da implementação |
| GREEN | Executar ciclo de duas páginas até confirmação e repetir; o total não aumenta. | Prova principal com payload sintético e contagens |
| REFACTOR/REGRESSÃO | Fonte vazia, timeout, 401/403, 429, 5xx, payload inválido, clínico e repetição após reinício. Rota declarada inviável permanece como não aplicável, nunca como teste API aprovado. | Roteiro de cenários com resultado e evidência de cada caso |

**Dados/fixtures:** F2-API-CONFIG-001, F2-API-001, F2-API-ERROR-001, F2-API-HANDOFF-001; identificadores ENTITY-TEST-001, UNIT-TEST-001, PROF-TEST-001 e fonte fixture://fase-2. Não criar identidades reais ausentes.

**Comandos:** registrar runner real do ambiente autorizado antes de rodar. Build/lint são auxiliares; script de test sem cenários funcionais não comprova aceite. Roteiros acima são a prova mínima reproduzível. Guardar ação, entrada, resultado, versão e evidência em 05_entregas.

## Teste humano do cliente

Origem: demonstração do ciclo da Fase 2 (não há checklist CL-NNN registrado). Quem testa: champion. Acompanhar o ciclo da origem até a central, ver origem e competência; repetir, interromper a segunda página e recuperar com total estável. Evidência: registro de aceite explícito e versão, sem conteúdo clínico.

## Handoff e operação

Operar somente fontes autorizadas. Monitorar lotes pendentes/falhos, rejeições, duplicidade e versão ativa. Nenhuma exceção vira aprovação automática. Pendências herdadas: três unidades sem nome, catálogo vigente, endpoint/autenticação, política de transporte e armazenamento autorizado. Formato de fechamento/indicadores é tratado na fase de reconciliação.

## Tasks vinculadas

Referência por fase + título exato; UUID do portal pendente. Status vive nos cards de fase.md, sem duplicação nesta tabela.

| Task da Fase 2 | Critério | Prova | Pré-condições |
|---|---|---|---|
| Confirmar de onde virão os dados e se há conexão automática | CA-2-005 | F2-API-CONFIG-001 | Handoff F1-HANDOFF-001 |
| Trazer um conjunto de dados de teste pela conexão aprovada | CA-2-006 | F2-API-001 | Confirmar de onde virão os dados e se há conexão automática; Guardar um ciclo de teste na central |
| Conferir o que acontece quando a conexão falha | CA-2-007 | F2-API-ERROR-001 | Trazer um conjunto de dados de teste pela conexão aprovada |
| Ver e aprovar o resultado da entrada pela conexão | CA-2-008 | F2-API-HANDOFF-001 | Conferir o que acontece quando a conexão falha |

## Emendas

Nenhuma. Qualquer mudança de contrato ou autorização material deve ser registrada antes de executar.

## Emenda de condução e referências recebidas — 10/11/2026

## Como o Maestro deve conduzir esta fase (ajuste de clareza de 10/11/2026)

Este ajuste é apenas documental. Não refaz o sistema, não altera escopo, critérios, dependências, IDs, prazos ou aprovações existentes. Não executa migrações, não acessa banco do cliente e não importa pacientes reais. A Fase 1, os arquivos de prova e as tarefas já marcadas como concluídas ficam intactos.

O sistema organiza os atendimentos realizados, os valores e a conferência financeira por profissional, serviço, unidade, pagador e mês. Não é um CRM comercial nem prontuário clínico completo. A expressão "centralizador" da primeira conversa não autoriza criar funções comerciais novas.

- **Quem faz a parte técnica:** o Maestro do projeto, reaproveitando o que já existe. Fábio responde perguntas da operação, testa a tela e confirma o resultado. Não pedir a ele comandos, código, configuração de banco, chaves técnicas, logs ou edição de arquivos.
- **Antes de agir:** ler os registros da tarefa e explicar em poucas frases o que já existe, o que falta e o que será testado. Fazer uma pergunta por vez, somente quando a resposta ainda não estiver documentada. Uma tarefa por vez dentro da fase liberada. O consultor não é aprovador obrigatório entre tarefas; escalar apenas mudança de escopo/plano ou bloqueio fora da autonomia.
- **Ao mostrar a tela:** dizer "Fábio, olha o Skip agora"; fornecer o link do projeto correto e o nome da seção que existe de fato. Mostrar uma ação por vez: onde clicar, o que deve aparecer e como reconhecer um erro. Esperar a resposta antes de pedir o próximo passo.
- **Sem tela inventada:** os botões da T002 abaixo constam da prova de 15/10, mas não foram conferidos ao vivo nesta revisão. Nas tarefas seguintes, os passos descrevem o comportamento esperado; o Maestro deve primeiro identificar o controle real e explicar seu nome e posição. Se não existir, registrar a lacuna da própria tarefa, nunca mandar o cliente procurar um botão inexistente.
- **Teste humano:** confirmar explicitamente "Testei e está correto" ou equivalente; registrar ação, resultado, evidência sem dados pessoais e versão. Silêncio ou teste automático não é aprovação humana. Não marcar tarefa nem seguir automaticamente.
- **Segurança:** preparar testes técnicos com dados fictícios em ambiente isolado. Não conectar, consultar ou testar banco do cliente, mesmo em homologação. Se a comprovação exigir acesso real, parar e pedir ao consultor o caminho autorizado; o teste da interface é conduzido pelo próprio Fábio. Não usar as planilhas com pacientes como carga de teste. Não publicar, apagar, alterar permissões ou expor credenciais.

### Qual registro seguir agora

A Jornada do plano no Drive conserva T001 e T002 marcadas como concluídas e T003 em andamento (registro de 22/10). O repositório/pasta operacional conserva 1/12, com T002 em `aguardando_teste_humano` desde 15/10. **Nenhuma marcação foi rebaixada, promovida ou reconciliada nesta revisão.** A primeira ação segura é verificar se há aprovação posterior da T002. Se não houver, guiar somente o teste pendente descrito abaixo, sem reimplementar a T002 nem iniciar T003. Se a aprovação já existir, registrar sua referência e comunicar a divergência para sincronização, sem inventar aceite.

### O que os documentos recebidos já esclarecem

- A primeira call confirma que Fábio pretendia sair do MDMED para um sistema próprio. Não comprova qual sistema já substituiu o MDMED, seu endereço ou uma API funcionando. Não voltar a pedir a API do MDMED como premissa.
- Ricardo, Isis e Victor Hugo já fornecem exemplos de data, paciente, pagador, serviço e valores, com contas diferentes por profissional/serviço. Na amostra de Ricardo: `Abril 26!F2 = E2*0.5` e `G4 = E4*0.8`; na de Victor Hugo: `G2 = F2*0.8` e `I2 = G2*0.5`; Isis: `E2 = D2*0.8`. Essas fórmulas demonstram o histórico, não a vigência nem a aprovação atual da regra.
- O Modelo Faturamento Qualivida mostra a organização mensal por profissional/serviço e pagador; em `Mar26` há **21 categorias pagadoras mais a coluna Total**, não 22 pagadores. Inclui particular e prefeituras, não só convênios.
- QLV2 já mostra valores, quantidades, médias, faixas de metas, bônus e capacidade. Há exemplos de bônus diferentes por pessoa, inclusive 150/300/450 e 100/150/200. Não aplicar uma faixa universal nem chamar capacidade de meta aprovada.
- Spike2 organiza quantidades por nome, unidade, exame e mês, além de indicadores internos. A função de cada nome (solicitante, executor ou laudador) deve ser confirmada; o arquivo sozinho não prova que são laudadores. Seu histórico se estende até 2026, não apenas 2022.
- Há divergência entre a aba e datas, nomes com acentuação corrompida, cabeçalhos diferentes e meses sem aba. Tratar como pendências de qualidade, não corrigir datas nem completar valores por suposição. Aba ausente não prova atendimento ausente.

**Não recobrar os seis arquivos já entregues.** Usá-los como referência. Ainda faltam: identificação da aplicação própria e responsável técnico, viabilidade/documentação da conexão, amostra e permissão do PDF caso necessário, catálogo e identidades vigentes antes de dados reais. Regras de repasse/comissão, período de validade, significado de cada valor e baseline de tempo só serão confirmados no momento previsto das fases seguintes; não criar cálculo financeiro na Fase 2. Importação automática de Excel não passa a fazer parte do escopo por esta revisão.

### Onde Fábio responde

Responder no chat do seu Maestro, na conversa da tarefa. Exemplo da T005: "Meu sistema próprio é [nome/link]. Quem cuida dele é [nome]. Ele já está em uso/em construção/não sei". O Maestro registra a resposta na tarefa; se precisar documentação técnica, explica quem deve fornecê-la e por quê. Senhas/chaves não vão no chat, no repositório ou na descrição da tarefa.

### F2-T005 — Confirmar de onde virão os dados e se há conexão automática

**Para que serve:** descobrir de onde virão os atendimentos e valores e se existe uma maneira segura de recebê-los automaticamente. API significa essa conexão entre sistemas; não é tarefa do Fábio escrevê-la.
**Já temos:** exemplos dos campos nas seis planilhas e a fala de que ele sairia do MDMED para um sistema próprio. Não perguntar de novo quais são todos os campos nem pedir reenvio dessas planilhas.
**O que Fábio responde no chat do Maestro, uma pergunta por vez:** (1) qual é o nome/link do sistema próprio que mencionou e se já usa ou ainda está construindo; (2) quem consegue explicar tecnicamente como os dados saem dele. Se não souber, "não sei" é resposta válida. O Maestro primeiro verifica se essas respostas já existem nos registros e não as recobra.
**O Maestro faz:** distingue a aplicação de origem da central, mesmo se forem partes do mesmo sistema. A API do banco usado pela central não comprova uma API da aplicação de origem. Com o responsável indicado, reúne a documentação necessária de endereço de entrada, acesso mínimo, campos, páginas e limites; usa os exemplos já recebidos para comparar nomes de campos. Não pede token no chat nem testa banco real. Registra conexão viável/documentada, pendente ou inviável com motivo. Ausência de documentação não é inviabilidade provada.
**Como Fábio aprova:** o Maestro apresenta em linguagem simples "Os dados saem de X e entram em Y por Z; isto está confirmado; isto ainda falta". Fábio confirma se a origem está correta. Se a conexão for inviável, explica a alternativa PDF já prevista; Excel é referência, não uma nova integração autorizada.
**Depende:** handoff da F1. **Parar:** origem indefinida, acesso/política não aprovados ou necessidade de mudar o escopo. **Prova:** F2-API-CONFIG-001 / CA-2-005, SPEC-2-002. Não declarar integração pronta.

### F2-T006 — Trazer um conjunto de dados de teste pela conexão aprovada

**Para que serve:** mostrar um conjunto completo de dados fictícios entrando pela conexão que foi documentada.
**O Maestro faz:** só começa com configuração da T005 aprovada e persistência da T002 confirmada. Prepara o exemplo sintético, confere todas as páginas de resultado e mostra recebidos, aceitos, repetidos e recusados. Não importa as planilhas com pacientes nem ativa dados reais.
**Fábio confere:** (1) recebe o link da tela e localiza a área indicada pelo Maestro; (2) usa a ação real apresentada para trazer o exemplo; (3) vê de onde veio, de qual mês e quantos itens apareceram; (4) compara a lista com o exemplo fictício mostrado pelo Maestro, incluindo campos sem valor, que devem continuar vazios. Só então responde se está correto. Se a rota foi declarada inviável, registrar não aplicável com a decisão; não simular uma conexão inexistente como implementada.
**Depende:** T005 e T002. **Parar:** lote incompleto, campo inventado, acesso indevido ou ausência de configuração. **Prova:** F2-API-001 / CA-2-006, SPEC-2-002.

### F2-T007 — Conferir o que acontece quando a conexão falha

**Para que serve:** se a fonte estiver lenta, fora do ar, sem autorização ou com resposta incompleta, o sistema deve avisar e não perder nem duplicar os dados.
**O Maestro faz:** simula falhas em ambiente isolado: demora, acesso negado, excesso de pedidos, erro do serviço, página ausente, resposta inválida e fonte vazia. Prepara uma demonstração segura; Fábio não precisa saber códigos HTTP nem provocar falha no serviço real.
**Fábio confere:** (1) vê a situação de teste e a mensagem simples de erro; (2) verifica que o sistema não chama o resultado incompleto de concluído; (3) após o Maestro preparar a recuperação, tenta novamente pela ação real explicada; o total fica correto e sem duplicação; (4) no caso de fonte vazia, a tela diz que não havia dados, em vez de fingir que importou. Alternativa PDF é oferecida sem buscar pastas automaticamente.
**Depende:** T006. **Parar:** retry infinito, perda/duplicação ou ampliação de permissão. **Prova:** F2-API-ERROR-001 / CA-2-007, SPEC-2-002.

### F2-T008 — Ver e aprovar o resultado da entrada pela conexão

**Para que serve:** Fábio entender e aprovar o que a conexão realmente entregou, sem relatório cheio de termos técnicos.
**O Maestro faz:** reúne provas anteriores e apresenta origem, mês, quantidade, versões, campos vazios e resultado da repetição e da recuperação. Não confundir conexão documentada com conexão implementada.
**Fábio confere:** (1) olha a origem e o mês na tela indicada; (2) compara os totais com o resumo do Maestro; (3) acompanha a repetição sem aumentar contagem e a recuperação já demonstrada; (4) diz se o exemplo representa a entrada esperada. Se a API foi inviável, o Maestro mostra o motivo e aponta o teste equivalente de PDF; o aceite não pode afirmar API construída. Pedir confirmação explícita somente do caminho comprovado.
**Depende:** T007, ou decisão formal de não aplicabilidade prevista na SPEC. **Parar:** ausência de prova ou caminho não usado contado como entregue. **Prova:** F2-API-HANDOFF-001 / CA-2-008, SPEC-2-002.
