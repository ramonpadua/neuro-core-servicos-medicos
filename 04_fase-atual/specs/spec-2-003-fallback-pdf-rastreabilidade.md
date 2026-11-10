# SPEC-2-003 — Fallback de PDF, exceções e handoff do ciclo

**Fase:** 2 — Central financeira e ingestão multi-fonte  
**Status:** planejada; implementação condicionada às pré-condições abaixo  
**Dono operacional:** Maestro na execução técnica; Fábio Schneider nas respostas, teste e aceite (orientação de 10/11; registros históricos preservados)
**Origem no escopo:** Fase 2; D-003/D-004; RQ-002, RQ-003, RQ-004, RQ-006, RQ-012  
**Degrau da solução:** reuso do sistema existente e recursos nativos autorizados — preservar contrato e decisões da Fase 1.  
**Data:** 2026-10-02

## Contexto e decisões fechadas

- Estado atual: Fase 1 concluída no registro do cliente; handoff em 05_entregas/F1-T012-handoff-fase-2.md. Esta SPEC não declara nova reexecução de testes da Fase 1.
- Estado desejado: Um demonstrativo de teste é importado com origem, página/linha e hash preservados; arquivo repetido não duplica e arquivo ilegível fica em exceção. O ciclo possui pacote de evidências para a próxima fase.
- Decisões: Qualivida é entidade operacional principal; central consumidora é o próprio sistema; catálogo por clínica, guia + record_id, TUSS opcional, retenção permanente do registro, vigência preserva histórico. production-record.v1 e chave sistema:record_id:versão não mudam.
- Bloqueios: Permissão de pasta em leitura, amostra e layout aprovados, mapeamento e política de tratamento administrativo precisam estar registrados em F2-PDF-CONFIG-001. OCR/extração incerta exige revisão humana, sem aprovação automática. API viável não exige ativar fallback real; exige demonstrar a alternativa com fixture autorizada.

## Resultado observável

Um demonstrativo de teste é importado com origem, página/linha e hash preservados; arquivo repetido não duplica e arquivo ilegível fica em exceção. O ciclo possui pacote de evidências para a próxima fase.

## Limites e dependências

- Inclui: Leitura restrita a pasta/fonte autorizada; prévia do demonstrativo; layout versionado, normalização e relatório; fallback visível e recuperação; handoff final do ciclo.
- Fora de escopo: pagamento/Open Banking, migração MDMED, prontuário clínico, reconciliação/cálculo de repasse, go-live com dados reais e contratação de serviço novo.
- Entradas: handoff aprovado para transição pelo consultor; somente fixtures sintéticas inicialmente. Uso real exige fonte, mapeamento, acesso e política autorizados.
- Saídas: ciclo/relatório com proveniência, versão e estados explícitos; provas abaixo; nenhuma credencial em documentos.
- Atores/permissões: Fábio como administrador/champion; herdar RLS por entidade/unidade/profissional e mínima leitura/escrita necessária. Serviço sem usuário não herda acesso global.
- Superfícies: sistema existente em 07_sistemas; referências do handoff src/lib/operational-record.ts, service-catalog.ts, rls-fixture.ts e demonstração em src/pages/Index.tsx. Antes de alterar, confirmar cópia real e ambiente; ausência não autoriza recriação silenciosa.
- Plano B: configuração ausente fica documentada e etapa técnica bloqueada. API inviável encaminha para PDF autorizado. PDF desconhecido vira exceção.
- Rollback: reversão lógica auditada; preservação do original, versões, chaves e evidências; versão anterior continua consultável.

## Dados e integrações

Demonstrativo: source_ref, source_hash, document_version, layout_version, mapping_version, page_or_line, entity_id, unit_id, professional_id, competency, service_ref, quantity/unit_value/gross_value, responsible_user_id, evidence_ref e estado. Chave durável do item de documento combina hash/layout/mapeamento/localização; relação com production-record.v1 exige mapeamento aprovado, sem adulterar sua chave.

