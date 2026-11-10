# SPEC-F2-03 — Checklist por contratante, fotos e laudo

**Fase:** 2  
**Status:** liberada (10/11/2026)  
**Dono:** implementação Skip/Supabase; conteúdo dos checklists pela Comboio e pelas fábricas  
**Origem no escopo:** §§5.6, 7, 11 (D9) e 14; ata de 18/09; DOCX de 26/09 §7.1  
**Degrau da solução:** nativo da plataforma: Postgres, RLS, Supabase Storage privado e página de laudo para impressão/PDF no navegador.

## Contexto e decisões fechadas

- **Estado atual:** a inspeção é feita no papel ou em planilha, e a fábrica recebe o resultado por e-mail manual.
- **Estado desejado:** cada fábrica tem o seu modelo de checklist; a logística executa item a item no celular, com foto; o sistema emite um laudo com data, hora, fotos e responsável.
- **Decisões fechadas:** o checklist é **dado configurável**, não código (D9 deixa de bloquear o desenvolvimento); a não conformidade registra, fotografa e avisa a fábrica, mas **não bloqueia** entrada nem saída, porque quem decide é o contratante; o laudo emitido não é editado (correção gera um novo laudo que substitui o anterior).
- **Modelo inicial** (citado pelo Jean em 18/09): parte elétrica e setas; sirene de ré; para-brisa sem trinca; emissão de poluentes; pneus; limpeza da carroceria, compatível com a carga anterior.

## Resultado observável

A logística faz o checklist de um veículo no celular, com foto por item; se tudo estiver OK, a triagem vai para `pronto`; se houver não conformidade, a fábrica vê o problema e decide; o laudo fica disponível para a logística e para a fábrica.

## Limites e dependências

- **Inclui:** modelos versionados por fábrica, execução, fotos, decisão da fábrica sobre não conformidade e laudo.
- **Fora de escopo:** envio automático do laudo (fase 4), certificado de limpeza como serviço cobrado à parte (a validar), IA de análise de foto.
- **Entradas:** triagem `cadastrado` (SPEC-F2-02).
- **Saídas:** respostas, fotos, decisão e laudo.
- **Plano B/rollback:** sem rede, a logística anota no papel e lança depois, marcando "registro posterior".

## Dados e integrações

- `checklist_templates(id, company_id, version, active, created_by)` e `checklist_template_items(id, template_id, order, label, requires_photo bool, help_text)`. Mudar o modelo cria uma nova versão; triagens antigas continuam apontando para a versão usada.
- `checklist_runs(id, triage_id UNIQUE ativo, template_id, started_by, started_at, finished_at, result[aprovado,nao_conforme])`.
- `checklist_answers(run_id, item_id, value[ok,nao_conforme,nao_se_aplica], note, photo_path)`. Para não conformidade, a nota é obrigatória; a foto é obrigatória quando o item exige.
- Fotos no bucket privado `triagem/{triage_id}/…`, com acesso pela RLS da triagem (logística e a fábrica dona). Até 5 MB por foto, comprimida no celular. A retenção é definida antes do go-live (§9).
- `triage_reports(id, triage_id, run_id, version, issued_at, issued_by, superseded_by nullable)`.

## Fluxo e regras

1. A logística inicia o checklist (`cadastrado` → `em_triagem`); o sistema usa o modelo ativo da fábrica.
2. A logística responde cada item; não dá para concluir com item sem resposta.
3. Na conclusão: tudo OK ou "não se aplica" → `aprovado`, e a triagem vai para `pronto`. Com não conformidade → `nao_conforme`: a triagem fica em `em_triagem` com o aviso "aguardando decisão da fábrica".
4. A fábrica (portal) decide sobre a não conformidade: **liberar assim mesmo** (→ `pronto`, com motivo) ou **recusar** (→ `cancelado`, com motivo). A logística pode registrar a decisão em nome da fábrica, com o nome de quem decidiu.
5. O laudo é emitido na conclusão: página com o cabeçalho da Comboio, a placa, o motorista, a fábrica/unidade, a operação, os itens com resultado, nota e foto, e o horário e o responsável; pode ser impresso ou salvo em PDF pelo navegador.

