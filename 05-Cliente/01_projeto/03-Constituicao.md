# Constituição do projeto

## Responsáveis

**Comboio:** dono do negócio e aprovação de regras; operação/logística para validação de fluxos; responsável por contas Skip, Supabase e, futuramente, AWS/equipamentos; responsáveis pela aprovação de cadastros e pelo piloto.

**Equipe de implementação:** construir as telas no Skip, configurar Supabase Auth/Postgres/RLS/RPC/Edge Functions, executar testes, registrar evidências por task e apresentar os fluxos para aceite. Não assume compra, instalação ou manutenção de cancelas.

## Fontes de verdade

O [escopo definitivo](02-Escopo-Definitivo.md) define resultado, limites e fases. As [SPECs da fase 1](../04_fase-atual/specs/00-INDICE.md) definem contratos implementáveis; [tasks](../04_fase-atual/fase.md) definem sequência e prova. Decisões posteriores devem atualizar o [registro de decisões](04-Decisoes.md) e, se afetarem contrato, gerar emenda da SPEC antes de alterar a aplicação.

## Gates

- Desenvolvimento da fase 1: liberado conforme SPECs, começando pela F1-001.
- Operação real: só após testes de segurança, isolamento entre empresas, backup/restauração, contingência e aceite da F1-024.
- Fases 2–4: não liberadas por este pacote; exigem decisão explícita sobre checklists/cliente, biometria e hardware, respectivamente.
- Publicação, convites externos, compra de recursos pagos e conexão com contas reais exigem autorização do titular da conta/projeto.

## Proteção do trabalho

Dados pessoais reais não entram em evidências de teste. Chaves secretas não entram no Skip, em prompts, nesta pasta nem no Git. Toda tabela exposta no Supabase tem RLS testada por perfil. O acesso físico sempre conserva um procedimento humano de contingência.
