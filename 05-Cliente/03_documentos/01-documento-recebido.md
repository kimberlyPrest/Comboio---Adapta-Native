# Leitura do documento de escopo recebido

O DOCX de 26/09/2026 descreve um sistema amplo de estacionamento, cadastro, cobrança, triagem, portal do cliente, chamadas, notificações, bonificação por abastecimento e integração futura com sistemas do posto. É uma fonte de requisitos **propostos**, não um contrato técnico pronto para execução integral.

## Aproveitado no escopo atual

- Separação entre estacionamento e triagem.
- Cadastro de empresa, motorista e veículo; permissões por perfil.
- Visão de ocupação por categoria sem vaga física numerada.
- Histórico de entrada/saída e tratamento de exceções.
- Na evolução: checklist por contratante, filas de carga/descarga, chamada e visão do cliente.

## Deixado para fases futuras ou fora do recorte

- Reconhecimento facial, OCR de placa, cancela e notificações automáticas: dependem de hardware, segurança e validação; facial previsto via AWS após fase 1.
- Cobrança automática, bonificação por combustível, contratos a prazo, CRM/PDV e campanhas: não integram o início do sistema de cadastro/previsão.
- Emissão de NF-e e conciliação: fora deste escopo inicial.

## Divergências a resolver

O documento descreve primeira estadia “acima de 2h até 24h”, mas sua fórmula `teto((permanência − 2h) ÷ 24h)` só mudaria de faixa após 26h. Também distribui cadastro/liberação entre caixa, comercial e logística de modo diferente do relato da reunião. Esses pontos constam como decisões pendentes; não foram implementados por suposição.