| Origem/destino | Fonte de verdade | Permissão | Falha/recuperação |
|---|---|---|---|
| Produção → central lógica | production-record.v1 do handoff | Escopo mínimo registrado; autenticação do ambiente autorizado | Nunca confirmar parcial; repetição com mesma chave |
| Documento → demonstrativo central | Original autorizado + hash/layout/mapeamento | Pasta em leitura; confirmação por ator autorizado | Exceção por layout/identidade; original não alterado |

| Regra | Condição | Resultado | Fonte |
|---|---|---|---|
| RN-2-010 | Layout desconhecido ou PDF ilegível | Exceção; não extrair por suposição nem confirmar parcial | Escopo fase 2 |
| RN-2-011 | Arquivo repetido, mesmos hash e versão | Reutilizar importação registrada; total inalterado | RQ-003 |
| RN-2-012 | Identidade não mapeada ou catálogo não vigente | Bloquear uso real; somente fixture sintética isolada | F1-T001/F1-T005/F1-T012 |
| RN-2-013 | Fonte original | Leitura somente; hash antes/depois iguais | Escopo fase 2 |
| RN-2-014 | Caminho de fase encerrado | Pacote registra usada/não aplicável/bloqueada por rota, sem afirmar integração ausente | F1-HANDOFF-001/escopo fase 2 |

## Fluxo e regras

1. Registrar fonte autorizada em leitura, amostra sintética/anonimizada aprovada e versão do layout. Não buscar em outras pastas nem ler PDFs clínicos.
2. Reusar recurso de extração já disponível no ambiente autorizado. Se parser/OCR não existir ou precisar de serviço externo, parar e registrar dependência; não instalar ou enviar documento a serviço novo por inferência.
3. Preservar original e hash; selecionar arquivos do escopo e detectar repetição por hash + versão do layout/mapeamento. Mesmo nome com bytes diferentes é versão distinta que demanda validação.
4. Extrair apenas campos administrativos reconhecidos. Cada item mantém página/linha ou localização equivalente, fonte, hash e versões de layout/mapeamento. Desconhecido, ambíguo ou ilegível vira exceção.
5. Aplicar catálogo por clínica e identidades aprovadas. Três unidades sem nome e catálogo desatualizado não autorizam inferência de identidades ou preços reais. Valores ausentes permanecem null.
6. Mostrar prévia por lote com campos, origem e pendências. Confirmar somente lote completamente validado pela central da SPEC-2-001. Não promover resultado incerto de OCR automaticamente.
7. Reimportar o mesmo documento, interromper um lote e recuperar; contagem confirmada permanece correta. Reversão preserva original, evidência e histórico.
8. Consolidar pacote final com rota usada, decisão API/fallback, contagens, chaves, exceções, permissões, provas de falha e rollback. A central financeira é o mesmo sistema; reconciliação e cálculo pertencem à fase seguinte.

| Cenário | Entrada | Resultado |
|---|---|---|
| Positivo | F2-PDF-001: PDF sintético layout aprovado | Prévia íntegra e lote confirmado com hash e página/linha |
| Vazio/acesso | Pasta vazia ou sem permissão | Vazio/negado explícito; nenhuma varredura fora do escopo |
| Qualidade | F2-PDF-ERROR-001: ilegível, layout novo, identidade ambígua | Exceção visível, sem dado inventado ou confirmação parcial |
| Repetição | Mesmo arquivo duas vezes e após reinício | Uma importação, total inalterado |
| Mudança | Mesmo nome, hash diferente | Versão nova em staging, revisão explícita e histórico preservado |
| Final | F2-HANDOFF-001: rota viável comprovada | Pacote com proveniência, versão, idempotência e recuperação |

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

- [ ] **CA-2-009:** Pasta/fonte, amostra, layout e mapeamentos autorizados registrados ou bloqueio formalizado. Prova: F2-PDF-CONFIG-001.
- [ ] **CA-2-010:** Amostra administrativa importada com original intacto, hash, origem e localização por item. Prova: F2-PDF-001.
- [ ] **CA-2-011:** Ilegíveis/ambíguos não confirmam lote; repetição/reinício não duplicam; versão nova preserva histórico. Prova: F2-PDF-ERROR-001.
- [ ] **CA-2-012:** Pacote comprova ciclo por API ou fallback documentado, proveniência, idempotência, falha recuperável e aceite. Prova: F2-HANDOFF-001.

