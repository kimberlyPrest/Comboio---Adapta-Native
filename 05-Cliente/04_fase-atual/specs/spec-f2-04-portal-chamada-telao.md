# SPEC-F2-04 — Portal da fábrica, chamada, filas e telão

**Fase:** 2  
**Status:** liberada (10/11/2026)  
**Dono:** implementação Skip/Supabase; validação com uma fábrica piloto  
**Origem no escopo:** §§6, 7 e 14; DOCX de 26/09 §§7.1, 7.4 e 7.5; ata de 18/09 (chamada para carregar)  
**Degrau da solução:** nativo da plataforma: portal no Skip, Supabase Realtime para o telão e link de WhatsApp montado na tela (envio manual, sem API).

## Contexto e decisões fechadas

- **Estado atual:** a fábrica recebe e-mail manual de chegada e de "pode descer"; o motorista espera sem saber a ordem.
- **Estado desejado:** a fábrica acompanha os seus veículos e chama pelo portal; a logística vê duas filas; o telão da sala de espera mostra quem foi chamado; o motorista recebe o aviso por WhatsApp com um toque da logística.
- **Decisões fechadas:** filas de carga e descarga independentes; a chamada é da fábrica (ou da logística em nome dela, com motivo); o envio automático por WhatsApp e e-mail fica para a fase 4; o telão é uma tela do sistema num monitor comum, sem hardware especial.

## Resultado observável

Um veículo `pronto` é chamado pela fábrica, aparece na fila certa e no telão em segundos, e a logística avisa o motorista com um toque; a fábrica vê a evolução até a saída.

## Limites e dependências

- **Inclui:** portal da fábrica (lista, detalhe, laudo, chamada), filas, telão e aviso manual ao motorista.
- **Fora de escopo:** disparo automático (fase 4), estimativa de horário, doca com capacidade garantida.
- **Entradas:** triagens `pronto` (SPEC-F2-03); membros da fábrica (SPEC-F2-02).
- **Saídas:** chamadas, posições nas filas e eventos.
- **Plano B/rollback:** se o Realtime falhar, o telão recarrega a cada 30 s e mostra "atualizado às hh:mm"; se o portal cair, a logística chama em nome da fábrica, com motivo.

## Dados e integrações

- Chamada = transição `pronto` → `chamado_carga` ou `chamado_descarga` (RPC da SPEC-F2-02), com `called_by`, `called_at`, `on_behalf_reason` e destino (unidade/doca, texto livre).
- A fila é derivada: triagens `chamado_*` ordenadas por `called_at`, uma por operação. Não há tabela de fila separada.
- Telão: consulta segura de leitura que expõe **apenas** a placa, o primeiro nome do motorista, a operação, o destino e o horário da chamada. Acesso por um usuário de exibição (papel `display`) sem permissão de escrita.
- Aviso manual: o botão monta a mensagem ("Veículo ABC-1234 chamado para carregamento na Fábrica X") e abre `wa.me/<telefone>` com o texto; o evento registra que o aviso foi aberto, não que foi entregue.

## Fluxo e regras

| Cenário | Condição | Resultado esperado | Erro/recuperação |
| --- | --- | --- | --- |
| S801 | Usuário da fábrica abre o portal | Vê só os veículos das suas unidades, por fase, com laudo | — |
| S802 | Fábrica chama um veículo `pronto` | Estado `chamado_*`; entra no fim da fila certa; aparece no telão | Chamada duplicada é idempotente |
| S803 | Fábrica tenta chamar um veículo não `pronto` ou de outra unidade | Negado | Mensagem clara |
| S804 | Logística inicia a operação do 1º da fila | `em_operacao`; sai do telão | — |
| S805 | Telão sem conexão | Mostra o último estado com o horário e tenta reconectar | Nunca uma lista vazia falsa |
| S806 | Logística toca em "avisar motorista" | Abre o WhatsApp com o texto pronto; o evento é registrado | Sem telefone: botão desabilitado com motivo |
| S807 | Usuário `viewer` da fábrica tenta chamar | Negado (só `caller` chama) | — |

