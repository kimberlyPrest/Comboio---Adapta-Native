# AP-2026-10-15-1600 — Tres caminhos antes de julgar o gate

- Status: candidato
- Escopo: projeto do cliente
- Task/SPEC: F1-003 / SPEC-F1-01
- Sinal: teste humano devolveu "Failed to fetch" no login; antes de tratar como defeito do gate, a verificação por 3 caminhos independentes (DNS local, MCP do banco, navegador do champion) mostrou indisponibilidade do serviço Supabase. Um só caminho falhando seria inconclusivo; três convergindo isola a causa.
- Evidência: `06_notas/debug/debug-2026-10-15-f1-003-supabase-indisponivel.md` (15/10/2026).
- Regra reutilizável: quando um teste humano devolver erro de conexão (Failed to fetch / timeout / 000), provar por ao menos 2 caminhos independentes (ex.: curl direto + MCP + navegador) se o SERVIÇO está fora antes de mexer em código; erro de rede nunca deve ser exibido cru ao usuário — traduzir para mensagem própria.
- Quando aplicar: qualquer falha relatada com erro de rede em qualquer task da fase.
- Quando não aplicar: quando o erro é de negócio (credencial, RLS, estado) e não de transporte.
- Confiança: alta — convergência de três caminhos independentes documentada.
- Privacidade: sem segredo, dado pessoal ou conteúdo bruto.
