# Fase 2 — Tarefas

<!-- fase-format:2 -->

**Objetivo:** trazer um ciclo de teste à central financeira do próprio sistema por API preferencial ou fallback de PDF documentado, com proveniência, idempotência durável e recuperação.

**Abertura:** 2026-10-02. Uma task por vez. As tasks de preparação podem ser iniciadas; as demais aguardam suas dependências e acesso autorizado. Somente fixtures sintéticas até autorização de fonte real. UUIDs novos aguardam o portal; responsável e prazo dos cards não foram inventados.

**Registro histórico de autorização (2026-10-05; não usar como lista atual de elegibilidade):** o consultor Felipe Navaar autorizou a execução da Fase 2 pelo champion **Fábio Schneider**. A task **F2-T001 — Confirmar o ambiente e a fonte do primeiro ciclo** está **AUTORIZADA** e é a única elegível agora; uma task por vez, com prova da SPEC e teste humano do champion antes da próxima. As demais tasks permanecem aguardando dependências e nova autorização task a task.

**Progresso: 1/12 concluídas (F2-T001 em 2026-10-15).**

A condução atual está na nota de 10/11 ao fim deste documento: verificar a pendência documental de segurança e o teste da T002; não voltar à T001 nem exigir liberação do consultor para cada tarefa. Aceite humano continua obrigatório.

## Cards

- [x] Confirmar o ambiente e a fonte do primeiro ciclo #projeto
  > SPEC-2-001 — CA-2-001; prova F2-CONFIG-001. Pré-condições: Handoff F1-HANDOFF-001. Registrar ambiente do sistema existente, armazenamento durável autorizado, escopo de acesso e origem sintética do ciclo. Anexar configuração sem segredos, fonte oficial, responsáveis e relação origem → destino. Não escolher nova stack nem criar dados reais. A ausência de acesso fica registrada e impede a task de persistência. **AUTORIZADA em 2026-10-05 pelo consultor Felipe Navaar; execução pelo champion Fábio Schneider.** **CONCLUÍDA em 2026-10-15:** CA-2-001 PASSOU; prova `05_entregas/F2-CONFIG-001.md` (commits 0abc345 e bb09ee7) com ambiente confirmado (Skip 51227 v0.0.44, VITE_POCKETBASE_URL provisionada, backend PocketBase ativo), decisão de armazenamento durável (PocketBase nativo do Skip), estratégia de unicidade/transação/reversão e fonte sintética fixture://fase-2; varredura de segredos limpa; teste humano aprovado pelo champion ("Testei e está correto", 2026-10-15); verificação independente do zero OK. Pendência de higiene registrada: `.env` commitado no repo do sistema (remover/rotacionar antes da F2-T002).

- [ ] Guardar um ciclo de teste na central @Maestro #projeto
  > SPEC-2-001; CA-2-002; prova F2-CENTRAL-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Antes do teste:** o Maestro confere somente os registros do encaminhamento da pendência de segurança do arquivo `.env`, já anotada na T001. Não abre valores, não solicita comandos ao Fábio e não presume segredo exposto. Se o tratamento não estiver comprovado, registra a pendência e pede ao consultor o caminho autorizado antes de executar o teste, sem reconstruir nada.
  > **O que já existe:** a prova de 15/10 registra o ciclo de teste guardado e recuperável. Não pedir implementação de novo. **O que falta verificar:** se Fábio já aprovou o teste depois dessa data.
  > **Passo a passo com Fábio:**
  > 1. O Maestro explica que vamos conferir apenas dados inventados e diz: "Fábio, olha o Skip agora". Informa o link registrado na prova: https://pagina-em-branco-ee3ec--preview.goskip.app . É referência histórica; se o projeto ou a tela mudou, parar e pedir que Fábio indique a página atual. Não iniciar outro app.
  > 2. Peça para localizar a seção "F2-T002 · Ciclo persistido na central" e clicar em "Atualizar consulta". Esperado: lote `F2-LOT-2026-10-001`, situação CONFIRMADO e uma emissão. Se não aparecer, pedir um print recortado da área, sem dados pessoais, e parar.
  > 3. Explique: "Agora atualize a página pelo botão de recarregar do navegador". Depois peça "Atualizar consulta" novamente. Esperado: o mesmo lote e a mesma emissão continuam lá; não sumiram nem aumentaram.
  > 4. Aponte quantidade 1 e os campos de valor sem preenchimento (`null`). Explique que vazio não significa valor zero e que não vamos inventar valor.
  > 5. Peça clicar uma vez em "Reimportar mesmo lote". Esperado: mensagem "DUPLICADO" (significa que o sistema reconheceu o mesmo dado e não criou outro). O total continua com uma emissão.
  > 6. Pergunte se tudo apareceu como descrito. Se sim, pedir "Testei e está correto" e registrar a aprovação vinculada à prova existente. Se der erro vermelho, sumir dado ou aumentar total, manter a pendência e tratar só a falha. Não concluir T003 junto.
  > **Dono:** Maestro na orientação/verificação documental; Fábio no teste e aceite. **Prova:** F2-CENTRAL-001 / CA-2-002; nenhuma prova antiga é editada.

