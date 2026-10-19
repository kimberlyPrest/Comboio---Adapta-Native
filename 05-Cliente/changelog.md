# Histórico do pacote do cliente

## 19/10/2026 — F1-014 concluída

- 2026-10-19 · [Jean Borges Tavares] · Task F1-014 concluída: previsão da empresa — migrações `f1_014_company_arrival_request` + `f1_014_fix_recursion_v2` + `f1_014_dvl_company_policy`: caminho EMPRESA na `create_arrival_request` (usuário company ativo cria previsão para motorista com vínculo ativo empresa↔motorista e veículo vinculado àquele motorista; `source=company` auditável; mesmas regras de janela 4h/60d e sem sobreposição; evento "criação pela empresa"); policies de embed para o contexto da empresa via funções SECURITY DEFINER `company_sees_driver`/`company_sees_vehicle`. Skip 62225 (v0.0.24→v0.0.25, QA verde): seção "Previsões da frota" na home da empresa — lista com motorista/placa/janela/status e formulário em cascata. Provas: 12 verdes — RED PGRST202; Zeta cria p/ Motorista Novo (vínculo próprio, source=company); Zeta p/ motorista SEM vínculo P0001; motorista no caminho company P0001; anon 42501; isolamento de leitura; embeds verdes; regressão 4 perfis; UI mobile: dropdowns populam; previsão criada pela tela ("Solicitada"). Defeitos próprios corrigidos (3): embeds nulos (policies da F1-006 não cobriam a empresa); **RECURSÃO RLS (42P17)** — a primeira policy de embed criou ciclo drivers→cdl→drivers e QUEBROU a leitura de drivers para TODOS os perfis; corrigido com SECURITY DEFINER e regressão verificada nos 4 perfis; dropdown de veículos não filtrava pelo motorista. **APRENDIZADO DURÁVEL: policy RLS que consulta tabela cuja policy volta à primeira = recursão infinita que quebra a tabela INTEIRA; checagem de contexto entre tabelas relacionadas sempre via SECURITY DEFINER.** Teste humano do champion aprovado em 19/10 08:40. Evidência: `artifacts/f1-014-evidencia.md` (workspace ETHOS). Fixtures: conta zeta.gestora.teste@comboiologistica.com.br (F1014-Zeta!), previsões de Motorista Novo · GO9X8Y7.
- Progresso da fase 1: 14/24 (58%); F1-015 (agenda/decisão interna) é a próxima elegível.

## 16/10/2026 — F1-013 concluída

- 2026-10-16 · [Jean Borges Tavares] · Task F1-013 concluída: autosserviço do motorista — migrações `f1_013_driver_selfservice` + `f1_013_fix_vehicle_ref`: RPC `create_arrival_request` (motorista ATIVO cria previsão própria: veículo com vínculo ativo a SI; janela 4h/60d; triagem exige empresa vinculada; SEM sobreposição; nasce `requested` + evento) e RPC `change_arrival_request` (titular ou interno edita/cancela antes do check-in; cancelamento preserva histórico; EDIÇÃO MATERIAL de confirmado VOLTA A REQUESTED). Skip v0.0.23: "Minhas previsões" na home do motorista — lista com status colorido, Cancelar, formulário com aviso "não é garantia de vaga". Provas: 15 verdes (incl. regra de ouro confirmado→requested e trilha de eventos). Defeitos próprios (2): p_vehicle.id na RPC; marcador de conflito órfão no patch (QA pegou). DEBUG do teste humano: "não aparece nada" → cache do navegador; produção publicada às 21:41 e verificada na tela; aprovado em 16/10 21:45. Evidência: `artifacts/f1-013-evidencia.md`.
- Progresso da fase 1: 13/24 (54%); F1-014 e F1-015 são as próximas elegíveis.

## 16/10/2026 — F1-012 concluída

- 2026-10-16 · [Jean Borges Tavares] · Task F1-012 concluída: modelo e estados de previsão — migrações `f1_012_arrival_requests` + `f1_012_fix_audit_trigger`: enums arrival_source/kind/status, `arrival_requests` (janela 4h, trilha de decisão, version), `request_events` (histórico imutável), RLS por perfil, auditoria por trigger, RPC `review_arrival_request` (confirmar/recusar c/ motivo/no_show pós-janela, transições ilegais negadas, evento atômico). Provas: 13 verdes. Defeito próprio: trigger de auditoria com colunas do esquema real de audit_events. AP: ler information_schema de tabela de outra task antes de trigger/RPC. Defaults do piloto p/ validação: janela 4h, horizonte 60 dias, confirmado editado volta a requested. Teste humano aprovado em 16/10 21:35. Evidência: `artifacts/f1-012-evidencia.md`.
- Progresso da fase 1: 12/24 (50%); F1-013, F1-014 e F1-015 são as próximas elegíveis.

## 16/10/2026 — F1-011 concluída

