# AP-2026-10-15-1640 — Credencial de teste humano em bloco copiavel

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-003 / SPEC-F1-01
- Sinal: relato "deu senha incorreta" apos 8 provas automaticas verdes; as credenciais revalidadas por API no mesmo minuto deram LOGIN OK nas tres fixtures — causa foi copia/digitacao da senha (caractere `!` final perdido ou espaco extra), nao defeito do sistema.
- Evidência: ciclo de debug 15/10/2026 ~16:35 (changelog); revalidacao de API 16:33 e 16:39.
- Regra reutilizável: roteiro de teste humano com credencial entrega e-mail e senha em blocos de codigo separados (linha a linha, sem espaço); relato de "senha incorreta" com provas de API verdes → primeiro revalidar a credencial por API e checar copia antes de suspeitar do produto.
- Quando aplicar: todo roteiro de teste humano que envolva login de fixture.
- Quando não aplicar: quando o relato traz erro de negocio visivel (gate, redirect, mensagem de tela) — ai debugar direto.
- Confiança: alta — revalidacao de API no mesmo instante isola a causa.
- Privacidade: sem segredo real (credenciais sao fixtures descartaveis de teste), sem dado pessoal.
