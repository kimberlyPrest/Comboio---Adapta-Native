# Ator de auditoria: JWT do chamador, nunca parâmetro

- **Data**: 2026-10-15T17:31:00-03:00 · **Task**: F1-004 · **Capturado por**: Severino (agente)

## O que aconteceu

A Edge Function `admin-manage-internal` recebe o JWT do usuário no header e usa DOIS clientes: `service_role` só para operar `auth.users` (criar/deletar usuário — não existe outra via) e um client com o JWT do próprio chamador para as RPCs de perfil. Assim o banco identifica o ator via `auth.uid()` e os triggers de auditoria gravam quem fez de verdade.

## Por que importa

- Parâmetro "ator" em RPC é spoofável; `auth.uid()` não. Toda RPC de mudança de estado com auditoria deve usar `auth.uid()`.
- RPC `SECURITY DEFINER` com validação `is_admin_active()` no servidor fecha o círculo: mesmo chamada direto por REST por operador, rejeita (P0001 provado).
- O `service_role` nunca sai do servidor (env da função) — linha vermelha respeitada.

## Aplicação futura

Padrão para F1-008/012/015/018 (RPCs transacionais de estado): revalidar papel via `is_admin_active()` no corpo da função, `grant` só a `authenticated` após `revoke` de PUBLIC, ator sempre via `auth.uid()`.