## Perguntas ao cliente (o agente pergunta antes de implementar)

1. **Na F2-018:** "Já existe TV ou monitor na sala de espera dos motoristas? Ele tem internet e um navegador (ou um computador ligado nele)?" Sem TV, entregue o telão e teste num computador comum.
2. **Na F2-018:** "No telão, pode aparecer o primeiro nome do motorista ou só a placa?" Até a resposta, mostre só a placa.
3. **Na F2-016:** "Ao chamar, a fábrica precisa informar a doca ou o local (ex.: 'Doca 3')?"

## Instruções de execução para o implementador

1. **Ler:** escopo §§6 e 14; SPEC-F2-02 e 03.
2. **Alterar somente:** portal, filas, telão e aviso manual.
3. **Não alterar:** a RPC de estados, exceto para usá-la.
4. **Executar:** RED → portal → chamada → filas → telão (Realtime) → aviso manual → GREEN.
5. **Modelo sugerido no Maestro:** Ultra para RLS, Realtime e a consulta do telão; Pro para telas.
6. **Parar:** se a exibição no telão exigir mais dados pessoais que os previstos.

## Checklist de execução

- [ ] Portal da fábrica com isolamento por unidade.
- [ ] Chamada com papel `caller` e idempotência.
- [ ] Filas na tela da logística.
- [ ] Telão em tela cheia, letra grande, separado por operação, com Realtime e fallback.
- [ ] Aviso manual por WhatsApp.
- [ ] S801–S807 comprovados.

## Critérios de aceite

- [ ] **CA-F2-04-01:** a fábrica vê só as suas unidades, em todas as fases, com laudo (S801, S803).
- [ ] **CA-F2-04-02:** a chamada leva o veículo à fila certa, na ordem, e só um `caller` chama (S802, S807).
- [ ] **CA-F2-04-03:** o telão mostra as chamadas em até 5 s, sem dados além dos previstos, e trata a falta de conexão (S802, S805).
- [ ] **CA-F2-04-04:** a logística opera as filas e avisa o motorista com um toque, com registro (S804, S806).

## TDD da SPEC

| Etapa | Prova | Comando/ação | Resultado esperado | Evidência |
| --- | --- | --- | --- | --- |
| RED | S801–S807 | `supabase test db` + roteiro | Falhas esperadas | Saída inicial |
| GREEN | S801–S807 | Telas, RLS, Realtime | CA-F2-04-01 a 04 passam | Testes + teste de queda do telão |
| REGRESSÃO | S603, S705 | Testes anteriores | Sem vazamento | Saída |

## Tasks vinculadas

| ID | Task | Dono | SPEC | Critério | Recorte da prova | Evidência esperada | Pré-condições | Status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F2-015 | Portal da fábrica (lista, detalhe, laudo) | Implementação | F2-04 | CA-F2-04-01 | S801/S803 | Testes RLS | F2-014 | ☐ |
| F2-016 | Chamada para carga/descarga | Implementação | F2-04 | CA-F2-04-02 | S802/S803/S807 | Testes | F2-015 | ☐ |
| F2-017 | Filas da logística | Implementação | F2-04 | CA-F2-04-04 | S804 | Roteiro | F2-016 | ☐ |
| F2-018 | Telão da sala de espera | Implementação | F2-04 | CA-F2-04-03 | S802/S805 | Teste de queda | F2-016 | ☐ |
| F2-019 | Aviso manual ao motorista por WhatsApp | Implementação | F2-04 | CA-F2-04-04 | S806 | Evento registrado | F2-017 | ☐ |

## Emendas

| Data | Origem do sinal | Micro-spec/task | Motivo |
| --- | --- | --- | --- |
| — | Nenhuma | — | — |
