# SPEC-F1-04 — Chegada, permanência, saída e prova operacional

**Fase:** 1  
**Status:** planejada; pronta para execução técnica após SPEC-F1-01 a 03  
**Dono:** implementação Skip/Supabase; aceite operacional pela Comboio  
**Origem no escopo:** §§2, 5.4–5.5, 6–7, 9 e 12 do escopo definitivo  
**Degrau da solução:** nativo da plataforma — telas Skip, Postgres/RLS e RPC transacional de visita; sem hardware.

## Contexto e decisões fechadas

- **Estado atual:** chegada e saída têm controles manuais; motoristas novos podem chegar sem registro prévio.
- **Estado desejado:** operador registra presença real com ou sem previsão, conclui cadastro e encerra visita sem perder histórico.
- **Decisões fechadas:** reserva não ocupa vaga; só visita `inside` ocupa; fase 1 não aciona cancela nem processa pagamento.
- **Bloqueios:** uso real exige responsável pela exceção e procedimento de contingência; desenvolvimento usa massa fictícia.

## Resultado observável

Painel interno mostra veículos presentes e histórico; operador faz check-in/out de previstos e espontâneos; ocupação deriva apenas de visitas abertas.

## Limites e dependências

- **Inclui:** visita real, cadastro provisório, reconciliação, saída manual, ocupação, exceção e piloto de aceite.
- **Fora de escopo:** cancela, facial, pagamento online, checklist de triagem e disponibilidade garantida de vaga.
- **Entradas:** SPEC-F1-01/02/03, placa/tipo, pessoa conhecida ou provisória, ticket/referência e operador.
- **Saídas:** visita, eventos, ocupação e histórico auditável.
- **Atores:** só interno cria/encerra visita; empresa/motorista não leem visitas alheias.
- **Superfícies:** Skip interno; tabelas `visits`/`visit_events`; RLS/RPC; painel de ocupação.
- **Plano B/rollback:** formulário físico/ticket numerado em pane; posterior lançamento marcado; despublicar versão defeituosa sem apagar histórico.

## Dados e integrações

Operador registra chegada com ou sem previsão e encerra visita real. `visits(id, arrival_request_id nullable UNIQUE, driver_id nullable, provisional_driver_name nullable, provisional_contact nullable, vehicle_id nullable, plate_snapshot, vehicle_type, ticket_reference, checkin_idempotency_key UNIQUE, checkout_idempotency_key UNIQUE nullable, status[inside,closed], arrived_at, departed_at, checkin_by, checkout_by, release_state[not_recorded,recorded,exception], release_note, created_at, updated_at)`; `visit_events` imutável. Para motorista novo, cadastro provisório mínimo inclui nome informado, placa/tipo, contato quando disponível e referência de ticket; deve ser reconciliado ao cadastro completo sem perder `visit.id`. A identificação mínima exata para liberar saída é responsabilidade operacional validada no piloto; até decisão, saída com dados incompletos exige `exception`, motivo e usuário interno, nunca bloqueio físico automático.

| Origem/destino | Fonte de verdade | Campos/contrato | Permissão | Erro/recuperação |
| --- | --- | --- | --- | --- |
| Skip → `check_in_visit` | `visits`, `arrival_requests`, eventos | previsão opcional, placa/tipo, operador, idempotency key | interno ativo | Duplicata retorna visita existente; transação evita estado parcial. |
| Skip → `check_out_visit` | `visits`, eventos | ID, condição, motivo de exceção | interno ativo | Repetição idempotente; incompleto exige exceção. |
| Painel Skip → consulta segura | visitas `inside` | tipo, contagem, filtros | interno ativo | Erro não inventa vagas; mostrar indisponibilidade. |

## Fluxo e regras

- RPC `check_in_visit`: procura solicitação confirmada por ID/placa e permite associação; se não houver, cria visita `walk-in`. Só interno, horário do banco, uma única visita `inside` por placa. Solicitação associada muda a `arrived` na mesma transação. Requisição repetida com chave de idempotência retorna a visita existente.
- RPC `check_out_visit`: verifica `inside`, registra horário do banco, liberação manual e responsável; status vira `closed`; repetição idempotente não cria segunda saída. Sem acerto/identificação completos, exige exceção com motivo e autorização registrada. Não aciona cancela.
- Consulta de visitas via RLS para internos; visão/consulta segura de ocupação conta `inside`, não solicitações. Capacidade (500/100/100 no DOCX) é **referência não validada**; mostrar contagem real, não “vagas disponíveis” até confirmação. Sem vaga numerada.
- RPC `link_visit_driver` reconcilia provisório com motorista validado, mantendo evento e foto temporal. Correções exigem papel, motivo e trilha; não editar evento original. Todas as RPCs têm execução restrita, checagem de papel/vínculo no banco e testes de chamada direta por usuário indevido.

**Telas e contingência:** painel de previstos, presentes, saídos e pendências; check-in por placa/CPF; saída com condição/exceção. Em pane, ticket físico numerado e posterior lançamento marcado como `registro posterior`; falha de rede mostra indisponibilidade, nunca ocupação zero falsa.