- 2026-10-16 · [Jean Borges Tavares] · Task F1-011 concluída: E2E dos três cadastros (regressão, sem build novo) — jornadas completas com contas novas NA PRODUÇÃO (v0.0.21): EMPRESA (signup→onboarding→/company-request→aprovação→home ativa), MOTORISTA (onboarding→/driver-request→ativação→Meus veículos com E2EPL8X), INTERNO (convite→ativação). S205 isolamento verde; S206 mobile+desktop com capturas, sem biometria; lacuna mobile do form de empresa fechada; nenhuma lacuna nova. DEBUG: champion não recebia e-mail de confirmação → conta jean.e2e@ criada/confirmada por SQL; aprovado em 16/10 21:21. Evidência: `artifacts/f1-011-evidencia.md` (7 capturas).
- Progresso da fase 1: 11/24 (46%); **SPEC-F1-02 100% concluída**.

## 16/10/2026 — F1-010 concluída

- 2026-10-16 · [Jean Borges Tavares] · Task F1-010 concluída: veículos e vínculos — migrações `f1_010_vehicles_links` + `f1_010_fix_review_link_enum`: RPCs `register_vehicle` (placa normalizada, idempotente), `link_driver_vehicle` (motorista→si; interno→qualquer; reativa vínculo encerrado com histórico), `review_link` (approve→active/refuse→ended, link_state). Skip v0.0.21: "Meus veículos" (motorista) + fila de vínculos (admin). Provas: 16 verdes. Defeito próprio: enum link_state vs link_status. DEBUG do teste humano: produção nunca publicada → publicada (ref bcf3b4e) e verificada; 2º relato era cache; aprovado em 16/10 15:15. REGRA NOVA: produção publicada em todo fechamento de task. Evidência: `artifacts/f1-010-evidencia.md`.
- Progresso da fase 1: 10/24 (42%); F1-011 e F1-012 são as próximas elegíveis.

## 16/10/2026 — F1-009 concluída

- 2026-10-16 · [Jean Borges Tavares] · Task F1-009 concluída: autoinscrição/ativação de motorista — migração `f1_009_driver_registration`: RPC `request_driver_registration` (CPF normalizado único, idempotente por auth.uid(), mensagem segura na duplicata) + RPC `set_driver_status` (só por interno ativo). Skip v0.0.18: formulário /driver-request, DriverPendingRouter, fila de ativação na home admin. Provas: 14 verdes (mobile 390px); sem defeitos próprios. Teste humano aprovado em 16/10 11:46. Evidência: `artifacts/f1-009-evidencia.md`.
- Progresso da fase 1: 9/24 (38%); F1-010 e F1-012 são as próximas elegíveis.

## 16/10/2026 — F1-008 concluída

- 2026-10-16 · [Jean Borges Tavares] · Task F1-008 concluída: aprovar/rejeitar empresa — RPC transacional `review_company`: aprovação muda empresa+vínculo+perfil JUNTOS (ou nada muda); rejeição exige motivo (empresa rejected, vínculo disabled, perfil mantém pending); idempotente; rollback natural. Skip v0.0.17: fila de aprovação na home admin com modal de motivo obrigatório e erro no próprio modal. Provas: 12 verdes; pedido real do champion "comboio logistica" aprovado pela tela. Teste humano aprovado (Zeta aprovada pela tela). Evidência: `artifacts/f1-008-evidencia.md`.
- Progresso da fase 1: 8/24 (33%); F1-009 e F1-012 são as próximas elegíveis.

## 16/10/2026 — F1-007 concluída

- 2026-10-16 · [Jean Borges Tavares] · Task F1-007 concluída: solicitação de empresa — RPC transacional `request_company_registration`: cria companies(pending) + company_users(pending) juntos, idempotente por auth.uid(), CNPJ duplicado → mensagem segura. Skip v0.0.13→v0.0.14: onboarding de papel no 1º login, formulário /company-request com guard, Dashboard redireciona company/pending sem vínculo, refreshProfile no hook. Provas: 11 verdes. Teste humano: jornada real do champion; DEBUG (3 ciclos, 2 defeitos corrigidos): link de e-mail redirecionava p/ localhost (Site URL, pendência de painel); defeito REAL v0.0.14: gate tratava "sem perfil" como bloqueio — corrigido. Pendências de painel p/ Raysson: Site URL + Redirect URLs e leaked-password protection. Evidência: `artifacts/f1-007-evidencia.md`.
- Progresso da fase 1: 7/24 (29%); F1-008 é a próxima elegível.

## 15/10/2026 — F1-006 concluída

- 2026-10-15 · [Jean Borges Tavares] · Task F1-006 concluída: migrações `f1_006_cadastros_vinculos_v2` + `f1_006_harden_admin_rpcs_anon` (aditivas): companies completa (CNPJ único, trilha de revisão), drivers (CPF único 1:1), vehicles (placa única), 3 tabelas de vínculo com validade, RLS por perfil nas 5 superfícies, auditoria transacional, índices. Provas: 13 verdes (duplicatas 23505, concorrência de placa 201×409, isolamento, anon 42501). Achado advisors: anon tinha EXECUTE explícito nas RPCs admin — corrigido. Teste humano aprovado em 15/10 19:01 (1º relato era colagem de queries no batch). Evidência: `artifacts/f1-006-evidencia.md`.
- Progresso da fase 1: 6/24 (25%); F1-007 é a próxima elegível.

