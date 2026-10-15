# Prova de negação roda em transação separada da fixture

- **Data**: 2026-10-15T19:05:00-03:00 · **Task**: F1-006 · **Capturado por**: Severino (agente)

## O que aconteceu

Um batch SQL continha a fixture de motorista + a prova proposital de CPF duplicado. A prova falhou (como esperado, 23505) e reverteu o BATCH INTEIRO — a fixture sumiu junto. Diagnóstico confuso no início: `on conflict do nothing` mascarou a ausência. Refeito em chamadas separadas: fixture primeiro (commit), prova depois (falha isolada). O mesmo vale para o teste humano: as 3 queries do roteiro foram coladas juntas e o ERRO proposital da query 2 abortou a query 3 — relato de "erro no teste 3" sem defeito nenhum.

## Por que importa

- Prova de negação e fixture compartilham transação → a negação revertida apaga a fixture e gera falso negativo.
- Roteiro humano com erro proposital no meio: tudo que vem depois não roda; o testador acha que a ÚLTIMA query falhou.

## Aplicação futura

- Fixture e prova de negação sempre em chamadas SQL separadas.
- Roteiro de teste humano: erro proposital SEMPRE na última query, ou instrução explícita de rodar query por query.
