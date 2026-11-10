# SPEC-F2-01 — Layout e usabilidade das telas operacionais

**Fase:** 2  
**Status:** liberada (10/11/2026)  
**Dono:** implementação Skip; validação de uso por Jean e Rayson  
**Origem no escopo:** §§6 e 14; pedido do cliente na call de acompanhamento de 28/10/2026  
**Degrau da solução:** nativo da plataforma: componentes e estilos do Skip (React + Tailwind + shadcn/ui); sem mudança de regra de negócio.

## Contexto e decisões fechadas

- **Relato do cliente (28/10):** telas longas, com muito scroll; o resultado de uma ação aparece longe; o conteúdo não usa a largura do computador; a letra fica fina e pequena no celular; os botões do card ficam encavalados; houve tela branca em erro (27 e 28/10).
- **Diagnóstico do código (repo `Login---Projeto`, v0.0.50, sincronizado em 10/11):**

| # | Causa encontrada | Onde | Efeito relatado |
| --- | --- | --- | --- |
| C1 | As telas de operação usam o mesmo `Layout` da tela de login: `main` com `flex items-center justify-center` (conteúdo centralizado na vertical), fundo animado, brilho e silhueta de bomba de combustível | `src/components/Layout.tsx` | Informação "lá em cima/no meio", rolagem longa, fundo distraindo |
| C2 | Todas as homes (admin, operador, empresa, motorista) ficam dentro de um cartão de **560 px** (`HomeShell`, `max-w-[560px]`), inclusive no computador | `src/pages/homes.tsx` (`HomeShell`) | "Tela reduzida, não ocupa o espaço" |
| C3 | Cerca de 130 textos em `text-xs`, `text-[11px]` ou `text-[10px]`, inclusive dados e botões | `homes.tsx` | Letra fina e pequena no celular |
| C4 | Botões dos cards com 32 px de altura (`h-8`), texto de 11 px e lado a lado: Confirmar, Recusar, No-show, Registrar chegada e Reabrir; o "Reabrir" abre o mesmo diálogo de recusa | `homes.tsx` (`InternalAgenda`, perto da linha 575) | Botões encavalados e risco de toque errado |
| C5 | Nenhum tratamento de erro global (*error boundary*). Os erros de 27–28/10 eram variáveis duplicadas ou fora de escopo (`cNotes`, `reqBusy`), que derrubam o app inteiro | Commits v0.0.48–v0.0.50 | Tela branca |
| C6 | `src/lib/supabase/types.ts` não tem nenhuma tabela, então o build não acusa campo ou tabela errados | `types.ts` | Erros só aparecem em produção |
| C7 | Uma única página de **3.131 linhas** concentra os quatro perfis e a agenda (1.166 linhas) | `homes.tsx` | Cada ajuste arrisca quebrar outro perfil (a origem dos "fixes" de 28/10) |

- **Estado desejado:** o operador encontra o que precisa sem rolar a tela, vê o resultado da ação no mesmo lugar, lê bem no celular e nunca vê tela branca.
- **Decisões fechadas:** manter a identidade verde e preto; o fundo animado e a silhueta continuam **só** no login e nos cadastros públicos; nenhuma regra de negócio, RPC ou RLS muda nesta SPEC.

## Resultado observável

As telas da fase 1 passam no roteiro de usabilidade no celular (390 px) e no computador (1440 px), e qualquer erro mostra uma mensagem com "tentar de novo".

## Limites e dependências

- **Inclui:** um layout próprio para as telas logadas, tratamento de erro, tipos gerados do Supabase, escala de texto, cards e ações, divisão do `homes.tsx` e foco no resultado.
- **Fora de escopo:** mudar campos, estados ou permissões.
- **Plano B/rollback:** publicar por etapa; se uma tela regredir, voltar a versão anterior no Skip sem tocar no banco.

## Regras de layout (contrato)

| ID | Regra | Como verificar |
| --- | --- | --- |
| L1 | Telas logadas usam um `AppShell` próprio: barra superior fixa (marca, perfil, sair), conteúdo a partir do topo, sem fundo animado, largura útil de até 1440 px; o painel interno usa grade de 2–3 colunas no computador | Captura em 1440 px sem faixa vazia maior que 25% da largura |
| L2 | Texto base de 16 px no celular; dados-chave (placa, status, nome) em peso 600–700; `text-[10px]`/`text-[11px]` só em rótulos auxiliares, nunca em dado ou botão | Busca no código + captura em 390 px |
| L3 | Placa sempre em destaque (fonte maior, negrito, caixa alta) em cards, listas e detalhes | Captura |
| L4 | Card com cabeçalho (placa + status), corpo e **barra de ações separada**: alvo de toque de 44 px ou mais (`h-11`), espaço mínimo de 8 px, quebra em coluna no celular, ação principal destacada; as de risco (Recusar, No-show) ficam num menu "mais ações" e pedem confirmação; cada botão abre o diálogo que o nome promete | Captura em 390 px; teste do "Reabrir" |
| L5 | Depois de qualquer ação, o card atualiza no lugar e aparece um aviso curto, sem rolar para o topo; formulários longos em etapas ou seções recolhíveis | Roteiro: registrar chegada e saída sem procurar o resultado |
| L6 | Busca por placa/CPF fixa no topo do painel interno | Captura |
| L7 | *Error boundary* global e um por seção do painel: erro mostra mensagem em português com "tentar de novo", **nunca tela branca**; falha registrada no console sem dado pessoal | Forçar erro de render, de rede e de permissão |
| L8 | Estados vazios e de carregamento explícitos, nunca zero falso | Captura |
| L9 | `types.ts` gerado do Supabase (`supabase gen types`) e usado nas consultas; o build falha com campo ou tabela inexistentes | Build com um campo errado proposital falha |
| L10 | `homes.tsx` dividido em arquivos por perfil e por seção (ex.: `pages/internal/Agenda.tsx`, `pages/company/Home.tsx`), sem mudar o comportamento | Nenhum arquivo de página acima de 600 linhas; roteiros da fase 1 passam |

