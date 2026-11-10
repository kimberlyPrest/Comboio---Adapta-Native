# SPEC-F2-02 — Perfis operacionais, fábricas e serviço de triagem

**Fase:** 2  
**Status:** liberada (10/11/2026)  
**Dono:** implementação Skip/Supabase; validação pela logística da Comboio  
**Origem no escopo:** §§3, 5.6, 7 e 14; DOCX de 26/09 §§5 e 7.1–7.2  
**Degrau da solução:** nativo da plataforma: tabelas Postgres, RLS e RPC transacional, como na fase 1.

## Contexto e decisões fechadas

- **Estado atual:** a visita (`visits`) registra entrada e saída de qualquer veículo; o sistema tem os perfis admin, operador, empresa e motorista; "empresa" não distingue fábrica, transportadora e frota.
- **Estado desejado:** um veículo que vem para carga ou descarga ganha um **serviço de triagem** ligado à sua visita, com fábrica/unidade de destino, dados da carga e estados controlados pela logística.
- **Decisões fechadas (DOCX 26/09, confirmar na liberação):** a logística executa o cadastro operacional e a triagem; a fábrica chama pelo portal (SPEC-F2-04); a não conformidade não bloqueia; o pagador da triagem é definido por contratante.

## Resultado observável

A logística abre a triagem de um veículo presente, preenche o cadastro operacional e move os estados até "concluído"; cada fábrica só enxerga as triagens das suas unidades.

## Limites e dependências

- **Inclui:** papéis `logistica`, `caixa` e `fabrica`; tipo de relação da empresa; unidades; usuários da fábrica vinculados a unidades; serviço de triagem; estados e eventos.
- **Fora de escopo:** checklist (SPEC-F2-03), portal e chamada (SPEC-F2-04), valores (SPEC-F2-05), OCR de NF (fase 4), perfil comercial e contratos (fase 3).
- **Entradas:** visitas `inside` da fase 1; empresas aprovadas.
- **Saídas:** `triage_services`, `triage_events`, unidades e papéis.
- **Plano B/rollback:** se a triagem falhar, a visita continua válida e a logística volta à planilha temporária; nunca apagar eventos.

## Dados e integrações

- `companies` (fase 1) ganha `relation_types[]` ∈ {`fabrica`, `transportadora`, `frota`, `estacionamento`} e `triage_payer` ∈ {`motorista`, `transportadora`, `fabrica`} para fábricas.
- `company_units(id, company_id, name, address, active)`.
- `unit_members(user_id, unit_id, role[viewer,caller])`: usuário de portal da fábrica, convidado por admin interno.
- `triage_services(id, visit_id UNIQUE, unit_id, carrier_company_id nullable, operation[carga,descarga], order_number, product, quantity, invoice_number, sender, recipient, document_photo_path nullable, status, payer, created_by, created_at, updated_at)`. Na carga, pedido, produto e quantidade são obrigatórios; na descarga, NF, remetente e destinatário.
- `triage_events` imutável (de, para, ator, motivo, horário do banco).
- Papéis novos no `profiles.role` da fase 1 (hoje `admin | operator | company | driver`): `logistics`, `cashier`, `factory` e `display` (telão, SPEC-F2-04). Só o admin atribui. Atualize `UserRole` em `src/hooks/use-auth.tsx`.
- **Linha de base do banco:** o repo de código só tem a migration de seed (`20260929182622_seed_initial_user.sql`); o schema da fase 1 foi aplicado direto no Supabase. Antes de qualquer migration nova, exporte o schema atual (`supabase db dump --schema-only`) para `supabase/migrations/<timestamp>_baseline_fase1.sql`, sem dados, e confira que recriar um banco vazio a partir dela reproduz tabelas, RLS e RPC.

## Fluxo e regras

Estados: `chegou` → `cadastrado` → `em_triagem` → `pronto` → `chamado_carga` | `chamado_descarga` → `em_operacao` → `concluido`. Também existe `cancelado`, com motivo. A saída do pátio é registrada pelo check-out da visita.

| Transição | Quem | Regra |
| --- | --- | --- |
| abrir (`chegou`) | logística | Só para visita `inside` e sem triagem aberta |
| → `cadastrado` | logística | Campos obrigatórios da operação preenchidos |
| → `em_triagem` / `pronto` | logística | Ver SPEC-F2-03 (checklist) |
| → `chamado_*` | fábrica (portal) ou logística em nome dela, com motivo | Ver SPEC-F2-04 |
| → `em_operacao` → `concluido` | logística | Ordem da fila; horário do banco |
| → `cancelado` | logística/admin | Motivo obrigatório |

