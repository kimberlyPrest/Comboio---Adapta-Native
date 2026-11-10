# SPEC-F3-01 — Comercial, verificação presencial e consentimento

**Fase:** 3  
**Status:** liberada (10/11/2026); execução depois da F2-024  
**Dono:** implementação Skip/Supabase; validação pelo comercial e pelo caixa da Comboio  
**Origem no escopo:** §§7 e 14; DOCX de 26/09 §§3.1, 4.2–4.4 e 5; transcrição de 29/09 (cadastro presencial contra fraude)  
**Degrau da solução:** nativo da plataforma: Postgres, RLS e Storage privado.

## Contexto e decisões fechadas

- **Estado atual:** o motorista pode se cadastrar sozinho e é ativado depois; não há foto de documento; não há consentimento de marketing; não existem os perfis comercial e gerência.
- **Estado desejado:** o motorista só fica **verificado** depois de conferência presencial, com foto da CNH e do documento do veículo; o consentimento de marketing fica registrado com data e hora; o comercial faz o cadastro completo e o caixa só o cadastro à vista.
- **Decisões fechadas (DOCX):** o caixa cadastra somente à vista; o comercial cadastra à vista e a prazo; o Instagram é opcional; sem consentimento, o contato não entra na base de marketing; o motivo da verificação presencial é evitar documento e cheque clonados.

## Resultado observável

O caixa cadastra um motorista à vista em menos de 2 minutos, com foto da CNH; o comercial vê os verificados e os pendentes; um relatório lista só quem consentiu, com a data do aceite.

## Limites e dependências

- **Inclui:** papéis `commercial` e `manager`; status de verificação; fotos de documentos; consentimento versionado; cadastro rápido no caixa.
- **Fora de escopo:** OCR de documento; checagem em órgão externo; disparo de campanha (fase 4); facial (fase 5).
- **Entradas:** motoristas, veículos e empresas da fase 1; papéis da SPEC-F2-02.
- **Saídas:** `driver_verifications`, `vehicle_documents` e `marketing_consents`.
- **Plano B/rollback:** se o Storage falhar, a verificação fica "pendente de foto" e a operação segue com a conferência visual registrada.

## Dados e integrações

- `driver_verifications(driver_id, status[pendente,verificado,recusado], cnh_photo_path, verified_by, verified_at, note)`; a foto fica no bucket privado `documentos/`, visível só para comercial, gerência e admin.
- `vehicle_documents(vehicle_id, doc_photo_path, checked_by, checked_at)`.
- `marketing_consents(id, person_id, channel[whatsapp,email,instagram], granted bool, text_version, granted_at, revoked_at, collected_by)`; somente inserção; a revogação é um novo registro.
- `drivers.instagram` opcional; `drivers.payment_condition` ∈ {`a_vista`, `a_prazo`}, e só o comercial define `a_prazo` (SPEC-F3-02).

## Perguntas ao cliente (o agente pergunta antes de implementar)

1. **Na F3-002:** "A foto da CNH é obrigatória para todo motorista, ou só para quem vai ter contrato a prazo? E a foto do documento do veículo?"
2. **Na F3-002:** "Por quanto tempo guardamos as fotos de documentos? Quem pode ver?" Sem resposta: 5 anos, visíveis só para comercial, gerência e admin; registre como pendente.
3. **Na F3-003:** "Quem escreve e aprova o texto do consentimento de marketing?" Use um texto-modelo marcado "rascunho, aprovar com o responsável por privacidade" até a resposta.
4. **Na F3-001:** "Quem é comercial e quem é gerência hoje (nomes e e-mails)?"

## Fluxo e regras

| Cenário | Condição | Resultado esperado |
| --- | --- | --- |
| S1001 | Caixa cadastra motorista à vista, com foto da CNH | Motorista `verificado`, `a_vista`, com ator e horário |
| S1002 | Caixa tenta definir `a_prazo` | Negado |
| S1003 | Motorista autoinscrito na fase 1 | Fica `pendente` até a conferência presencial |
| S1004 | Fábrica, motorista ou logística tenta abrir foto de documento | Negado (RLS e Storage) |
| S1005 | Motorista consente WhatsApp e depois revoga | Dois registros; o relatório mostra o estado atual |
| S1006 | Relatório de marketing | Só consentimentos ativos, com versão do texto e data |

## Instruções de execução para o implementador

1. **Ler:** escopo §§7 e 14; SPEC-F1-02; SPEC-F2-02.
2. **Alterar somente:** papéis, verificação, documentos, consentimento e o cadastro rápido no caixa.
3. **Modelo sugerido no Maestro:** Ultra para Storage e RLS; Pro para telas.
4. **Parar:** se o cliente pedir para guardar documento sem prazo de retenção definido para produção.

## Critérios de aceite

- [ ] **CA-F3-01-01:** papéis comercial e gerência com RLS testada (S1002, S1004).
- [ ] **CA-F3-01-02:** verificação presencial com foto e ator; autoinscrito fica pendente (S1001, S1003).
- [ ] **CA-F3-01-03:** consentimento versionado, revogável e relatório correto (S1005, S1006).
- [ ] **CA-F3-01-04:** cadastro à vista no caixa em até 2 minutos no roteiro (S1001).

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S1001–S1006 | `supabase test db` | Falhas esperadas | Saída inicial |
| GREEN | S1001–S1006 | Migrações, Storage e telas | CA-F3-01-01 a 04 passam | Testes e capturas |
| REGRESSÃO | Fase 1 e 2 | Testes completos | Sem quebra | Saída |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F3-001 | Papéis comercial e gerência + RLS | Implementação | F3-01 | CA-F3-01-01 | S1002/S1004 | Testes RLS | F2-024 | ☐ |
| F3-002 | Verificação presencial com fotos de documentos | Implementação | F3-01 | CA-F3-01-02 | S1001/S1003/S1004 | Testes Storage + captura | F3-001 | ☐ |
| F3-003 | Consentimento de marketing versionado | Implementação | F3-01 | CA-F3-01-03 | S1005/S1006 | Testes + relatório | F3-001 | ☐ |
| F3-004 | Cadastro rápido à vista no caixa | Implementação | F3-01 | CA-F3-01-04 | S1001/S1002 | Roteiro cronometrado | F3-002 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