## Perguntas ao cliente (o agente pergunta antes de implementar)

1. **Antes da F2-001:** "A fase 1 terminou em 28/10. Pode me mandar um resumo do piloto (as 4 jornadas testadas e o que ficou pendente) para registrar a F1-024?" Registre a resposta em `05_entregas/fase-1/f1-024-piloto-aceite.md`. Sem resposta, registre "aceite verbal em 28/10, ata pendente" e siga.
2. **Na F2-005:** "Quem vai validar as telas novas: Jean, Rayson ou os dois? Em que celular a equipe usa o sistema?"

## Fluxo e regras

1. Ordem: L7 + L9 (proteção) → L1 (AppShell) → L2–L3 → L4 → L10 → L5–L6.
2. Uma etapa por publicação; depois de cada uma, rodar os roteiros da fase 1 (S401–S406) para conferir que nada mudou.
3. Validar com Jean ou Rayson antes das telas novas da fase 2, que já nascem com estas regras.

| Cenário | Condição | Resultado esperado |
| --- | --- | --- |
| S501 | Operador registra chegada no celular | Resultado visível sem rolar; botões separados e legíveis |
| S502 | Rede cai, permissão é negada ou um componente quebra | Mensagem clara e "tentar de novo"; o resto da tela continua funcionando |
| S503 | Painel no computador com 30 veículos presentes | Grade usa a largura; busca no topo; placa legível |
| S504 | Card confirmado no celular | Ações sem sobreposição; Recusar e No-show pedem confirmação; "Reabrir" faz o que diz |
| S505 | Admin de empresa e motorista abrem as homes que deram tela branca em 28/10 | Carregam ou mostram o erro tratado |

## Instruções de execução para o implementador

1. **Ler:** esta SPEC e os arquivos citados no diagnóstico.
2. **Alterar somente:** layout, componentes, estilos, tratamento de erro, tipos gerados e a organização de arquivos do front.
3. **Não alterar:** tabelas, RLS, RPC, campos ou estados.
4. **Modelo sugerido no Maestro:** Pro; Ultra para a divisão do `homes.tsx` (L10).
5. **Parar:** se uma mudança visual exigir mudar regra ou dado; registre a dúvida em `01_projeto/04-Decisoes.md`.

## Checklist de execução

- [ ] F1-024 registrada (pergunta 1).
- [ ] L1–L10 aplicadas, com capturas antes e depois.
- [ ] S501–S505 comprovados.
- [ ] Roteiro de usabilidade aprovado pelo cliente.

## Critérios de aceite

- [ ] **CA-F2-01-01:** nenhuma tela fica branca em erro forçado e o build pega campo inexistente (S502, S505, L9).
- [ ] **CA-F2-01-02:** chegada e saída registradas no celular sem procurar o resultado (S501).
- [ ] **CA-F2-01-03:** cards e ações sem sobreposição em 390 px; ações de risco confirmadas; botões fazem o que dizem (S504).
- [ ] **CA-F2-01-04:** telas logadas usam a largura em 1440 px, com busca fixa e placa em destaque (S503).
- [ ] **CA-F2-01-05:** `homes.tsx` dividido sem regressão, e o cliente aprova o roteiro de usabilidade (L10).

## TDD da SPEC

| Etapa | Prova | Ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S501–S505 nas telas atuais | Capturas e erro forçado | Problemas C1–C7 reproduzidos | Capturas antes |
| GREEN | S501–S505 | Aplicar L1–L10 | Critérios 01–04 passam | Capturas depois |
| REGRESSÃO | S401–S406 | Rodar os roteiros da fase 1 | Nenhuma regra quebrada | Roteiro assinado |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F2-001 | Registrar a F1-024; *error boundary* e tipos gerados (L7, L9) | Implementação | F2-01 | CA-F2-01-01 | S502/S505 | Captura de erro tratado + build falhando com campo errado | — | ☐ |
| F2-002 | `AppShell` das telas logadas (L1, L8) | Implementação | F2-01 | CA-F2-01-04 | S503 | Capturas 390/1440 | F2-001 | ☐ |
| F2-003 | Escala de texto e placa (L2, L3) | Implementação | F2-01 | CA-F2-01-04 | S503 | Capturas antes/depois | F2-002 | ☐ |
| F2-004 | Cards e barra de ações (L4) | Implementação | F2-01 | CA-F2-01-03 | S504 | Capturas 390 px | F2-003 | ☐ |
| F2-005 | Dividir `homes.tsx`; resultado no lugar e busca fixa; aprovação (L5, L6, L10) | Implementação + Comboio | F2-01 | CA-F2-01-02/05 | S501/S503 | Roteiro aprovado | F2-004 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| 10/11/2026 | Leitura do código v0.0.50 | F2-001 a F2-005 | Diagnóstico C1–C7 substitui a task de inventário |
