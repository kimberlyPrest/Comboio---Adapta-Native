# Revoke de PUBLIC não cobre grants explícitos de anon

- **Data**: 2026-10-15T19:06:00-03:00 · **Task**: F1-006 · **Capturado por**: Severino (agente)

## O que aconteceu

A F1-004 fez `revoke ... from public` nas RPCs admin e assumiu a superfície fechada. Os advisors de segurança do Supabase (rodados na F1-006) apontaram que `anon` ainda tinha EXECUTE: o Supabase concede grants EXPLÍCITOS a `anon` e `authenticated` via default privileges, e revoke de PUBLIC não os remove. Ação sempre foi barrada pela revalidação `is_admin_active()` no corpo da função, mas a superfície existia. Corrigido com `revoke ... from anon` explícito; verificado por REST: anon → 42501, admin → executa com validação.

## Por que importa

- Defense in depth real: a validação interna é a última barreira, mas cada papel só deve ENXERGAR o que pode chamar.
- Advisors não são enfeite: rodar após toda migração DDL pegou um buraco que a prova funcional não via (a prova sempre usava token válido).

## Aplicação futura

- Toda RPC SECURITY DEFINER nova: `revoke from public, anon` + `grant to authenticated` (se aplicável) — nunca confiar só no revoke de PUBLIC.
- Rodar `get_advisors(security)` como passo fixo do fechamento de task com DDL.
