# SPEC-F3-04 — Bonificação por abastecimento, painel gerencial e aceite da fase 3

**Fase:** 3  
**Status:** liberada (10/11/2026); execução depois da F2-024  
**Dono:** implementação Skip/Supabase; regra de bonificação pelo Jean e pelo Rayson  
**Origem no escopo:** §§7, 11 (D12) e 14; DOCX de 26/09 §§2.1, 8 e 11  
**Degrau da solução:** nativo da plataforma: Postgres e RPC; os litros são digitados pelo caixa (a integração Promoflex/WebPosto é da fase 4).

## Contexto e decisões fechadas

- **Estado atual:** não há ligação entre o abastecimento e o pátio; a gerência não vê receita.
- **Estado desejado:** o caixa lança os litros; o abastecimento acima do mínimo gera **uma** estadia grátis, que é usada automaticamente na próxima saída; a gerência vê receita, uso por cliente e pendências.
- **Decisões fechadas (DOCX):** 200 L ou mais por abastecimento = 1 estadia grátis; **não acumulativa** (500 L = 1, sem sobra); 199 L não ganha; o mínimo é configurável; o crédito fica vinculado ao cliente/placa; a validade sugerida é de 30 dias (decisão do CEO, D12).

## Resultado observável

Um caminhão abastece 250 L, o caixa lança, e na saída o sistema desconta uma estadia; o painel gerencial mostra a receita do dia (à vista e a prazo), o uso por cliente e as faturas vencidas.

## Limites e dependências

- **Inclui:** lançamento de abastecimento, crédito, validade, uso na saída, painel gerencial, segurança e piloto da fase 3.
- **Fora de escopo:** leitura automática do PDV/Promoflex (fase 4); crédito no app de fidelidade (fase 4).
- **Entradas:** cobranças da SPEC-F2-05; contratos e faturas (SPECs F3-02 e F3-03).
- **Saídas:** `fuelings`, `stay_credits` e as consultas do painel.
- **Plano B/rollback:** se um lançamento estiver errado, o estorno é um novo registro; o crédito usado volta, com motivo.

## Dados e integrações

- `fuelings(id, vehicle_id, driver_id nullable, liters, receipt_ref, entered_by, entered_at, reversal_of)`.
- `stay_credits(id, vehicle_id, fueling_id UNIQUE, granted_at, expires_at, used_charge_id nullable, used_at)`.
- Parâmetros na `pricing_params` (SPEC-F2-05): `bonus_min_liters=200`, `bonus_validity_days=30`.
- Na saída: se existe crédito válido da placa e a cobrança é de estadia, o crédito mais antigo abate 1 estadia (registra `used_charge_id`).

## Perguntas ao cliente (o agente pergunta antes de implementar)

1. **Na F3-013:** "O crédito fica com a placa ou com o motorista?" O DOCX diz "cliente/placa"; sem resposta, use a placa.
2. **Na F3-014 (obrigatória):** "Qual a validade do crédito de estadia: 30 dias? O mínimo continua 200 litros?" O Rayson ia validar pela margem do diesel.
3. **Na F3-014:** "O crédito vale para a triagem também, ou só para a estadia?" Sem resposta, só para a estadia.
4. **Na F3-015:** "Quais 3 números a gerência quer ver primeiro ao abrir o painel?"
5. **Na F3-017:** "Qual dia podemos fazer o piloto da fase 3 com um cliente a prazo real?"

## Fluxo e regras

| Cenário | Condição | Resultado esperado |
| --- | --- | --- |
| S1301 | Abastecimento de 199 L | Sem crédito |
| S1302 | Abastecimento de 500 L | Exatamente 1 crédito |
| S1303 | Saída com crédito válido e 2 estadias | Cobra 1 estadia; crédito usado |
| S1304 | Crédito vencido | Não é usado |
| S1305 | Estorno do abastecimento com crédito ainda não usado | Crédito cancelado |
| S1306 | Painel gerencial do dia | Receita à vista por forma, a prazo do período, ocupação, uso por cliente e faturas vencidas |
| S1307 | Caixa ou logística abre o painel gerencial | Negado |
| S1308 | Piloto da fase 3 | Cliente a prazo com fatura fechada e baixa; motorista à vista com bonificação; placa bloqueada; painel conferido com o caixa do dia |

## Instruções de execução para o implementador

1. **Ler:** escopo §§7, 11 e 14; SPECs F2-05, F3-02 e F3-03.
2. **Alterar somente:** abastecimentos, créditos, uso na saída, painel, segurança e piloto.
3. **Modelo sugerido no Maestro:** Ultra para RPC e testes; Pro para o painel.
4. **Parar:** se a regra de bonificação mudar (ex.: virar acumulativa); trate como emenda.

## Critérios de aceite

- [ ] **CA-F3-04-01:** crédito correto, não acumulativo, com validade e estorno (S1301–S1305).
- [ ] **CA-F3-04-02:** painel gerencial com receita, uso e pendências, só para a gerência (S1306, S1307).
- [ ] **CA-F3-04-03:** segurança, backup e o piloto da fase 3 têm prova; uso real só com o aceite da Comboio (S1308).

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S1301–S1307 | `supabase test db` | Falhas esperadas | Saída inicial |
| GREEN | S1301–S1307 | Migrações, RPC e painel | CA-F3-04-01 e 02 passam | Testes e capturas |
| REGRESSÃO | Fases 1–3 + S1308 | Testes completos + piloto | Sem quebra; aceite em ata | Relatório e ata |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F3-013 | Lançamento de abastecimento pelo caixa | Implementação | F3-04 | CA-F3-04-01 | S1301/S1302/S1305 | Testes + captura | F3-001 | ☐ |
| F3-014 | Crédito de estadia e uso na saída | Implementação | F3-04 | CA-F3-04-01 | S1303/S1304 | Testes SQL | F3-013 | ☐ |
| F3-015 | Painel gerencial | Implementação | F3-04 | CA-F3-04-02 | S1306/S1307 | Captura + conferência | F3-012, F3-014 | ☐ |
| F3-016 | Segurança, backup e regressão da fase 3 | Implementação | F3-04 | CA-F3-04-03 | S1004/S1107/S1207/S1307 | Relatório + restore | F3-015, F3-004, F3-008 | ☐ |
| F3-017 | Piloto e aceite da fase 3 | Operação Comboio | F3-04 | CA-F3-04-03 | S1308 | Ata com pendências | F3-016, F2-025 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
