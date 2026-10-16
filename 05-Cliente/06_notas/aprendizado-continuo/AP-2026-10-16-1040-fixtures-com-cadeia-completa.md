# Fixture de prova precisa da cadeia completa de vínculos

- **Data**: 2026-10-16T10:40:00-03:00 · **Task**: F1-007/F1-008 · **Capturado por**: Severino (agente)

## O que aconteceu

Na F1-008, a rejeição pela UI "não passava" duas vezes. A RPC estava CERTA em recusar: as fixtures usadas (Empresa A/B legadas da F1-002 e a probe da Epsilon) não tinham a cadeia completa — `company_users` sem linha em `profiles` (ou sem vínculo). A guarda de rollback da RPC ("perfil do responsável não encontrado" / "pedido sem responsável pendente") era o comportamento de segurança, mas parecia um bug de UI. No teste humano, o champion relatou "Empresa A e B não rejeita" — de novo a guarda correta recusando fixture incompleta.

## Por que importa

- Fixture de prova de um fluxo que envolve múltiplas entidades precisa reproduzir a CADEIA COMPLETA (auth.users → profiles → company_users → companies), senão as guardas de integridade falham a prova e parecem defeito.
- O roteiro de teste humano herdado de fixtures antigas engana o champion — o teste dele deve usar fixture criada pelo fluxo real (F1-007) ou fixture SQL completa.

## Aplicação futura

- Antes de provar um fluxo transacional por UI, conferir a cadeia completa da fixture (todas as FKs com linha existente).
- Fixtures legadas de fases anteriores que não passaram pelo fluxo novo: marcar `inactive` com motivo (limpeza auditável), não deixar na fila confundindo o operador.