## 15/10/2026 — F1-005 concluída

- 2026-10-15 · [Jean Borges Tavares] · Task F1-005 concluída: homes por perfil no Skip 62225 (v0.0.8) — Dashboard ramifica por profile.role APÓS o gate: homes Admin/Operador/Empresa/Motorista; administração de internos na home admin via Edge Function da F1-004; admin não desativa a si mesmo. Provas: 11 verdes (mobile 390px + desktop 1440px). Teste humano aprovado em 15/10 18:40. SPEC-F1-01 100%. Evidência: `artifacts/f1-005-evidencia.md`.
- Progresso da fase 1: 5/24 (21%); F1-006 é a próxima elegível.

## 15/10/2026 — F1-004 concluída

- 2026-10-15 · [Jean Borges Tavares] · Task F1-004 concluída: autorização e convite de interno — Edge Function `admin-manage-internal` (service_role só no servidor, ator via JWT do chamador); RPCs `admin_register_internal`/`admin_set_internal_status` com revalidação `is_admin_active()`; convite cria operator/pending; idempotência por e-mail; rollback real provado (0 órfãos). Provas S105: 10 verdes. Teste humano aprovado em 15/10 17:34. Evidência: `artifacts/f1-004-evidencia.md`.
- Progresso da fase 1: 4/24 (17%); F1-005 é a próxima elegível.

## 15/10/2026 — F1-003 concluída + DEBUG

- 2026-10-15 · [Jean Borges Tavares] · Task F1-003 concluída: gate de perfil no Skip 62225 (v0.0.7) — disabled é deslogado com aviso, pending não opera, falha de leitura bloqueia; logout revoga; senha mínima 8; falha de rede traduzida. Provas S103: 8 verdes. Teste humano aprovado em 15/10 16:37. Evidência: `artifacts/f1-003-evidencia.md`. Pendência de painel: leaked-password protection.
- 2026-10-15 · DEBUG F1-003 (2º ciclo): "senha incorreta" → digitação/cópia (caractere `!` perdido), não defeito → credenciais reentregues em blocos copiáveis.
- 2026-10-15 · DEBUG F1-003 (1º ciclo): "Failed to fetch" → Supabase indisponível (521) confirmada por 3 caminhos; defeitos próprios corrigidos: erro de rede cru (v0.0.6) e logout sem revoger acesso (v0.0.7).
- Progresso da fase 1: 3/24 (12%); F1-004 é a próxima elegível.

## 03/10/2026 — Progresso da implementação

- 2026-10-03 · [Jean Borges Tavares] · Task F1-002 concluída: migrações `f1_002_profiles_company_users_audit_rls` + `f1_002_harden_functions_v2` (enums, profiles, company_users, companies, audit_events, RLS em todas, auditoria transacional, admin inicial por procedimento controlado, fixtures A/B); RED comprovado e GREEN verificado (S102 anon `[]`, S104 isolamento, INSERT anônimo 42501). Teste humano aprovado em 03/10. Evidência: `artifacts/f1-002-evidencia.md`.
- 2026-10-03 · DEBUG task F1-002: falha relatada sem descrição → não reproduzido em 9 provas de regressão → roteiro reformulado em caminho único; reaprovado.
- Progresso da fase: 2/24 (8%); F1-003 é a próxima elegível.

## 29/09/2026 — Progresso da implementação

- 2026-09-29 · [Jean Borges Tavares] · Task F1-001 concluída: conexão Skip↔Supabase por tela de teste (/connection-test, projeto Skip 62225, v0.0.4) lendo o Supabase `lwkqsukxefwpreqjdkzg` só com a chave anon; S101 verde; teste humano aprovado no preview. Evidências: `artifacts/f1-001-evidencia.md` e capturas no workspace ETHOS.
- Progresso da fase 1: 1/24 (4%); F1-002 é a próxima elegível.

## 29/09/2026 — Adequação documental de SPECs e tasks

- Quatro SPECs reformatadas com contexto, degrau, limites, dados, fluxo e regras, checklist, critérios, TDD, handoff, tasks vinculadas e emendas.
- Tabela de 24 tasks ampliada com dono, SPEC, critério, recorte da prova, evidência, pré-condições, ponto de parada e status.
- Matriz de rastreabilidade da fase 1 incluída e cópias sincronizadas com o plano. Escopo e gates preservados sem alteração.

## 29/09/2026 — Completação da pasta operacional

- Adicionada a cópia com nome explícito `01_projeto/02-Escopo-Definitivo.md`, conferida byte a byte com o escopo do plano.
- Adicionados objetivo, visão, constituição, decisões, roadmap, sínteses de reunião, índice de documentos, status, notas e área de entregas.
- Incluída cópia íntegra do DOCX recebido do cliente em `03_documentos/`.
- Mantidas as quatro SPECs e 24 tasks liberadas para desenvolvimento da fase 1, agora também na estrutura canônica `04_fase-atual/`.
- Cópias duplicadas do pacote inicial (`01-Escopo.md` e `04-Fase-Atual/`) removidas após conferência; originais e novas cópias permanecem íntegros no plano e nesta pasta.
