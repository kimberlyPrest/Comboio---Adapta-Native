# SPEC-F3-03 — Faturas do período e pendências

**Fase:** 3  
**Status:** liberada (10/11/2026); execução depois da SPEC-F3-02  
**Dono:** implementação Skip/Supabase; conferência pelo financeiro da Comboio  
**Origem no escopo:** §§7 e 14; DOCX de 26/09 §§3.2 e 8  
**Degrau da solução:** nativo da plataforma: Postgres, RPC, `pg_cron` no Supabase para o fechamento e página de fatura para impressão/PDF.

## Contexto e decisões fechadas

- **Estado atual:** as cobranças `a_faturar` se acumulam sem fatura.
- **Estado desejado:** no vencimento, a fatura do contrato fecha sozinha com todas as estadias e triagens do período; o financeiro baixa o pagamento e vê o que está vencido.
- **Decisões fechadas (DOCX):** fechamento automático no vencimento; **envio manual** (o e-mail automático é da fase 4); alerta de faturas vencidas e em aberto, com dias de atraso. Isso também ataca a dor original do DMO (faturamento e contas a receber).

## Resultado observável

No dia do vencimento, a fatura do contrato aparece pronta na tela, com a lista de visitas; o financeiro imprime, envia e registra o envio; ao receber, dá baixa; o painel mostra o que está vencido.

## Limites e dependências

- **Inclui:** fechamento por contrato e período, itens, PDF, registro de envio, baixa e atraso.
- **Fora de escopo:** boleto, PIX cobrança, NF-e, conciliação bancária, e-mail automático (fase 4), juros e multa.
- **Entradas:** `visit_charges` `a_faturar` com `contract_id` (SPEC-F3-02).
- **Saídas:** `invoices`, `invoice_items`, `invoice_payments` e `invoice_sends`.
- **Plano B/rollback:** se o job falhar, o financeiro roda "fechar agora" para o contrato; o fechamento é idempotente por contrato e período.

## Dados e integrações

- `invoices(id, contract_id, period_start, period_end, due_date, total, status[aberta,enviada,paga,parcial,vencida,cancelada], closed_at, closed_by[system|user])`, com `UNIQUE(contract_id, period_start)`.
- `invoice_items(invoice_id, charge_id UNIQUE)`: cada cobrança entra em uma única fatura.
- `invoice_sends(invoice_id, sent_by, sent_at, emails[], note)`, `invoice_payments(invoice_id, amount, method, received_by, received_at, reversal_of)`.
- O job `pg_cron` diário chama a RPC `close_due_invoices()`, que lê segredo do Vault, e não de código, se precisar de algum.

## Perguntas ao cliente (o agente pergunta antes de implementar)

1. **Na F3-010:** "O que precisa aparecer na fatura além da lista de visitas (placa, entrada, saída, valor)? CNPJ da Comboio, dados bancários, observação?"
2. **Na F3-011:** "O cliente pode pagar uma fatura em partes?" Sem resposta, permita baixa parcial com status `parcial`.
3. **Na F3-012:** "Quem do financeiro vai dar baixa nas faturas?"

## Fluxo e regras

| Cenário | Condição | Resultado esperado |
| --- | --- | --- |
| S1201 | Vencimento chega com 12 cobranças `a_faturar` | Uma fatura com 12 itens e o total correto |
| S1202 | Job roda duas vezes no mesmo dia | Uma única fatura |
| S1203 | Cobrança criada depois do fechamento | Entra na fatura do próximo período |
| S1204 | Financeiro registra o envio | `enviada`, com ator, data e e-mails |
| S1205 | Pagamento parcial e depois o restante | `parcial` → `paga` |
| S1206 | Vencimento + 1 dia sem pagamento | `vencida`, com dias de atraso no painel |
| S1207 | Fábrica ou caixa tenta ver fatura de outro cliente | Negado |

## Instruções de execução para o implementador

1. **Ler:** escopo §§7 e 14; SPEC-F3-02.
2. **Alterar somente:** faturas, job, envio, baixa e pendências.
3. **Modelo sugerido no Maestro:** Ultra para o job, a RPC e a idempotência; Pro para telas.
4. **Parar:** se o cliente pedir emissão fiscal ou boleto; isso é outro escopo.

## Critérios de aceite

- [ ] **CA-F3-03-01:** fechamento automático e idempotente no vencimento (S1201–S1203).
- [ ] **CA-F3-03-02:** fatura imprimível com envio registrado (S1204).
- [ ] **CA-F3-03-03:** baixa total e parcial, atraso calculado e isolamento entre clientes (S1205–S1207).

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S1201–S1207 | `supabase test db` | Falhas esperadas | Saída inicial |
| GREEN | S1201–S1207 | Migrações, job, RPC e telas | CA-F3-03-01 a 03 passam | Testes, `cron.job` e fatura exemplo |
| REGRESSÃO | S1104 | Testes da SPEC-F3-02 | Cobrança a prazo intacta | Saída |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F3-009 | Fechamento automático de faturas (job) | Implementação | F3-03 | CA-F3-03-01 | S1201–S1203 | Testes + `cron.job` | F3-007 | ☐ |
| F3-010 | Fatura imprimível e registro de envio | Implementação | F3-03 | CA-F3-03-02 | S1204 | Fatura exemplo (dados fictícios) | F3-009 | ☐ |
| F3-011 | Baixa de pagamento | Implementação | F3-03 | CA-F3-03-03 | S1205/S1207 | Testes + captura | F3-009 | ☐ |
| F3-012 | Pendências e atraso | Implementação | F3-03 | CA-F3-03-03 | S1206 | Captura | F3-011 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
