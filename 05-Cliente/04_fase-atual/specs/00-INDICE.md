# Fase 1 — índice de SPECs

Fonte: [escopo definitivo](../../01_projeto/02-Escopo-Definitivo.md). As quatro SPECs seguem a estrutura documental do plugin (contexto, degrau, limites, dados, fluxo, instruções, checklist, aceite, TDD, handoff, tasks e emendas). Status `planejada` significa ainda não implementada. O escopo de quatro fases e os gates foram deixados fora desta adequação por decisão do cliente.

| Ordem | SPEC | Resultado |
| --- | --- | --- |
| 1 | [SPEC-F1-01 Plataforma, identidade e isolamento](spec-f1-01-plataforma-acesso.md) | Login, perfis, base persistente, isolamento e auditoria |
| 2 | [SPEC-F1-02 Cadastros](spec-f1-02-cadastros.md) | Empresa, motorista, veículo e vínculos |
| 3 | [SPEC-F1-03 Previsões](spec-f1-03-previsoes.md) | Autosserviço do motorista, solicitações e agenda |
| 4 | [SPEC-F1-04 Operação e aceite](spec-f1-04-operacao-aceite.md) | Check-in/out, ocupação e prova ponta a ponta |

Execução: cenários de teste definidos antes de cada build; consultar [tarefas gerais](../fase.md) e [matriz de rastreabilidade](../matriz-de-rastreabilidade.md). A Jornada com UUIDs permanece no plano do projeto; a primeira task conecta Skip ao Supabase em contas autorizadas da Comboio. Fases 2–4 não têm SPECs de execução neste pacote.
