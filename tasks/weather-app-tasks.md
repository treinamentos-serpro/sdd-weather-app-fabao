# Weather App — Backlog de tarefas

Backlog derivado de [weather-app-plan.md](../plans/weather-app-plan.md) e
alinhado aos requisitos de [weather-app-spec.md](../specs/weather-app-spec.md).
As tarefas seguem a ordem de implementação: tipos, funções puras, serviços,
hook, componentes, integração, testes e hardening. As dependências explícitas
indicam os pré-requisitos de cada tarefa. Cada tarefa delimita uma
responsabilidade e, sempre que possível, um ou dois arquivos relevantes;
implementação e testes ficam em tarefas separadas.

## Rastreabilidade com a especificação

A spec identifica os requisitos como RF1–RF5 e RNF1–RNF8, mas não numera
individualmente os critérios de aceite dentro de cada requisito. A matriz
associa cada tarefa aos requisitos cujos comportamentos seus critérios cobrem;
os critérios detalhados permanecem descritos junto de cada tarefa. T-45 deriva
da estratégia de validação do plano e não corresponde a um requisito de produto
isolado.

| Tarefa | Requisitos relacionados | Prioridade | Tamanho |
| --- | --- | --- | --- |
| T-01 | RF1–RF5, RNF5 | P0 | M |
| T-02 | RF1–RF3 | P0 | P |
| T-03 | RF2–RF3 | P0 | P |
| T-04 | RF3, RNF5 | P0 | M |
| T-05 | RF4 | P0 | P |
| T-06 | RF4, RNF8 | P0 | P |
| T-07 | RF1–RF3 | P0 | P |
| T-08 | RF2–RF3 | P0 | P |
| T-09 | RF3, RNF5 | P0 | M |
| T-10 | RF4 | P0 | P |
| T-11 | RF4, RNF8 | P0 | P |
| T-12 | RF5, RNF4–RNF5 | P0 | M |
| T-13 | RF1, RF5 | P0 | M |
| T-14 | RF2–RF3, RF5, RNF5 | P0 | G |
| T-15 | RF5, RNF4–RNF5 | P0 | M |
| T-16 | RF1, RF5 | P0 | M |
| T-17 | RF2–RF3, RF5, RNF5 | P0 | G |
| T-18 | RF1, RF5 | P0 | G |
| T-19 | RF2–RF3, RF5 | P0 | G |
| T-20 | RF4 | P0 | M |
| T-21 | RF1, RF5 | P0 | M |
| T-22 | RF2–RF3, RF5 | P0 | M |
| T-23 | RF4 | P0 | P |
| T-24 | RF1–RF5, RNF3 | P0 | G |
| T-25 | RNF1–RNF2, RNF6–RNF8 | P0 | M |
| T-26 | RF1, RNF3 | P0 | P |
| T-27 | RF1, RNF2–RNF3 | P0 | M |
| T-28 | RF2, RNF2, RNF8 | P0 | P |
| T-29 | RF3–RF4, RNF6, RNF8 | P0 | P |
| T-30 | RF3, RNF2, RNF6 | P0 | P |
| T-31 | RF4, RNF3 | P1 | P |
| T-32 | RF5, RNF3–RNF4 | P0 | P |
| T-33 | RF1–RF3, RF5, RNF3, RNF5 | P0 | M |
| T-34 | RF5, RNF3 | P0 | P |
| T-35 | RF1, RNF3 | P0 | P |
| T-36 | RF1, RNF2–RNF3 | P0 | P |
| T-37 | RF2, RNF3 | P0 | P |
| T-38 | RF3–RF4, RNF6, RNF8 | P0 | M |
| T-39 | RF4, RNF3 | P1 | P |
| T-40 | RF5, RNF3–RNF4 | P0 | P |
| T-41 | RF1–RF3, RF5, RNF3 | P0 | M |
| T-42 | RF1–RF3 | P0 | G |
| T-43 | RF5, RNF4–RNF5 | P2 | M |
| T-44 | RF4, RNF1, RNF3, RNF6, RNF8 | P2 | G |
| T-45 | Plano técnico — estratégia de testes e critério de conclusão | P1 | G |

**Escala:** P0 reúne o mínimo necessário para entregar o fluxo principal
utilizável; P1 completa requisitos e validações importantes antes de considerar
a entrega pronta; P2 são verificações adicionais que podem vir depois do fluxo
principal, mas continuam no escopo. P, M e G indicam esforço relativo pequeno,
médio e grande, respectivamente — não são estimativas em horas. G sinaliza
trabalho com várias camadas, estados concorrentes ou integração ampla.

