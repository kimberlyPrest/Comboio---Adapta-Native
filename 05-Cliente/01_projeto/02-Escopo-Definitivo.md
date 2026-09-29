# Comboio — escopo definitivo do sistema de cadastros e previsões de chegada

Versão 1.0 · 29/09/2026 · Documento de execução sujeito às decisões explicitamente marcadas como **a validar**.

## 1. Resultado esperado e recorte

Construir um sistema web, utilizável no celular e no computador, que concentre os cadastros de empresas, motoristas, veículos e usuários internos; permita registrar previsões de chegada; e dê à operação uma visão única das solicitações, chegadas e atendimentos. O primeiro uso não depende de cancela, leitor de placa, reconhecimento facial, PDV nem faturamento.

**Stack definida pelo cliente para a fase 1:** Skip (plataforma web da Adapta) para construção da interface e Supabase para autenticação, banco PostgreSQL, regras de acesso por linha e funções de backend. A integração facial futura usará API da AWS, mediada por backend seguro; chaves AWS não ficam no Skip/navegador. O projeto Supabase deve ser da Comboio ou de titularidade formalmente definida, nunca apenas de conta pessoal da equipe de implementação.

O termo **reserva** neste documento significa uma intenção de chegada com data, janela de horário, veículo e finalidade, que pode ser confirmada pela operação. Não pressupõe vaga numerada nem proíbe chegada espontânea. Essa distinção preserva a operação relatada por Jean: hoje não há agendamento prévio e o caminhão inicia o processo quando chega.

**Jornadas abrangidas:** (A) estacionamento avulso; (B) veículo de triagem para carga/descarga. A jornada B pode exigir confirmação do contratante ou da equipe interna antes de ser considerada autorizada. Um motorista não pode aprovar por conta própria um serviço encomendado por uma empresa.

**Objetivo mensurável proposto:** após implantação do piloto, 100% das previsões e chegadas usadas no teste devem estar registradas em uma base única; cada veículo deve ter um histórico consultável de solicitação, confirmação, check-in e encerramento. Tempo de cadastro, taxa de comparecimento e ocorrências sem cadastro serão medidos no piloto para estabelecer metas operacionais definitivas.

## 2. Princípios de produto

1. O motorista pode chegar sem reserva. A operação registra um atendimento avulso, sem criar uma previsão retroativa fictícia.
2. Cadastro é distinto de autenticação: empresa, motorista e colaborador têm formulários e permissões próprias.
3. Uma reserva não é entrada no pátio. A ocupação só muda no check-in real e na saída real.
4. O sistema mantém trilha de quem alterou dados e status, inclusive cancelamentos e liberações excepcionais.
5. O reconhecimento facial é previsto em fase futura, com entrada alternativa não biométrica. Nenhuma decisão de acesso físico será automatizada antes de homologação dos equipamentos e validação operacional.
6. Dados compartilhados e persistentes são obrigatórios desde a primeira fase; `localStorage` por navegador não é solução de produção.

## 3. Perfis e permissões

| Perfil | Cadastro e acesso | Pode fazer | Não pode fazer |
| --- | --- | --- | --- |
| Administrador interno | Criado por administrador autorizado; sem autoinscrição pública | Gerir usuários, perfis, empresas, motoristas, veículos, parâmetros, agenda e auditoria | Apagar histórico operacional sem rastreabilidade |
| Operador interno | Criado pelo administrador | Cadastrar e corrigir dados operacionais, criar reserva em nome de terceiro, confirmar/recusar conforme política, registrar chegada e saída, tratar exceções | Administrar perfis/permissões ou dados de outras funções restritas |
| Empresa contratante | Solicita cadastro com CNPJ; ativação pela equipe interna | Manter dados próprios, vincular usuários da empresa, cadastrar motoristas/veículos associados, criar e consultar solicitações da própria empresa | Visualizar outras empresas, aprovar o próprio cadastro, criar usuário interno |
| Motorista | Autoinscrição ou convite/registro pela empresa ou equipe interna; ativação após verificação | Atualizar dados próprios, cadastrar/vincular veículo permitido, solicitar previsão de chegada, consultar e cancelar suas solicitações elegíveis | Confirmar triagem do contratante, alterar dados de outro motorista, ver reservas de terceiros |

