# SPEC-2-001 — Central financeira, proveniência e idempotência durável

**Fase:** 2 — Central financeira e ingestão multi-fonte  
**Status:** planejada; implementação condicionada às pré-condições abaixo  
**Dono operacional:** Maestro na execução técnica; Fábio Schneider nas respostas, teste e aceite (orientação de 10/11; registros históricos preservados)
**Origem no escopo:** Fase 2; D-003/D-004; RQ-002, RQ-003, RQ-004, RQ-012  
**Degrau da solução:** reuso do sistema existente e recursos nativos autorizados — preservar contrato e decisões da Fase 1.  
**Data:** 2026-10-02

## Contexto e decisões fechadas

- Estado atual: Fase 1 concluída no registro do cliente; handoff em 05_entregas/F1-T012-handoff-fase-2.md. Esta SPEC não declara nova reexecução de testes da Fase 1.
- Estado desejado: Um ciclo sintético permanece consultável na central após reinício; reimportação e concorrência não duplicam emissões; cada item aponta para fonte e versão.
- Decisões: Qualivida é entidade operacional principal; central consumidora é o próprio sistema; catálogo por clínica, guia + record_id, TUSS opcional, retenção permanente do registro, vigência preserva histórico. production-record.v1 e chave sistema:record_id:versão não mudam.
- Bloqueios: Antes da persistência: ambiente e armazenamento durável autorizados, permissões mínimas e mecanismo de unicidade/transação registrados em F2-CONFIG-001. Se indisponíveis, executar apenas o registro de pré-condições; não substituir por localStorage ou Map.

## Resultado observável

Um ciclo sintético permanece consultável na central após reinício; reimportação e concorrência não duplicam emissões; cada item aponta para fonte e versão.

## Limites e dependências

- Inclui: Modelo mínimo de produção, referência financeira, demonstrativo, regra/mapeamento e evidência; persistência durável de lotes/chaves; normalização aprovada; histórico e reversão lógica.
- Fora de escopo: pagamento/Open Banking, migração MDMED, prontuário clínico, reconciliação/cálculo de repasse, go-live com dados reais e contratação de serviço novo.
- Entradas: handoff aprovado para transição pelo consultor; somente fixtures sintéticas inicialmente. Uso real exige fonte, mapeamento, acesso e política autorizados.
- Saídas: ciclo/relatório com proveniência, versão e estados explícitos; provas abaixo; nenhuma credencial em documentos.
- Atores/permissões: Fábio como administrador/champion; herdar RLS por entidade/unidade/profissional e mínima leitura/escrita necessária. Serviço sem usuário não herda acesso global.
- Superfícies: sistema existente em 07_sistemas; referências do handoff src/lib/operational-record.ts, service-catalog.ts, rls-fixture.ts e demonstração em src/pages/Index.tsx. Antes de alterar, confirmar cópia real e ambiente; ausência não autoriza recriação silenciosa.
- Plano B: configuração ausente fica documentada e etapa técnica bloqueada. API inviável encaminha para PDF autorizado. PDF desconhecido vira exceção.
- Rollback: reversão lógica auditada; preservação do original, versões, chaves e evidências; versão anterior continua consultável.

## Dados e integrações

lot_id; source_kind (PRODUCTION_API, PDF_FALLBACK); source_ref e source_version; source_hash quando arquivo; entity_id, unit_id, professional_id, competency; evidence_ref; contract_version; idempotency_key; estado do lote; ator, data e evento de auditoria. Preservar quantity/unit_value/gross_value como null se ausentes. Referências de regras e demonstrativos armazenam versão e origem, sem calcular repasses nesta fase.

| Origem/destino | Fonte de verdade | Permissão | Falha/recuperação |
|---|---|---|---|
| Produção → central lógica | production-record.v1 do handoff | Escopo mínimo registrado; autenticação do ambiente autorizado | Nunca confirmar parcial; repetição com mesma chave |
| Documento → demonstrativo central | Original autorizado + hash/layout/mapeamento | Pasta em leitura; confirmação por ator autorizado | Exceção por layout/identidade; original não alterado |

| Regra | Condição | Resultado | Fonte |
|---|---|---|---|
| RN-2-001 | production-record.v1 recebido | Preservar envelope, chave e valores null; campo novo exige versão/aprovação | F1-T009/F1-T012 |
| RN-2-002 | Mesma chave e mesmo conteúdo | Uma única emissão persistida; devolver referência existente | RQ-003/F1-T011 |
| RN-2-003 | Mesma chave e conteúdo diferente | CONFLITO_IDEMPOTENCIA, zero sobrescrita | Integridade do contrato aprovado |
| RN-2-004 | Lote incompleto ou identidade ambígua | Lote não confirmado; registrar exceção sem conteúdo clínico | Escopo fase 2 |
| RN-2-005 | Reversão de lote confirmado | Histórico append-only; restaurar referência ativa anterior | Escopo fase 2/rollback |

## Fluxo e regras