- [ ] Conferir que repetir a entrada não cria registros duplicados @Maestro #projeto
  > SPEC-2-001; CA-2-003; prova F2-IDEMP-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Para que serve:** garantir que clicar de novo, recarregar a página ou receber o mesmo dado ao mesmo tempo não faça a clínica contar o atendimento duas vezes.
  > **O Maestro faz:** confere primeiro o aceite humano da T002. Reaproveita a proteção já documentada; completa apenas os cenários faltantes de repetição após reinício, duas entradas simultâneas e mesmo identificador com conteúdo diferente. Prepara as simulações técnicas em ambiente isolado com dados fictícios; Fábio não precisa abrir código nem fazer requisições.
  > **Fábio confere, uma etapa por vez:** (1) o Maestro mostra o total inicial do exemplo e explica o controle real a usar; (2) Fábio repete a entrada de teste e recarrega; o total fica igual; (3) o Maestro apresenta o resultado da simulação de duas entradas juntas: continua um único registro; (4) mostra uma tentativa de alterar o conteúdo usando a mesma identificação: aparece aviso de conflito e o original continua igual. Ao final, pedir aprovação explícita. Se não existir demonstração segura, registrar a falta; não pedir ao cliente uma operação técnica.
  > **Depende:** T002 com teste humano comprovado. **Parar:** duplicata, alteração do original, teste que exija banco real ou evidência ausente. **Prova:** F2-IDEMP-001 / CA-2-003, SPEC-2-001.

- [ ] Conferir que uma falha não deixa dados pela metade @Maestro #projeto
  > SPEC-2-001; CA-2-004; prova F2-ROLLBACK-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Para que serve:** uma entrada interrompida não deve deixar metade dos dados aparecendo como se estivesse tudo certo.
  > **O Maestro faz:** prepara exemplos fictícios de interrupção no meio, usuário sem acesso, identidade sem correspondência e conteúdo clínico proibido; demonstra recuperação e retorno à versão anterior sem apagar histórico. Não pede que Fábio derrube conexão, altere permissões ou mexa no banco.
  > **Fábio confere:** (1) observa o conjunto de teste antes; (2) vê a falha simulada e confirma que nenhum resultado pela metade aparece como concluído; (3) acompanha a repetição após corrigir o exemplo e vê o conjunto completo; (4) acompanha a volta à versão anterior e verifica que o histórico continua disponível. O Maestro informa nomes dos controles reais e o esperado antes de cada clique; pede aceite ao fim.
  > **Depende:** T003 comprovada. **Parar:** perda de original, exposição parcial ou necessidade de acesso real não autorizado. **Prova:** F2-ROLLBACK-001 / CA-2-004, SPEC-2-001.

- [ ] Confirmar de onde virão os dados e se há conexão automática @Maestro #projeto
  > SPEC-2-002; CA-2-005; prova F2-API-CONFIG-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Para que serve:** descobrir de onde virão os atendimentos e valores e se existe uma maneira segura de recebê-los automaticamente. API significa essa conexão entre sistemas; não é tarefa do Fábio escrevê-la.
  > **Já temos:** exemplos dos campos nas seis planilhas e a fala de que ele sairia do MDMED para um sistema próprio. Não perguntar de novo quais são todos os campos nem pedir reenvio dessas planilhas.
  > **O que Fábio responde no chat do Maestro, uma pergunta por vez:** (1) qual é o nome/link do sistema próprio que mencionou e se já usa ou ainda está construindo; (2) quem consegue explicar tecnicamente como os dados saem dele. Se não souber, "não sei" é resposta válida. O Maestro primeiro verifica se essas respostas já existem nos registros e não as recobra.
  > **O Maestro faz:** distingue a aplicação de origem da central, mesmo se forem partes do mesmo sistema. A API do banco usado pela central não comprova uma API da aplicação de origem. Com o responsável indicado, reúne a documentação necessária de endereço de entrada, acesso mínimo, campos, páginas e limites; usa os exemplos já recebidos para comparar nomes de campos. Não pede token no chat nem testa banco real. Registra conexão viável/documentada, pendente ou inviável com motivo. Ausência de documentação não é inviabilidade provada.
  > **Como Fábio aprova:** o Maestro apresenta em linguagem simples "Os dados saem de X e entram em Y por Z; isto está confirmado; isto ainda falta". Fábio confirma se a origem está correta. Se a conexão for inviável, explica a alternativa PDF já prevista; Excel é referência, não uma nova integração autorizada.
  > **Depende:** handoff da F1. **Parar:** origem indefinida, acesso/política não aprovados ou necessidade de mudar o escopo. **Prova:** F2-API-CONFIG-001 / CA-2-005, SPEC-2-002. Não declarar integração pronta.

