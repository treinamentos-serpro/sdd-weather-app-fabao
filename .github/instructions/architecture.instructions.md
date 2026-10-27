---
description: "Use when creating or changing files in src/ and deciding where React presentation, application state, API access, domain types, or pure functions belong. Defines the Weather App layer boundaries and test seams."
applyTo: "src/**/*.{ts,tsx}"
---

# Instruções — Arquitetura em camadas

Organize `src/` para manter apresentação, orquestração, acesso a dados e regras
puras separados. Preserve a direção das dependências:

```text
components → hooks → services → Open-Meteo
     └──────────────→ lib
hooks ──────────────→ lib
services ──────────→ lib
```

`types/` contém contratos compartilhados pelas camadas; não deve depender delas.
Não introduza estado global, cache ou persistência sem requisito correspondente.

## Estrutura de referência

```text
src/
  components/       componentes de apresentação
  hooks/            estado local e orquestração dos fluxos
  services/         comunicação HTTP e adaptação das respostas externas
  lib/              regras puras, conversões e formatação reutilizáveis
  types/            tipos de domínio e estados de operação
  App.tsx           composição da página
  main.tsx          inicialização da aplicação
```

Crie arquivos ou subpastas conforme a necessidade real da implementação; evite
adicionar abstrações sem uso.

## Limites de responsabilidade

- **`components/`:** renderize dados recebidos por props e comunique ações por
  callbacks. Componentes não chamam `fetch`, não conhecem o formato bruto da
  Open-Meteo e não implementam regras de concorrência. Trate visualmente os
  estados de carregamento, vazio, erro e dados parciais, com semântica acessível.
- **`hooks/`:** coordene estado React local e operações da tela, como busca,
  seleção explícita de cidade, consulta meteorológica, unidade e retry. Use
  identificadores de operação e, quando apropriado, `AbortController` para
  impedir que respostas obsoletas atualizem a tela. Não replique nele a lógica
  de transporte ou normalização dos serviços.
- **`services/`:** encapsule `fetch`, construção de URLs, timeout,
  `AbortSignal` e tratamento de falhas de rede, HTTP e JSON. Converta respostas
  externas para os tipos de domínio, delegando validações e transformações
  puras a `lib/`. Não importe componentes nem hooks.
- **`lib/`:** mantenha funções sem efeitos colaterais, com entradas explícitas
  e resultados previsíveis. Coloque aqui regras como normalização de dados,
  mapeamento de códigos WMO, conversão de Celsius/Fahrenheit e formatação de
  datas ou números. Não acesse React, rede ou estado global.
- **`types/`:** declare contratos de domínio compartilhados, como `City`,
  `WeatherData`, `Unit` e estados de operação. A interface deve consumir os
  tipos normalizados, não os DTOs da API.

## Regras do domínio meteorológico

- Solicite temperaturas em Celsius e converta para Fahrenheit localmente, na
  apresentação dos valores; alternar unidade não deve disparar nova consulta.
- Represente valores meteorológicos ausentes ou inválidos como indisponíveis,
  em vez de substituí-los por zero ou estimativas.
- Mapeie códigos WMO conhecidos para descrições em pt-BR; não exponha códigos
  brutos como descrições ao usuário.
- Preserve os dados geográficos opcionais retornados pela geocodificação, sem
  inventar informações ausentes.
- Valide fuso e datas da previsão antes de disponibilizar os dados à UI; não
  descarte um dia válido apenas porque alguma métrica está indisponível.

## Testabilidade

- Teste funções de `lib/` diretamente, cobrindo valores válidos, limites e
  entradas inválidas, sem mocks de React ou rede.
- Teste `services/` isolando `fetch` e verificando conversão de respostas,
  erros e timeout, sem depender de chamadas reais à Open-Meteo.
- Teste hooks com serviços simulados para verificar transições de estado,
  retry, mudança de unidade e descarte de respostas obsoletas.
- Teste componentes por comportamento observável, acessibilidade e interação,
  sem acoplá-los à implementação interna dos hooks ou serviços.