## TDD da SPEC

| Etapa | Ação | Resultado/evidência |
|---|---|---|
| RED | Usar PDF sintético de layout aprovado e repetir importação; prova deve detectar perda de localização/proveniência e duplicação antes da solução. | Captura/registro da falha esperada antes da implementação |
| GREEN | Extrair amostra, mostrar prévia validada e confirmar ciclo; comparar hash do original e contagem na repetição. | Prova principal com payload sintético e contagens |
| REFACTOR/REGRESSÃO | Pasta vazia/negada, PDF ilegível, layout desconhecido, identidade ambígua, nulos, conteúdo clínico e arquivo repetido após reinício; provar rollback da central. | Roteiro de cenários com resultado e evidência de cada caso |

**Dados/fixtures:** F2-PDF-CONFIG-001, F2-PDF-001, F2-PDF-ERROR-001, F2-HANDOFF-001; identificadores ENTITY-TEST-001, UNIT-TEST-001, PROF-TEST-001 e fonte fixture://fase-2. Não criar identidades reais ausentes.

**Comandos:** registrar runner real do ambiente autorizado antes de rodar. Build/lint são auxiliares; script de test sem cenários funcionais não comprova aceite. Roteiros acima são a prova mínima reproduzível. Guardar ação, entrada, resultado, versão e evidência em 05_entregas.

## Teste humano do cliente

Origem: demonstração do ciclo da Fase 2 (não há checklist CL-NNN registrado). Quem testa: champion. Abrir prévia, localizar um valor no PDF de origem, confirmar ciclo e reimportar; conferir total estável e original intacto. Revisar pacote final e registrar aceite explícito. Evidência: registro de aceite explícito e versão, sem conteúdo clínico.

## Handoff e operação

Operar somente fontes autorizadas. Monitorar lotes pendentes/falhos, rejeições, duplicidade e versão ativa. Nenhuma exceção vira aprovação automática. Pendências herdadas: três unidades sem nome, catálogo vigente, endpoint/autenticação, política de transporte e armazenamento autorizado. Formato de fechamento/indicadores é tratado na fase de reconciliação.

## Tasks vinculadas

Referência por fase + título exato; UUID do portal pendente. Status vive nos cards de fase.md, sem duplicação nesta tabela.

| Task da Fase 2 | Critério | Prova | Pré-condições |
|---|---|---|---|
| Confirmar qual arquivo PDF pode ser usado como alternativa | CA-2-009 | F2-PDF-CONFIG-001 | Handoff F1-HANDOFF-001 |
| Trazer um PDF de teste e conferir os dados na tela | CA-2-010 | F2-PDF-001 | Confirmar qual arquivo PDF pode ser usado como alternativa; Guardar um ciclo de teste na central |
| Conferir arquivos ilegíveis, incompletos ou repetidos | CA-2-011 | F2-PDF-ERROR-001 | Trazer um PDF de teste e conferir os dados na tela |
| Revisar o ciclo de teste e registrar a aprovação da fase | CA-2-012 | F2-HANDOFF-001 | Conferir que uma falha não deixa dados pela metade; Ver e aprovar o resultado da entrada pela conexão; Conferir arquivos ilegíveis, incompletos ou repetidos |

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

### F2-T009 — Confirmar qual arquivo PDF pode ser usado como alternativa