1. Ler o contrato production-record.v1 e as decisões da F1-T005/F1-T009; confirmar o ambiente já utilizado, sem criar stack nova.
2. Criar área de staging do ciclo com lote, origem e versão; validar todos os itens antes da confirmação.
3. Reusar validação, RLS e elegibilidade VALIDO + ATENDIDO do sistema existente. Origem financeira/demonstrativo possui envelope de ingestão próprio e não é reinterpretada como produção.
4. Persistir lote, itens e registro de idempotência usando armazenamento autorizado e unicidade no servidor. Uma chave idêntica devolve o resultado persistido; conteúdo divergente sob mesma chave rejeita o lote.
5. Tornar lote visível apenas depois de todos os itens e suas evidências persistirem. Usar transação nativa; se indisponível, estado de lote e barreira de leitura impedem exposição parcial. Registrar a estratégia no F2-CONFIG-001 antes de implementar.
6. Reiniciar e consultar o mesmo ciclo; reimportar e enviar duas chamadas simultâneas. A versão corrigida do registro usa chave nova e mantém a anterior.
7. Reversão marca a versão/lote como revertido e restaura a referência ativa anterior. Mantém trilha, originais e chaves: nunca limpa a idempotência para repetir uma emissão.

| Cenário | Entrada | Resultado |
|---|---|---|
| Positivo | F2-CENTRAL-001: envelope válido sintético | Lote confirmado e consultável após reinício |
| Repetição/concorrência | F2-IDEMP-001: mesma chave, incluindo duas chamadas simultâneas | Uma emissão, mesma referência; nenhum aumento de total |
| Conflito | Mesma chave, payload diferente | Rejeição estruturada; original preservado |
| Falha/rollback | F2-ROLLBACK-001: interrupção no segundo item | Nenhum parcial visível; retry seguro e reversão auditada |
| Permissão/privacidade | Usuário sem escopo ou payload clínico | Negação/rejeição sem persistência ou vazamento |

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

- [ ] **CA-2-001:** Configuração sem segredos registrada, acesso e armazenagem aprovados ou bloqueio formalizado. Prova: F2-CONFIG-001.
- [ ] **CA-2-002:** Ciclo persistido é recuperado após reinício com origem e versão completas e null preservado. Prova: F2-CENTRAL-001.
- [ ] **CA-2-003:** Repetição após reinício e concorrência deixam uma emissão; conflito não sobrescreve. Prova: F2-IDEMP-001.
- [ ] **CA-2-004:** Falha, permissão e rollback não expõem lote parcial nem apagam trilha. Prova: F2-ROLLBACK-001.

## TDD da SPEC

| Etapa | Ação | Resultado/evidência |
|---|---|---|
| RED | Reiniciar a demonstração volátil da fase 1 e evidenciar a falta de idempotência durável; configurar teste que detecta duplicação ou perda da referência persistida. | Captura/registro da falha esperada antes da implementação |
| GREEN | Persistir um ciclo sintético completo, reiniciar, recuperar e repetir a mesma chave; comparar contagem e conteúdo. | Prova principal com payload sintético e contagens |
| REFACTOR/REGRESSÃO | Executar concorrência, mesma chave divergente, falha intermediária, reversão, permissão e privacidade; repetir cenários da fase 1 sem regressão. | Roteiro de cenários com resultado e evidência de cada caso |

**Dados/fixtures:** F2-CONFIG-001, F2-CENTRAL-001, F2-IDEMP-001, F2-ROLLBACK-001; identificadores ENTITY-TEST-001, UNIT-TEST-001, PROF-TEST-001 e fonte fixture://fase-2. Não criar identidades reais ausentes.

**Comandos:** registrar runner real do ambiente autorizado antes de rodar. Build/lint são auxiliares; script de test sem cenários funcionais não comprova aceite. Roteiros acima são a prova mínima reproduzível. Guardar ação, entrada, resultado, versão e evidência em 05_entregas.

## Teste humano do cliente

Origem: demonstração do ciclo da Fase 2 (não há checklist CL-NNN registrado). Quem testa: champion. Abrir ciclo, consultar origem/versão, reiniciar, reimportar e conferir que o total é igual; observar reversão com histórico preservado. Evidência: registro de aceite explícito e versão, sem conteúdo clínico.

## Handoff e operação

Operar somente fontes autorizadas. Monitorar lotes pendentes/falhos, rejeições, duplicidade e versão ativa. Nenhuma exceção vira aprovação automática. Pendências herdadas: três unidades sem nome, catálogo vigente, endpoint/autenticação, política de transporte e armazenamento autorizado. Formato de fechamento/indicadores é tratado na fase de reconciliação.

## Tasks vinculadas

Referência por fase + título exato; UUID do portal pendente. Status vive nos cards de fase.md, sem duplicação nesta tabela.

| Task da Fase 2 | Critério | Prova | Pré-condições |
|---|---|---|---|
| Confirmar o ambiente e a fonte do primeiro ciclo | CA-2-001 | F2-CONFIG-001 | Handoff F1-HANDOFF-001 |
| Guardar um ciclo de teste na central | CA-2-002 | F2-CENTRAL-001 | Confirmar o ambiente e a fonte do primeiro ciclo |
| Conferir que repetir a entrada não cria registros duplicados | CA-2-003 | F2-IDEMP-001 | Guardar um ciclo de teste na central |
| Conferir que uma falha não deixa dados pela metade | CA-2-004 | F2-ROLLBACK-001 | Conferir que repetir a entrada não cria registros duplicados |

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

