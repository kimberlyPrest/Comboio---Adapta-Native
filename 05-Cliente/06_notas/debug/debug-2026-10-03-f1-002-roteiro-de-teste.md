# Debug Summary — F1-002 · 03/10/2026

**Task e problema:** teste humano da F1-002 relatado como falha, com relato vazio (mensagem terminou em "Descrevo o problema:").

**Reprodução:** não reproduzida. Em 03/10 08:15, re-execução completa: S102 (anon `[]` nas 4 tabelas), S104 Empresa A (só a própria empresa), Motorista A (só si), auditoria só admin, admin vê 4 perfis, INSERT anônimo negado (42501), 3 fixtures logando. Em 08:19, revalidação final do fechamento: novamente verde.

**Causa raiz:** não confirmada tecnicamente — a falha mais provável foi do roteiro de teste, que misturava SQL no painel Supabase com login no preview num único passo confuso. Sem relato do champion, nenhuma alteração de produto foi feita.

**Correção:** roteiro de teste reformulado em caminho único (Teste 1: login no preview com fixture; Teste 2 opcional: SQL de isolamento). Champion repetiu o teste e aprovou ("ok, deu certo", 08:18).

**Verificação automática:** passou — 9 provas de regressão (08:15) + revalidação de fechamento (08:19:20).

**Gate atual:** fechada — teste humano aprovado e task concluída.
