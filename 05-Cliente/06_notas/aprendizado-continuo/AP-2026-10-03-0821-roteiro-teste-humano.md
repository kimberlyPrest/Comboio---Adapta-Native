# AP-2026-10-03-0821 — Roteiro de teste humano: um caminho só

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-002 / SPEC-F1-01
- Sinal: teste humano devolvido como "falha" sem descrição; 9 provas técnicas verdes não reproduziram o problema; o roteiro original misturava SQL no painel Supabase com login no preview num passo confuso. Roteiro reformulado em caminho único (login no preview primeiro, SQL opcional) foi aprovado de imediato.
- Evidência: `06_notas/debug/debug-2026-10-03-f1-002-roteiro-de-teste.md`; changelog 03/10/2026.
- Regra reutilizável: roteiro de teste humano para não técnico = um caminho só, um ambiente por passo, resultado esperado por passo; se o relato vier sem descrição, pedir evidência mínima (o que fez / esperava / apareceu) antes de alterar produto.
- Quando aplicar: todo portão de teste humano de tasks com prova fora do app.
- Quando não aplicar: quando o relato já traz erro concreto reproduzível — nesse caso debugar direto.
- Confiança: média — causa presumida do roteiro, não confirmada pelo champion; o efeito (reaprovação imediata) é observado.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