- [ ] Trazer um conjunto de dados de teste pela conexão aprovada @Maestro #projeto
  > SPEC-2-002; CA-2-006; prova F2-API-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Para que serve:** mostrar um conjunto completo de dados fictícios entrando pela conexão que foi documentada.
  > **O Maestro faz:** só começa com configuração da T005 aprovada e persistência da T002 confirmada. Prepara o exemplo sintético, confere todas as páginas de resultado e mostra recebidos, aceitos, repetidos e recusados. Não importa as planilhas com pacientes nem ativa dados reais.
  > **Fábio confere:** (1) recebe o link da tela e localiza a área indicada pelo Maestro; (2) usa a ação real apresentada para trazer o exemplo; (3) vê de onde veio, de qual mês e quantos itens apareceram; (4) compara a lista com o exemplo fictício mostrado pelo Maestro, incluindo campos sem valor, que devem continuar vazios. Só então responde se está correto. Se a rota foi declarada inviável, registrar não aplicável com a decisão; não simular uma conexão inexistente como implementada.
  > **Depende:** T005 e T002. **Parar:** lote incompleto, campo inventado, acesso indevido ou ausência de configuração. **Prova:** F2-API-001 / CA-2-006, SPEC-2-002.

- [ ] Conferir o que acontece quando a conexão falha @Maestro #projeto
  > SPEC-2-002; CA-2-007; prova F2-API-ERROR-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Para que serve:** se a fonte estiver lenta, fora do ar, sem autorização ou com resposta incompleta, o sistema deve avisar e não perder nem duplicar os dados.
  > **O Maestro faz:** simula falhas em ambiente isolado: demora, acesso negado, excesso de pedidos, erro do serviço, página ausente, resposta inválida e fonte vazia. Prepara uma demonstração segura; Fábio não precisa saber códigos HTTP nem provocar falha no serviço real.
  > **Fábio confere:** (1) vê a situação de teste e a mensagem simples de erro; (2) verifica que o sistema não chama o resultado incompleto de concluído; (3) após o Maestro preparar a recuperação, tenta novamente pela ação real explicada; o total fica correto e sem duplicação; (4) no caso de fonte vazia, a tela diz que não havia dados, em vez de fingir que importou. Alternativa PDF é oferecida sem buscar pastas automaticamente.
  > **Depende:** T006. **Parar:** retry infinito, perda/duplicação ou ampliação de permissão. **Prova:** F2-API-ERROR-001 / CA-2-007, SPEC-2-002.

- [ ] Ver e aprovar o resultado da entrada pela conexão @Maestro #projeto
  > SPEC-2-002; CA-2-008; prova F2-API-HANDOFF-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Para que serve:** Fábio entender e aprovar o que a conexão realmente entregou, sem relatório cheio de termos técnicos.
  > **O Maestro faz:** reúne provas anteriores e apresenta origem, mês, quantidade, versões, campos vazios e resultado da repetição e da recuperação. Não confundir conexão documentada com conexão implementada.
  > **Fábio confere:** (1) olha a origem e o mês na tela indicada; (2) compara os totais com o resumo do Maestro; (3) acompanha a repetição sem aumentar contagem e a recuperação já demonstrada; (4) diz se o exemplo representa a entrada esperada. Se a API foi inviável, o Maestro mostra o motivo e aponta o teste equivalente de PDF; o aceite não pode afirmar API construída. Pedir confirmação explícita somente do caminho comprovado.
  > **Depende:** T007, ou decisão formal de não aplicabilidade prevista na SPEC. **Parar:** ausência de prova ou caminho não usado contado como entregue. **Prova:** F2-API-HANDOFF-001 / CA-2-008, SPEC-2-002.

