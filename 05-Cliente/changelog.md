# Histórico do pacote do cliente

## 15/10/2026 — F1-003 concluída + DEBUG

- 2026-10-15 · [Jean Borges Tavares] · Task F1-003 concluída: gate de perfil no Skip 62225 (v0.0.7, QA verde) — sessão consulta `profiles`: `disabled` é deslogado com aviso, `pending` não opera ("Aguardando aprovação"), falha na leitura do perfil bloqueia operação; sem sessão não há acesso (logout encerra de fato); senha mínima 8 no cadastro; falha de rede traduzida para mensagem própria. Provas S103: 8 verdes (4 API + 4 UI) — pendente/desativado/ativo com perfis corretos, credencial inválida genérica, logout revoga, `/dashboard` sem sessão redireciona. Teste humano aprovado pelo champion em 15/10 16:37. Evidência: `artifacts/f1-003-evidencia.md` (workspace ETHOS). Pendência de painel: ativar leaked-password protection (Security → Authentication) — não exposta pelo MCP.
- 2026-10-15 · [Jean Borges Tavares] · DEBUG task F1-003 (2º ciclo): relato "senha incorreta" → credenciais revalidadas por API nas três fixtures (todas LOGIN OK) → causa: digitação/cópia da senha no formulário (caractere `!` perdido ou espaço extra), não defeito → credenciais reentregues em blocos copiáveis linha a linha → teste aprovado.
- 2026-10-15 · [Jean Borges Tavares] · DEBUG task F1-003 (1º ciclo): "Failed to fetch" no login → causa raiz: Supabase indisponível (521/Bad Gateway) confirmada por 3 caminhos independentes → defeitos próprios achados e corrigidos: erro de rede cru em inglês (v0.0.6) e logout sem encerrar acesso no dashboard (v0.0.7) → ver `06_notas/debug/debug-2026-10-15-f1-003-supabase-indisponivel.md`.
- Progresso da fase 1: 3/24 (12%); F1-004 é a próxima elegível.

## 03/10/2026 — Progresso da implementação

- 2026-10-03 · [Jean Borges Tavares] · Task F1-002 concluída: migrações `f1_002_profiles_company_users_audit_rls` + endurecimento `f1_002_harden_functions_v2` aplicadas no Supabase `lwkqsukxefwpreqjdkzg` (enums, `profiles`, `company_users`, `companies`, `audit_events`, RLS em todas, auditoria transacional por trigger, admin inicial por procedimento controlado, fixtures fictícios A/B/motorista); RED comprovado (anon leu tabela sem RLS via REST) e GREEN verificado por SQL/REST diretos: S102 (anon `[]` nas 4 tabelas), S104 (Empresa A só a própria; Motorista A só si; auditoria só admin), INSERT anônimo negado (42501); advisors críticos limpos pós-revoke de PUBLIC. Teste humano aprovado pelo champion em 03/10 (login no preview com fixture + SQL de isolamento). SQL versionado no Supabase; evidência no workspace ETHOS (`artifacts/f1-002-evidencia.md`).
- 2026-10-03 · [Jean Borges Tavares] · DEBUG task F1-002: teste humano relatado como falha sem descrição → não reproduzido em 9 provas de regressão (S102/S104/insert/login/auditoria verdes em 03/10 08:15) → roteiro de teste reformulado em caminho único (Skip primeiro, SQL opcional); teste reaprovado. Corrigido.
- Progresso da fase 1: 2/24 (8%); F1-003 é a próxima elegível.

## 29/09/2026 — Progresso da implementação

- 2026-09-29 · [Jean Borges Tavares] · Task F1-001 concluída: conexão Skip↔Supabase verificada por tela de teste (`/connection-test`, projeto Skip 62225, versão 0.0.4, QA verde) lendo o projeto Supabase `lwkqsukxefwpreqjdkzg` somente com a chave publicável (anon); S101 verde (Conexão OK, resposta PGRST205 do próprio banco) e caminho de erro exercitado (falha de rede exibida com data/hora); teste humano aprovado pelo champion no preview. Evidências: `artifacts/f1-001-evidencia.md` e capturas `f1-001-*.png` no workspace ETHOS; rota no preview `login-posto-comboio-a8a05--preview.goskip.app`.
- Progresso da fase 1: 1/24 tasks concluídas (4%); F1-002 é a próxima elegível.

## 29/09/2026 — Adequação documental de SPECs e tasks

- Quatro SPECs reformatadas com contexto, degrau, limites, dados, fluxo, checklist, critérios, TDD, handoff, tasks vinculadas e emendas.
- Tabela de 24 tasks ampliada com dono, SPEC, critério, recorte da prova, evidência, pré-condições, ponto de parada e status.
- Matriz de rastreabilidade da fase 1 incluída e cópias sincronizadas com o plano. Escopo e gates preservados sem alteração.

## 29/09/2026 — Completação da pasta operacional

- Adicionada a cópia com nome explícito `01_projeto/02-Escopo-Definitivo.md`, conferida byte a byte com o escopo do plano.
- Adicionados objetivo, visão, constituição, decisões, roadmap, sínteses de reunião, índice de documentos, status, notas e área de entregas.
- Incluída cópia íntegra do DOCX recebido do cliente em `03_documentos/`.
- Mantidas as quatro SPECs e 24 tasks liberadas para desenvolvimento da fase 1, agora também na estrutura canônica `04_fase-atual/`.
- Cópias duplicadas do pacote inicial (`01-Escopo.md` e `04-Fase-Atual/`) removidas após conferência; originais e novas cópias canônicas permanecem íntegros no plano e nesta pasta.
