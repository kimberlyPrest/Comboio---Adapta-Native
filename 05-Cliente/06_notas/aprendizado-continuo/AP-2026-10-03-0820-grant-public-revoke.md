# AP-2026-10-03-0820 — Grant padrão a PUBLIC anula revoke direcionado

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-002 / SPEC-F1-01
- Sinal: `revoke execute ... from anon, authenticated` não bloqueou chamada RPC anônima a função SECURITY DEFINER; o privilégio vinha do grant padrão a `PUBLIC`. Só `revoke ... from public` (+ grant explícito a `authenticated`) endureceu de fato.
- Evidência: chamada RPC como anon retornou `false` após o primeiro revoke; após revoke de PUBLIC retornou `42501 permission denied` (02/10, curl direto); advisors `anon_security_definer_function_executable` e `function_search_path_mutable` zerados.
- Regra reutilizável: ao endurecer funções no Supabase, revogar de `public` (e conferir com chamada RPC real), não só dos papéis anon/authenticated — e validar por tentativa, não pelo advisor.
- Quando aplicar: toda migração que criar funções SECURITY DEFINER ou auxiliares expostas no schema public.
- Quando não aplicar: funções que devem ser RPC públicas intencionais.
- Confiança: alta — comportamento observado duas vezes (antes e depois do revoke correto).
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
