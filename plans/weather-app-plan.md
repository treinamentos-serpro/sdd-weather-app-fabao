# Weather App — Plano técnico

## Architecture

Aplicação web client-side, sem backend próprio. A interface React coordena os
fluxos de busca e consulta; serviços isolam as chamadas à Open-Meteo e convertem
as respostas externas para modelos de domínio. A interface não consome
diretamente o formato bruto da API.

```text
Componentes React
  Busca → seleção explícita de cidade → clima atual e previsão
          controles de unidade e estados acessíveis
                 ↕
Estado local da aplicação
                 ↕
Serviços de geocodificação e meteorologia
                 ↕ HTTPS, sem API key
Open-Meteo
```

Buscar uma cidade não a seleciona. A geocodificação apresenta opções com os
detalhes retornados; somente uma seleção explícita inicia a consulta
meteorológica. Isso evita escolher incorretamente cidades homônimas (**RF1**).

Os serviços validam e normalizam as respostas antes de disponibilizá-las à UI.
Campos meteorológicos incompletos são representados como indisponíveis; dados
inválidos ou impossíveis de interpretar geram erro recuperável. O estado local
identifica cada operação para impedir que respostas obsoletas substituam
resultados atuais (**RF2**, **RF3**, **RF5**, **RNF5**).

Não há autenticação, localização automática, persistência de servidor ou
histórico nesta versão (**RNF7** e itens fora de escopo). A composição visual
deve ser responsiva, legível e operável por teclado (**RNF1**, **RNF3**,
**RNF6**).

## Tech Stack

- **React 19 e TypeScript strict:** interface componentizada e contratos
  explícitos para respostas parciais; já estão nas dependências do projeto.
- **Vite:** desenvolvimento e build; já configurado no scaffold.
- **Tailwind CSS:** estilos responsivos e tema dark glassmorphism, conforme as
  convenções do projeto.
- **`fetch` e `AbortController`:** chamadas HTTP, cancelamento e timeout sem
  adicionar cliente HTTP.
- **`Intl` do navegador:** formatação de datas e números em `pt-BR` e no fuso
  horário da cidade, sem dependência adicional.
- **Vitest e Testing Library:** testes unitários e de componentes.
- **Playwright:** testes dos fluxos no navegador.
- **pnpm e Biome:** ferramentas já configuradas para dependências, lint e
  formatação.

Não se propõe estado global, cache ou persistência: estado React local é
suficiente para o escopo da primeira versão.

## Project Structure

O scaffold ainda não contém `src/` nem `tests/`; a estrutura abaixo é uma
proposta para a implementação, não um inventário de arquivos existentes:

```text
src/
  components/       apresentação da busca, resultados, clima e estados da UI
  hooks/            estado local e orquestração dos fluxos da tela
  services/         transporte HTTP e adaptação das respostas Open-Meteo
  lib/              regras puras, normalização, conversão e formatação
  types/            contratos de domínio e estados de operação
  App.tsx           composição da página e conexão entre hook e componentes
  main.tsx          inicialização da aplicação
tests/
  unit/             testes de lib, services e hooks
  components/       testes de comportamento e acessibilidade
  e2e/
    weather.spec.ts fluxos principais no navegador
```

### Responsabilidades e dependências

- **`components/`:** renderiza dados recebidos por props, comunica ações por
  callbacks e apresenta estados de carregamento, vazio, erro e dados parciais.
  Manter um componente por arquivo. Não chama `fetch`, não conhece os DTOs da
  API nem coordena concorrência.
- **`hooks/`:** coordena o estado React local e os fluxos da tela, incluindo
  busca, seleção explícita de cidade, consulta meteorológica, unidade e retry.
  Descarta respostas obsoletas e delega transporte e normalização às camadas
  correspondentes.
- **`services/`:** encapsula chamadas HTTP, URLs, timeout, cancelamento e erros
  de rede, HTTP ou JSON. Adapta as respostas externas aos tipos de domínio e
  não importa componentes ou hooks.
- **`lib/`:** contém funções sem efeitos colaterais, como normalização,
  mapeamento de códigos WMO, conversão de temperatura e formatação. Não acessa
  React, rede ou estado global.
- **`types/`:** define contratos compartilhados pelas camadas, sem depender
  delas. A interface consome modelos normalizados em vez de formatos brutos da
  API.