- [ ] Confirmar qual arquivo PDF pode ser usado como alternativa @Maestro #projeto
  > SPEC-2-003; CA-2-009; prova F2-PDF-CONFIG-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Para que serve:** preparar a alternativa por PDF se a conexão automática não puder ser usada.
  > **Já temos:** as seis planilhas Excel. Elas explicam o processo, mas não comprovam o formato de um PDF nem dão autorização para ler uma pasta.
  > **Fábio responde no chat, só se ainda faltar:** (1) de qual documento PDF viriam os dados; (2) onde esse documento fica e quem pode autorizar a leitura. O Maestro explica que precisa apenas do modelo correspondente ao teste, não de prontuário clínico. Se a amostra for necessária, solicitar cópia fictícia ou anonimizada na pasta de contexto já usada e pedir que informe o nome; nunca solicitar senha. Não pedir todos os arquivos históricos.
  > **O Maestro faz:** identifica campos e layout, registra original, versão, permissão de leitura e correspondência com as referências já entregues. Havendo texto que dependa de leitura de imagem, explica o que precisaria de revisão humana; serviço externo novo exige decisão, não envio automático.
  > **Fábio aprova:** o Maestro mostra quais campos serão lidos e para onde vão; Fábio confirma que esse é o documento certo e o uso permitido. PDF é alternativa prevista, não rota automaticamente ativa.
  > **Depende:** handoff da F1. **Parar:** PDF não recebido/aprovado, dado clínico, permissão ausente ou extração não disponível. **Prova:** F2-PDF-CONFIG-001 / CA-2-009, SPEC-2-003.

- [ ] Trazer um PDF de teste e conferir os dados na tela @Maestro #projeto
  > SPEC-2-003; CA-2-010; prova F2-PDF-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Para que serve:** ler um PDF de teste e mostrar os dados antes de confirmar a entrada.
  > **O Maestro faz:** usa somente modelo sintético aprovado na T009 e guarda a referência da página/linha de cada informação. Não escolhe um Excel no lugar do PDF nem altera o arquivo original.
  > **Fábio confere:** (1) abre o PDF fictício indicado e a tela do Skip pelo link informado; (2) segue o botão real apresentado pelo Maestro para carregar o exemplo; (3) compara mês, profissional/unidade, quantidade e valores com a página de origem; (4) verifica que ausência de valor fica vazia, não zero; (5) somente com a prévia completa e correta confirma pela ação existente indicada. O Maestro explica cada campo e espera uma resposta por etapa; se houver divergência, não confirmar.
  > **Depende:** T009 e T002. **Parar:** layout desconhecido, dado ambíguo, arquivo real não autorizado ou confirmação parcial. **Prova:** F2-PDF-001 / CA-2-010, SPEC-2-003.

- [ ] Conferir arquivos ilegíveis, incompletos ou repetidos @Maestro #projeto
  > SPEC-2-003; CA-2-011; prova F2-PDF-ERROR-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Para que serve:** impedir que arquivo repetido conte duas vezes e que documento ilegível vire dado inventado.
  > **O Maestro faz:** prepara exemplos fictícios de PDF ilegível, modelo desconhecido, identidade ambígua, valor ausente, arquivo repetido e arquivo com mesmo nome mas conteúdo alterado; cobre também pasta vazia/acesso negado em simulação. Não expõe pacientes nem muda permissões reais.
  > **Fábio confere:** (1) repete o PDF de teste da T010 pela ação real indicada; o total não muda; (2) recarrega e verifica o mesmo total; (3) acompanha um PDF ilegível e vê aviso com motivo, sem confirmação silenciosa; (4) vê a versão alterada apresentada como nova versão para revisão, preservando a anterior. Pedir aceite explícito ao final e explicar os avisos sem jargão.
  > **Depende:** T010. **Parar:** duplicação, original perdido, preenchimento por suposição ou parcial confirmado. **Prova:** F2-PDF-ERROR-001 / CA-2-011, SPEC-2-003.

