# SPEC-F2-05 — Acerto à vista na saída, ocupação e aceite da fase 2

**Fase:** 2  
**Status:** liberada (10/11/2026)  
**Dono:** implementação Skip/Supabase; regras de preço por Jean; aceite pela Comboio  
**Origem no escopo:** §§5.5, 7, 11 (D6, D7) e 14; DOCX de 26/09 §§2, 3.1 e 7.3  
**Degrau da solução:** nativo da plataforma: tabela de parâmetros, função de cálculo no Postgres e RPC de quitação; sem TEF nem pagamento online.

## Contexto e decisões fechadas

- **Estado atual:** o check-out da fase 1 registra a saída e um "estado de liberação" manual, sem valor; o caixa calcula por conta própria.
- **Estado desejado:** na saída, o sistema mostra quanto cobrar (estadia ou triagem), o caixa ou a logística registra como foi pago, e a saída só é liberada quitada, "a faturar" ou com exceção justificada; a gerência vê o recebido do dia por forma de pagamento.
- **Decisões fechadas (DOCX):** tolerância de 2 h grátis; R$ 80 por estadia de 24 h, estadia cheia e sem rateio; triagem a R$ 100 por veículo, incluindo toda a permanência; capacidade de 500 caminhões, 100 carros e 100 motos, com alerta acima de 90% e sem bloqueio.
- **Regra provisória (D7):** passou de 2 h → `estadias = ceil(permanência / 24 h)`, que segue a tabela do DOCX (até 24 h = 1; até 48 h = 2). Se o Jean escolher a fórmula "−2 h", só muda um parâmetro (`desconto_tolerancia_na_contagem`). Todos os valores são parâmetros versionados, nunca fixos no código.

## Resultado observável

Um carro que ficou 26 h sai com R$ 160 calculado e a quitação registrada em PIX pelo caixa; um caminhão de triagem sai com R$ 100 recebidos pela logística, ou "a faturar" quando o pagador é a fábrica; a gerência vê o fechamento do dia.

## Limites e dependências

- **Inclui:** parâmetros de preço, cálculo, quitação à vista, "a faturar", isenção com aprovação, saída condicionada, ocupação com capacidade e fechamento do dia, segurança e piloto da fase.
- **Fora de escopo:** contrato a prazo, fatura do período, bloqueio de placa, bonificação por litros (fase 3); TEF e integração de PDV (não previstos).
- **Entradas:** visitas (fase 1), triagens (SPEC-F2-02), papéis caixa e logística.
- **Saídas:** `visit_charges`, `payments` e o relatório do dia.
- **Plano B/rollback:** se o cálculo falhar, o caixa registra o valor manualmente como exceção, com motivo; valores nunca são apagados (estorno é um novo lançamento).

## Dados e integrações

- `pricing_params(id, version, valid_from, tolerance_minutes=120, stay_price=80.00, triage_price=100.00, subtract_tolerance=false, capacity_truck=500, capacity_car=100, capacity_moto=100, alert_pct=90, created_by)`; só a gerência/admin cria uma versão nova.
- `visit_charges(id, visit_id UNIQUE, params_version, minutes, stays, kind[estadia,triagem,isento], amount, payer, status[aberto,quitado,a_faturar,isento,excecao], computed_at)`; recalculado no check-out pelo horário do banco.
- `payments(id, charge_id, method[dinheiro,pix,credito,debito], amount, received_by, received_at, reversal_of nullable, note)`; somente inserção.
- A triagem substitui a estadia da mesma visita (`kind=triagem`). Quando o pagador da fábrica não é o motorista, o status é `a_faturar` (a cobrança vem na fase 3).

## Fluxo e regras

1. O operador, o caixa ou a logística abre a saída da visita: o sistema calcula e mostra o tempo, o número de estadias e o valor.
2. O caixa (estadia) ou a logística (triagem) registra a forma de pagamento e o valor; com o total quitado → `quitado`.
3. Isenção: só gerência/admin, com motivo. Exceção (sistema fora do ar ou valor divergente): obrigatório informar motivo e ator.
4. O check-out só fecha a visita com o status `quitado`, `a_faturar`, `isento` ou `excecao`.
5. Painel: ocupação por tipo contra a capacidade, com alerta acima do percentual; fechamento do dia com entradas, saídas, valor calculado, recebido por forma e pendências.

| Cenário | Condição | Resultado esperado |
| --- | --- | --- |
| S901 | Permanência de 2h00 | R$ 0 (dentro da tolerância) |
| S902 | 2h01 | 1 estadia, R$ 80 |
| S903 | 24h00 / 24h01 | R$ 80 / R$ 160 |
| S904 | 48h01 | 3 estadias, R$ 240 |
| S905 | Triagem de 30 h, pagador motorista | R$ 100, sem estadia |
| S906 | Triagem, pagador fábrica | `a_faturar`; saída liberada |
| S907 | Caixa tenta mudar o preço ou isentar | Negado |
| S908 | Check-out com valor aberto e sem exceção | Bloqueado com mensagem |
| S909 | Ocupação de caminhões em 455/500 | Alerta visual de 91%, sem bloqueio |
| S910 | Mudança de preço com visita em curso | Usa a versão vigente no check-out, registrada na cobrança |
| S911 | Piloto da fase 2 | Jornadas: triagem completa com chamada e telão; triagem com não conformidade; estadia avulsa paga; estadia com exceção; layout aprovado |