A direção das dependências é `components → hooks → services → Open-Meteo`.
`components`, `hooks` e `services` podem usar funções de `lib/`; `types/` não
depende das demais camadas. `App.tsx` compõe a página e `main.tsx` inicializa a
aplicação. Não adicionar estado global, cache ou persistência sem requisito.

Essa separação define pontos de teste independentes: testar `lib/` diretamente
com entradas válidas, limites e inválidas; testar `services/` com `fetch`
simulado para respostas, erros e timeout; testar `hooks/` com serviços
simulados para transições de estado, retry, unidade e respostas obsoletas; e
testar `components/` pelo comportamento observável, interação e acessibilidade,
sem acoplá-los aos detalhes internos das outras camadas.

## Data Model

Os contratos de domínio estão documentados em
[Interfaces TypeScript](./weather-app-types.md): `City`, `CurrentWeather`,
`ForecastDay`, `WeatherData`, `Unit` e o auxiliar `WeatherValue<T>`, com
comentários sobre os campos e sua origem na Open-Meteo. Esse documento é a
referência dos contratos para a implementação.

Os tipos abaixo complementam esses contratos com erros e estados de operação.
São decisões de interface, não código de implementação:

```ts
type WeatherErrorKind =
  | "timeout"
  | "network"
  | "http"
  | "api"
  | "invalid-response";

type OperationState<T> =
  | { status: "idle" }
  | { status: "loading"; operationId: number }
  | { status: "success"; data: T }
  | { status: "empty" }
  | { status: "error"; kind: WeatherErrorKind; message: string };
```

`WeatherValue` torna explícito quando um valor está indisponível e impede que
ausências sejam convertidas em zero ou estimativas. Uma resposta parcial válida
preserva os campos válidos e marca cada campo ausente como indisponível (**RF2**,
**RF3**).

Os estados de operação têm significado distinto: `idle` indica que ainda não
há operação ativa ou resultado; `loading` identifica a chamada vigente;
`success` contém dados interpretáveis; `empty` representa uma busca concluída
sem correspondências; e `error` representa uma falha que impede interpretar a
operação. Dados meteorológicos parciais continuam em `success`, com cada campo
ausente marcado como indisponível. Uma resposta meteorológica sem estrutura
essencial ou sem dados interpretáveis é `error`, não `empty`.

Se nenhum campo de `current` puder ser exibido, mas houver previsão válida e
estrutura essencial interpretável, manter a operação em `success` parcial.
Representar os campos atuais como indisponíveis e exibir, nessa seção, a
mensagem de falha das condições atuais com ação de nova tentativa; preservar
a previsão válida da mesma cidade. Derivar essa condição dos campos de
`CurrentWeather`, sem novo estado independente. O retry repete a consulta
meteorológica completa para a cidade selecionada, usando o mesmo endpoint
combinado, sem criar requests separados. Se não houver nenhum valor
meteorológico exibível em toda a resposta, usar `error`; datas isoladas não
contam como valores meteorológicos (**RF2**, **RF3**, **RF5**).

`City` contém dados devolvidos pela geocodificação; `id` é uma chave interna
para identificar cada correspondência, não um dado de apresentação. `admin1`
e `admin2` representam subdivisões retornadas, e `country` e `countryCode`
permanecem opcionais. A UI exibe os detalhes geográficos disponíveis sem
deduzir ou inventar os ausentes (**RF1**). A descrição de condição é derivada
de códigos WMO reconhecidos; códigos desconhecidos ficam indisponíveis, sem
descrição inventada.

Temperaturas ficam normalizadas em Celsius. A unidade escolhida é aplicada na
renderização, derivada dos valores em Celsius e da unidade atual, não armazenada
nos dados meteorológicos. A conversão usa a função pura de `lib/` no caminho de
apresentação; alternar a unidade apenas atualiza o estado local e renderiza os
valores derivados, sem novo request ou cópia Fahrenheit dos dados. `date` é uma
data de calendário `YYYY-MM-DD` no fuso da cidade; `forecast` contém exatamente
cinco itens em ordem cronológica, sem repetição (**RF3**, **RF4**).