- [ ] Revisar o ciclo de teste e registrar a aprovação da fase @Maestro #projeto
  > SPEC-2-003; CA-2-012; prova F2-HANDOFF-001. A parte técnica é do Maestro; Fábio responde, testa e aprova.
  > **Para que serve:** encerrar a fase sabendo o que funciona, o que não se aplica e o que continua pendente.
  > **O Maestro faz:** reúne as provas já produzidas sem editá-las, informa qual caminho foi usado (conexão ou PDF), o que não foi usado e por quê. Mostra origem, mês, totais, campos vazios, proteção contra repetição, recuperação e histórico. Não chama teste com dados fictícios de operação real.
  > **Fábio confere:** (1) o Maestro mostra um exemplo completo desde a origem até a tela; (2) Fábio confere os totais e a identificação do exemplo; (3) acompanha a repetição e a recuperação previamente preparadas; (4) recebe uma lista simples do que falta antes de dados reais; (5) confirma ou aponta a etapa que não funcionou. Cada etapa deve ter o controle real e o resultado esperado explicados antes. Só registrar aprovação com sua confirmação explícita; não abrir fase seguinte automaticamente.
  > **Depende:** T004, T008 e T011 conforme as regras de aplicabilidade das SPECs. **Parar:** evidência ausente ou teste humano pendente. **Prova:** F2-HANDOFF-001 / CA-2-012, SPEC-2-003.

## Tasks — referência operacional

Esta tabela é uma projeção de critérios e dependências para compatibilidade. O status editável vive apenas nas caixas dos cards acima. Identificadores F2-Txxx são referências locais de evidência, não UUIDs do portal.

| Referência | Task | Papel de execução | SPEC | Critério | Prova | Evidência | Pré-condições | Ponto de parada |
|---|---|---|---|---|---|---|---|---|
| F2-T001 | Confirmar o ambiente e a fonte do primeiro ciclo | Champion + Ethos, conforme aceite | SPEC-2-001 | CA-2-001 | F2-CONFIG-001 | 05_entregas/F2-CONFIG-001.md | Handoff F1-HANDOFF-001 | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T002 | Guardar um ciclo de teste na central | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-001 | CA-2-002 | F2-CENTRAL-001 | 05_entregas/F2-CENTRAL-001.md | Confirmar o ambiente e a fonte do primeiro ciclo | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T003 | Conferir que repetir a entrada não cria registros duplicados | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-001 | CA-2-003 | F2-IDEMP-001 | 05_entregas/F2-IDEMP-001.md | Guardar um ciclo de teste na central | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T004 | Conferir que uma falha não deixa dados pela metade | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-001 | CA-2-004 | F2-ROLLBACK-001 | 05_entregas/F2-ROLLBACK-001.md | Conferir que repetir a entrada não cria registros duplicados | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T005 | Confirmar de onde virão os dados e se há conexão automática | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-002 | CA-2-005 | F2-API-CONFIG-001 | 05_entregas/F2-API-CONFIG-001.md | Handoff F1-HANDOFF-001 | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T006 | Trazer um conjunto de dados de teste pela conexão aprovada | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-002 | CA-2-006 | F2-API-001 | 05_entregas/F2-API-001.md | Confirmar de onde virão os dados e se há conexão automática; Guardar um ciclo de teste na central | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T007 | Conferir o que acontece quando a conexão falha | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-002 | CA-2-007 | F2-API-ERROR-001 | 05_entregas/F2-API-ERROR-001.md | Trazer um conjunto de dados de teste pela conexão aprovada | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T008 | Ver e aprovar o resultado da entrada pela conexão | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-002 | CA-2-008 | F2-API-HANDOFF-001 | 05_entregas/F2-API-HANDOFF-001.md | Conferir o que acontece quando a conexão falha | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T009 | Confirmar qual arquivo PDF pode ser usado como alternativa | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-003 | CA-2-009 | F2-PDF-CONFIG-001 | 05_entregas/F2-PDF-CONFIG-001.md | Handoff F1-HANDOFF-001 | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T010 | Trazer um PDF de teste e conferir os dados na tela | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-003 | CA-2-010 | F2-PDF-001 | 05_entregas/F2-PDF-001.md | Confirmar qual arquivo PDF pode ser usado como alternativa; Guardar um ciclo de teste na central | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T011 | Conferir arquivos ilegíveis, incompletos ou repetidos | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-003 | CA-2-011 | F2-PDF-ERROR-001 | 05_entregas/F2-PDF-ERROR-001.md | Trazer um PDF de teste e conferir os dados na tela | Acesso/regra ausente; nenhum dado real por inferência |
| F2-T012 | Revisar o ciclo de teste e registrar a aprovação da fase | Maestro: técnica; Fábio: respostas, teste e aceite | SPEC-2-003 | CA-2-012 | F2-HANDOFF-001 | 05_entregas/F2-HANDOFF-001.md | Conferir que uma falha não deixa dados pela metade; Ver e aprovar o resultado da entrada pela conexão; Conferir arquivos ilegíveis, incompletos ou repetidos | Acesso/regra ausente; nenhum dado real por inferência |
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