| Cenário | Condição | Resultado esperado | Erro/recuperação |
| --- | --- | --- | --- |
| S701 | Checklist completo e OK | Triagem `pronto` e laudo emitido | — |
| S702 | Item em não conformidade sem nota ou foto | Conclusão bloqueada | Mensagem do item |
| S703 | Não conformidade concluída | Fábrica vê o aviso; triagem aguarda a decisão | — |
| S704 | Fábrica libera assim mesmo | `pronto`, com motivo e ator | — |
| S705 | Fábrica B tenta abrir foto ou laudo da fábrica A | Negado (RLS e Storage) | — |
| S706 | Modelo alterado durante uma triagem | A triagem em curso mantém a versão antiga | — |

## Perguntas ao cliente (o agente pergunta antes de implementar)

1. **Na F2-011:** "Podem me mandar o checklist de uma fábrica real (foto da folha ou lista dos itens)? Quais itens exigem foto?" Sem resposta, use o modelo inicial desta SPEC e marque-o como "modelo provisório".
2. **Na F2-012:** "Por quanto tempo precisamos guardar as fotos do checklist?" Sem resposta, use 12 meses e registre em `04-Decisoes.md` como pendente de confirmação.
3. **Na F2-014:** "A limpeza da carroceria gera um certificado separado, cobrado à parte, ou entra no laudo da triagem?" Até a resposta, a limpeza é um item do checklist e não é cobrada.

## Instruções de execução para o implementador

1. **Ler:** escopo §§5.6 e 9; SPEC-F2-02.
2. **Alterar somente:** modelos, execução, fotos, decisão e laudo.
3. **Não alterar:** estados fora do previsto na SPEC-F2-02.
4. **Executar:** RED → modelos → execução mobile → fotos/Storage → decisão → laudo → GREEN.
5. **Modelo sugerido no Maestro:** Ultra para Storage/RLS; Pro para telas.
6. **Parar:** se a política de retenção das fotos não estiver definida até o piloto; registre em decisões.

## Checklist de execução

- [ ] Modelos versionados e modelo inicial cadastrado para uma fábrica de teste.
- [ ] Execução no celular com câmera e compressão.
- [ ] Storage privado testado entre fábricas.
- [ ] Laudo imprimível.
- [ ] S701–S706 comprovados.

## Critérios de aceite

- [ ] **CA-F2-03-01:** admin cria e versiona o modelo de checklist por fábrica (S706).
- [ ] **CA-F2-03-02:** a logística executa no celular, com foto e nota obrigatórias quando exigidas (S701, S702).
- [ ] **CA-F2-03-03:** a não conformidade não bloqueia; a fábrica decide, com ator e motivo (S703, S704).
- [ ] **CA-F2-03-04:** laudo emitido, imprimível e visível só para a logística e a fábrica dona (S701, S705).

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S701–S706 | `supabase test db` + roteiro | Falhas esperadas | Saída inicial |
| GREEN | S701–S706 | Migrações, Storage, telas e laudo | CA-F2-03-01 a 04 passam | Testes, capturas e laudo exemplo |
| REGRESSÃO | S603, S605 | Testes da SPEC-F2-02 | Sem vazamento | Saída |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F2-011 | Modelos de checklist por fábrica (versionados) | Implementação | F2-03 | CA-F2-03-01 | S706 | SQL + captura | F2-007 | ☐ |
| F2-012 | Execução do checklist no celular com fotos | Implementação | F2-03 | CA-F2-03-02 | S701/S702/S705 | Testes Storage + captura | F2-009, F2-011 | ☐ |
| F2-013 | Não conformidade e decisão da fábrica | Implementação | F2-03 | CA-F2-03-03 | S703/S704 | Eventos + captura | F2-012 | ☐ |
| F2-014 | Laudo imprimível | Implementação | F2-03 | CA-F2-03-04 | S701/S705 | Laudo exemplo (dados fictícios) | F2-012 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