### Sequência sugerida em fatias verticais

| Ordem | Fatia e resultado visível | Tarefas |
| --- | --- | --- |
| 1 | **Busca utilizável:** tela responsiva, busca de cidade, seleção explícita, feedback de loading/vazio/erro e retry. | T-01, T-02, T-12, T-13, T-18, T-25, T-26, T-27, T-32, T-33, T-34, T-07, T-15, T-16, T-21, T-35, T-36, T-40, T-41, T-24 |
| 2 | **Clima completo:** ao selecionar uma cidade, mostrar condições atuais e cinco dias locais; validar datas e respostas parciais. | T-03, T-04, T-14, T-19, T-20, T-28, T-29, T-30, T-05, T-06, T-08, T-09, T-10, T-11, T-17, T-22, T-23, T-37, T-38, T-42 |
| 3 | **Unidade preferida:** habilitar a alternância Celsius/Fahrenheit sem nova consulta. | T-31, T-39 |
| 4 | **Confiança para release:** cobrir recuperação/concorrência no navegador, experiência mobile/teclado e rodar a validação completa. | T-43, T-44, T-45 |

Para antecipar o resultado da primeira fatia, executar T-24 como integração
incremental, conectando cada componente assim que estiver pronto. Como está
redigida, a lista de dependências de T-24 inclui componentes das fatias
posteriores e pode bloquear a primeira tela; revisar essa dependência antes da
execução, sem adiar a composição inicial até a conclusão de todo o backlog.

### Requisitos funcionais → tarefas

| Requisito | Tarefas de implementação | Tarefas de teste | Cobertura |
| --- | --- | --- | --- |
| RF1 — Buscar e selecionar uma cidade | T-01, T-02, T-13, T-18, T-24, T-26, T-27, T-33 | T-07, T-16, T-21, T-35, T-36, T-41, T-42 | Coberto |
| RF2 — Consultar as condições atuais | T-01, T-02, T-03, T-14, T-19, T-24, T-28, T-33 | T-08, T-17, T-22, T-37, T-41, T-42 | Coberto |
| RF3 — Consultar a previsão de cinco dias | T-01, T-02, T-03, T-04, T-14, T-19, T-24, T-29, T-30, T-33 | T-08, T-09, T-17, T-22, T-38, T-41, T-42 | Coberto |
| RF4 — Alterar a unidade de temperatura | T-01, T-05, T-06, T-20, T-24, T-29, T-31 | T-10, T-11, T-23, T-38, T-39, T-44 | Coberto |
| RF5 — Comunicar estados da consulta | T-01, T-12, T-13, T-14, T-18, T-19, T-24, T-32, T-33, T-34 | T-15, T-16, T-17, T-21, T-22, T-40, T-41, T-43 | Coberto |

Todos os requisitos funcionais RF1–RF5 têm tarefas de implementação e de
teste correspondentes; não há requisito funcional sem tarefa no backlog.

## Fase 1 — Tipos
### T-01 — Definir os contratos de domínio

- **Descrição:** Criar os tipos compartilhados para cidades, condições atuais,
  previsão diária, dados meteorológicos, unidade, valores indisponíveis e
  estados/erros de operação.
- **Critérios de aceite:**
  - Existem `City`, `CurrentWeather`, `ForecastDay`, `WeatherData`, `Unit` e
    `WeatherValue<T>` conforme `plans/weather-app-types.md`.
  - Os estados distinguem `idle`, `loading`, `success`, `empty` e `error`; os
    erros distinguem timeout, rede, HTTP, API e resposta inválida.
  - Os contratos não dependem de React, serviços ou componentes, aceitam
    campos geográficos opcionais e representam dados meteorológicos
    indisponíveis.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/types/weather.ts`, `src/types/operation.ts`.
- **Tipo:** Data.


## Fase 2 — Funções puras
### T-02 — Normalizar termo de busca e valores meteorológicos

- **Descrição:** Criar funções puras para aparar o termo de busca e normalizar
  valores meteorológicos disponíveis ou indisponíveis.
- **Critérios de aceite:**
  - A busca aplica apenas `trim()`, preservando acentos e demais caracteres.
  - Valores ausentes, nulos ou não finitos são representados como
    indisponíveis, nunca como zero ou estimativas.
  - As funções não acessam React, serviços ou rede.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/lib/normalize.ts`.
- **Tipo:** Data.

### T-03 — Mapear códigos WMO

