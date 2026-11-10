# STATUS — Projeto Qualivida / Neuro Core

**Atualizado em:** 2026-10-15 · **Por:** Adapta Labs

## Onde estamos

- **Fase atual:** 2 — Central financeira e ingestão multi-fonte, aberta em 2026-10-02.
- **Objetivo:** importar um ciclo de teste pela API preferencial ou pelo fallback de PDF documentado, preservando origem, versão, idempotência durável e recuperação.
- **Tasks:** 1/12 (8%). Cards e critérios em 04_fase-atual/fase.md; três SPECs em 04_fase-atual/specs/.
- **Última task concluída:** F2-T001 — Confirmar o ambiente e a fonte do primeiro ciclo (2026-10-15): CA-2-001 PASSOU; prova `05_entregas/F2-CONFIG-001.md`; armazenamento durável decidido (PocketBase nativo do Skip); teste humano aprovado pelo champion.
- **Próxima task:** Guardar um ciclo de teste na central (F2-T002) — pré-condição satisfeita; aguarda autorização task a task do champion.
- **Execução:** uma task por vez; dependências e permissões conferidas antes de implementar.
- **Autorização:** Fase 2 autorizada para execução em 2026-10-05; F2-T002 é a próxima elegível após novo pedido do champion.

## Contexto confirmado

- Fábio Schneider é champion e responsável único. Qualivida é entidade operacional principal.
- A central consumidora é o próprio sistema, conforme decisão da F1-T009.
- Preservar production-record.v1 e chave sistema:record_id:versão; valores ausentes continuam null.
- Execução assistida pelo Ethos no ambiente existente; submódulo 07_sistemas preservado.
- **Armazenamento durável (decisão F2-T001):** PocketBase nativo do Skip Cloud; `VITE_POCKETBASE_URL` provisionada; somente collection `users` existe — collections do ciclo serão criadas na F2-T002 com os campos da SPEC-2-001.

## Pré-condições da Fase 2

| Item | Quem confirma | Quando |
|---|---|---|
| Ambiente existente, cópia de implementação, armazenamento durável e escopo de acesso | Champion/Ethos | ✅ Feito — F2-CONFIG-001 (2026-10-15) |
| Endpoint, autenticação segura, esquema, paginação e limites da API; ou inviabilidade documentada | Champion/administrador | Antes do transporte pela API |
| Pasta/fonte em leitura, amostra e layout reconhecido, mapeamento e revisão de OCR | Champion/financeiro | Antes da importação PDF |
| Catálogo vigente e nomes das três unidades restantes | Champion/financeiro | Antes de dados reais |

Nenhum segredo é gravado no repositório. Fixtures sintéticas isoladas até autorização de fonte real.

**Pendência de higiene (antes da F2-T002):** `.env` commitado na raiz do repo do sistema `fabioschneider9/p-gina-em-branco-njnhgvesg` — remover do repositório e rotacionar o valor (dono: Fábio/administrador).

## Entregas preservadas

- **Fase 1:** 12/12 concluídas no registro do cliente; fase encerrada administrativamente por autorização expressa do consultor em 2026-10-02. Unidade arquivada em 05_entregas/fase-1/.
- Evidências F1-T001 a F1-T012 continuam em 05_entregas; pacote F1-HANDOFF-001 em 05_entregas/F1-T012-handoff-fase-2.md.
- A liberação não registra nova reexecução técnica ou novo teste humano da Fase 1; conserva os aceites e limitações existentes.

## Próxima demonstração

F2-T002: persistir o ciclo sintético na central (PocketBase), reiniciar e mostrar que o ciclo continua consultável com origem/versão completas e nulls preservados. Aceite do champion registrado por task.
## Nota de orientação — 10/11/2026

Revisão apenas documental das tarefas abertas, com teste humano guiado e referências já recebidas. Estado anterior preservado.

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
