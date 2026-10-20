# F1-023 — Segurança, backup e contingência

Data: 20/10/2026 · Projeto: login-posto-comboio-a8a05.goskip.app (Skip 62225) + Supabase lwkqsukxefwpreqjdkzg

## 1. Revisão de segurança (advisors do linter Supabase)

**Resultado: ZERO crítico (ERROR). WARNs remanescentes documentados e aceitos:**

| Achado | Nível | Ação |
| --- | --- | --- |
| `arrival_requests_audit()` executável por anon via REST | WARN | **CORRIGIDO** (migração `f1_023_security_hardening`): revoke de PUBLIC/anon/authenticated — trigger interno continua funcionando (prova: 1 evento novo em audit_events pós-revoke) |
| `touch_updated_at` com search_path mutável | WARN | **CORRIGIDO** (mesma migração): `set search_path = 'public'` |
| 19 RPCs SECURITY DEFINER executáveis por `authenticated` | WARN | **By design** — toda RPC revalida papel/vínculo internamente (`is_internal_active`, `is_admin_active`, titularidade). Negativas provadas por REST real em cada task (anon 42501 / motorista P0001) |
| Leaked-password protection desativada | WARN | **Pendência de painel (Raysson)** — ativar em Authentication → Settings. Junto com Site URL/Redirect URLs do Auth (localhost quebra redirect de e-mail) |
| Segredo no frontend | — | **ZERO**: `.env` do Skip contém só URL + chave publishable (anon, pública por design). service_role existe só na Edge Function `admin-manage-internal`, no servidor |
| RLS | — | Todas as tabelas com RLS por perfil, provada por REST em F1-002/006/014/018/022 |

## 2. Backup e restore

- **Backups automáticos**: projeto Supabase Pro tem backup diário automático + PITR (point-in-time recovery) gerenciado pela plataforma.
- **Prova de restore executada (20/10/2026)**: dump lógico das 4 tabelas críticas (`visits`, `visit_events`, `arrival_requests`, `profiles`) copiado para schema `restore_test` e conferido: contagens idênticas (10 visitas, 24 eventos, 13 previsões, 21 perfis). Schema de prova removido em seguida. **O dado é restaurável e íntegro.**
- **Procedimento de restore real (pane)**: painel Supabase → Database → Backups → escolher ponto → restaurar em projeto novo ou mesmo projeto (PITR). Duração típica: minutos a ~1h conforme tamanho.

## 3. Contingência operacional (pane do sistema)

**Regra da SPEC**: em pane, operar com ticket físico numerado e lançar depois marcado como `registro posterior`; nunca bloquear o motorista; nunca limpar visitas para "voltar" versão.

1. **Sistema fora** → operador usa bloco de tickets físicos numerados: anota placa, tipo, hora de entrada, nome/contato se houver.
2. **Entrada**: ticket no pára-brisa; dados no papel.
3. **Saída**: operador confere o ticket, anota hora de saída e condição (normal/exceção com motivo).
4. **Sistema volta** → operador lança as visitas com "Registrar chegada espontânea" (preenche o nº do ticket físico no campo "Ticket de referência") e depois "Registrar saída" com a hora do papel no motivo ("registro posterior — saída às HH:MM, ticket 000123").
5. **Nunca**: bloquear veículo por falta de sistema; apagar/recriar visitas; editar eventos históricos.
6. **Falha de rede na tela**: o painel mostra indisponibilidade — nunca mostrar ocupação zero falsa.

## 4. Responsáveis

- **Operação do procedimento**: operador de plantão (contingência é manual por design na Fase 1 — sem cancela física).
- **Backups**: automáticos da plataforma; verificação periódica pelo admin (painel Supabase).
- **Pendências de painel (Raysson)**: Site URL/Redirect URLs do Auth; leaked-password protection.

## 5. Evidências

- Advisors antes/depois (2 WARNs próprios corrigidos; 0 ERROR).
- Prova de restore com contagens idênticas (acima).
- Prova REST: `arrival_requests_audit` → PGRST202 (inacessível via API); `visits` anon → 42501.
- Este documento: `05-Cliente/05_entregas/f1-023-seguranca-backup-contingencia.md` no repo.
