# Fases 2 e 3 — tarefas

Versão 1.0 · 10/11/2026. Stack: Skip (goskip.dev) + Supabase. **Fase 2 (25 tasks) e fase 3 (17 tasks) liberadas.** A fase 1 (24/24) está em [05_entregas/fase-1](../05_entregas/fase-1/fase.md); o registro da F1-024 é o primeiro passo da F2-001.

Como executar: pegue a primeira task com todas as pré-condições `☑`. Antes de implementar, leia a SPEC inteira e faça as **perguntas ao cliente** marcadas para aquela task. Defina o cenário de teste antes do build, prove e só então marque `☑`. Status `☐` = planejada; `◐` = em andamento; `☑` = concluída com prova. Modelo no Maestro: **Pro** como padrão; **Ultra** para migração, RLS, RPC, Storage, Realtime e jobs.

## Fase 2 — Triagem, portal da fábrica e acerto à vista

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F2-001 | Registrar a F1-024; *error boundary* e tipos gerados (L7, L9) | Implementação | [F2-01](specs/spec-f2-01-layout-usabilidade.md) | CA-F2-01-01 | S502/S505 | Captura de erro tratado + build falhando com campo errado | — | ☐ |
| F2-002 | `AppShell` das telas logadas (L1, L8) | Implementação | [F2-01](specs/spec-f2-01-layout-usabilidade.md) | CA-F2-01-04 | S503 | Capturas 390/1440 | F2-001 | ☐ |
| F2-003 | Escala de texto e placa (L2, L3) | Implementação | [F2-01](specs/spec-f2-01-layout-usabilidade.md) | CA-F2-01-04 | S503 | Capturas antes/depois | F2-002 | ☐ |
| F2-004 | Cards e barra de ações (L4) | Implementação | [F2-01](specs/spec-f2-01-layout-usabilidade.md) | CA-F2-01-03 | S504 | Capturas 390 px | F2-003 | ☐ |
| F2-005 | Dividir `homes.tsx`; resultado no lugar e busca fixa; aprovação (L5, L6, L10) | Implementação + Comboio | [F2-01](specs/spec-f2-01-layout-usabilidade.md) | CA-F2-01-02/05 | S501/S503 | Roteiro aprovado | F2-004 | ☐ |
| F2-006 | Linha de base do schema; papéis logística, caixa, fábrica e telão + RLS | Implementação | [F2-02](specs/spec-f2-02-perfis-fabricas-triagem.md) | CA-F2-02-01 | S603/S604 | Baseline + testes RLS | F2-001 | ☐ |
| F2-007 | Tipos de empresa, unidades e membros da fábrica | Implementação | [F2-02](specs/spec-f2-02-perfis-fabricas-triagem.md) | CA-F2-02-02 | S603 | SQL + captura | F2-006 | ☐ |
| F2-008 | Modelo e abertura da triagem com cadastro operacional | Implementação | [F2-02](specs/spec-f2-02-perfis-fabricas-triagem.md) | CA-F2-02-03 | S601/S602 | Testes + captura | F2-007 | ☐ |
| F2-009 | RPC de estados da triagem e eventos | Implementação | [F2-02](specs/spec-f2-02-perfis-fabricas-triagem.md) | CA-F2-02-04 | S605/S606 | Testes SQL | F2-008 | ☐ |
| F2-010 | Tela de trabalho da logística | Implementação | [F2-02](specs/spec-f2-02-perfis-fabricas-triagem.md) | CA-F2-02-03/04 | S601/S605 | Roteiro + capturas | F2-009, F2-004 | ☐ |
| F2-011 | Modelos de checklist por fábrica (versionados) | Implementação | [F2-03](specs/spec-f2-03-checklist-laudo.md) | CA-F2-03-01 | S706 | SQL + captura | F2-007 | ☐ |
| F2-012 | Execução do checklist no celular com fotos | Implementação | [F2-03](specs/spec-f2-03-checklist-laudo.md) | CA-F2-03-02 | S701/S702/S705 | Testes Storage + captura | F2-009, F2-011 | ☐ |
| F2-013 | Não conformidade e decisão da fábrica | Implementação | [F2-03](specs/spec-f2-03-checklist-laudo.md) | CA-F2-03-03 | S703/S704 | Eventos + captura | F2-012 | ☐ |
| F2-014 | Laudo imprimível | Implementação | [F2-03](specs/spec-f2-03-checklist-laudo.md) | CA-F2-03-04 | S701/S705 | Laudo exemplo (dados fictícios) | F2-012 | ☐ |
| F2-015 | Portal da fábrica (lista, detalhe, laudo) | Implementação | [F2-04](specs/spec-f2-04-portal-chamada-telao.md) | CA-F2-04-01 | S801/S803 | Testes RLS + capturas | F2-014 | ☐ |
| F2-016 | Chamada para carga/descarga | Implementação | [F2-04](specs/spec-f2-04-portal-chamada-telao.md) | CA-F2-04-02 | S802/S803/S807 | Testes + captura | F2-015 | ☐ |
| F2-017 | Filas da logística | Implementação | [F2-04](specs/spec-f2-04-portal-chamada-telao.md) | CA-F2-04-04 | S804 | Roteiro + captura | F2-016 | ☐ |
| F2-018 | Telão da sala de espera | Implementação | [F2-04](specs/spec-f2-04-portal-chamada-telao.md) | CA-F2-04-03 | S802/S805 | Vídeo curto + teste de queda | F2-016 | ☐ |
| F2-019 | Aviso manual ao motorista por WhatsApp | Implementação | [F2-04](specs/spec-f2-04-portal-chamada-telao.md) | CA-F2-04-04 | S806 | Captura + evento | F2-017 | ☐ |
| F2-020 | Parâmetros de preço e função de cálculo | Implementação | [F2-05](specs/spec-f2-05-acerto-saida-aceite.md) | CA-F2-05-01 | S901–S905/S910 | Testes SQL | F2-006 | ☐ |
| F2-021 | Quitação à vista, "a faturar", isenção e exceção | Implementação | [F2-05](specs/spec-f2-05-acerto-saida-aceite.md) | CA-F2-05-02 | S906/S907 | Testes + captura | F2-020, F2-009 | ☐ |
| F2-022 | Check-out condicionado ao acerto | Implementação | [F2-05](specs/spec-f2-05-acerto-saida-aceite.md) | CA-F2-05-03 | S908 | Testes + captura | F2-021 | ☐ |
| F2-023 | Ocupação com capacidade e fechamento do dia | Implementação | [F2-05](specs/spec-f2-05-acerto-saida-aceite.md) | CA-F2-05-04 | S909 | Captura + conferência | F2-022, F2-003 | ☐ |
| F2-024 | Segurança, backup e regressão da fase 2 | Implementação | [F2-05](specs/spec-f2-05-acerto-saida-aceite.md) | CA-F2-05-05 | S603/S705/S803/S907 | Relatório + restore | F2-019, F2-023 | ☐ |
| F2-025 | Piloto e aceite da fase 2 | Operação Comboio | [F2-05](specs/spec-f2-05-acerto-saida-aceite.md) | CA-F2-05-05 | S911 | Ata com pendências | F2-024, F2-005 | ☐ |

