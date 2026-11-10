# SPEC-F1-03 — Previsões de chegada e autosserviço

**Fase:** 1  
**Status:** planejada; pronta para execução técnica após SPEC-F1-01/02  
**Dono:** implementação Skip/Supabase; aceite de negócio pela operação Comboio  
**Origem no escopo:** §§1, 5.3, 6, 7 e 12 do escopo definitivo  
**Degrau da solução:** nativo da plataforma — formulários Skip, Postgres/RLS e RPC transacional para estados/agenda.

## Contexto e decisões fechadas

- **Estado atual:** a operação descrita não usa agendamento; caminhões chegam sem aviso. A previsão é capacidade nova e opcional.
- **Estado desejado:** motorista registrado e empresa criam intenção de chegada, interno decide e todos veem status pertinente.
- **Decisões fechadas para o piloto:** previsão sem vaga garantida; toda solicitação fica pendente de confirmação interna; chegada espontânea continua válida.
- **Bloqueios:** mudança da política de confirmação, cota ou janela exige emenda antes de execução; parâmetros de 60 dias/4h abaixo são defaults do piloto a validar com a Comboio antes do uso real.

## Resultado observável

Motorista ativo envia previsão no celular, empresa solicita para vínculo válido, interno confirma/recusa e a agenda reflete o status sem alterar a ocupação física.

## Limites e dependências

- **Inclui:** criação por motorista, empresa **e operador interno** em nome de terceiro, consulta, edição, cancelamento, decisão interna, no-show, histórico e conflito de janela.
- **Fora de escopo:** promessa de vaga, reserva de doca, cobrança, mensagens automáticas e check-in físico.
- **Entradas:** identidade/vínculos ativos de F1-01/02; data, janela, veículo, motorista, finalidade, unidade quando triagem.
- **Saídas:** previsão e eventos consultáveis; agenda interna e status nos portais.
- **Atores:** motorista só seu vínculo; empresa só sua organização; interno cria/revisa em nome de terceiro com motivo/autor registrados.
- **Superfícies:** Skip nos três portais, `arrival_requests`, `request_events`, RLS, RPC.
- **Plano B/rollback:** suspender novas solicitações e manter leitura/histórico; chegada espontânea permanece na F1-04.

### Política do piloto para a fase 1

“Reserva” é previsão de chegada, **não garantia de vaga**. Estacionamento solicitado pelo motorista/empresa fica `requested` até confirmação interna; triagem sempre exige confirmação interna, mesmo se originada pela empresa. Sem capacidade por janela nesta fase: a agenda informa volume, mas não bloqueia automaticamente. O motorista pode solicitar uma janela futura de até 60 dias e não pode sobrepor duas solicitações ativas para a mesma placa na mesma janela. Uma previsão confirmada modificada em data, veículo, finalidade ou destino volta a `requested`. Essas regras são parâmetros do piloto, não afirmação de regra histórica do cliente.

## Dados e integrações

`arrival_requests(id, source[driver,company,internal], kind[parking,triage], company_id nullable, driver_id, vehicle_id, destination_unit nullable, starts_at, ends_at, status[draft,requested,confirmed,rejected,cancelled,no_show,arrived], notes, decision_by, decision_at, decision_reason, created_at, updated_at, version)`; `request_events(id, request_id, actor_id, from_status, to_status, reason, created_at)`. Janela com início < fim, duração máxima configurada inicialmente em 4h. Data/hora persistida em UTC; tela no fuso da operação. Concorrência otimista por `version`; tentativa com versão antiga retorna conflito e pede atualização da tela.

| Origem/destino | Fonte de verdade | Campos/contrato | Permissão | Falha/recuperação |
| --- | --- | --- | --- | --- |
| Portais Skip → RPC `create_arrival_request` | `arrival_requests` | motorista, veículo, finalidade, janela, empresa/unidade | vínculo validado ou operador | Conflito/duplicidade retorna erro; nenhuma ocupação criada. |
| Skip → RPC `change_arrival_request` | pedido + evento | ID, versão, mudança/cancelamento | titular/vínculo ou interno | Versão antiga retorna conflito e recarrega. |
| Interno → RPC `review_arrival_request` | pedido + evento | decisão, ator, motivo | interno ativo | Estado inválido/dupla decisão negado atomicamente. |

## Fluxo e regras

- RPC `create_arrival_request`: motorista ativo escolhe veículo vinculado, finalidade, data/janela e, para triagem, empresa/unidade autorizada; usuário de empresa ativa pode solicitar para motorista/veículo vinculados; **operador interno** pode registrar em nome de terceiro com `source=internal`, ator e motivo/observação. A função verifica `auth.uid()`, papel e vínculo no banco; não aceita veículo de terceiro para usuário externo.
- RPC `change_arrival_request`: edição/cancelamento por autor/vínculo permitido, antes do check-in. Alteração material desfaz confirmação. Cancelamento preserva histórico.
- Consulta da agenda via visão segura/consulta com RLS, filtros por data, status, tipo, empresa e placa, paginação e ordenação por janela. RPC `review_arrival_request`: somente interno confirma/recusa/marca `no_show`; recusa exige motivo; `no_show` somente após fim da janela e ausência de visita.
- RPCs devem ter `GRANT EXECUTE` restrito a `authenticated`, validar ator/vínculo internamente e gravar mudança, evento e auditoria na mesma transação. Funções `SECURITY DEFINER`, se inevitáveis, usam `search_path` fixo e revisão de privilégios. Solicitação cancelada/recusada não é reativada.

