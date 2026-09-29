# Visão da solução

O sistema terá três experiências conectadas pela mesma base de dados:

1. **Empresa:** solicita cadastro, mantém seus vínculos e registra previsões de chegada de motoristas/veículos autorizados. Na evolução da triagem, acompanha somente as próprias operações.
2. **Motorista:** cadastra-se, mantém dados e veículos permitidos, informa previsão de chegada pelo celular e acompanha o status. Uma previsão não garante vaga ou entrada automática.
3. **Equipe interna:** aprova cadastros, vê a agenda, confirma solicitações, registra quem chega espontaneamente, faz check-in/out e trata exceções com trilha de auditoria.

As fases seguintes acrescentam checklists e chamada de carga/descarga, identificação facial via AWS e integração homologada com equipamentos de acesso. Cada capacidade entra quando suas dependências operacionais, de segurança e de infraestrutura forem confirmadas. O [roadmap](05-Roadmap.md) mostra as fronteiras de cada fase.