O vínculo motorista–empresa pode ser múltiplo, mas cada solicitação deve indicar **qual empresa/contratante e qual veículo** a justificam. Quando o motorista não tiver empresa associada, poderá solicitar estacionamento avulso; triagem dependerá de associação/validação. Usuário de empresa enxerga apenas sua organização e unidades autorizadas. Exclusão de usuário não apaga registros históricos; aplica-se desativação.

## 4. Cadastros e campos

### 4.1 Empresa

Formulário próprio para solicitação de cadastro. Campos mínimos: razão social, nome fantasia, CNPJ, endereço básico, nome e e-mail do responsável, telefone, tipo de relação com a Comboio (contratante de triagem, transportadora ou cliente de estacionamento), unidade/fábrica quando aplicável e status (`pendente`, `ativa`, `rejeitada`, `inativa`). CNPJ deve ser único entre empresas ativas; o sistema valida formato e duplicidade. A aprovação interna vincula quem aprovou, data e motivo de rejeição quando houver. Documentos comerciais, crédito e preços negociados ficam fora da primeira fase.

### 4.2 Motorista

Formulário enxuto, mobile-first: nome completo, CPF, telefone, e-mail para acesso, senha ou convite seguro, vínculo com empresa quando houver, status de verificação e aceite dos termos de uso/aviso de privacidade. CPF e e-mail não podem gerar contas duplicadas ativas; correções por homônimo ou erro devem ser feitas pela operação com trilha. CNH, imagem facial e documento do veículo **não são campos obrigatórios da fase 1**. A necessidade de CNH será validada com a operação antes de coleta.

### 4.3 Veículo

Placa, tipo (caminhão, carro ou moto), identificação complementar opcional e vínculos autorizados com motorista/empresa. Placa é normalizada para busca e duplicidade. O vínculo pode ser alterado sem apagar o histórico de quem dirigiu em cada visita. Carreta/reboque, cavalo e composição exigem definição da equipe operacional antes de se tornarem campos obrigatórios.

### 4.4 Usuário interno

Nome, e-mail corporativo, função, perfil de permissão, status ativo/inativo, criador e data de criação. Contas internas são provisionadas apenas por administrador. Mudanças de perfil, bloqueio e reativação são auditadas. Não existe formulário público para “virar interno”.

### 4.5 Dados de solicitação e visita

Identificador, origem (`motorista`, `empresa`, `interno`, `chegada espontânea`), tipo de atendimento (`pátio` ou `triagem`), empresa/contratante quando aplicável, motorista, veículo, unidade de destino, data prevista, janela de horário, observações operacionais, status, motivo de recusa/cancelamento, autor e datas de cada transição. Na visita real: horário de chegada, operador responsável, ticket/referência, horário de saída e exceções. Pedido/NF, produto e quantidade para carga/descarga só entram quando o fluxo de triagem for detalhado e validado; **não há emissão de NF-e** neste escopo.

## 5. Fluxos funcionais

### 5.1 Solicitação de cadastro de empresa

1. Responsável informa os campos da empresa e cria acesso inicial.
2. Sistema valida campos, duplicidade e registra pedido `pendente`.
3. Interno autorizado aprova/rejeita, com registro de responsável e data.
4. Após aprovação, a empresa pode manter seus usuários, veículos e solicitações, dentro do limite de permissões.

**Exceções:** empresa duplicada, CNPJ inválido, representante que não consegue comprovar vínculo, empresa inativada e solicitação de correção cadastral. A comprovação e o procedimento de análise serão definidos pela Comboio; o sistema não assume aprovação automática.

### 5.2 Cadastro e acesso do motorista

1. Motorista cria a conta ou recebe convite da empresa/Comboio.
2. Informa dados mínimos; o sistema valida duplicidades e conclui a verificação de acesso.
3. Pode cadastrar/vincular veículo e solicitar previsão de chegada para pátio.
4. Para triagem, escolhe empresa/unidade autorizada e encaminha solicitação para confirmação.

**Exceções:** motorista sem e-mail, telefone compartilhado, CPF já cadastrado, troca de empresa, veículo de terceiro. O canal alternativo de ativação sem e-mail é decisão pendente; até lá, a operação poderá criar a solicitação em nome do motorista.

### 5.3 Criação e confirmação da previsão

Empresa, motorista ou interno informa atendimento, veículo, motorista, destino e data/janela. O sistema mostra conflito com solicitações ativas do mesmo veículo e horário. Para estacionamento, a política inicial sugerida é **confirmação operacional simples**, sem promessa de vaga; para triagem, confirmação por empresa contratante ou operação autorizada. A política exata, prazo de resposta e eventuais limites por janela serão configurados após validação da capacidade real.

