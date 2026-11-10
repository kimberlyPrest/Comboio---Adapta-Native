# SPEC-F3-02 — Contratos a prazo, preço negociado e bloqueio de placa

**Fase:** 3  
**Status:** liberada (10/11/2026); execução depois da F2-024  
**Dono:** implementação Skip/Supabase; regras de crédito pelo Jean (CEO)  
**Origem no escopo:** §§7 e 14; DOCX de 26/09 §§3.2, 4.1, 5 e 8  
**Degrau da solução:** nativo da plataforma: Postgres, RLS e RPC transacional.

## Contexto e decisões fechadas

- **Estado atual:** na fase 2, quem não paga à vista fica `a_faturar` sem contrato por trás.
- **Estado desejado:** empresa ou frota com contrato ativo usa o pátio sem passar pelo caixa, ao preço negociado; placa com pendência aparece bloqueada para o caixa e para o operador.
- **Decisões fechadas (DOCX):** contrato a prazo **sem garantia real registrada fica bloqueado**; exceção de crédito só pelo CEO; o comercial cria o contrato depois da análise de crédito; o caixa vê o bloqueio, mas não desbloqueia; o bloqueio é no sistema, não na cancela (cancela é fase 5).

## Resultado observável

O comercial cria um contrato com garantia, o Jean aprova, e as placas autorizadas saem do pátio com a cobrança "a faturar" no preço do contrato; uma placa bloqueada mostra o aviso no check-in e no check-out.

## Limites e dependências

- **Inclui:** contratos, garantias, preços negociados, placas e motoristas autorizados, aprovação do CEO, cobrança a prazo na saída, bloqueio e desbloqueio.
- **Fora de escopo:** análise de crédito automatizada; juros e multa (só se o cliente pedir); bloqueio físico.
- **Entradas:** empresas, veículos e motoristas; `visit_charges` da SPEC-F2-05; papéis da SPEC-F3-01.
- **Saídas:** `contracts`, `contract_vehicles`, `contract_drivers` e `plate_blocks`.
- **Plano B/rollback:** contrato suspenso faz a placa voltar a pagar à vista, sem apagar histórico.

## Dados e integrações

- `contracts(id, company_id, status[rascunho,aguardando_ceo,ativo,suspenso,encerrado], guarantee_type, guarantee_description, guarantee_doc_path, stay_price, triage_price, billing_cycle, due_day, billing_emails[], created_by, approved_by, approved_at, exception_reason)`.
- `contract_vehicles(contract_id, vehicle_id)` e `contract_drivers(contract_id, driver_id)`.
- `plate_blocks(id, vehicle_id, reason, blocked_by, blocked_at, unblocked_by, unblocked_at)`; somente inserção e atualização do desbloqueio.
- Na saída (RPC da SPEC-F2-05): placa em contrato `ativo` → `visit_charges.amount` pelo preço do contrato, `status=a_faturar`, `contract_id` preenchido.

## Perguntas ao cliente (o agente pergunta antes de implementar)

1. **Na F3-005:** "Que tipos de garantia real vocês aceitam (cheque, imóvel, fiança, outro)? Precisa guardar foto ou cópia do documento?"
2. **Na F3-005:** "O fechamento da fatura é mensal? Todos no mesmo dia ou cada contrato tem o seu dia de vencimento?"
3. **Na F3-006:** "A aprovação do CEO é sempre do Jean? Mais alguém pode aprovar quando ele não estiver?"
4. **Na F3-008:** "Bloqueio por inadimplência: a partir de quantos dias de atraso o comercial deve ser avisado?" O bloqueio continua manual; o sistema só sugere.

## Fluxo e regras

| Cenário | Condição | Resultado esperado |
| --- | --- | --- |
| S1101 | Comercial cria contrato sem garantia | Não sai de `rascunho` |
| S1102 | Contrato com garantia enviado | `aguardando_ceo`; só o CEO aprova |
| S1103 | CEO aprova exceção sem garantia | `ativo` com `exception_reason` e ator |
| S1104 | Placa autorizada sai do pátio | Cobrança `a_faturar` no preço do contrato; sem passar pelo caixa |
| S1105 | Placa não autorizada da mesma empresa | Cobrança à vista normal |
| S1106 | Placa bloqueada chega | Aviso vermelho no check-in e no check-out com o motivo; a entrada é registrada (a decisão é humana) |
| S1107 | Caixa tenta desbloquear | Negado |
| S1108 | Contrato suspenso | Placas voltam ao à vista; histórico intacto |

## Instruções de execução para o implementador

1. **Ler:** escopo §§7 e 14; SPEC-F2-05; SPEC-F3-01.
2. **Alterar somente:** contratos, garantias, vínculos, bloqueios e o ramo a prazo do cálculo da saída.
3. **Não alterar:** o cálculo à vista da fase 2.
4. **Modelo sugerido no Maestro:** Ultra para RPC e RLS; Pro para telas.
5. **Parar:** se o cliente pedir juros ou multa; registre e trate como emenda.

## Critérios de aceite

- [ ] **CA-F3-02-01:** contrato só fica ativo com garantia ou exceção do CEO (S1101–S1103).
- [ ] **CA-F3-02-02:** placa autorizada gera `a_faturar` no preço do contrato; a não autorizada paga à vista (S1104, S1105, S1108).
- [ ] **CA-F3-02-03:** bloqueio visível no check-in e no check-out; só comercial e gerência desbloqueiam (S1106, S1107).

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S1101–S1108 | `supabase test db` | Falhas esperadas | Saída inicial |
| GREEN | S1101–S1108 | Migrações, RPC e telas | CA-F3-02-01 a 03 passam | Testes e capturas |
| REGRESSÃO | S901–S910 | Testes da SPEC-F2-05 | À vista intacto | Saída |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F3-005 | Contratos, garantias e preços negociados | Implementação | F3-02 | CA-F3-02-01 | S1101/S1102 | Testes + captura | F3-001 | ☐ |
| F3-006 | Aprovação e exceção do CEO | Implementação | F3-02 | CA-F3-02-01 | S1102/S1103 | Testes + eventos | F3-005 | ☐ |
| F3-007 | Cobrança a prazo na saída | Implementação | F3-02 | CA-F3-02-02 | S1104/S1105/S1108 | Testes SQL | F3-006 | ☐ |
| F3-008 | Bloqueio e desbloqueio de placa | Implementação | F3-02 | CA-F3-02-03 | S1106/S1107 | Testes + captura | F3-001 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