### F2-T002 — Guardar um ciclo de teste na central

**Antes do teste:** o Maestro confere somente os registros do encaminhamento da pendência de segurança do arquivo `.env`, já anotada na T001. Não abre valores, não solicita comandos ao Fábio e não presume segredo exposto. Se o tratamento não estiver comprovado, registra a pendência e pede ao consultor o caminho autorizado antes de executar o teste, sem reconstruir nada.
**O que já existe:** a prova de 15/10 registra o ciclo de teste guardado e recuperável. Não pedir implementação de novo. **O que falta verificar:** se Fábio já aprovou o teste depois dessa data.
**Passo a passo com Fábio:**
1. O Maestro explica que vamos conferir apenas dados inventados e diz: "Fábio, olha o Skip agora". Informa o link registrado na prova: https://pagina-em-branco-ee3ec--preview.goskip.app . É referência histórica; se o projeto ou a tela mudou, parar e pedir que Fábio indique a página atual. Não iniciar outro app.
2. Peça para localizar a seção "F2-T002 · Ciclo persistido na central" e clicar em "Atualizar consulta". Esperado: lote `F2-LOT-2026-10-001`, situação CONFIRMADO e uma emissão. Se não aparecer, pedir um print recortado da área, sem dados pessoais, e parar.
3. Explique: "Agora atualize a página pelo botão de recarregar do navegador". Depois peça "Atualizar consulta" novamente. Esperado: o mesmo lote e a mesma emissão continuam lá; não sumiram nem aumentaram.
4. Aponte quantidade 1 e os campos de valor sem preenchimento (`null`). Explique que vazio não significa valor zero e que não vamos inventar valor.
5. Peça clicar uma vez em "Reimportar mesmo lote". Esperado: mensagem "DUPLICADO" (significa que o sistema reconheceu o mesmo dado e não criou outro). O total continua com uma emissão.
6. Pergunte se tudo apareceu como descrito. Se sim, pedir "Testei e está correto" e registrar a aprovação vinculada à prova existente. Se der erro vermelho, sumir dado ou aumentar total, manter a pendência e tratar só a falha. Não concluir T003 junto.
**Dono:** Maestro na orientação/verificação documental; Fábio no teste e aceite. **Prova:** F2-CENTRAL-001 / CA-2-002; nenhuma prova antiga é editada.

### F2-T003 — Conferir que repetir a entrada não cria registros duplicados

**Para que serve:** garantir que clicar de novo, recarregar a página ou receber o mesmo dado ao mesmo tempo não faça a clínica contar o atendimento duas vezes.
**O Maestro faz:** confere primeiro o aceite humano da T002. Reaproveita a proteção já documentada; completa apenas os cenários faltantes de repetição após reinício, duas entradas simultâneas e mesmo identificador com conteúdo diferente. Prepara as simulações técnicas em ambiente isolado com dados fictícios; Fábio não precisa abrir código nem fazer requisições.
**Fábio confere, uma etapa por vez:** (1) o Maestro mostra o total inicial do exemplo e explica o controle real a usar; (2) Fábio repete a entrada de teste e recarrega; o total fica igual; (3) o Maestro apresenta o resultado da simulação de duas entradas juntas: continua um único registro; (4) mostra uma tentativa de alterar o conteúdo usando a mesma identificação: aparece aviso de conflito e o original continua igual. Ao final, pedir aprovação explícita. Se não existir demonstração segura, registrar a falta; não pedir ao cliente uma operação técnica.
**Depende:** T002 com teste humano comprovado. **Parar:** duplicata, alteração do original, teste que exija banco real ou evidência ausente. **Prova:** F2-IDEMP-001 / CA-2-003, SPEC-2-001.

### F2-T004 — Conferir que uma falha não deixa dados pela metade

**Para que serve:** uma entrada interrompida não deve deixar metade dos dados aparecendo como se estivesse tudo certo.
**O Maestro faz:** prepara exemplos fictícios de interrupção no meio, usuário sem acesso, identidade sem correspondência e conteúdo clínico proibido; demonstra recuperação e retorno à versão anterior sem apagar histórico. Não pede que Fábio derrube conexão, altere permissões ou mexa no banco.
**Fábio confere:** (1) observa o conjunto de teste antes; (2) vê a falha simulada e confirma que nenhum resultado pela metade aparece como concluído; (3) acompanha a repetição após corrigir o exemplo e vê o conjunto completo; (4) acompanha a volta à versão anterior e verifica que o histórico continua disponível. O Maestro informa nomes dos controles reais e o esperado antes de cada clique; pede aceite ao fim.
**Depende:** T003 comprovada. **Parar:** perda de original, exposição parcial ou necessidade de acesso real não autorizado. **Prova:** F2-ROLLBACK-001 / CA-2-004, SPEC-2-001.