Status da solicitação: `rascunho` → `solicitada` → `confirmada` ou `recusada`; de `solicitada/confirmada` pode ir a `cancelada` ou `não compareceu`; `confirmada` pode ir a `chegou`. Não se permite saltar para `chegou` sem registro de check-in. Cada transição tem data, autor e motivo quando aplicável. A edição relevante de uma solicitação confirmada exige nova confirmação ou justificativa de exceção.

### 5.4 Chegada sem reserva e check-in

Operador pesquisa por placa/CPF; se houver previsão, associa a chegada ao registro certo; se não houver, abre visita espontânea. Se o motorista ainda não estiver cadastrado, registra dados mínimos para identificação e o encaminha para completar cadastro durante a permanência, conforme operação relatada. A visita fica `em atendimento`, com horário real de entrada. O painel de ocupação conta apenas visitas presentes, separadas por tipo de veículo. Não atribui vaga física individual.

### 5.5 Saída e encerramento

Interno autorizado registra horário de saída e condição de liberação operacional. Na fase inicial, cobrança e quitação podem ser **referência/estado lançado manualmente**, sem processar pagamento, conciliar meios de pagamento ou emitir documento fiscal. O sistema impede encerramento silencioso sem identificação suficiente e registra a justificativa de exceção. Nenhuma falha de cadastro ou reconhecimento facial poderá deixar o motorista sem rota de atendimento humano.

### 5.6 Triagem posterior

Quando ativada a fase de triagem, a visita passa pelos estados de cadastro operacional, checklist por contratante, tratamento de não conformidade, `pronto para carga/descarga`, chamada e conclusão. Filas de carga e descarga são separadas. Quem pode aprovar checklist, chamar motorista, resolver não conformidade e liberar saída precisa ser validado por contratante; a especificação do checklist será derivada dos formulários e vídeos reais.

## 6. Telas e experiência

**Portal da empresa:** solicitação de cadastro; painel das solicitações próprias; nova previsão em nome de motorista/veículo; equipe e veículos vinculados; detalhe e histórico; na fase de triagem, visualização de status e chamada de veículos de suas unidades.

**Portal do motorista:** cadastro e acesso; dados pessoais; veículos vinculados; solicitação em até poucos passos; lista de próximas chegadas; detalhe com status, endereço/instruções e opção de cancelamento; visualização clara de que a previsão não substitui o check-in. A interface deve ser utilizável em celular e não expor dados de outros motoristas.

**Painel interno:** agenda por dia/janela, filtros por tipo/status/empresa/placa, solicitações pendentes, chegada espontânea, check-in/check-out, ocupação atual, cadastros para aprovação e histórico auditável. A fase de triagem acrescenta fila, checklist, laudo e exceções.

**Notificações:** a primeira fase mostra alterações dentro do sistema e permite compartilhar confirmação manualmente. Envio automático por e-mail/WhatsApp depende de canal, fornecedor, custo e consentimentos definidos; não é pré-requisito para começar a operar.

## 7. Entrega por fases

| Fase | Capacidades entregues | Resultado demonstrável e aceite |
| --- | --- | --- |
| **1 — Uso imediato: cadastro e previsão** | Base central; login e permissões; formulários separados de empresa, motorista e interno; veículo; empresa com aprovação; motorista mobile; criação/consulta/cancelamento de previsão; agenda interna; registro de chegada espontânea e check-in/check-out manual; auditoria básica. | Empresa solicita cadastro e é aprovada; motorista previamente cadastrado faz uma previsão; interno confirma, recebe veículo com e sem previsão e encerra visita; cada perfil só vê os dados autorizados. |
| **2 — Operação logística e triagem** | Cadastro operacional, checklists configuráveis por contratante, fotos/laudo, estados, filas, visão da empresa e chamada; painel de ocupação e exceções refinado. | Veículo de triagem percorre chegada, checklist, aprovação, chamada e saída, com histórico e isolamento de dados entre contratantes. |
| **3 — Identificação facial com AWS** | Cadastro biométrico opcional, verificação facial por integração com Amazon Rekognition, política de confiança, revisão/fallback humano, exclusão de referência biométrica e trilha; estudo e, se justificado, prova de vivacidade. | Piloto controlado identifica motorista recorrente sem impedir o fluxo por falha ou recusa da biometria; falsos positivos/negativos e tempos de resposta medidos; privacidade aprovada antes de ativação. |
| **4 — Acesso físico e comunicações** | Integração homologada com equipamento escolhido, sinal de autorização, leitura de ticket/placa se aprovada, notificações automáticas e procedimentos de contingência. | Teste presencial de entrada/saída com equipamento, falha de rede, falta de energia, ticket inválido e liberação manual documentada. |

