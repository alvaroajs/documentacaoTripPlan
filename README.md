#  Trip Plan
**Análise de requisitos · app Integração com o App de Assinatura**

## Objetivo

Permitir que clientes e atendentes simulem viagens longas com carros elétricos e recebam um plano com paradas de recarga, carga ao chegar e sair de cada parada, tempo de recarga e horário de chegada.

**Fora do escopo:** navegação própria, reserva e pagamento de recarga, disponibilidade em tempo real.

## Requisitos funcionais

| ID | Requisito | Fase |
|---|---|:---:|
| RF01 | Informar origem e destino da viagem | 1 |
| RF02 | Calcular rota e consumo considerando relevo, velocidade e temperatura
| RF03 | Definir paradas com carga de chegada, carga de saída e tempo de recarga 
| RF04 | Indicar um eletroposto alternativo (plano B) para cada parada 
| RF05 | Informar quando não há plano viável e em qual trecho
| RF06 | Modo agência: simular com qualquer modelo da frota, sem login do cliente 
| RF07 | Abrir a rota no Google Maps ou no Waze 
| RF08 | Mostrar quanto da franquia de km a viagem consome 
| RF09 | Ler a carga da bateria pela telemetria no app de assinatura 
| RF10 | Recalcular o plano durante a viagem com a carga real 

## Regras de negócio

- Chegar a cada parada e ao destino com no mínimo **15%** de carga.
- Carregar só o necessário até a próxima parada, **limitado a 80%**.
- Usar apenas eletropostos compatíveis, com **40 kW** ou mais e a até **5 km** da rota.
- Aplicar **10%** de margem de segurança sobre o consumo estimado.
- No app, usar a carga lida pela telemetria; no modo agência, considerar carga inicial de **100%**.

## Requisitos não funcionais

- Gerar o plano em até **5 s**.
- Atualizar a base de eletropostos **diariamente**.
- Seguir a **LGPD**, com consentimento do cliente para uso dos dados de telemetria.
- Continuar funcionando se um provedor externo falhar.

## Integrações

| Dado | Fonte | Custo |
|---|---|---|
| Rota e altitude | [OpenRouteService](https://openrouteservice.org) | Grátis (2.000 rotas/dia) |
| Eletropostos | [Open Charge Map](https://openchargemap.org) + OpenStreetMap | Grátis |
| Clima | [Open-Meteo](https://open-meteo.com) | Grátis (uso não comercial) |
| Ficha dos veículos | [Open EV Data](https://github.com/KilowattApp/open-ev-data) | Grátis |
| Carga da bateria (telemetria) e contrato | Sistemas Localiza | Interno |

## Arquitetura

```mermaid
flowchart LR
    APP["App / modo agência"] --> BFF["API Gateway"]
    BFF --> TP["Trip Planner Service"]
    TP --> RT["Rotas"]
    TP --> DB[("Eletropostos")]
    TP --> WX["Clima"]
    TP --> SOC["Telemetria e contrato"]
    ING["Ingestão diária"] --> DB
```

## Principal risco

As bases abertas de eletropostos podem estar desatualizadas e não informam se o carregador está livre. Mitigação: plano B em toda parada, reserva mínima de 15% e ingestão diária dos dados.

