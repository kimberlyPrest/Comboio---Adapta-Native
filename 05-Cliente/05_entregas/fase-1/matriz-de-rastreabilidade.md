# Matriz de rastreabilidade — fase 1

Versão 1.0 · 29/09/2026. Projeção das mesmas 24 tasks da tabela operacional, Jornada e seções “Tasks vinculadas” das quatro SPECs. `☐` = planejada; `◐` = em andamento; `☑` = concluída com prova. Escopo de quatro fases preservado conforme orientação do cliente; esta matriz cobre apenas a fase em foco. Não constitui gate de aprovação.

| Fase | Origem no escopo | SPEC | Critério | Task | Prova da SPEC | Evidência | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | §§1–3, 7, 9 | F1-01 | CA-F1-01-01 | F1-001 | S101 | Captura/configuração | ☐ |
| 1 | §§1–3, 7, 9 | F1-01 | CA-F1-01-02/03 | F1-002 | S102/S104 | SQL/testes RLS | ☐ |
| 1 | §§1–3, 7, 9 | F1-01 | CA-F1-01-02/05 | F1-003 | S103 | Roteiro Auth | ☐ |
| 1 | §§1–3, 7, 9 | F1-01 | CA-F1-01-03/04 | F1-004 | S104/S105 | Policies/função/auditoria | ☐ |
| 1 | §§1–3, 7, 9 | F1-01 | CA-F1-01-05 | F1-005 | S103/S105 | Capturas por perfil | ☐ |
| 1 | §§3–5, 9 | F1-02 | CA-F1-02-01/04 | F1-006 | S201/S204 | SQL/testes de unicidade | ☐ |
| 1 | §§3–5, 9 | F1-02 | CA-F1-02-01 | F1-007 | S201 | Captura/teste retomada | ☐ |
| 1 | §§3–5, 9 | F1-02 | CA-F1-02-02 | F1-008 | S202 | RPC/auditoria | ☐ |
| 1 | §§3–5, 9 | F1-02 | CA-F1-02-03 | F1-009 | S203 | Captura mobile/teste | ☐ |
| 1 | §§3–5, 9 | F1-02 | CA-F1-02-04 | F1-010 | S204/S205 | Teste vínculo/RLS | ☐ |
| 1 | §§3–5, 9 | F1-02 | CA-F1-02-05 | F1-011 | S205/S206 | E2E/capturas | ☐ |
| 1 | §§1, 5.3, 6–7 | F1-03 | CA-F1-03-03 | F1-012 | S304 | SQL/testes de estados | ☐ |
| 1 | §§1, 5.3, 6–7 | F1-03 | CA-F1-03-01 | F1-013 | S301/S307 | Captura mobile/teste | ☐ |
| 1 | §§1, 5.3, 6–7 | F1-03 | CA-F1-03-02 | F1-014 | S302/S307 | Teste A→B/captura | ☐ |
| 1 | §§1, 5.3, 6–7 | F1-03 | CA-F1-03-02/03 | F1-015 | S303/S304 | RPC/eventos/agenda | ☐ |
| 1 | §§1, 5.3, 6–7 | F1-03 | CA-F1-03-04 | F1-016 | S305/S306 | Teste concorrência | ☐ |
| 1 | §§1, 5.3, 6–7 | F1-03 | CA-F1-03-05 | F1-017 | S307 | E2E/capturas | ☐ |
| 1 | §§2, 5.4–5.5, 6–7, 9 | F1-04 | CA-F1-04-01/04 | F1-018 | S403/S407 | SQL/testes RLS | ☐ |
| 1 | §§2, 5.4–5.5, 6–7, 9 | F1-04 | CA-F1-04-01 | F1-019 | S401/S402/S403 | Eventos/roteiro | ☐ |
| 1 | §§2, 5.4–5.5, 6–7, 9 | F1-04 | CA-F1-04-02 | F1-020 | S404 | Evento/teste vínculo | ☐ |
| 1 | §§2, 5.4–5.5, 6–7, 9 | F1-04 | CA-F1-04-03 | F1-021 | S405 | Teste/captura | ☐ |
| 1 | §§2, 5.4–5.5, 6–7, 9 | F1-04 | CA-F1-04-04 | F1-022 | S406/S407 | Contagem/RLS | ☐ |
| 1 | §§2, 5.4–5.5, 6–7, 9 | F1-04 | CA-F1-04-05 | F1-023 | S403/S407 | Relatório/restore | ☐ |
| 1 | §§2, 5.4–5.5, 6–7, 9 | F1-04 | CA-F1-04-05 | F1-024 | S408 | Ata do piloto | ☐ |

## Regras de sincronização

Ao mudar status/ID, atualizar nesta ordem: `01.Fase_1/00-Tasks_Gerais.md`, `00.tasks_per_fase/fase_1.md`, seção `## Tasks vinculadas` da SPEC e esta matriz. IDs de task `F1-xxx` e UUIDs da Jornada são imutáveis. Critério ou prova novos exigem alteração da SPEC antes da task.
