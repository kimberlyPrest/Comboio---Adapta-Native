# Rollback real provado pela camada que ainda não existe

- **Data**: 2026-10-15T17:30:00-03:00 · **Task**: F1-004 · **Capturado por**: Severino (agente)

## O que aconteceu

Para provar "ação parcial é revertida" (S105) sem código de teste artificial, ordenei o build assim: publiquei a Edge Function ANTES de criar a RPC de perfil. A primeira chamada de convite então criou o usuário no Auth, falhou no passo 2 (RPC inexistente — PGRST202) e o caminho de rollback executou de verdade: `deleteUser` + "criação revertida". Confirmação independente por SQL: 0 usuários em `auth.users` para o e-mail da prova.

## Por que importa

- A prova de rollback é exercitada pelo caminho real de produção, não por mock ou flag de teste.
- A falha é induzida naturalmente pela sequência de build (função antes da permissão).
- Zero superfície de teste no produto (D17: código além do aceite é superfície não verificada).

## Aplicação futura

Sempre que uma task exigir "ação parcial revertida": montar a sequência de build para que o passo posterior ainda não exista na primeira chamada; essa primeira tentativa é a prova de rollback; depois aplicar a camada que faltava e seguir para o GREEN.