Uma resposta é **inválida** se não puder ser interpretada com segurança, se
contiver erro HTTP/API, ou se faltar informação estrutural necessária para
identificar localidade, fuso ou datas. É **parcial** quando a estrutura e as
datas são interpretáveis, mas algum valor meteorológico individual está
ausente, nulo, não finito ou tem código de condição desconhecido.

## Data Flow

1. A pessoa informa um nome e inicia a busca. Aplicar somente `trim()` para
   remover espaços externos; preservar acentos e os demais caracteres. Se
   restar texto vazio, não chamar a geocodificação e informar que é necessário
   digitar uma cidade (**RF1**).
2. O serviço envia o termo normalizado como parâmetro de URL e converte as
   correspondências para `City`. Resultados vazios mostram estado vazio;
   múltiplas localidades são apresentadas para seleção explícita.
3. Ao selecionar uma cidade, a aplicação registra essa localização e consulta
   clima atual e previsão para suas coordenadas.
4. O serviço valida a resposta, mapeia os códigos de condição conhecidos e
   normaliza valores válidos ou indisponíveis. Para a previsão, produz cinco
   datas no fuso retornado pela fonte.
5. A interface atualiza somente a operação vigente. Respostas de buscas
   substituídas ou consultas iniciadas para outra cidade são ignoradas.
6. A UI apresenta o nome da cidade selecionada, condições, temperaturas e
   cinco datas. Valores indisponíveis são identificados; datas e números são
   formatados em `pt-BR`.
7. Ao mudar a unidade, converter localmente todas as temperaturas disponíveis.
   A renderização consulta a unidade atual, inclusive se a resposta da chamada
   chegar depois da alternância; nenhuma chamada adicional é necessária
   (**RF4**).

## External APIs

As chamadas são feitas do navegador por HTTPS para os endpoints públicos da
Open-Meteo, sem API key.