## Perguntas ao cliente (o agente pergunta antes de implementar)

1. **Na F2-020 (obrigatória):** "Para fechar a regra de cobrança: um veículo que fica 25 horas paga 1 ou 2 estadias? O documento de 26/09 diz 'acima de 2 h até 24 h = 1 estadia', mas a fórmula só cobraria a 2ª depois de 26 h." Resposta "2" → `subtract_tolerance=false` (padrão); resposta "1" → `true`. Registre a resposta em `04-Decisoes.md`.
2. **Na F2-020:** "Os valores continuam R$ 80 por estadia, R$ 100 por triagem e 2 horas grátis? A moto paga o mesmo valor do caminhão?"
3. **Na F2-021:** "Quem pode dar isenção de estadia: só o Jean, ou a gerência também (Rayson, Selton)?"
4. **Na F2-025:** "O condomínio já está recebendo veículos? Qual dia podemos fazer o piloto com a operação real?"

## Instruções de execução para o implementador

1. **Ler:** escopo §§5.5, 7, 11 e 14; SPEC-F1-04 e SPEC-F2-02.
2. **Alterar somente:** parâmetros, cobrança, pagamentos, regra do check-out, painel e fechamento do dia.
3. **Não alterar:** a idempotência do check-out da fase 1, que deve ser preservada.
4. **Executar:** RED (S901–S910) → parâmetros → cálculo → quitação → check-out → painel → segurança → piloto.
5. **Modelo sugerido no Maestro:** Ultra para cálculo, RPC e RLS; Pro para telas.
6. **Parar:** se o Jean mudar a regra de estadia, ajustar o parâmetro e refazer S901–S904 antes de seguir.

## Checklist de execução

- [ ] Parâmetros versionados, com a versão inicial do DOCX.
- [ ] Função de cálculo com testes de borda S901–S905.
- [ ] Quitação, "a faturar", isenção e exceção com papéis corretos.
- [ ] Check-out condicionado sem quebrar a idempotência.
- [ ] Painel com capacidade e fechamento do dia.
- [ ] Revisão de segurança da fase 2 (RLS entre fábricas, Storage, papéis) e backup/restauração.
- [ ] Piloto S911 registrado em ata.

## Critérios de aceite

- [ ] **CA-F2-05-01:** o cálculo acerta todos os casos de borda com parâmetros versionados (S901–S905, S910).
- [ ] **CA-F2-05-02:** a quitação é registrada por papel, somente com inserção; isenção só pela gerência (S906, S907).
- [ ] **CA-F2-05-03:** a saída só fecha acertada ou com exceção justificada (S908).
- [ ] **CA-F2-05-04:** o painel mostra a ocupação contra a capacidade, com alerta, e o fechamento do dia (S909).
- [ ] **CA-F2-05-05:** segurança, backup e o piloto das cinco jornadas têm prova; o uso real da fase 2 só começa com o aceite da Comboio (S911).

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S901–S910 | `supabase test db` | Falhas esperadas | Saída inicial |
| GREEN | S901–S910 | Migrações, função, RPC e telas | CA-F2-05-01 a 04 passam | Testes + roteiro |
| REGRESSÃO | Fase 1 + SPECs F2-02 a 04 + S911 | Testes completos + piloto | Sem quebra; aceite em ata | Relatório e ata |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F2-020 | Parâmetros de preço e função de cálculo | Implementação | F2-05 | CA-F2-05-01 | S901–S905/S910 | Testes SQL | F2-006 | ☐ |
| F2-021 | Quitação à vista, "a faturar", isenção e exceção | Implementação | F2-05 | CA-F2-05-02 | S906/S907 | Testes | F2-020, F2-009 | ☐ |
| F2-022 | Check-out condicionado ao acerto | Implementação | F2-05 | CA-F2-05-03 | S908 | Testes | F2-021 | ☐ |
| F2-023 | Ocupação com capacidade e fechamento do dia | Implementação | F2-05 | CA-F2-05-04 | S909 | Conferência com o cliente | F2-022, F2-003 | ☐ |
| F2-024 | Segurança, backup e regressão da fase 2 | Implementação | F2-05 | CA-F2-05-05 | S603/S705/S803/S907 | Relatório + restore | F2-019, F2-023 | ☐ |
| F2-025 | Piloto e aceite da fase 2 | Operação Comboio | F2-05 | CA-F2-05-05 | S911 | Ata com pendências | F2-024, F2-005 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
