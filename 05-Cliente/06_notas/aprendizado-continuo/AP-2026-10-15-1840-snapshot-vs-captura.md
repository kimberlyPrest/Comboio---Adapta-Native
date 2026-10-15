# Snapshot de acessibilidade é a prova; captura é ilustração

- **Data**: 2026-10-15T18:40:00-03:00 · **Task**: F1-005 · **Capturado por**: Severino (agente)

## O que aconteceu

Na prova mobile da home admin, a captura de tela (viewport 390px) mostrava apenas 1 interno na lista, sugerindo dado faltando. O snapshot de acessibilidade confirmou os 3 internos presentes (botões Desativar desabilitado/Desativar/Ativar) — a captura só cortava o conteúdo fora da viewport. A conclusão de prova foi tirada do snapshot, não da imagem.

## Por que importa

- Captura de tela é evidência VISUAL (layout, responsividade, cores) — não é prova de conteúdo completo.
- Snapshot de acessibilidade lista toda a árvore renderizada, sem corte de viewport.
- Declarar "dado faltando" com base em captura gera falso negativo e retrabalho.

## Aplicação futura

Em toda prova de UI: conteúdo e comportamento se provam pelo snapshot (ou `get text`); a captura acompanha como anexo visual do que foi renderizado. Se a captura e o snapshot divergirem, o snapshot manda.