**Telas:** motorista vê nova previsão, lista e detalhe/status; empresa vê solicitações dos vínculos; interno vê agenda diária/semanal e ações de decisão. Mostrar conflito, janela inválida, inatividade e falha de rede. Sem notificação automática externa.

| Cenário | Condição | Resultado esperado | Erro/recuperação |
| --- | --- | --- | --- |
| S301 | Motorista ativo com veículo vinculado solicita janela válida | Pedido `requested`, visível ao titular/interno; ocupação inalterada. | Janela inválida/sobreposta rejeitada. |
| S302 | Empresa ativa solicita para vínculo próprio | Pedido criado só na empresa/unidade autorizada. | Empresa A não cria/lê B. |
| S303 | Operador interno registra previsão em nome de terceiro | `source=internal`, ator e motivo/observação auditados. | Operador inativo/externo negado. |
| S304 | Interno confirma/recusa/no-show | Estado e evento mudam juntos; recusa tem motivo. | No-show antes do fim da janela negado. |
| S305 | Pedido confirmado é editado/cancelado | Mudança material volta a `requested`; cancelamento preserva histórico. | Pedido `arrived` não pode ser cancelado. |
| S306 | Duas ações simultâneas na mesma versão | Apenas uma vence; outra recebe conflito explícito. | Recarregar e reaplicar se ainda permitido. |
| S307 | Motorista, empresa e interno navegam telas | Cada perfil vê seu recorte; status claro, sem promessa de vaga. | Erro de rede não inventa confirmação. |

## Instruções de execução para o implementador

1. **Ler:** escopo §§1, 5.3, 6 e 7; SPEC-F1-01/02; tasks F1-012 a F1-017.
2. **Alterar somente:** pedido, eventos, RPC/RLS, agenda e telas de previsão.
3. **Não alterar:** ocupação, visita real, cobrança, doca ou notificações externas.
4. **Executar:** testes RED → modelo/estados → autosserviço motorista/empresa/interno → decisão → concorrência → regressão.
5. **Parar:** solicitação autoconfirmada, promessa de vaga ou regra comercial nova não aprovada.
6. **Estado válido ao parar:** pedidos anteriores permanecem legíveis; chegada espontânea continua possível.

## Checklist de execução

- [ ] Modelo, eventos e RLS aplicados.
- [ ] Criação pelos três atores, decisão interna e idempotência testadas.
- [ ] S301–S307, inclusive erro e acesso cruzado, exercitados.
- [ ] Evidências de status e histórico anexadas.
- [ ] Default de janela/prazo marcado para validação antes do uso real.

## Critérios de aceite

- [ ] **CA-F1-03-01:** motorista ativo solicita/consulta previsão própria e não altera ocupação.
- [ ] **CA-F1-03-02:** empresa e operador interno criam solicitações somente dentro de seus poderes, com origem/ator registrados.
- [ ] **CA-F1-03-03:** confirmação, recusa e no-show seguem estados permitidos e geram evento atômico.
- [ ] **CA-F1-03-04:** edição, cancelamento, conflito de janela e concorrência não perdem histórico.
- [ ] **CA-F1-03-05:** telas de agenda e autosserviço mostram status correto sem vazar dados entre perfis.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S301–S306 antes da RPC/telas | `supabase test db` + roteiro Skip | Falhas esperadas para regra não implementada. | Saída inicial. |
| GREEN | S301–S307 | Aplicar RPC/RLS e construir telas | CA-F1-03-01 a 05 passam. | Testes, eventos e capturas. |
| REFACTOR/REGRESSÃO | Concorrência, fuso e acesso cruzado | `supabase test db` + três perfis na UI | Sem sobrescrita ou vazamento. | Relatório e roteiro datado. |

**Fixtures:** empresas/motoristas/veículos A/B fictícios; janelas passadas, futuras e sobrepostas. **Evidência exigida:** eventos antes/depois, teste SQL e capturas mobile.

## Handoff e operação

Demonstrar S301–S307 à operação. Interno acompanha pendentes diariamente; no-show só após a janela. Sem acesso ao Supabase, usar contingência de chegada espontânea, sem criar confirmação fictícia. Mudança de confirmação/capacidade exige emenda antes de implementar.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1-012 | Modelo/estados | Implementação | F1-03 | CA-F1-03-03 | S304 | SQL + teste | F1-006 | ☐ |
| F1-013 | Portal motorista | Implementação | F1-03 | CA-F1-03-01 | S301/S307 | Capturas + teste | F1-009/010/012 | ☐ |
| F1-014 | Portal empresa | Implementação | F1-03 | CA-F1-03-02 | S302/S307 | Capturas + teste | F1-008/010/012 | ☐ |
| F1-015 | Agenda/decisão interna, inclusive criação em nome de terceiro | Implementação | F1-03 | CA-F1-03-02/03 | S303/S304 | RPC + roteiro | F1-004/012 | ☐ |
| F1-016 | Concorrência/idempotência | Implementação | F1-03 | CA-F1-03-04 | S305/S306 | Testes concorrentes | F1-013–015 | ☐ |
| F1-017 | E2E de previsões | Implementação | F1-03 | CA-F1-03-05 | S307 | Relatório + capturas | F1-013–016 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