Cada fase exige demonstração com dados de teste, validação dos atores operacionais e correção de defeitos impeditivos antes da próxima. As fases 3 e 4 não são requisitos para a fase 1 entrar em uso. Nenhum prazo de calendário ou preço de implantação é fixado sem acesso ao ambiente, responsáveis e volume esperado.

## 8. Reconhecimento facial com AWS — desenho e limites

O serviço candidato é **Amazon Rekognition**: coleção de referências faciais para usuários previamente cadastrados e busca/verificação por imagem. O cadastro associa o identificador técnico da face ao motorista no banco do sistema; a imagem ou vetor não vira identificador público do usuário. A checagem pode apoiar a identificação na chegada/saída, mas não substitui a autorização de reserva, a condição de pagamento nem a segurança da cancela. A escolha entre comparação 1:1, busca 1:N e prova de vivacidade será decidida em teste com o equipamento e cenário real. A AWS documenta `IndexFaces` para inserir referências, `SearchFacesByImage` para busca em coleção e Face Liveness como verificação probabilística de presença; nenhum desses recursos garante acerto perfeito.

**Pré-condições da fase 3:** equipamento/câmera e iluminação adequados; conectividade; conta e região AWS definidas; orçamento de uso; controles de acesso e criptografia; política de limiar testada; procedimento de cadastro, atualização e exclusão; avaliação de privacidade, finalidade e base legal; alternativa não biométrica funcional. Dados biométricos vinculados à pessoa são dados pessoais sensíveis segundo a ANPD. O sistema não coleta fotografia facial na fase 1 “para usar depois”. Falha de correspondência não equivale a fraude e deve encaminhar para conferência alternativa.

**Fronteira de responsabilidade:** Adapta desenvolve a integração de software, sujeita a credenciais e custos AWS fornecidos/aprovados pela Comboio. A Comboio contrata/adquire câmeras, leitores, cancelas, instalação e manutenção; o fornecedor de hardware precisa expor interface documentada e participar da homologação. A integração entre software e acionamento físico só entra após fornecedor definido e testes conjuntos.

## 9. Regras de segurança, dados e operação

- Autenticação individual, recuperação segura de acesso, sessão com expiração e permissões verificadas no servidor; proibição de contas compartilhadas para ações auditáveis.
- Separação de dados por empresa/unidade; empresa e motorista não consultam dados de outro contratante por manipulação de URL ou identificador.
- Validação de CPF/CNPJ, placa, e-mail, datas e status na interface e no servidor; prevenção de duplicatas e registro de correções.
- Criptografia em trânsito e proteção de dados em repouso; segredos/credenciais fora do código; backups testados; registro de alterações sensíveis e acessos administrativos.
- Política de retenção e exclusão definida antes da produção, especialmente para documentos, fotos de checklist e biometria. Dados comerciais/marketing devem ter finalidade e preferências separadas do cadastro operacional.
- A agenda deve funcionar para múltiplos operadores simultâneos sem sobrescrever dados. Fuso horário da operação deve ser explícito nos registros e telas.
- Disponibilidade, volume de usuários, veículos por dia e tempo-alvo de resposta serão quantificados no levantamento inicial; não se promete operação offline nem acionamento autônomo de cancela na fase 1.

## 10. Fora do escopo inicial

Pagamento online, TEF, conciliação bancária, faturamento automático, emissão de NF-e, integração com WebPosto/Promoflex, campanhas de marketing, cálculo definitivo de estadia/bonificação, OCR de nota fiscal, agendamento de doca com capacidade garantida, atribuição de vaga numerada, reconhecimento facial e abertura física de cancela. Funcionalidades de fases futuras deste documento não estão incluídas no aceite da fase 1.

## 11. Decisões pendentes e dependências

