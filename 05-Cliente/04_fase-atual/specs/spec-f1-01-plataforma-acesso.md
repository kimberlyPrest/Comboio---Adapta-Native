# SPEC-F1-01 — Plataforma, identidade e isolamento

**Fase:** 1  
**Status:** planejada; pronta para execução técnica conforme pré-condições  
**Dono:** implementação Skip/Supabase; aceite de negócio pelo administrador indicado da Comboio  
**Origem no escopo:** §§1–3, 7, 9 e 12 do escopo definitivo  
**Degrau da solução:** nativo da plataforma — Skip para interface e Supabase Auth/Postgres/RLS; Edge Function somente para ações privilegiadas.

## Contexto e decisões fechadas

- **Estado atual:** não há aplicação desta fase nem contas/ambientes confirmados neste workspace; os controles relatados são planilhas e processos manuais.
- **Estado desejado:** três experiências autenticadas, dados persistentes e isolamento entre empresas/motoristas.
- **Decisões fechadas:** stack Skip + Supabase; autenticação pelo Supabase Auth; políticas RLS no banco; interno não tem autoinscrição pública.
- **Bloqueio localizado:** F1-001 não começa sem acesso autorizado às contas da Comboio; isso não bloqueia a redação desta SPEC.

## Resultado observável

Aplicação web responsiva construída no **Skip (goskip.dev, plataforma web da Adapta)**, ligada a um projeto **Supabase** da Comboio. O Skip entrega as telas e a experiência; Supabase fornece Auth, Postgres, políticas RLS, migrações e, quando necessário, Edge Functions para operações privilegiadas. A configuração e os prompts devem ser guardados como documentação versionada; exportação para GitHub, se disponível no plano, é recomendada para revisão e histórico, mas não é pré-condição do piloto. O implementador deve confirmar plano, ambiente de teste e acesso ao Skip/Supabase antes da F1-001. Este “Skip” não é o framework homônimo de apps iOS/Android.

## Limites e dependências

- **Inclui:** conexão Skip–Supabase, identidade, sessão, perfis, contexto de empresa, RLS, convite de internos, auditoria e telas-base.
- **Fora de escopo:** biometria, cancela, cobrança, triagem completa, importação de dados reais.
- **Entradas:** projetos Skip/Supabase autorizados, administrador inicial indicado, domínios/ambiente de teste.
- **Saídas:** aplicação de teste, migrações/policies, Edge Function administrativa, roteiro de acesso e evidências.
- **Atores:** admin gerencia internos; operador usa funções operacionais; empresa e motorista só acessam o próprio contexto.
- **Superfícies:** projeto Skip; Supabase Auth; schema `public`; policies RLS; Edge Functions; configuração de segredos.
- **Plano B e rollback:** acesso negado por padrão; em falha, suspender versão publicada e restaurar migração/backup ensaiado, sem apagar usuários.

## Dados e integrações

Identidade e sessão pertencem a `auth.users`/Supabase Auth; **não criar tabelas próprias de senha ou sessão**. Tabelas de domínio no schema `public`: `profiles(user_id UUID PK REFERENCES auth.users, role ENUM[admin,operator,company,driver], status ENUM[pending,active,disabled], created_at, updated_at)`; `company_users(user_id, company_id, status)`; `audit_events(id, actor_user_id nullable, entity_type, entity_id, action, before_json, after_json, created_at, request_id)`. Datas armazenadas em UTC e exibidas em `America/Sao_Paulo`; jamais gravar senha, token de sessão, chave secreta ou imagem facial no log. Um usuário de empresa pode ter vínculos múltiplos, mas deve escolher contexto de empresa ativo; toda consulta é protegida por RLS e, em operações privilegiadas, por validação na Edge Function.

| Origem/destino | Fonte de verdade | Campos/contrato | Permissão | Falha e recuperação |
| --- | --- | --- | --- | --- |
| Skip → Supabase Auth | `auth.users` | e-mail, sessão e `auth.uid()` | titular da sessão | Erro genérico; repetir login/recuperação, sem expor existência de conta. |
| Skip → Postgres | `profiles`, `company_users` | papel, status, contexto | RLS por usuário/vínculo | Acesso negado por padrão; não usar chave secreta no browser. |
| Edge Function → Auth/Postgres | funções admin | convite/desativação de interno | admin verificado no servidor | Idempotência por usuário/e-mail e auditoria; erro não cria acesso parcial. |

## Fluxo e regras

- Autenticação: usar **Supabase Auth e-mail/senha** com confirmação de e-mail configurada; login/logout/recuperação pela API oficial de Auth. O Skip não armazena hash/senha. Perfil `pending` não acessa operação. Limite de tentativas e proteção contra abuso devem ser configurados e testados no serviço.
- Primeiro admin: criado por procedimento controlado no Supabase, fora de formulário público. Demais internos: convite/criação via **Edge Function autenticada de admin** usando chave secreta apenas no servidor. A função valida o papel do solicitante no banco, cria/desativa usuário e audita a ação. Nunca expor chave `service_role`/secret no Skip ou no navegador.
- Habilitar **RLS em todas as tabelas expostas**. Policies usam `auth.uid()` e vínculos ativos; `anon` não lê cadastros, reservas ou visitas. Empresa A não lê B; motorista lê apenas próprios dados; operador/admin têm políticas explícitas por ação. Funções privilegiadas revalidam papel/vínculo no servidor e são auditadas.
- Operações de mudança de estado com múltiplas tabelas usam função SQL/RPC transacional ou Edge Function que chama RPC; falha não pode deixar evento sem estado ou vice-versa. Mensagens de erro não expõem dados de terceiros. Cada escrita relevante emite `audit_event` na mesma transação.