- **Descrição:** Criar a função pura que converte códigos meteorológicos WMO
  reconhecidos em descrições em pt-BR.
- **Critérios de aceite:**
  - Códigos suportados resultam em descrições legíveis em pt-BR.
  - Códigos desconhecidos ou indisponíveis resultam em condição indisponível,
    sem descrição inventada.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/lib/weather-codes.ts`.
- **Tipo:** Data.

### T-04 — Validar fuso e alinhar datas da previsão

- **Descrição:** Criar funções puras para validar o fuso IANA, determinar as
  cinco datas esperadas e alinhar as métricas diárias por data.
- **Critérios de aceite:**
  - A sequência contém hoje e os quatro dias seguintes no fuso informado, em
    ordem e sem duplicatas, sem depender do fuso do navegador.
  - Fuso ausente/inválido e datas impossíveis, duplicadas ou ininterpretáveis
    são rejeitados.
  - Um dia esperado sem métricas permanece na sequência, com os campos
    meteorológicos indisponíveis.
- **Dependências:** T-01, T-02.
- **Arquivos prováveis:** `src/lib/forecast-dates.ts`.
- **Tipo:** Data.

### T-05 — Converter temperaturas

- **Descrição:** Criar funções puras de conversão entre Celsius e Fahrenheit.
- **Critérios de aceite:**
  - A conversão cobre valores negativos e fracionários e limites como
    `0 °C → 32 °F`, `100 °C → 212 °F` e `-40 °C → -40 °F`.
  - Valores indisponíveis permanecem indisponíveis.
  - A conversão sempre parte do valor Celsius original, sem acumular
    conversões.
- **Dependências:** T-01, T-02.
- **Arquivos prováveis:** `src/lib/temperature.ts`.
- **Tipo:** Data.

### T-06 — Formatar temperaturas e unidades

- **Descrição:** Criar funções puras para formatar temperaturas e rótulos de
  unidade para exibição em pt-BR.
- **Critérios de aceite:**
  - Valores disponíveis são arredondados a uma casa decimal e exibem unidade
    identificável.
  - Um valor arredondado a zero não é apresentado com sinal negativo.
  - Valores indisponíveis não são formatados como números.
- **Dependências:** T-01, T-05.
- **Arquivos prováveis:** `src/lib/format.ts`.
- **Tipo:** Data.


## Fase 3 — Serviços
### T-12 — Criar transporte HTTP com timeout e erros tipados

- **Descrição:** Implementar o transporte compartilhado dos serviços com
  timeout, cancelamento e classificação explícita de falhas.
- **Critérios de aceite:**
  - Cada chamada tem deadline de 10 segundos e usa `AbortController`.
  - Falhas de rede, HTTP, erro explícito da API e JSON inválido são
    distinguíveis; cancelamento por substituição não é apresentado como
    timeout.
  - Timers e listeners são liberados em sucesso, erro e cancelamento.
  - Respostas concluídas após o deadline não são tratadas como sucesso.
- **Dependências:** T-01.
- **Arquivos prováveis:** `src/services/http.ts`, `src/types/operation.ts`.
- **Tipo:** Data.

### T-13 — Implementar o serviço de geocodificação

- **Descrição:** Consultar a API de geocodificação e converter as respostas
  para opções do domínio, sem selecionar uma cidade automaticamente.
- **Critérios de aceite:**
  - A URL usa `URLSearchParams` com termo aparado, `count=10`, `language=pt`
    e `format=json`.
  - Cada correspondência vira `City`, com `id` string, coordenadas e somente
    os detalhes geográficos opcionais retornados.
  - Resultados ausentes ou vazios representam busca sem correspondências;
    respostas inválidas e falhas não são mascaradas como resultado vazio.
- **Dependências:** T-01, T-02, T-12.
- **Arquivos prováveis:** `src/services/geocoding.ts`.
- **Tipo:** Data.

### T-14 — Implementar o serviço de previsão meteorológica

- **Descrição:** Consultar clima atual e previsão combinados e normalizar o
  resultado para `WeatherData`.
- **Critérios de aceite:**
  - A chamada usa as coordenadas da cidade, solicita Celsius,
    `timezone=auto`, clima atual, variáveis diárias e cinco dias.
  - A cidade selecionada é mantida em `WeatherData`; o fuso retornado e as
    datas são validados antes de disponibilizar a previsão.
  - São produzidos cinco dias no fuso da cidade, com arrays alinhados por
    data e valores ausentes marcados como indisponíveis.
  - Respostas parciais preservam campos interpretáveis. Sem qualquer valor
    meteorológico exibível, a resposta é erro.
  - Se não houver campos atuais exibíveis, uma previsão válida continua
    disponível como sucesso parcial para a mesma cidade.
- **Dependências:** T-01, T-02, T-03, T-04, T-12.
- **Arquivos prováveis:** `src/services/weather.ts`.
- **Tipo:** Data.


## Fase 4 — Hook
### T-18 — Orquestrar busca e retry no hook

- **Descrição:** Implementar no hook o ciclo da busca de cidades, incluindo
  texto editável, termo submetido, concorrência e retry.
- **Critérios de aceite:**
  - O texto editável é separado do termo submetido; busca vazia não chama o
    serviço e informa a validação.
  - A busca distingue carregamento, vazio, sucesso e erro; retry usa o termo
    submetido mesmo que o campo tenha sido editado.
  - Reenvio do mesmo termo normalizado durante loading não cria chamada nem
    troca o identificador vigente; outro termo pode substituir a busca ativa.
  - Respostas obsoletas são ignoradas e resultados não selecionam cidade
    automaticamente.
- **Dependências:** T-01, T-13.
- **Arquivos prováveis:** `src/hooks/useWeatherApp.ts`.
- **Tipo:** Data.

### T-19 — Orquestrar seleção de cidade e consulta meteorológica

- **Descrição:** Integrar seleção explícita de cidade, consulta combinada,
  estados meteorológicos, concorrência e retry da cidade selecionada.
- **Critérios de aceite:**
  - Selecionar uma cidade inicia a consulta; carregamentos de busca e clima
    são independentes.
  - Somente a operação vigente atualiza o estado; dados de cidade anterior
    não aparecem associados à nova seleção ou a uma consulta que falhou.
  - Retry repete a consulta completa da cidade selecionada, não cria
    requisições separadas e fica indisponível durante loading.
  - Um resultado parcial mantém os dados válidos; condições atuais sem
    valores exibíveis não descartam previsão válida da mesma resposta.
- **Dependências:** T-14, T-18.
- **Arquivos prováveis:** `src/hooks/useWeatherApp.ts`.
- **Tipo:** Data.

### T-20 — Manter e derivar unidade no hook

- **Descrição:** Manter a unidade escolhida no estado local e expor a
  apresentação das temperaturas sem alterar os dados originais.
- **Critérios de aceite:**
  - A unidade inicial é Celsius; alternar afeta temperaturas atuais, mínimas
    e máximas já exibidas ou recebidas posteriormente.
  - Alterar unidade não dispara consulta, reinicia loading ou armazena cópia
    Fahrenheit.
  - Valores indisponíveis permanecem assim em ambas as unidades.
- **Dependências:** T-05, T-06, T-19.
- **Arquivos prováveis:** `src/hooks/useWeatherApp.ts`.
- **Tipo:** Data.


## Fase 5 — Componentes
### T-25 — Definir a base visual responsiva

- **Descrição:** Configurar tema dark glassmorphism, estilos globais e
  estrutura de layout responsiva.
- **Critérios de aceite:**
  - A interface usa o tema do projeto e se adapta a desktop e viewport estreita.
  - A estrutura inicial não causa rolagem horizontal.
  - Não adiciona autenticação, geolocalização automática, persistência,
    favoritos ou histórico.
- **Dependências:** Nenhuma.
- **Arquivos prováveis:** `src/index.css`, `src/App.tsx`.
- **Tipo:** UI.

### T-26 — Criar formulário acessível de busca

- **Descrição:** Apresentar entrada de cidade e ação de busca com validação
  acessível.
- **Critérios de aceite:**
  - Campo e ação têm nomes acessíveis e funcionam por teclado, incluindo Enter.
  - Busca em branco informa que é necessário digitar uma cidade e não chama
    geocodificação.
  - O texto de busca e a ação ficam disponíveis para nova tentativa ou
    correção.
- **Dependências:** T-18, T-25.
- **Arquivos prováveis:** `src/components/CitySearch.tsx`.
- **Tipo:** UI.

### T-27 — Criar lista de resultados de cidade

- **Descrição:** Exibir correspondências de geocodificação como opções para
  seleção explícita.
- **Critérios de aceite:**
  - Cada resultado é selecionável e apresenta nome e detalhes devolvidos,
    incluindo região e país quando disponíveis.
  - Detalhes ausentes não são inventados e nenhuma opção é selecionada
    automaticamente.
  - Opções homônimas podem ser distinguidas pelos detalhes disponíveis.
- **Dependências:** T-13, T-18, T-25.
- **Arquivos prováveis:** `src/components/CityResults.tsx`.
- **Tipo:** UI.

### T-28 — Exibir condições meteorológicas atuais

- **Descrição:** Apresentar cidade selecionada, temperatura e condição atual,
  incluindo campos indisponíveis.
- **Critérios de aceite:**
  - Exibe cidade, temperatura com unidade e descrição legível quando
    disponíveis.
  - Campos indisponíveis são identificados, sem estimativa ou substituição
    por zero.
  - A seção não associa dados a uma cidade diferente da selecionada.
- **Dependências:** T-19, T-20, T-25.
- **Arquivos prováveis:** `src/components/CurrentWeather.tsx`.
- **Tipo:** UI.

### T-29 — Criar apresentação de um dia da previsão

- **Descrição:** Apresentar os dados de um único dia da previsão diária.
- **Critérios de aceite:**
  - Exibe data, condição e temperaturas mínima/máxima acompanhadas da unidade.
  - Data e números seguem `pt-BR`; métricas ou condições ausentes são
    identificadas como indisponíveis.
  - O dia permanece visível quando uma ou mais métricas estão ausentes.
- **Dependências:** T-04, T-06, T-20, T-25.
- **Arquivos prováveis:** `src/components/ForecastDay.tsx`.
- **Tipo:** UI.

### T-30 — Exibir a lista da previsão de cinco dias

- **Descrição:** Apresentar a sequência de dias normalizada para a cidade
  consultada.
- **Critérios de aceite:**
  - Exibe exatamente cinco dias em ordem cronológica.
  - Cada dia é apresentado pelo componente de dia e mantém seus dados
    associados à cidade consultada.
- **Dependências:** T-14, T-19, T-29.
- **Arquivos prováveis:** `src/components/Forecast.tsx`.
- **Tipo:** UI.

### T-31 — Criar controle acessível de unidade

- **Descrição:** Permitir alternar entre Celsius e Fahrenheit na interface.
- **Critérios de aceite:**
  - As opções têm rótulos acessíveis e estado selecionado perceptível.
  - A interação funciona por teclado e comunica a unidade atual.
  - O controle chama apenas a ação de alteração de unidade do hook.
- **Dependências:** T-20, T-25.
- **Arquivos prováveis:** `src/components/UnitToggle.tsx`.
- **Tipo:** UI.

### T-32 — Apresentar estado de carregamento

- **Descrição:** Exibir feedback acessível para operações de busca e consulta
  em andamento.
- **Critérios de aceite:**
  - Busca e consulta exibem carregamento próprio e imediato.
  - O estado usa semântica apropriada a tecnologias assistivas e não confunde
    operações distintas.
- **Dependências:** T-18, T-19, T-25.
- **Arquivos prováveis:** `src/components/LoadingStatus.tsx`.
- **Tipo:** UI.

### T-33 — Apresentar estados vazio e de erro

- **Descrição:** Exibir mensagens próprias para busca sem resultados e
  operações com falha.
- **Critérios de aceite:**
  - Busca sem resultados preserva o termo e permite pesquisar novamente.
  - Erros comunicam qual operação falhou; timeout é informado como demora
    excessiva.
  - A apresentação não mostra dados antigos como atuais após falha ou troca
    de cidade.
- **Dependências:** T-18, T-19, T-25.
- **Arquivos prováveis:** `src/components/OperationStatus.tsx`.
- **Tipo:** UI.

### T-34 — Criar ação de retry acessível

- **Descrição:** Expor a ação de nova tentativa para a operação recuperável
  vigente.
- **Critérios de aceite:**
  - O controle tem nome acessível, funciona por teclado e aciona o retry da
    operação correta.
  - Fica indisponível durante loading e não gera chamadas duplicadas.
- **Dependências:** T-18, T-19, T-25.
- **Arquivos prováveis:** `src/components/RetryButton.tsx`, `src/App.tsx`.
- **Tipo:** UI.


## Fase 6 — Integração
### T-24 — Compor e inicializar a aplicação

- **Descrição:** Conectar o hook à aplicação e configurar a inicialização da
  árvore React.
- **Critérios de aceite:**
  - `App.tsx` compõe os fluxos e passa dados/ações por props.
  - Componentes não acessam `fetch` nem DTOs da API.
  - `main.tsx` inicializa a aplicação sem adicionar funcionalidades fora do
    escopo.
- **Dependências:** T-18, T-19, T-20, T-25, T-26, T-27, T-28, T-29, T-30,
  T-31, T-32, T-33, T-34.
- **Arquivos prováveis:** `src/App.tsx`, `src/main.tsx`.
- **Tipo:** UI.


## Fase 7 — Testes
### T-07 — Testar normalização

- **Descrição:** Criar testes unitários para a normalização do termo de busca e
  dos valores meteorológicos.
- **Critérios de aceite:**
  - Cobrem texto com espaços externos, entrada vazia, acentos e preservação
    dos demais caracteres.
  - Cobrem valores válidos, ausentes, nulos e não finitos.
  - Passam isoladamente com Vitest, sem mocks de React ou rede.
- **Dependências:** T-02.
- **Arquivos prováveis:** `tests/unit/normalize.test.ts`.
- **Tipo:** Test.

### T-08 — Testar códigos WMO

- **Descrição:** Criar testes unitários do mapeamento de códigos WMO.
- **Critérios de aceite:**
  - Cobrem códigos reconhecidos, código desconhecido e valor indisponível.
  - Confirmam descrições em pt-BR e ausência de descrições inventadas.
- **Dependências:** T-03.
- **Arquivos prováveis:** `tests/unit/weather-codes.test.ts`.
- **Tipo:** Test.

### T-09 — Testar datas e fuso da previsão

- **Descrição:** Criar testes unitários de validação de fuso e alinhamento de
  datas da previsão.
- **Critérios de aceite:**
  - Cobrem fuso diferente do navegador, virada de dia e cinco datas
    consecutivas.
  - Cobrem fuso inválido, datas inválidas ou duplicadas e métricas ausentes
    para uma data esperada.
  - Passam isoladamente com Vitest.
- **Dependências:** T-04.
- **Arquivos prováveis:** `tests/unit/forecast-dates.test.ts`.
- **Tipo:** Test.

### T-10 — Testar conversão de temperatura

- **Descrição:** Criar testes unitários dedicados das funções puras de
  conversão entre Celsius e Fahrenheit.
- **Critérios de aceite:**
  - Cobrem conversões nos dois sentidos, valores negativos, fracionários e
    valores indisponíveis.
  - Incluem os limites `0 °C`, `100 °C` e `-40 °C`.
  - Confirmam a precisão das conversões sem arredondamentos acumulados ao
    alternar entre Celsius e Fahrenheit.
  - Passam isoladamente com Vitest, sem mocks de React ou rede.
- **Dependências:** T-05.
- **Arquivos prováveis:** `tests/unit/temperature.test.ts`.
- **Tipo:** Test.

### T-11 — Testar formatação de temperaturas

- **Descrição:** Criar testes unitários da formatação de valores e unidades.
- **Critérios de aceite:**
  - Cobrem arredondamento a uma casa decimal, rótulos de unidade e valor
    arredondado a zero sem sinal negativo.
  - Confirmam que valores indisponíveis não são convertidos em números.
  - Passam isoladamente com Vitest.
- **Dependências:** T-06.
- **Arquivos prováveis:** `tests/unit/format.test.ts`.
- **Tipo:** Test.

### T-15 — Testar transporte HTTP

- **Descrição:** Criar testes do transporte HTTP com `fetch` simulado e
  temporizadores controlados.
- **Critérios de aceite:**
  - Cobrem sucesso, falhas de rede/HTTP/API/JSON, cancelamento e timeout aos
    10 segundos.
  - Verificam que uma resposta tardia não resulta em sucesso e que timers e
    listeners são liberados nos caminhos de término.
- **Dependências:** T-12.
- **Arquivos prováveis:** `tests/unit/http.test.ts`.
- **Tipo:** Test.

### T-16 — Testar geocodificação

- **Descrição:** Criar testes do serviço de geocodificação com `fetch` mockado
  e respostas controladas.
- **Critérios de aceite:**
  - Mockam `fetch` diretamente e verificam endpoint, parâmetros, mapeamento de
    cidade e preservação de campos geográficos opcionais.
  - Cobrem resultados vazios e respostas inválidas sem chamadas reais à API.
- **Dependências:** T-13, T-15.
- **Arquivos prováveis:** `tests/unit/geocoding.test.ts`.
- **Tipo:** Test.

### T-17 — Testar serviço meteorológico

- **Descrição:** Criar testes do serviço meteorológico com `fetch` mockado e
  respostas controladas.
- **Critérios de aceite:**
  - Mockam `fetch` diretamente e verificam parâmetros da chamada,
    normalização e alinhamento por data.
  - Cobrem respostas parciais, fuso/datas inválidos, condições atuais sem
    valores com previsão válida e ausência total de valores meteorológicos.
  - Cobrem falhas HTTP/API/JSON e timeout por meio do transporte testado,
    sem repetir os testes de implementação de timeout e sem chamadas reais à
    API.
- **Dependências:** T-14, T-15.
- **Arquivos prováveis:** `tests/unit/weather.test.ts`.
- **Tipo:** Test.

### T-21 — Testar busca e concorrência do hook

- **Descrição:** Testar o ciclo de busca do hook com serviço de geocodificação
  simulado.
- **Critérios de aceite:**
  - Cobrem estados da busca, entrada vazia e retry do termo submetido.
  - Verificam reenvio duplicado, substituição por termo novo e descarte de
    respostas tardias.
  - Confirmam que resultados não selecionam cidade automaticamente.
- **Dependências:** T-18.
- **Arquivos prováveis:** `tests/unit/useWeatherSearch.test.ts`.
- **Tipo:** Test.

### T-22 — Testar seleção e consulta meteorológica do hook

- **Descrição:** Testar a consulta meteorológica e sua concorrência com
  serviços simulados.
- **Critérios de aceite:**
  - Cobrem seleção explícita, loading, sucesso, erro e retry da cidade correta.
  - Verificam seleção de outra cidade, descarte de respostas obsoletas e
    ausência de dados antigos após falha.
  - Cobrem resultado parcial com previsão válida e campos atuais indisponíveis.
- **Dependências:** T-19.
- **Arquivos prováveis:** `tests/unit/useWeatherQuery.test.ts`.
- **Tipo:** Test.

### T-23 — Testar unidade no hook

- **Descrição:** Testar a unidade selecionada e sua interação com consultas
  em andamento.
- **Critérios de aceite:**
  - Verificam unidade inicial Celsius e alternância de todas as temperaturas
    disponíveis.
  - A alternância durante e após consulta não dispara chamada meteorológica
    adicional nem altera valores indisponíveis.
- **Dependências:** T-20.
- **Arquivos prováveis:** `tests/unit/useWeatherUnit.test.ts`.
- **Tipo:** Test.

### T-35 — Testar formulário de busca

- **Descrição:** Testar o comportamento observável do formulário de busca com
  Testing Library e `user-event`.
- **Critérios de aceite:**
  - Cobrem entrada, envio por botão e Enter e validação de entrada vazia.
  - Usam roles, labels e texto visível, sem inspecionar detalhes internos do
    hook.
- **Dependências:** T-26.
- **Arquivos prováveis:** `tests/components/CitySearch.test.tsx`.
- **Tipo:** Test.

### T-36 — Testar resultados e seleção de cidade

- **Descrição:** Testar a apresentação e seleção dos resultados de cidade.
- **Critérios de aceite:**
  - Cobrem seleção, cidades homônimas e detalhes geográficos presentes ou
    ausentes.
  - Confirmam que nenhuma cidade é selecionada automaticamente.
- **Dependências:** T-27.
- **Arquivos prováveis:** `tests/components/CityResults.test.tsx`.
- **Tipo:** Test.

### T-37 — Testar condições atuais

- **Descrição:** Testar a apresentação de condições atuais completas e
  parciais.
- **Critérios de aceite:**
  - Cobrem campos disponíveis e indisponíveis, unidade e cidade correspondente.
  - Não dependem de detalhes internos de hooks ou serviços.
- **Dependências:** T-28.
- **Arquivos prováveis:** `tests/components/CurrentWeather.test.tsx`.
- **Tipo:** Test.

### T-38 — Testar previsão diária

- **Descrição:** Testar a apresentação do componente de dia e da lista de
  previsão.
- **Critérios de aceite:**
  - Verificam cinco dias em ordem, data e temperaturas com unidade.
  - Cobrem condições e métricas indisponíveis sem remover o dia.
- **Dependências:** T-29, T-30.
- **Arquivos prováveis:** `tests/components/Forecast.test.tsx`.
- **Tipo:** Test.

### T-39 — Testar alternância de unidade

- **Descrição:** Testar controle e comportamento observável da seleção de
  unidade.
- **Critérios de aceite:**
  - Cobrem alternância por teclado, unidade selecionada e rótulos acessíveis.
  - Confirmam que a ação de unidade é acionada sem submissão de busca ou
    consulta.
- **Dependências:** T-31.
- **Arquivos prováveis:** `tests/components/UnitToggle.test.tsx`.
- **Tipo:** Test.

### T-40 — Testar estados de carregamento

- **Descrição:** Criar testes de componente dedicados aos estados de loading
  de busca e consulta meteorológica.
- **Critérios de aceite:**
  - Cobrem loading de busca e consulta meteorológica.
  - Verificam semântica acessível e distinção entre as operações.
- **Dependências:** T-32.
- **Arquivos prováveis:** `tests/components/LoadingStatus.test.tsx`.
- **Tipo:** Test.

### T-41 — Testar estados vazio, erro e retry

- **Descrição:** Criar testes de componente dedicados aos estados vazio e de
  erro das operações e à ação de nova tentativa.
- **Critérios de aceite:**
  - Cobrem estados vazio e erro de busca e consulta.
  - Verificam retry acessível e indisponível durante loading.
  - Usam interações e conteúdo visíveis, sem acoplamento a detalhes internos
    de hooks ou serviços.
- **Dependências:** T-33, T-34.
- **Arquivos prováveis:** `tests/components/OperationStatus.test.tsx`,
  `tests/components/RetryButton.test.tsx`.
- **Tipo:** Test.

### T-42 — Testar fluxo integrado principal

- **Descrição:** Cobrir no navegador busca, seleção explícita e apresentação
  de clima atual e previsão com respostas controladas da Open-Meteo.
- **Critérios de aceite:**
  - Intercepta a rede com `page.route`; cobre no Chromium desktop a busca,
    seleção explícita e apresentação do clima atual e dos cinco dias de
    previsão.
  - Verifica entrada vazia, resultados vazios e cidade homônima com detalhes
    opcionais.
  - Confirma que não há request meteorológico antes da seleção explícita.
- **Dependências:** T-16, T-17, T-21, T-22, T-24, T-35, T-36, T-37, T-38,
  T-40, T-41.
- **Arquivos prováveis:** `tests/e2e/weather.spec.ts`.
- **Tipo:** Test.

### T-43 — Testar recuperação e concorrência no navegador

- **Descrição:** Cobrir erros recuperáveis, timeout, retry e substituição de
  operações em andamento.
- **Critérios de aceite:**
  - Intercepta respostas de erro e tardias e verifica mensagem e retry.
  - Confirma que timeout encerra o loading e que respostas obsoletas não
    substituem o estado vigente.
  - Verifica que retry durante loading não duplica chamadas.
- **Dependências:** T-42.
- **Arquivos prováveis:** `tests/e2e/weather.spec.ts`.
- **Tipo:** Test.

### T-44 — Testar fluxo E2E mobile, unidade e teclado

- **Descrição:** Validar o fluxo principal E2E no viewport mobile, além da
  alternância de unidade, operação por teclado e responsividade.
- **Critérios de aceite:**
  - A alternância de unidade não gera chamada adicional à API.
  - Busca, seleção, mensagens e dados são operáveis por teclado.
  - No projeto mobile iPhone 13, completa por toque o fluxo principal de busca,
    seleção explícita e exibição do clima atual e da previsão de cinco dias.
  - O viewport mobile não apresenta rolagem horizontal, corte impeditivo ou
    sobreposição; os controles e dados permanecem visíveis ou alcançáveis por
    rolagem vertical.
- **Dependências:** T-23, T-31, T-39, T-42.
- **Arquivos prováveis:** `tests/e2e/weather.spec.ts`, `playwright.config.ts`.
- **Tipo:** Test.


## Fase 8 — Hardening
### T-45 — Validar lint, build e suítes do projeto

- **Descrição:** Executar os comandos de verificação definidos pelo projeto e
  corrigir falhas relacionadas às entregas implementadas.
- **Critérios de aceite:**
  - `pnpm lint`, `pnpm build` e `pnpm test` concluem sem falhas.
  - `pnpm test:e2e` conclui nos projetos Chromium e mobile configurados.
  - Falhas são corrigidas na tarefa responsável, sem desabilitar ou enfraquecer
    critérios de teste.
- **Dependências:** T-07, T-08, T-09, T-10, T-11, T-15, T-16, T-17, T-21,
  T-22, T-23, T-24, T-35, T-36, T-37, T-38, T-39, T-40, T-41, T-42, T-43, T-44.
- **Arquivos prováveis:** `src/`, `tests/`, `playwright.config.ts`,
  configurações existentes do projeto.
- **Tipo:** Infra.
