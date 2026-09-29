# Objetivo do projeto

Dar à Comboio uma base única para cadastrar empresas, motoristas e veículos, receber previsões de chegada e registrar o movimento real do pátio. O objetivo operacional é reduzir erros da planilha compartilhada, permitir que o motorista já cadastrado informe sua chegada pelo celular e dar à equipe visibilidade de quem foi previsto, chegou e saiu.

O sistema não deve obrigar reserva: a operação relatada pelo cliente recebe caminhões sem agendamento. A primeira entrega usa o Skip para a interface e o Supabase para autenticação, dados e regras de acesso. A triagem e a automação física são evoluções separadas.

Resultado verificável da fase 1: empresa aprovada e motorista ativo conseguem fazer solicitações dentro de seus perfis; a equipe interna recebe veículos previstos e espontâneos, registra entrada/saída e consulta o histórico sem exposição entre empresas. Os critérios completos estão no [escopo definitivo](02-Escopo-Definitivo.md) e nas [SPECs](../04_fase-atual/specs/00-INDICE.md).