## Fase 3 — Caixa, contratos e gestão

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F3-001 | Papéis comercial e gerência + RLS | Implementação | [F3-01](specs/spec-f3-01-comercial-verificacao-consentimento.md) | CA-F3-01-01 | S1002/S1004 | Testes RLS | F2-024 | ☐ |
| F3-002 | Verificação presencial com fotos de documentos | Implementação | [F3-01](specs/spec-f3-01-comercial-verificacao-consentimento.md) | CA-F3-01-02 | S1001/S1003/S1004 | Testes Storage + captura | F3-001 | ☐ |
| F3-003 | Consentimento de marketing versionado | Implementação | [F3-01](specs/spec-f3-01-comercial-verificacao-consentimento.md) | CA-F3-01-03 | S1005/S1006 | Testes + relatório | F3-001 | ☐ |
| F3-004 | Cadastro rápido à vista no caixa | Implementação | [F3-01](specs/spec-f3-01-comercial-verificacao-consentimento.md) | CA-F3-01-04 | S1001/S1002 | Roteiro cronometrado | F3-002 | ☐ |
| F3-005 | Contratos, garantias e preços negociados | Implementação | [F3-02](specs/spec-f3-02-contratos-bloqueio.md) | CA-F3-02-01 | S1101/S1102 | Testes + captura | F3-001 | ☐ |
| F3-006 | Aprovação e exceção do CEO | Implementação | [F3-02](specs/spec-f3-02-contratos-bloqueio.md) | CA-F3-02-01 | S1102/S1103 | Testes + eventos | F3-005 | ☐ |
| F3-007 | Cobrança a prazo na saída | Implementação | [F3-02](specs/spec-f3-02-contratos-bloqueio.md) | CA-F3-02-02 | S1104/S1105/S1108 | Testes SQL | F3-006 | ☐ |
| F3-008 | Bloqueio e desbloqueio de placa | Implementação | [F3-02](specs/spec-f3-02-contratos-bloqueio.md) | CA-F3-02-03 | S1106/S1107 | Testes + captura | F3-001 | ☐ |
| F3-009 | Fechamento automático de faturas (job) | Implementação | [F3-03](specs/spec-f3-03-faturas-pendencias.md) | CA-F3-03-01 | S1201–S1203 | Testes + `cron.job` | F3-007 | ☐ |
| F3-010 | Fatura imprimível e registro de envio | Implementação | [F3-03](specs/spec-f3-03-faturas-pendencias.md) | CA-F3-03-02 | S1204 | Fatura exemplo (dados fictícios) | F3-009 | ☐ |
| F3-011 | Baixa de pagamento | Implementação | [F3-03](specs/spec-f3-03-faturas-pendencias.md) | CA-F3-03-03 | S1205/S1207 | Testes + captura | F3-009 | ☐ |
| F3-012 | Pendências e atraso | Implementação | [F3-03](specs/spec-f3-03-faturas-pendencias.md) | CA-F3-03-03 | S1206 | Captura | F3-011 | ☐ |
| F3-013 | Lançamento de abastecimento pelo caixa | Implementação | [F3-04](specs/spec-f3-04-bonificacao-painel-aceite.md) | CA-F3-04-01 | S1301/S1302/S1305 | Testes + captura | F3-001 | ☐ |
| F3-014 | Crédito de estadia e uso na saída | Implementação | [F3-04](specs/spec-f3-04-bonificacao-painel-aceite.md) | CA-F3-04-01 | S1303/S1304 | Testes SQL | F3-013 | ☐ |
| F3-015 | Painel gerencial | Implementação | [F3-04](specs/spec-f3-04-bonificacao-painel-aceite.md) | CA-F3-04-02 | S1306/S1307 | Captura + conferência | F3-012, F3-014 | ☐ |
| F3-016 | Segurança, backup e regressão da fase 3 | Implementação | [F3-04](specs/spec-f3-04-bonificacao-painel-aceite.md) | CA-F3-04-03 | S1004/S1107/S1207/S1307 | Relatório + restore | F3-015, F3-004, F3-008 | ☐ |
| F3-017 | Piloto e aceite da fase 3 | Operação Comboio | [F3-04](specs/spec-f3-04-bonificacao-painel-aceite.md) | CA-F3-04-03 | S1308 | Ata com pendências | F3-016, F2-025 | ☐ |

## Definition of done comum

Cenário definido antes do build Skip; prova verde no Supabase ou no roteiro de UI; revisão das policies e funções; nenhum segredo no frontend; documentação de uso e rollback; evidência ligada ao ID. Toda migration nova fica versionada em `supabase/migrations/` do repo de código. Falha de teste, decisão estrutural nova ou dependência externa não resolvida mantém `☐`.