**Para que serve:** preparar a alternativa por PDF se a conexão automática não puder ser usada.
**Já temos:** as seis planilhas Excel. Elas explicam o processo, mas não comprovam o formato de um PDF nem dão autorização para ler uma pasta.
**Fábio responde no chat, só se ainda faltar:** (1) de qual documento PDF viriam os dados; (2) onde esse documento fica e quem pode autorizar a leitura. O Maestro explica que precisa apenas do modelo correspondente ao teste, não de prontuário clínico. Se a amostra for necessária, solicitar cópia fictícia ou anonimizada na pasta de contexto já usada e pedir que informe o nome; nunca solicitar senha. Não pedir todos os arquivos históricos.
**O Maestro faz:** identifica campos e layout, registra original, versão, permissão de leitura e correspondência com as referências já entregues. Havendo texto que dependa de leitura de imagem, explica o que precisaria de revisão humana; serviço externo novo exige decisão, não envio automático.
**Fábio aprova:** o Maestro mostra quais campos serão lidos e para onde vão; Fábio confirma que esse é o documento certo e o uso permitido. PDF é alternativa prevista, não rota automaticamente ativa.
**Depende:** handoff da F1. **Parar:** PDF não recebido/aprovado, dado clínico, permissão ausente ou extração não disponível. **Prova:** F2-PDF-CONFIG-001 / CA-2-009, SPEC-2-003.

### F2-T010 — Trazer um PDF de teste e conferir os dados na tela

**Para que serve:** ler um PDF de teste e mostrar os dados antes de confirmar a entrada.
**O Maestro faz:** usa somente modelo sintético aprovado na T009 e guarda a referência da página/linha de cada informação. Não escolhe um Excel no lugar do PDF nem altera o arquivo original.
**Fábio confere:** (1) abre o PDF fictício indicado e a tela do Skip pelo link informado; (2) segue o botão real apresentado pelo Maestro para carregar o exemplo; (3) compara mês, profissional/unidade, quantidade e valores com a página de origem; (4) verifica que ausência de valor fica vazia, não zero; (5) somente com a prévia completa e correta confirma pela ação existente indicada. O Maestro explica cada campo e espera uma resposta por etapa; se houver divergência, não confirmar.
**Depende:** T009 e T002. **Parar:** layout desconhecido, dado ambíguo, arquivo real não autorizado ou confirmação parcial. **Prova:** F2-PDF-001 / CA-2-010, SPEC-2-003.

### F2-T011 — Conferir arquivos ilegíveis, incompletos ou repetidos

**Para que serve:** impedir que arquivo repetido conte duas vezes e que documento ilegível vire dado inventado.
**O Maestro faz:** prepara exemplos fictícios de PDF ilegível, modelo desconhecido, identidade ambígua, valor ausente, arquivo repetido e arquivo com mesmo nome mas conteúdo alterado; cobre também pasta vazia/acesso negado em simulação. Não expõe pacientes nem muda permissões reais.
**Fábio confere:** (1) repete o PDF de teste da T010 pela ação real indicada; o total não muda; (2) recarrega e verifica o mesmo total; (3) acompanha um PDF ilegível e vê aviso com motivo, sem confirmação silenciosa; (4) vê a versão alterada apresentada como nova versão para revisão, preservando a anterior. Pedir aceite explícito ao final e explicar os avisos sem jargão.
**Depende:** T010. **Parar:** duplicação, original perdido, preenchimento por suposição ou parcial confirmado. **Prova:** F2-PDF-ERROR-001 / CA-2-011, SPEC-2-003.

### F2-T012 — Revisar o ciclo de teste e registrar a aprovação da fase

**Para que serve:** encerrar a fase sabendo o que funciona, o que não se aplica e o que continua pendente.
**O Maestro faz:** reúne as provas já produzidas sem editá-las, informa qual caminho foi usado (conexão ou PDF), o que não foi usado e por quê. Mostra origem, mês, totais, campos vazios, proteção contra repetição, recuperação e histórico. Não chama teste com dados fictícios de operação real.
**Fábio confere:** (1) o Maestro mostra um exemplo completo desde a origem até a tela; (2) Fábio confere os totais e a identificação do exemplo; (3) acompanha a repetição e a recuperação previamente preparadas; (4) recebe uma lista simples do que falta antes de dados reais; (5) confirma ou aponta a etapa que não funcionou. Cada etapa deve ter o controle real e o resultado esperado explicados antes. Só registrar aprovação com sua confirmação explícita; não abrir fase seguinte automaticamente.
**Depende:** T004, T008 e T011 conforme as regras de aplicabilidade das SPECs. **Parar:** evidência ausente ou teste humano pendente. **Prova:** F2-HANDOFF-001 / CA-2-012, SPEC-2-003.