| ID | Decisão/insumo | Dono sugerido | Impacto |
| --- | --- | --- | --- |
| D1 | Uma previsão de estacionamento é automática, pendente ou exige aprovação? Há cota por janela? | Jean + operação | Política de reserva e mensagens ao motorista |
| D2 | Quem confirma triagem: contratante, logística ou ambos? Quem pode chamar veículo? | Jean + clientes-piloto | Permissões e fluxo da fase 2 |
| D3 | Qual canal viável para ativar motorista sem e-mail? | Operação + TI | Inclusão digital e autenticação |
| D4 | Quais campos são realmente obrigatórios para empresa, motorista, veículo, carga e descarga? | Operação + contratantes | Formulários, importação e privacidade |
| D5 | Como tratar não comparecimento, atrasos, reprogramação e cancelamento após check-in? | Operação | Estados e métricas da agenda |
| D6 | Números de vagas (500 caminhões, 100 carros, 100 motos) e comportamento em lotação. | Jean + operação | Ocupação e alertas |
| D7 | Regras de cobrança/isenção e responsável pelo acerto, se forem incorporadas depois. O DOCX conflita entre “até 24h” e fórmula que chega a 26h. | Jean + caixa | Futuro módulo financeiro; não bloqueia fase 1 |
| D8 | Fornecedor/equipamento, tipo de câmera, interface, rede, segurança física e custos AWS. | Comboio + TI + fornecedor | Fases 3 e 4 |
| D9 | Documentos de processos reais, planilha, formulários e checklists prometidos na reunião. | Comboio | Detalhamento e aceite da fase 2 |
| D10 | Quem é controlador/operador dos dados, política de retenção e fundamento do tratamento biométrico. | Comboio + responsável por privacidade | Liberação da fase 3 |

As decisões pendentes não autorizam improviso técnico. A fase 1 pode começar com parâmetros conservadores: reserva como previsão não garantida, triagem sempre pendente de confirmação e chegada espontânea sempre registrável.

## 12. Roteiros mínimos de aceite

1. **Empresa:** solicita cadastro; não acessa dados operacionais antes da aprovação; após ativação, vê apenas suas solicitações.
2. **Motorista recorrente:** acessa pelo celular, escolhe veículo e janela, envia previsão e acompanha confirmação; não vê dados de outros motoristas.
3. **Motorista novo sem previsão:** é recebido pela operação, entra como chegada espontânea e tem cadastro concluído sem perder o histórico da visita.
4. **Triagem:** pedido feito pelo motorista não se confirma sozinho; pessoa autorizada confirma; o atendimento percorre checklist, chamada e encerramento na fase 2.
5. **Conflito e exceção:** placa/CPF duplicado, mudança de horário, não comparecimento e falha de rede são tratados com mensagens claras e registro de responsável.
6. **Biometria futura:** usuário sem biometria, imagem não reconhecida e serviço AWS indisponível mantêm alternativa de identificação; nenhum falso match libera acesso físico por si só.

## 13. Base desta versão

- Reunião de mapeamento anexada pelo usuário: Jean declara ausência de agendamento (02:39–03:00), dois perfis de atendimento (02:58), entrada de novos motoristas com ticket e cadastro posterior (05:21–07:00), pátio sem vaga numerada (08:07–08:47) e compra de equipamentos pela Comboio (08:49–10:05).
- Documento do cliente “Sistema de Automação do Pátio de Estacionamento”, datado de 26/09/2026: referência de funcionalidades e regras **propostas**, não prova de validação operacional.
- Re-alinhamento de 18/09/2026 registrado na pasta do projeto: pivô do escopo financeiro para pátio e triagem.
- Referências técnicas e de privacidade: [AWS Rekognition — IndexFaces](https://docs.aws.amazon.com/rekognition/latest/APIReference/API_IndexFaces.html), [AWS Rekognition — SearchFacesByImage](https://docs.aws.amazon.com/rekognition/latest/APIReference/API_SearchFacesByImage.html), [AWS Rekognition Face Liveness](https://docs.aws.amazon.com/rekognition/latest/dg/face-liveness.html), [AWS AI Service Card — Face Matching](https://docs.aws.amazon.com/ai/responsible-ai/rekognition-face-matching/overview.html) e [Glossário da ANPD — dado pessoal sensível](https://www.gov.br/anpd/pt-br/documentos-e-publicacoes/glossario-anpd).
- Referências da stack: [Skip — integração com Supabase](https://docs.goskip.dev/introduction/faq/), [Supabase Auth](https://supabase.com/docs/guides/auth/passwords), [Supabase Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security) e [Supabase Edge Functions e segredos](https://supabase.com/docs/guides/functions/secrets).