| Cenário | Condição | Resultado esperado | Erro/recuperação |
| --- | --- | --- | --- |
| S401 | Previsão confirmada chega | Visita `inside`; pedido vira `arrived`; ocupação +1. | Veículo divergente exige revisão; não vincular automaticamente. |
| S402 | Veículo sem previsão chega | Visita espontânea com ticket, placa/tipo e dados mínimos. | Dados incompletos sinalizados para reconciliação. |
| S403 | Mesmo check-in é reenviado ou outra entrada compete | Uma visita aberta por placa; mesma chave retorna a existente. | Conflito explícito sem dupla ocupação. |
| S404 | Provisório é ligado a motorista validado | Mesmo `visit.id`, evento de vínculo e histórico preservados. | Vínculo errado é negado/corrigido com motivo. |
| S405 | Interno encerra visita | Saída única, horário e condição registrados; ocupação -1. | Sem identificação/acerto, exceção humana com motivo. |
| S406 | Interno consulta painel | Contagem igual a visitas `inside`, sem contar previsões. | Falha de rede mostra indisponibilidade, não zero. |
| S407 | Usuário externo invoca RPC/consulta | Acesso negado por RLS/função. | Registrar tentativa sem dado sensível. |
| S408 | Piloto com quatro jornadas | Cliente executa previsto, recorrente, triagem prevista e espontâneo. | Falha impeditiva mantém uso real bloqueado. |

## Instruções de execução para o implementador

1. **Ler:** escopo §§2, 5.4–5.5, 6–7, 9 e 12; SPEC-F1-01 a 03; tasks F1-018 a F1-024.
2. **Alterar somente:** visitas/eventos, RPC/RLS e telas/painel internos desta SPEC.
3. **Não alterar:** hardware, facial, pagamento, checklist de triagem ou preços.
4. **Executar:** RED → modelo → check-in → reconciliação → saída → painel → segurança/backup → piloto.
5. **Parar:** falta de responsável por exceção, vazamento, dupla entrada ou backup irrestaurável.
6. **Estado válido ao parar:** visitas existentes intactas e contingência manual disponível.

## Checklist de execução

- [ ] Visitas e eventos migrados, RLS/RPC testadas.
- [ ] Check-in previsto/espontâneo, reconciliação e saída exercitados.
- [ ] S401–S407 com erro, concorrência e acesso cruzado comprovados.
- [ ] Backup/restore, contingência e revisão de segurança concluídos.
- [ ] S408 registrado com aceite ou pendências, sem fingir go-live.

## Critérios de aceite

- [ ] **CA-F1-04-01:** previsto e espontâneo geram visita única e ocupação correta.
- [ ] **CA-F1-04-02:** provisório é reconciliado sem trocar ID/apagar histórico.
- [ ] **CA-F1-04-03:** saída manual é única; exceção exige ator e motivo.
- [ ] **CA-F1-04-04:** painel reflete apenas visitas abertas; externo não acessa dados internos.
- [ ] **CA-F1-04-05:** backup/restore, contingência e quatro jornadas têm prova; go-live só com aceite Comboio.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S401–S407 antes das RPC/telas | `supabase test db` + roteiro Skip | Falhas esperadas dos casos ainda não implementados. | Saída inicial. |
| GREEN | S401–S407 | Aplicar migrações/RPC e construir telas | CA-F1-04-01 a 04 passam. | SQL, testes e capturas. |
| REFACTOR/REGRESSÃO | S403/S405/S406/S407 e S408 | Testes de concorrência/RLS + piloto guiado | Sem dupla entrada/saída/vazamento; aceite documentado. | Relatório e ata. |

**Fixtures:** visitas fictícias previstas/espontâneas, placas A/B e contas por perfil. **Evidência exigida:** testes SQL, capturas de painel sem PII real, prova de restauração e ata de piloto.

## Handoff e operação

Demonstrar S401–S408 à Comboio. Operador acompanha presentes e exceções; administrador verifica backup e privilégios. Operação real requer HTTPS, persistência, restauração ensaiada, contingência e aceite. Em falha, suspender novas escritas, operar manualmente e reconciliar tickets antes de reabrir; nunca limpar visitas para voltar versão.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1-018 | Modelo de visitas | Implementação | F1-04 | CA-F1-04-01/04 | S403/S407 | SQL + testes | F1-012 | ☐ |
| F1-019 | Check-in | Implementação | F1-04 | CA-F1-04-01 | S401/S402/S403 | Roteiro + eventos | F1-018/011 | ☐ |
| F1-020 | Reconciliação | Implementação | F1-04 | CA-F1-04-02 | S404 | Teste + evento | F1-019 | ☐ |
| F1-021 | Check-out | Implementação | F1-04 | CA-F1-04-03 | S405 | Teste + captura | F1-019 | ☐ |
| F1-022 | Painel/ocupação | Implementação | F1-04 | CA-F1-04-04 | S406/S407 | Teste + captura | F1-019/021 | ☐ |
| F1-023 | Segurança/backup | Implementação | F1-04 | CA-F1-04-05 | S403/S407 | Relatório + restore | F1-017/022 | ☐ |
| F1-024 | Piloto/aceite | Operação Comboio | F1-04 | CA-F1-04-05 | S408 | Ata de aceite | F1-023 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
