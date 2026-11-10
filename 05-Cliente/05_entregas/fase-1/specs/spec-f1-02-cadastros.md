# SPEC-F1-02 — Cadastros de empresa, motorista e veículo

**Fase:** 1  
**Status:** planejada; pronta para execução técnica após SPEC-F1-01  
**Dono:** implementação Skip/Supabase; aceite de negócio pela operação Comboio  
**Origem no escopo:** §§3–5, 9 e 12 do escopo definitivo  
**Degrau da solução:** nativo da plataforma — formulários Skip, Supabase Auth/Postgres/RLS e RPC transacional para aprovação/vínculos.

## Contexto e decisões fechadas

- **Estado atual:** cadastro relatado como manual/planilha; formulários de origem prometidos, ainda não disponíveis neste pacote.
- **Estado desejado:** empresa, motorista e veículo identificados e vinculados, com aprovação interna auditável.
- **Decisões fechadas:** empresa e motorista podem solicitar cadastro; interno só é provisionado por admin; biometria e CNH não são obrigatórias na fase 1.
- **Bloqueio localizado:** ativação real depende de responsável de aprovação indicado; usar dados fictícios no desenvolvimento.

## Resultado observável

Três entradas distintas: formulário público de solicitação de empresa; formulário público de motorista; criação de usuário interno exclusiva do admin (SPEC-F1-01). Empresa aprovada vê “Minha empresa”, usuários vinculados e veículos; motorista ativo vê “Meu cadastro” e “Meus veículos”; operador vê fila de aprovação, busca e correção auditada. Formulários são mobile-first, com campos obrigatórios assinalados e erros ao lado do campo.

## Limites e dependências

- **Inclui:** pedido de empresa, autoinscrição de motorista, ativação, cadastros de veículo e vínculos validados.
- **Fora de escopo:** contratos, crédito, preços, marketing, biometria e coleta de CNH sem validação.
- **Entradas:** Auth e auditoria F1-01; responsável interno por aprovação; campos definidos no escopo §4.
- **Saídas:** cadastros ativos/pendentes, vínculos e histórico de revisão.
- **Atores:** solicitante de empresa, motorista, operador/admin; cada qual limitado ao próprio contexto.
- **Superfícies:** telas Skip; tabelas `companies`, `drivers`, `vehicles`, vínculos; RLS; RPC de registro/revisão.
- **Plano B/rollback:** falha após signup permite retomar sem duplicar; rollback de UI não apaga cadastros e preserva vínculos históricos.

## Dados e integrações

`companies(id, legal_name, trade_name, cnpj_normalized UNIQUE, address, contact_name, contact_email, phone, relationship_type, status[pending,active,rejected,inactive], reviewed_by, reviewed_at, review_reason)`; `drivers(id, user_id UNIQUE REFERENCES auth.users, full_name, cpf_normalized UNIQUE, phone, company_link_status)`; `vehicles(id, plate_normalized UNIQUE, type[truck,car,motorcycle], status)`; `driver_vehicle_links(driver_id, vehicle_id, valid_from, valid_to)`; `company_driver_links(company_id, driver_id, status)`; `company_vehicle_links(company_id, vehicle_id, status)`. Placa/CPF/CNPJ são normalizados antes da comparação. Não se apaga vínculo histórico utilizado em visita; encerra-se validade. E-mail de login pertence ao Supabase Auth. Campos de CNH, imagem facial, documento do veículo e marketing não entram na fase 1. Toda tabela exposta tem RLS e policy testada por perfil.

O pedido de empresa deve criar `company_users(user_id, company_id, status=pending)` junto com a empresa; a aprovação ativa **ambos**, além do perfil do responsável. Sem esse vínculo, o usuário aprovado não conseguiria acessar a própria empresa sob RLS.

| Origem/destino | Fonte de verdade | Campos/contrato | Permissão | Erro/recuperação |
| --- | --- | --- | --- | --- |
| Supabase Auth → cadastro | `auth.users` e `profiles` | `auth.uid()`, e-mail confirmado, papel/status | próprio titular | Falha pós-signup é retomável por `auth.uid()`. |
| Skip → RPC de empresa/motorista | tabelas de domínio | CPF/CNPJ normalizados, contato, status | usuário autenticado | Duplicado retorna revisão segura, sem revelar terceiros. |
| Skip → RPC de vínculo | tabelas de vínculo | pessoa, empresa, veículo, validade | titular ou operador | Vínculo pendente até aceite/validação; não apropriar cadastro alheio. |

## Fluxo e regras

- `request_company_registration` (Edge Function/RPC): após `signUp` no Supabase Auth e confirmação de e-mail, recebe dados do responsável e empresa, valida CNPJ/duplicidade e cria perfil + pedido `pending` em transação. Deve ser idempotente por `auth.uid()`; se a segunda etapa falhar, usuário Auth permanece pendente e pode retomar a solicitação sem duplicar empresa.
- `review_company` (RPC chamada por interno autorizado): aprova/rejeita, registra ator/data/motivo e ativa empresa + perfil responsável na mesma transação. Pedido duplicado/representante incerto permanece pendente para análise, sem mesclar automaticamente.
- `request_driver_registration` (Edge Function/RPC): após `signUp`/confirmação de e-mail no Supabase Auth, cria motorista/perfil `pending` por `auth.uid()`, valida CPF/telefone e permite retomada idempotente. Operador ativa após conferência. Sem e-mail, operador registra chegada espontânea na SPEC-F1-04 e cria cadastro assistido, mas não inventa conta acessível.
- Consultas/edições “Meu cadastro”, “Minha empresa” e veículos usam SDK Supabase com RLS; criação de vínculos e revisão de CPF/placa usam RPC transacional com autorização no banco. Empresa pode propor vínculo com motorista existente; vínculo entra `pending` até aceite do motorista ou validação interna, evitando apropriação unilateral de identidade.
- Conflito de CPF/CNPJ/placa retorna mensagem segura e fluxo de revisão interna; não revela cadastro completo de terceiro. Correção administrativa é auditada com antes/depois. Desativação impede novas solicitações, preserva histórico.

