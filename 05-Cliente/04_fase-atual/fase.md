# Fase 1 — tabela operacional de tasks

Versão 2.0 · 29/09/2026. Stack: Skip (goskip.dev) + Supabase. **16 de 24 tasks concluídas (F1-001 em 29/09, F1-002 em 03/10, F1-003 a F1-006 em 15/10, F1-007 a F1-013 em 16/10, F1-014 a F1-016 em 19/10/2026); as demais estão planejadas.** A sequência indica pré-condições: apenas tarefas com todas as dependências fechadas podem entrar na leva ativa. Status `☐` = planejada; `◐` = em andamento; `☑` = concluída com prova. Cada task aponta para uma única SPEC e uma subseção exata. Não criar conta AWS nem comprar equipamento nesta fase. Uso real depende de F1-024.

## Plataforma e acesso — SPEC-F1-01

| ID | Task e subseção da SPEC | Dono | SPEC | Critério binário | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1-001 | Conectar Skip/Supabase — **Limites e dependências; Dados e integrações**. | Implementação | F1-01 | CA-F1-01-01: tela de teste lê Supabase sem segredo no browser. | S101; GREEN da SPEC | Captura, IDs de ambiente e configuração sem segredos. | Contas autorizadas da Comboio. | Sem acesso/titularidade; não criar em conta pessoal. | ☑ 29/09/2026 |
| F1-002 | Migrar perfis/auditoria e RLS — **Dados e integrações; Fluxo e regras**. | Implementação | F1-01 | CA-F1-01-02/03: migração e negação de anônimo/cross-tenant passam. | S102/S104; RED→GREEN | SQL versionado e resultado `supabase test db`. | F1-001 concluída. | RLS ausente ou segredo exposto. | ☑ 03/10/2026 |
| F1-003 | Configurar Auth/telas login — **Fluxo e regras; Handoff e operação**. | Implementação | F1-01 | CA-F1-01-02/05: pendente/inativo não opera e logout revoga. | S103; GREEN | Roteiro e capturas de sessão/erro. | F1-002 concluída. | Senha em tabela própria ou pendente com acesso. | ☑ 15/10/2026 |
| F1-004 | Autorização e convite de interno — **Dados e integrações; Fluxo e regras**. | Implementação | F1-01 | CA-F1-01-03/04: externo não lê alheio; só admin convida/desativa. | S104/S105; REGRESSÃO | Policies, função, auditoria e teste direto. | F1-003 concluída. | Controle apenas visual ou chave secreta no Skip. | ☑ 15/10/2026 |
| F1-005 | Homes por perfil — **Resultado observável; Critérios de aceite**. | Implementação | F1-01 | CA-F1-01-05: perfis entram/saem em celular e desktop. | S103/S105; GREEN | Capturas/roteiro UI por perfil. | F1-004 concluída. | Autoinscrição de usuário interno. | ☑ 15/10/2026 |

## Cadastros — SPEC-F1-02

| ID | Task e subseção da SPEC | Dono | SPEC | Critério binário | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1-006 | Migrar cadastros/vínculos — **Dados e integrações**. | Implementação | F1-02 | CA-F1-02-01/04: unicidade e RLS passam, sem duplicata. | S201/S204; RED→GREEN | SQL e testes de banco/concorrência. | F1-002 concluída. | Migração destrutiva sem plano de recuperação. | ☑ 15/10/2026 |
| F1-007 | Solicitação de empresa — **Fluxo e regras**. | Implementação | F1-02 | CA-F1-02-01: pedido pendente sem acesso; retomada idempotente. | S201; GREEN | Captura e teste de falha pós-signup. | F1-003 e F1-006 concluídas. | Pedido concede acesso antes da aprovação. | ☑ 16/10/2026 |
| F1-008 | Aprovar/rejeitar empresa — **Dados e integrações; Fluxo e regras**. | Implementação | F1-02 | CA-F1-02-02: empresa, perfil e `company_users` mudam atomicamente. | S202; GREEN | RPC, evento e teste de rollback. | F1-004 e F1-007 concluídas; aprovador indicado. | Sem responsável de aprovação. | ☑ 16/10/2026 |
| F1-009 | Autoinscrição/ativação motorista — **Fluxo e regras**. | Implementação | F1-02 | CA-F1-02-03: pendente não reserva; ativo vê só si. | S203; GREEN | Roteiro mobile e teste de duplicata. | F1-004 e F1-006 concluídas. | Sem verificação do solicitante. | ☑ 16/10/2026 |
| F1-010 | Veículos/vínculos — **Dados e integrações; Fluxo e regras**. | Implementação | F1-02 | CA-F1-02-04: placa única e vínculo indevido negado. | S204/S205; REGRESSÃO | Teste de banco e evento de vínculo. | F1-008 e F1-009 concluídas. | Vínculo sem aceite/validação. | ☑ 16/10/2026 |
| F1-011 | E2E dos três cadastros — **Checklist; Critérios de aceite**. | Implementação | F1-02 | CA-F1-02-05: formulários funcionam e não coletam biometria obrigatória. | S205/S206; REGRESSÃO | Relatório e capturas mobile/desktop. | F1-007–F1-010 concluídas. | Coleta extra sem decisão. | ☑ 16/10/2026 |

## Previsões e autosserviço — SPEC-F1-03