- **Geocoding —** [documentação oficial](https://open-meteo.com/en/docs/geocoding-api)
  - URL: `https://geocoding-api.open-meteo.com/v1/search`
  - Parâmetros: `name` com o termo aparado e preservado, `count=10` para limitar
    a lista, `language=pt` e `format=json`. Construir a URL com `URLSearchParams`;
    o termo é dado de busca, nunca marcação HTML.
  - Exemplo de resposta resumida:

    ```json
    {
      "results": [{
        "id": 3448439,
        "name": "São Paulo",
        "latitude": -23.5505,
        "longitude": -46.6333,
        "country": "Brasil",
        "country_code": "BR",
        "admin1": "São Paulo"
      }]
    }
    ```

  - Mapear cada item de `results` para `City`: `id` convertido para `string`,
    `name`, `latitude` e `longitude` diretamente; `country` para `country`,
    `country_code` para `countryCode`, `admin1` e `admin2` para os campos de
    mesmo nome. Os quatro últimos são opcionais e só devem ser incluídos quando
    fornecidos. Ausência ou lista vazia de `results` representa busca sem
    correspondências, não erro de transporte.

- **Forecast —** [documentação oficial](https://open-meteo.com/en/docs)
  - URL: `https://api.open-meteo.com/v1/forecast`
  - Parâmetros: `latitude` e `longitude` da `City` selecionada;
    `current=temperature_2m,weather_code`;
    `daily=weather_code,temperature_2m_max,temperature_2m_min`;
    `temperature_unit=celsius`; `timezone=auto`; `forecast_days=5`.
    `current` e `daily` especificam as variáveis incluídas nas respectivas
    seções da resposta. `timezone=auto` pede o fuso local das coordenadas; usar
    o `timezone` IANA retornado como fonte de verdade, nunca o fuso do navegador
    (**RF3**).
  - Exemplo de resposta resumida (valores ilustrativos):

    ```json
    {
      "timezone": "America/Sao_Paulo",
      "current": {
        "temperature_2m": 22.4,
        "weather_code": 2
      },
      "daily": {
        "time": [
          "2026-10-27",
          "2026-10-28",
          "2026-10-29",
          "2026-10-30",
          "2026-10-31"
        ],
        "weather_code": [2, 3, 61, 0, 1],
        "temperature_2m_max": [25.1, 24.0, 21.3, 26.0, 27.2],
        "temperature_2m_min": [17.2, 16.8, 15.4, 17.0, 18.1]
      }
    }
    ```

  - Mapear a resposta para `WeatherData`: `city` é o objeto `City` selecionado
    e `timezone` vem da resposta. Em `current`, mapear `temperature_2m` para
    `CurrentWeather.temperatureC` e `weather_code` para `condition`, convertendo
    o código WMO reconhecido em descrição pt-BR. Para cada índice `i` de
    `daily.time`, criar um `ForecastDay`: `time[i]` → `date`,
    `temperature_2m_min[i]` → `temperatureMinC`,
    `temperature_2m_max[i]` → `temperatureMaxC` e `weather_code[i]` → `condition`
    (também convertido de código WMO). Os arrays diários são alinhados pelo
    índice.
  - Solicitar sempre Celsius para manter os valores de domínio normalizados e
    permitir alternância de unidade sem nova consulta. Valores ausentes, nulos
    ou não finitos tornam-se `WeatherValue` com status `unavailable`; código WMO
    desconhecido também torna `condition` indisponível, nunca inventada.

Validar o envelope da resposta, o fuso e os dados de data antes de expor a
previsão. Calcular as cinco datas esperadas a partir da data atual no fuso da
cidade e associar os valores recebidos pela data. Datas duplicadas ou
ininterpretáveis, ou fuso ausente/inválido, invalidam a resposta. Para uma data
esperada sem valores, manter o slot e marcar os campos meteorológicos como
indisponíveis; não estimar valores nem reduzir a sequência de cinco dias.

## State Management

Manter o estado React local em um hook de orquestração usado por `App.tsx`;
`App.tsx` compõe a tela e os componentes recebem valores e ações por props.
Separar os estados da busca e da consulta meteorológica para que cada fluxo
tenha seu próprio ciclo explícito:

```ts
interface AppState {
  query: string;
  submittedQuery?: string;
  search: OperationState<City[]>;
  selectedCity?: City;
  weather: OperationState<WeatherData>;
  unit: Unit;
}
```

`query` é o texto editável do campo. `submittedQuery` guarda um snapshot do
termo normalizado da busca vigente e é definido antes de iniciar sua chamada;
permanece associado ao resultado ou erro até outra busca válida ser aceita.
Editar o campo não altera esse snapshot. O retry de busca usa `submittedQuery`,
não `query`, para repetir exatamente a operação que falhou.

A unidade começa em Celsius e permanece no estado local do hook; não persistir
entre sessões, pois a spec não decide essa persistência. A unidade selecionada
vale para resultados já exibidos e para respostas de chamadas em andamento. A
mudança de unidade não altera `weather.data` nem reinicia a operação.

Identificar operações com um contador monotônico e, quando aplicável,
`AbortController`. Uma nova busca torna obsoleta a anterior; somente o
identificador vigente pode atualizar estado. Ao selecionar outra cidade,
associar qualquer resultado meteorológico à cidade contida em `WeatherData`
e não exibir dados antigos como se fossem dessa seleção. O identificador
continua necessário mesmo com cancelamento, pois cancelamento não garante que
uma resposta tardia não chegue.

Ao enviar uma busca por botão ou Enter, comparar o termo após `trim()` com
`submittedQuery`: se a busca estiver em `loading` para o mesmo termo, não
iniciar outra chamada nem substituir seu identificador. Um termo normalizado
diferente inicia uma nova busca e invalida a anterior. Manter a possibilidade
de enviar outro termo durante loading; bloquear apenas duplicatas, não toda
a busca. A guarda no hook deve valer independentemente do controle utilizado.

Busca e consulta meteorológica têm estados de carregamento próprios. Não
armazenar cópias em Fahrenheit, não adicionar cache e impedir retry enquanto a
mesma operação estiver em andamento (**RF5**).

## Error Handling

- **Busca vazia:** não chamar geocoding; informar que é necessário digitar o
  nome e manter o campo disponível.
- **Sem resultados:** informar que nenhuma cidade foi encontrada, preservar o
  termo e permitir outra busca.
- **Carregamento:** dar feedback visual imediatamente e identificar a operação
  em andamento.
- **Timeout:** limitar cada chamada à fonte a 10 segundos com deadline e
  `AbortController`. Ao exceder o prazo, encerrar o carregamento, informar
  timeout e oferecer retry. Uma resposta posterior ao deadline não pode
  atualizar o estado, mesmo que o cancelamento de transporte não seja efetivo.
- **Falha recuperável:** classificar timeout, falha de rede, status HTTP não
  bem-sucedido, erro explícito retornado no payload pela API e resposta inválida
  como `timeout`, `network`, `http`, `api` e `invalid-response`,
  respectivamente. Apresentar mensagem compreensível em `pt-BR` e permitir
  repetir a mesma operação, preservando o termo ou a cidade selecionada.
- **Duplicação e concorrência:** desabilitar retry enquanto a operação
  correspondente estiver ativa. Reenviar o mesmo termo normalizado pelo botão
  de busca ou Enter durante loading não gera chamada adicional; um termo
  diferente substitui a busca anterior. Respostas obsoletas são ignoradas.
  Cada chamada aceita no máximo uma tentativa ativa por operação.
- **Resposta parcial:** tratar como sucesso parcial, não como erro: mostrar
  valores válidos e marcar os ausentes, nulos, não finitos ou não mapeáveis como
  indisponíveis. Na previsão, preservar os cinco dias e marcar individualmente
  os campos indisponíveis. Não renderizar dados anteriores de outra cidade como
  resultado da consulta que falhou. Quando nenhum campo atual for exibível,
  comunicar falha nessa seção e oferecer retry da consulta completa, sem
  descartar uma previsão válida da mesma resposta. Esse retry segue o loading
  e o bloqueio de duplicação da consulta meteorológica normal.
- **Resposta inválida:** erro de JSON, erro explícito da API, estrutura
  essencial malformada, fuso inválido ou datas impossíveis geram erro
  recuperável, não dados presumidos.

## Testing Strategy

Usar Vitest para regras, serviços, hooks e componentes; reservar Playwright
para a integração dos fluxos no navegador e comportamento responsivo. O
scaffold ainda não contém `src/` ou `tests/`: esta é uma estratégia para a
implementação, não uma declaração de cobertura já existente. Derivar os
cenários dos critérios de aceite de **RF1–RF5** e **RNF1–RNF6**, **RNF8**.

### Vitest — funções puras

Testar `lib/` diretamente, sem React ou rede:

- Conversão derivada de Celsius: `0 °C → 32 °F`, `100 °C → 212 °F`,
  `-40 °C → -40 °F`, temperaturas negativas e fracionárias. Ao voltar a
  Celsius, usar o valor original, sem acumular erro de conversões sucessivas.
- Arredondamento a uma casa decimal, formatação em `pt-BR` e ausência de zero
  negativo. Valores indisponíveis permanecem indisponíveis em ambas as
  unidades; não converter `null`, `undefined` ou valores não finitos em zero.
- Normalização da busca com apenas `trim()`, preservando acentos e caracteres.
- Mapeamento de códigos WMO conhecidos e tratamento de códigos desconhecidos.
- Normalização de dados parciais, fuso e datas: exatamente cinco dias
  consecutivos, em ordem e sem duplicatas; incluir virada do dia com o fuso da
  cidade diferente do navegador. Rejeitar fuso inválido e datas impossíveis;
  manter o dia quando apenas suas métricas estiverem indisponíveis.

### Vitest — services com mock de `fetch`

Isolar os serviços com `fetch` simulado, sem chamadas reais à Open-Meteo:

- Verificar endpoints, parâmetros de busca codificados, coordenadas,
  `temperature_unit=celsius` e os campos de clima atual/previsão solicitados.
- Converter respostas válidas em modelos de domínio; cobrir geocodificação
  vazia, cidades homônimas e campos geográficos opcionais ausentes.
- Distinguir rejeição de rede, status HTTP de erro, erro explícito da API,
  JSON inválido e estrutura essencial malformada. Verificar os tipos de erro
  entregues ao hook, sem mascarar falhas como sucesso vazio.
- Preservar campos válidos em respostas parciais e marcar os demais como
  indisponíveis; rejeitar fuso/datas inválidos. Cobrir `current` ausente ou sem
  campos exibíveis com previsão válida como sucesso parcial, e a ausência de
  qualquer valor meteorológico exibível na resposta inteira como erro.
- Com temporizadores simulados, verificar que aos 10 segundos a chamada
  termina em timeout, aborta o transporte e não aceita conclusão tardia.
  Distinguir cancelamento por substituição de operação de timeout e liberar
  temporizadores/listeners tanto no sucesso quanto na falha.

### Vitest — hooks e componentes

Testar hooks com serviços simulados; testar componentes com Testing Library
e `user-event`, usando roles, labels e texto visível, não detalhes internos:

- Hooks: `idle → loading → success`, `loading → empty` na busca e
  `loading → error`; retry preserva o termo enviado ou a cidade da operação
  que falhou, mesmo que o texto do campo seja alterado depois.
- Concorrência: nova busca ou seleção invalida a anterior; respostas antigas
  não sobrescrevem dados atuais nem associam clima à cidade errada. Retry
  durante loading não cria chamada duplicada. Reenviar o mesmo termo
  normalizado por botão ou Enter, inclusive com espaços externos diferentes,
  mantém uma única chamada e o identificador vigente; enviar outro termo
  substitui a busca em andamento.
- Unidade: Celsius inicial; alternar durante e após a consulta atualiza
  temperatura atual, mínimas e máximas, mantendo os dados em Celsius e a
  contagem de chamadas ao serviço inalterada.
- `idle`: campo e orientação inicial disponíveis, sem resultado fictício.
- `loading`: feedback imediato e acessível por operação; retry indisponível.
- `error`: mensagem em pt-BR e ação de nova tentativa; não apresentar dados
  antigos como resultado atual da operação que falhou.
- `empty`: mensagem de nenhuma cidade encontrada, termo preservado e
  possibilidade de outra busca. Entrada em branco mostra validação sem
  chamar o serviço.
- `success`: opções de cidade sem seleção automática; cidade selecionada,
  condições atuais e cinco previsões com unidades e formatação corretas.
- Sucesso parcial: manter valores válidos e cinco datas, sinalizando campos
  indisponíveis sem estimativas. Sem campos atuais exibíveis e com previsão
  válida, mostrar falha das condições atuais, preservar a previsão e permitir
  retry da consulta completa, com uma única chamada e recuperação da seção.
  Validar nomes acessíveis, anúncio de estados e operação por teclado em todos
  os estados relevantes.

Restaurar mocks e temporizadores após cada teste para evitar interferência
entre cenários. Usar relógio fixo quando o resultado depender de data.

### Playwright — fluxos E2E e viewport mobile

Executar os fluxos com a aplicação real e interceptar as chamadas Open-Meteo
com `page.route`, fornecendo respostas controladas. Não simular hooks ou
componentes: validar a integração completa entre UI, estado e serviços, sem
depender da rede externa. Fixar o relógio nos cenários de datas e timeout.

- Buscar, escolher explicitamente uma cidade homônima e visualizar o clima
  atual e exatamente cinco datas no fuso da cidade.
- Validar busca em branco sem request, resultado vazio e cidades com detalhes
  geográficos ausentes; corrigir a entrada e completar uma nova busca.
- Alternar Celsius/Fahrenheit durante e depois da consulta; conferir valores,
  arredondamento e rótulos na temperatura atual e nas mínimas/máximas. Contar
  requests ao endpoint de forecast e confirmar que a alternância não gera
  nenhum adicional.
- Simular dados parciais, falha de rede/API e timeout; verificar feedback,
  retry bem-sucedido e bloqueio de chamadas duplicadas. Incluir condições
  atuais sem campos exibíveis com previsão válida e recuperar as condições
  atuais pelo retry da consulta completa.
- Após falha na busca, editar o campo e acionar retry; confirmar que o request
  usa o termo original. Durante uma busca pendente, reenviar o mesmo termo
  por botão e Enter sem requests adicionais; depois enviar um termo diferente
  e confirmar que uma nova busca substitui a anterior.
- Entregar resposta obsoleta após uma nova busca ou seleção de cidade;
  confirmar que apenas a operação vigente determina os dados exibidos.
- Percorrer busca, seleção, unidade e retry por teclado, verificando nomes
  acessíveis e feedback de loading, vazio e erro.

Rodar a suíte nos projetos já definidos em
[`playwright.config.ts`](../playwright.config.ts): `chromium` (Desktop Chrome)
e `mobile` (iPhone 13, viewport de 390 × 664 CSS px). Em ambos, verificar que
campo, seleção, unidades, mensagens e cinco previsões permanecem visíveis ou
alcançáveis por rolagem vertical, sem sobreposição ou corte e sem rolagem
horizontal. Incluir nomes longos de cidade, mensagens extensas e aumento do
texto. No projeto mobile, completar o fluxo também com interação por toque.
A emulação valida o viewport e interações, não substitui teste em aparelho
físico (**RNF1**, **RNF3**, **RNF6**).

### Execução e critério de conclusão

Na implementação, executar `pnpm lint`, `pnpm build` e `pnpm test`; executar
`pnpm test:e2e` para os fluxos de navegador nos dois projetos.
`pnpm test:coverage` ajuda a identificar lacunas, mas percentual isolado não
substitui os cenários e saídas exigidos acima. Esta atualização documental não
exige executar essas suítes.

## Risks & Trade-offs

As alternativas abaixo foram avaliadas para a primeira versão; não são
funcionalidades adicionais aprovadas.

| Decisão do plano | Benefício e custo/risco | Alternativas consideradas e motivo para não adotar agora |
| --- | --- | --- |
| React + Vite client-side, sem backend | Menos infraestrutura e integração direta com API pública sem chave; depende da disponibilidade e das regras de acesso da fonte no navegador. | Backend/proxy ou framework com SSR: permitiria centralizar acesso e cache, mas acrescentaria hospedagem e operação sem requisito correspondente. |
| Estado local em hook, sem store global ou persistência | Mantém a orquestração próxima da tela e reduz dependências; cidade e unidade se perdem ao recarregar ou encerrar a sessão. | Context/store global e `localStorage`: úteis para múltiplas telas ou preferências persistidas, mas sem necessidade definida no escopo atual. |
| `fetch` com `AbortController` e identificadores de operação | Evita novo cliente HTTP e impede respostas obsoletas; exige implementar e testar deadline, limpeza e concorrência. | Axios ou biblioteca de server state como TanStack Query: reduziriam parte da infraestrutura, mas introduziriam dependências e políticas de cache/retry desnecessárias agora. Cancelamento isolado não garante proteção contra respostas tardias. |
| Open-Meteo sem cache, fallback ou retry automático | Fonte única e recuperação explícita, sem apresentar dado antigo como atual; falhas e limites externos interrompem a consulta até nova tentativa. | Cache, provedor alternativo e retry automático: melhorariam disponibilidade, mas exigiriam regras de validade, normalização e carga adicionais fora do escopo. |
| Celsius como fonte única, Fahrenheit derivado na renderização | Alternância imediata, inclusive durante loading, sem request; requer conversão e arredondamento consistentes. | Consultar a API por unidade aumenta latência e chamadas; guardar duas cópias permite divergência e erro de arredondamento acumulado (**RF4**). |
| Seleção explícita de cidade | Reduz erro com cidades homônimas, mas acrescenta uma interação. | Escolher automaticamente o primeiro resultado ou usar geolocalização contraria a seleção exigida e o escopo de privacidade (**RF1**, **RNF7**). |
| Normalização por campo e tabela local WMO em pt-BR | Preserva dados parciais sem inventar valores; exige manter mapeamentos e distinguir ausência de estrutura inválida. | Rejeitar toda resposta parcial descartaria valores úteis; preencher com zero, estimativas ou códigos brutos violaria os requisitos (**RF2**, **RF3**). |
| Datas calculadas no fuso IANA retornado, com validação estrita | Mantém hoje e os quatro dias seguintes corretos; fuso/datas inválidos impedem exibir a previsão com segurança. | Fuso do navegador é mais simples, mas pode deslocar o dia. Biblioteca de datas só se justifica se `Intl` e funções puras deixarem de atender às regras (**RF3**, **RNF5**). |
| Funções puras, serviços, hooks e apresentação separados | Permite testes isolados e contratos claros; adiciona limites e arquivos que devem ter uso real. | Concentrar rede e regras nos componentes reduz a estrutura inicial, mas acopla transporte, estado e renderização; camadas genéricas adicionais seriam complexidade sem benefício comprovado. |
| Vitest amplo e Playwright focado, com rede simulada | Feedback rápido e resultados determinísticos; mocks não detectam sozinhos mudanças no serviço externo e emulação mobile não equivale a aparelho físico. | Apenas E2E aumenta custo e dificulta localizar falhas; apenas unitários não valida integração e layout; API real nas suítes obrigatórias adiciona instabilidade. Verificação manual da fonte e em aparelho complementa, mas não substitui os testes automatizados. |