| Cenário | Condição | Resultado esperado | Erro/recuperação |
| --- | --- | --- | --- |
| S201 | Empresa com CNPJ válido solicita cadastro | Empresa, responsável e vínculo `pending`; sem acesso operacional. | CNPJ duplicado não cria segunda empresa. |
| S202 | Operador aprova/rejeita empresa | Status de empresa, perfil e vínculo mudam juntos, com auditoria. | Representante duvidoso fica pendente para revisão. |
| S203 | Motorista confirma e-mail e pede cadastro | Perfil `pending`; após conferência fica ativo e vê só próprio cadastro. | Falha pós-signup pode ser retomada. |
| S204 | Veículo/vínculo proposto | Placa única; vínculo exige aceite/validação; histórico preservado. | Associação indevida negada. |
| S205 | Empresa A/motorista A consulta dado alheio | RLS nega até por SDK/chave pública. | Mensagem sem identificação de terceiro. |
| S206 | Três formulários em celular/desktop | Campos e erros visíveis, sem biometria obrigatória. | Campo inválido não grava registro parcial. |

## Instruções de execução para o implementador

1. **Ler:** escopo §§3–5 e 9; SPEC-F1-01; tasks F1-006 a F1-011.
2. **Alterar somente:** cadastros/vínculos, policies/RPC associados e telas Skip desta SPEC.
3. **Não alterar:** reservas, visitas, faturamento ou facial.
4. **Executar:** testes RED → migrações/RLS → empresa/aprovação → motorista → veículos/vínculos → E2E.
5. **Parar:** ausência de aprovador, coleta de dado extra sem decisão ou vínculo que ignore validação.
6. **Estado válido ao parar:** cadastros já criados seguem consultáveis por seus titulares; pendentes não ganham acesso por falha parcial.

## Checklist de execução

- [ ] Estruturas e unicidade aplicadas em banco de teste.
- [ ] Cadastro de empresa cria `company_users` pendente e aprovação ativa o vínculo.
- [ ] Autoinscrição de motorista e vínculo de veículo validados.
- [ ] S201–S206 e caminhos de erro exercitados.
- [ ] Evidências e responsável de aprovação registrados.

## Critérios de aceite

- [ ] **CA-F1-02-01:** empresa solicita cadastro e não acessa operação antes da aprovação.
- [ ] **CA-F1-02-02:** aprovação/rejeição altera empresa, perfil e vínculo de modo atômico e auditável.
- [ ] **CA-F1-02-03:** motorista se cadastra, é ativado e consulta somente seus dados.
- [ ] **CA-F1-02-04:** CPF/CNPJ/placa duplicados e vínculos indevidos são bloqueados sem expor terceiros.
- [ ] **CA-F1-02-05:** formulários de empresa, motorista e interno funcionam em celular/desktop; interno segue exclusivo do admin via F1-01.

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S201–S205 sem tabelas/policies/RPC | `supabase test db` e roteiro Skip | Fluxo ainda falha pelo motivo esperado. | Saída datada/roteiro. |
| GREEN | S201–S206 | Aplicar migração/RPC, build Skip e repetir cenários | CA-F1-02-01 a 05 passam. | SQL, relatório e capturas. |
| REFACTOR/REGRESSÃO | Duplicidade concorrente, falha pós-signup e acesso cruzado | `supabase test db` + teste UI | Sem duplicata/vazamento; recuperação funciona. | Relatório de regressão. |

**Fixtures:** empresas A/B, motoristas A/B, placas fictícias e usuários por papel. **Evidência exigida:** migrações/policies, testes e capturas em celular/desktop, sem dados reais.

## Handoff e operação

Demonstrar S201–S206 à operação Comboio. Operador revisa pendências e conflitos; admin revisa permissões; cadastros duplicados não são mesclados automaticamente. Se o aprovador não estiver definido, desenvolvimento com dados fictícios pode prosseguir, mas ativação real para.

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1-006 | Migrar cadastros/vínculos | Implementação | F1-02 | CA-F1-02-01/04 | S201/S204 | SQL + testes | F1-002 | ☐ |
| F1-007 | Solicitação de empresa | Implementação | F1-02 | CA-F1-02-01 | S201 | Captura + teste | F1-003/006 | ☐ |
| F1-008 | Aprovação de empresa | Implementação | F1-02 | CA-F1-02-02 | S202 | RPC + auditoria | F1-004/007 | ☐ |
| F1-009 | Cadastro de motorista | Implementação | F1-02 | CA-F1-02-03 | S203 | Teste + captura mobile | F1-004/006 | ☐ |
| F1-010 | Veículos e vínculos | Implementação | F1-02 | CA-F1-02-04 | S204/S205 | SQL + roteiro | F1-008/009 | ☐ |
| F1-011 | E2E dos cadastros | Implementação | F1-02 | CA-F1-02-05 | S205/S206 | Relatório + capturas | F1-007–010 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
