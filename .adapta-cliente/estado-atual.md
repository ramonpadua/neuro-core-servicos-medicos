# Estado atual — Adapta Cliente

- fase: 2
- task_id: F2-T002 (referência local; UUID do portal pendente)
- task: Guardar um ciclo de teste na central
- champion: Fábio Schneider
- spec: 04_fase-atual/specs/spec-2-001-central-proveniencia-idempotencia.md
- etapa: aguardando_teste_humano
- autorizacao_implementacao: confirmada — 2026-10-15, champion Fábio Schneider: "Pode implementar a F2-T002" (após relatório de análise)
- teste_humano: pendente
- verificacao_automatica: passou — pipeline Skip v0.0.45→v0.0.49 verde (setup/estática/build/integrações); migrações 0001-0003 aplicadas; GREEN provado no preview: lote F2-LOT-2026-10-001 CONFIRMADO, consultável após reinício, nulos preservados, reimportação → DUPLICADO, contagem no servidor 1 lote/1 emissão; prova F2-CENTRAL-001 commitada (ed6afd7); 2 rodadas de debug documentadas na prova
- aprendizado: pendente
- ultima_acao: prova F2-CENTRAL-001 anexada em 05_entregas (commit ed6afd7); verificações automáticas concluídas
- proxima_acao: aguardar teste humano do champion (roteiro na prova §6)
- atualizado_em: 2026-10-15T16:35:00-03:00

## Orientação de retomada — 10/11/2026

Registro original acima preservado. Antes de qualquer tarefa nova, verificar aprovação posterior da T002; se não existir, conduzir apenas o teste humano descrito na seção F2-T002 da fase atual. Não reimplementar nem marcar tarefa automaticamente. A leitura das planilhas não prova uma integração ativa nem aprova fórmulas para uso atual.