Todas as transições passam por uma RPC que verifica papel, vínculo e estado de origem e grava o evento na mesma transação. A RPC é idempotente por chave.

| Cenário | Condição | Resultado esperado | Erro/recuperação |
| --- | --- | --- | --- |
| S601 | Logística abre triagem de visita presente | Triagem `chegou` ligada à visita | Visita fechada ou triagem duplicada é negada |
| S602 | Cadastro de carga sem pedido | Bloqueado com mensagem do campo | — |
| S603 | Usuário da fábrica A consulta triagem da fábrica B | Negado por RLS | Tentativa registrada sem dado sensível |
| S604 | Caixa tenta mover estado da triagem | Negado | — |
| S605 | Transição fora de ordem (`cadastrado` → `concluido`) | Negada pela RPC | Estado intacto |
| S606 | Mesmo pedido de transição reenviado | Um único evento | — |

## Perguntas ao cliente (o agente pergunta antes de implementar)

1. **Na F2-007:** "Quais fábricas e unidades vão usar a triagem primeiro? Para cada uma: nome, unidade e quem da fábrica vai acessar o portal (nome e e-mail)." Use dados fictícios no teste e cadastre os reais só no piloto.
2. **Na F2-007:** "Em cada fábrica, quem paga a triagem de R$ 100: o motorista, a transportadora ou a própria fábrica?" Isso preenche `triage_payer`. Sem resposta, use `motorista` e registre em `04-Decisoes.md`.
3. **Na F2-008:** "No cadastro operacional, além de pedido, produto e quantidade (carga) e NF, remetente e destinatário (descarga), há outro campo que a logística sempre anota?"

## Instruções de execução para o implementador

1. **Ler:** escopo §§3, 5.6 e 14; SPEC-F1-01 (perfis/RLS) e SPEC-F1-04 (visitas).
2. **Alterar somente:** papéis, empresas/unidades, triagem e as telas da logística.
3. **Não alterar:** regras de visita da fase 1, além de ler `visits`.
4. **Executar:** linha de base do schema → RED (S601–S606) → migração → RLS → RPC → telas → GREEN.
5. **Modelo sugerido no Maestro:** Ultra para migração, RLS e RPC; Pro para telas.
6. **Parar:** se faltar a definição de campo obrigatório de uma fábrica real; registre em decisões.

## Checklist de execução

- [ ] Papéis, unidades e triagem migrados com RLS.
- [ ] RPC de transição com testes de papel, ordem e idempotência.
- [ ] Tela da logística: fila de trabalho (abertas por estado) e formulário por operação.
- [ ] S601–S606 comprovados.

## Critérios de aceite

- [ ] **CA-F2-02-01:** admin cria usuários de logística, caixa e fábrica, e cada um vê só o que o papel permite (S603, S604).
- [ ] **CA-F2-02-02:** fábrica com unidades; usuário de portal vinculado a unidade (S603).
- [ ] **CA-F2-02-03:** triagem aberta sobre visita presente, com campos obrigatórios por operação (S601, S602).
- [ ] **CA-F2-02-04:** estados só avançam pela RPC, na ordem, com evento e idempotência (S605, S606).

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S601–S606 | `supabase test db` | Falhas esperadas | Saída inicial |
| GREEN | S601–S606 | Migrações, RLS, RPC e telas | CA-F2-02-01 a 04 passam | SQL e testes |
| REGRESSÃO | S401–S407 da fase 1 | Testes da fase 1 | Visitas intactas | Saída dos testes |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F2-006 | Linha de base do schema; papéis logística, caixa, fábrica e telão + RLS | Implementação | F2-02 | CA-F2-02-01 | S603/S604 | Baseline + testes RLS | F2-001 | ☐ |
| F2-007 | Tipos de empresa, unidades e membros da fábrica | Implementação | F2-02 | CA-F2-02-02 | S603 | SQL + testes | F2-006 | ☐ |
| F2-008 | Modelo e abertura da triagem com cadastro operacional | Implementação | F2-02 | CA-F2-02-03 | S601/S602 | Testes | F2-007 | ☐ |
| F2-009 | RPC de estados da triagem e eventos | Implementação | F2-02 | CA-F2-02-04 | S605/S606 | Testes SQL | F2-008 | ☐ |
| F2-010 | Tela de trabalho da logística | Implementação | F2-02 | CA-F2-02-03/04 | S601/S605 | Roteiro | F2-009, F2-004 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
