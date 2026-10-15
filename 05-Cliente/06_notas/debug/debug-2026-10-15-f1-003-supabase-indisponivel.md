# Debug Summary — F1-003 · 15/10/2026

**Task e problema:** teste humano da F1-003 (login da fixture desativada) devolveu "Failed to fetch" na tela.

**Reprodução:** o erro foi reproduzido fora do navegador do champion: DNS local não resolve mais `lwkqsukxefwpreqjdkzg.supabase.co` (apex `supabase.co` resolve; subdomínio do projeto não), curl direto retorna HTTP 000 e o MCP do Supabase retorna `Connection terminated due to connection timeout`. Três caminhos independentes falhando ao mesmo tempo.

**Causa raiz:** indisponibilidade do serviço Supabase (ou de rota até ele) no momento do teste — não é defeito do gate de perfil nem do build (preview servindo v0.0.5, textos do gate confirmados no bundle).

**Defeito paralelo achado e corrigido:** em falha de rede, as telas de login/cadastro/recuperação exibiam `error.message` cru em inglês ("Failed to fetch") ao usuário. Corrigido na v0.0.6: falhas de rede (failed to fetch / network / timeout) agora mostram mensagem própria em português ("Não foi possível conectar ao servidor. Verifique sua conexão..."). QA verde.

**Correção:** v0.0.6 aplicada (Index/SignUp/ForgotPassword); provas S103 seguem pendentes por indisponibilidade do serviço.

**Verificação automática:** parcial — QA do Skip verde (v0.0.5 e v0.0.6); provas S103 não executáveis enquanto o Supabase não responder; rede bloqueia o banco até o serviço voltar.

**Gate atual:** aguardando janela com o serviço de volta → novo teste humano + provas S103 → conclusão.