**Telas e estados:** login, sair, confirmação de e-mail/recuperação, acesso negado, home por perfil e administração de internos (lista, convidar, desativar). Mostrar carregamento, validação e falha; apoiar celular, teclado e leitores de tela nos controles principais.

| Cenário | Condição | Resultado esperado | Erro/recuperação |
| --- | --- | --- | --- |
| S101 | Ambiente Skip e Supabase de teste conectados | Consulta de teste funciona sem segredo no frontend. | Falha de conexão não publica app; conferir permissões/URL. |
| S102 | Admin inicial e migrações aplicadas | Perfil ativo acessa home admin; RLS nega anônimo. | Migração falha sem estado parcial; restaurar teste. |
| S103 | Login válido/inválido/pendente | Só ativo entra; logout/expiração revogam acesso. | Credencial inválida não revela conta. |
| S104 | Empresa A tenta ler B; motorista A tenta ler B | Consultas diretas com chave pública são negadas. | Registrar incidente sem revelar dados. |
| S105 | Admin convida/desativa operador | Usuário interno muda status com auditoria. | Operador não cria admin; ação parcial é revertida. |

## Instruções de execução para o implementador

1. **Ler antes de alterar:** escopo §§1–3, 7, 9 e 12, esta SPEC e tasks F1-001 a F1-005.
2. **Alterar somente:** ambiente Skip/Supabase da fase 1, Auth, perfis, policies, função admin e telas-base.
3. **Não alterar:** regras de reservas, cadastro de empresa/motorista, acesso físico ou dados biométricos.
4. **Executar:** cenários RED → migrações/RLS → Auth → função admin → telas Skip → regressão.
5. **Parar:** sem contas autorizadas, segredo exposto, RLS ausente ou acesso cruzado permitido.
6. **Estado válido ao parar:** nenhum dado de outra empresa visível; versão anterior continua utilizável ou ambiente de teste fica despublicado.

## Checklist de execução

- [ ] Projetos e titularidade conferidos; configuração sem segredos documentada.
- [ ] Migrações, grants e RLS aplicados em teste.
- [ ] Auth, perfis e função administrativa funcionando.
- [ ] S101–S105 exercitados, incluindo erro e acesso cruzado.
- [ ] Evidências anexadas e responsável da Comboio informado.

## Critérios de aceite

- [ ] **CA-F1-01-01:** Skip lê dados de teste do Supabase sem chave secreta no navegador.
- [ ] **CA-F1-01-02:** autenticação e estado de perfil impedem pendente/inativo de operar.
- [ ] **CA-F1-01-03:** RLS nega acesso cruzado mesmo por consulta direta ao Supabase.
- [ ] **CA-F1-01-04:** somente admin cria/desativa internos, com evento auditável.
- [ ] **CA-F1-01-05:** homes dos perfis funcionam em celular/desktop; logout e sessão expirada encerram acesso.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S102–S105 antes de policies/função/telas | `supabase test db` e roteiro UI | Acesso/fluxo ainda falha pelo motivo esperado. | Saída de teste inicial e roteiro datado. |
| GREEN | S101–S105 | Aplicar migrações, build no Skip, repetir testes | Todos os CA-F1-01 passam. | SQL versionado, resultado de testes e capturas. |
| REFACTOR/REGRESSÃO | Sessão expirada, acesso cruzado e migração reaplicada | `supabase test db` + teste UI por perfil | Nenhum vazamento ou regressão. | Relatório de regressão e matriz de permissões. |

**Fixtures:** admin, operador, empresas A/B e motoristas A/B fictícios; sem dados pessoais reais. **Evidência exigida:** SQL/policies, relatório de banco e capturas Skip. Se o Skip não permitir automação de UI, usar roteiro manual reproduzível.

## Handoff e operação

Demonstrar S101–S105 ao administrador Comboio. Após entrega, administrador revisa usuários internos e operador monitora falhas de acesso; incidentes de RLS/segredo suspendem publicação. Bloquear uso real se não houver projeto Supabase compartilhado, RLS testada, HTTPS ou backup. Rollback: despublicar versão Skip problemática e restaurar migração/backup testado, sem excluir usuários.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1-001 | Conectar Skip/Supabase | Implementação | F1-01 | CA-F1-01-01 | S101 | Captura + configuração sem segredos | Contas autorizadas | ☐ |
| F1-002 | Migrações, RLS e admin inicial | Implementação | F1-01 | CA-F1-01-02/03 | S102/S104 | SQL + testes de banco | F1-001 | ☐ |
| F1-003 | Auth no Skip | Implementação | F1-01 | CA-F1-01-02/05 | S103 | Roteiro UI | F1-002 | ☐ |
| F1-004 | Autorização e convite interno | Implementação | F1-01 | CA-F1-01-03/04 | S104/S105 | Policies + teste de função | F1-003 | ☐ |
| F1-005 | Homes por perfil | Implementação | F1-01 | CA-F1-01-05 | S103/S105 | Capturas mobile/desktop | F1-004 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
