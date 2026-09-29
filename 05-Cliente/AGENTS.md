# Instruções de execução para agentes e implementadores

Trabalhe a partir de `01_projeto/02-Escopo-Definitivo.md`, `04_fase-atual/specs/` e `04_fase-atual/fase.md`, nesta ordem. Execute apenas a fase 1 liberada. Não implemente facial, cancela, triagem completa, cobrança ou integrações comerciais por antecipação.

Stack obrigatória: Skip (goskip.dev) + Supabase. Use Supabase Auth para credenciais, RLS em todas as tabelas expostas e funções seguras para transições privilegiadas. Nunca ponha segredo Supabase/AWS no Skip ou no navegador. Defina cenário de teste antes de cada build; registre evidência e não marque task concluída sem prova. Mantenha dados reais fora dos testes e das capturas.

Se faltar uma decisão que mude regra, permissão ou dado obrigatório, pare essa task e registre a dúvida em `01_projeto/04-Decisoes.md`; não preencha o silêncio com uma suposição. Publicação, criação de contas pagas, convites externos e uso real dependem de autorização da Comboio. A fase 1 só entra em operação após F1-024.
