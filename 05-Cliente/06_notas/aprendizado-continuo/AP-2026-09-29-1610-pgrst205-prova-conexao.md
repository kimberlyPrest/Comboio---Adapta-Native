# AP-2026-09-29-1610 — PGRST205 como prova de conexão válida

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-001 / SPEC-F1-01
- Sinal: uma consulta REST a uma tabela inexistente no Supabase retorna `PGRST205` ("Could not find the table ... in the schema cache") quando a chave anon é válida e o projeto é o correto; a resposta é emitida pelo próprio PostgREST do projeto.
- Evidência: reproduzido três vezes — curl direto (29/09 ~15:40), tela `/connection-test` no preview (15:51, 896 ms) e revalidação pós-teste humano (16:01, 1196 ms); documentado em `artifacts/f1-001-evidencia.md` (workspace ETHOS).
- Regra reutilizável: para provar conectividade app↔Supabase em ambiente sem tabelas, consulte uma tabela inexistente e trate `PGRST205` como sucesso (chave aceita, projeto certo); qualquer outro código (401, 403, erro de rede) é falha e deve ser exibido como tal.
- Quando aplicar: telas de diagnóstico de conexão e health checks da fase 1 (ex.: regressão de S101 em F1-002+).
- Quando não aplicar: quando a tabela consultada deveria existir (aí `PGRST205` é defeito de migração); quando policies/RLS devem negar antes do schema cache (nesse caso espere 401/403, não PGRST205).
- Confiança: alta — mesmo comportamento em três execuções independentes, fora e dentro do app.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
