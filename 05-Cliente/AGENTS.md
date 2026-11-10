# Instruções de execução para agentes e implementadores

Trabalhe a partir de `01_projeto/02-Escopo-Definitivo.md` (v2.0), `04_fase-atual/specs/` e `04_fase-atual/fase.md`, nesta ordem. **Estão liberadas as fases 2 e 3.** Execute as tasks na ordem das pré-condições; a fase 3 começa depois da F2-024. Não implemente as fases 4 e 5 (notificações automáticas, integrações Promoflex/WebPosto, OCR, facial, cancela) por antecipação.

**Perguntas ao cliente:** cada SPEC tem a seção "Perguntas ao cliente", que diz em qual task perguntar. Antes de implementar essa task, faça a pergunta ao Jean ou ao Rayson em linguagem simples e registre a resposta em `01_projeto/04-Decisoes.md` ("Respostas registradas durante a execução"). Se não houver resposta, siga o padrão escrito na pergunta e registre-o como pendente. Perguntas marcadas como **obrigatórias** param a task até a resposta.

Stack obrigatória: Skip (goskip.dev) + Supabase. Use Supabase Auth para credenciais, RLS em todas as tabelas expostas e funções seguras para transições privilegiadas. Nunca ponha segredo Supabase/AWS no Skip ou no navegador. **Toda migration nova fica versionada em `supabase/migrations/` do repo de código** (a primeira task de banco da fase 2 cria a linha de base do schema). Defina o cenário de teste antes de cada build; registre a evidência e não marque task concluída sem prova. Mantenha dados reais fora dos testes e das capturas.

Modelo no Maestro: Pro como padrão; Ultra para migração, RLS, RPC, Storage, Realtime e jobs.

Se faltar uma decisão que mude regra, permissão ou dado obrigatório e ela não estiver nas perguntas da SPEC, pare essa task e registre a dúvida em `01_projeto/04-Decisoes.md`; não preencha o silêncio com uma suposição. Publicação, criação de contas pagas, convites externos e uso real dependem de autorização da Comboio. Cada fase só entra em operação real depois do seu piloto (F2-025, F3-017).