| ID | Task e subseção da SPEC | Dono | SPEC | Critério binário | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1-012 | Modelo e estados — **Dados e integrações; Fluxo e regras**. | Implementação | F1-03 | CA-F1-03-03: transição/evento atômicos e ilegais negadas. | S304; RED→GREEN | SQL/RPC e testes de estado. | F1-006 concluída. | Estado sem evento ou RPC aberta a externo. | ☑ 16/10/2026 |
| F1-013 | Autosserviço motorista — **Fluxo e regras; Resultado observável**. | Implementação | F1-03 | CA-F1-03-01: titular cria/edita/cancela sem ocupar vaga. | S301/S307; GREEN | Capturas mobile e teste de vínculo. | F1-009/010/012 concluídas. | Triagem autoconfirmada ou vaga prometida. | ☑ 16/10/2026 |
| F1-014 | Previsão da empresa — **Fluxo e regras**. | Implementação | F1-03 | CA-F1-02-02: empresa cria só para próprios vínculos. | S302/S307; GREEN | Teste A→B e capturas. | F1-008/010/012 concluídas. | Vínculo não validado. | ☑ 19/10/2026 |
| F1-015 | Agenda/decisão/criação interna — **Fluxo e regras**. | Implementação | F1-03 | CA-F1-03-02/03: interno cria por terceiro, confirma/recusa/no-show com ator. | S303/S304; GREEN | RPC, eventos e roteiro de agenda. | F1-004/012 concluídas. | Regra de confirmação alterada sem emenda. | ☑ 19/10/2026 |
| F1-016 | Concorrência/idempotência — **Dados e integrações; Fluxo e regras**. | Implementação | F1-03 | CA-F1-03-04: uma atualização vence, outra recebe conflito. | S305/S306; REGRESSÃO | Teste concorrente e histórico. | F1-013–15 concluídas. | Perda silenciosa de atualização. | ☑ 19/10/2026 |
| F1-017 | E2E de previsões — **Checklist; Critérios de aceite**. | Implementação | F1-03 | CA-F1-03-05: status/isolamento corretos nos três portais. | S307; REGRESSÃO | Roteiro datado e capturas. | F1-013–16 concluídas. | Vazamento ou ocupação criada por previsão. | ☐ |

## Operação, qualidade e aceite — SPEC-F1-04

| ID | Task e subseção da SPEC | Dono | SPEC | Critério binário | Recorte da prova | Evidência esperada | Pré-condições | Ponto de parada | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1-018 | Modelo de visitas — **Dados e integrações**. | Implementação | F1-04 | CA-F1-04-01/04: visita aberta única e RLS negando externo. | S403/S407; RED→GREEN | SQL e testes concorrentes/RLS. | F1-012 concluída. | Migração apaga histórico. | ☐ |
| F1-019 | Check-in previsto/espontâneo — **Fluxo e regras**. | Implementação | F1-04 | CA-F1-04-01: visita/ocupação únicas, sem reserva fictícia. | S401/S402/S403; GREEN | Eventos e roteiro Skip. | F1-018/011 concluídas. | Entrada depende de facial/cancela. | ☐ |
| F1-020 | Reconciliar provisório — **Dados e integrações; Fluxo e regras**. | Implementação | F1-04 | CA-F1-04-02: mesmo ID/histórico após vínculo. | S404; GREEN | Evento e teste de vínculo. | F1-019 concluída. | Histórico sobrescrito. | ☐ |
| F1-021 | Check-out/exceção — **Fluxo e regras**. | Implementação | F1-04 | CA-F1-04-03: saída única; incompleto exige ator/motivo. | S405; GREEN | Testes e captura. | F1-019 concluída. | Motorista retido sem rota humana. | ☐ |
| F1-022 | Painel/ocupação — **Resultado observável; Fluxo e regras**. | Implementação | F1-04 | CA-F1-04-04: só visitas abertas contadas; externo negado. | S406/S407; REGRESSÃO | Captura e teste de contagem/RLS. | F1-019/021 concluídas. | Capacidade não validada exibida como garantida. | ☐ |
| F1-023 | Segurança/backup/contingência — **Checklist; Handoff e operação**. | Implementação | F1-04 | CA-F1-04-05: sem crítico aberto; restore e contingência demonstrados. | S403/S407; REGRESSÃO | Relatório, restore e procedimento. | F1-017/022 concluídas. | Vazamento, segredo exposto ou backup irrestaurável. | ☐ |
| F1-024 | Piloto/aceite — **Critérios de aceite; Handoff e operação**. | Operação Comboio | F1-04 | CA-F1-04-05: quatro jornadas e decisão de uso documentadas. | S408; REGRESSÃO | Ata assinada com pendências. | F1-023 concluída. | Sem aceite explícito, não operar com dados reais. | ☐ |

## Definition of done comum

Para marcar `☑`: cenário definido antes do build Skip, prova verde no Supabase/roteiro UI, revisão das policies/funções, nenhum segredo no frontend, documentação de uso/rollback e evidência ligada ao ID. Cobertura automatizada mínima de 80% das regras de domínio em SQL/RPC/Edge Functions; roteiro ponta a ponta para caminhos principais. Se o Skip não permitir automação UI, registrar teste manual reproduzível. Falha de teste, decisão estrutural nova ou dependência externa não resolvida mantém `☐`.
