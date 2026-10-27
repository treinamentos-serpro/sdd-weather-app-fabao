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
  components/       busca, seleção, clima atual, previsão e estados da UI
  services/         chamadas Open-Meteo, validação e normalização
  types/            modelos de domínio e estados de operação
  utils/            conversão e formatação, se forem funções reutilizáveis
  App.tsx           composição dos fluxos e estado da página
  main.tsx          inicialização da aplicação
tests/
  unit/             testes de serviços, normalização e conversão
  components/       testes de comportamento e acessibilidade
e2e/
  weather.spec.ts   fluxos principais no navegador
```

Manter `components` independente do formato da API e `services` independente
da renderização. Criar hooks ou utilitários separados somente quando houver
lógica reutilizável, sem introduzir camadas adicionais por antecipação.

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

`City` contém dados devolvidos pela geocodificação; `id` é uma chave interna
para identificar cada correspondência, não um dado de apresentação. `admin1`
e `admin2` representam subdivisões retornadas, e `country` e `countryCode`
permanecem opcionais. A UI exibe os detalhes geográficos disponíveis sem
deduzir ou inventar os ausentes (**RF1**). A descrição de condição é derivada
de códigos WMO reconhecidos; códigos desconhecidos ficam indisponíveis, sem
descrição inventada.

Temperaturas ficam normalizadas em Celsius. A unidade escolhida é aplicada na
apresentação, não armazenada nos dados meteorológicos. `date` é uma data de
calendário `YYYY-MM-DD` no fuso da cidade; `forecast` contém exatamente cinco
itens em ordem cronológica, sem repetição (**RF3**, **RF4**).

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

Manter o estado na composição principal ou em um hook local simples, separado
entre busca e consulta meteorológica:

```ts
interface AppState {
  query: string;
  search: OperationState<City[]>;
  selectedCity?: City;
  weather: OperationState<WeatherData>;
  unit: Unit;
}
```

A unidade começa em Celsius e permanece em memória; não persistir entre
sessões, pois a spec não decide essa persistência. A unidade atual aplica-se
tanto a resultados exibidos quanto a chamadas em andamento.

Identificar operações com um contador monotônico e, quando aplicável,
`AbortController`. Uma nova busca torna obsoleta a anterior; somente o
identificador vigente pode atualizar estado. Ao selecionar outra cidade,
associar qualquer resultado meteorológico à cidade contida em `WeatherData`
e não exibir dados antigos como se fossem dessa seleção. O identificador
continua necessário mesmo com cancelamento, pois cancelamento não garante que
uma resposta tardia não chegue.

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
- **Falha recuperável:** distinguir timeout, rede, HTTP e resposta inválida;
  apresentar mensagem compreensível em `pt-BR` e permitir repetir a mesma
  operação, preservando o termo ou a cidade selecionada.
- **Duplicação e concorrência:** desabilitar retry enquanto a operação
  correspondente estiver ativa. Uma busca nova substitui a anterior; respostas
  obsoletas são ignoradas. Cada chamada aceita no máximo uma tentativa ativa
  por operação.
- **Dados parciais:** mostrar valores válidos e marcar os ausentes como
  indisponíveis. Não renderizar dados anteriores de outra cidade como
  resultado da consulta que falhou.
- **Resposta inválida:** erro de JSON, erro explícito da API, estrutura
  essencial malformada, fuso inválido ou datas impossíveis geram erro
  recuperável, não dados presumidos.

## Testing Strategy

Usar Vitest para testes unitários e de componentes e Playwright para fluxos
completos. O scaffold atual ainda não contém `src/` ou `tests/`; os testes
acompanham a implementação dessas áreas.

**Vitest — unitários e componentes**

- Busca: aparar apenas espaços externos, preservar acentos/caracteres e não
  chamar o serviço para entrada vazia.
- Geocoding: resultado vazio, localidades homônimas, campos opcionais ausentes
  e preservação do conteúdo como dado.
- Forecast: fuso ausente/inválido, datas inválidas ou duplicadas como erro;
  métricas ausentes como dados parciais; sempre cinco slots, ordenados e sem
  duplicatas, inclusive quando o fuso da cidade difere do navegador.
- Unidades: Celsius inicial, conversão nas duas direções, arredondamento a uma
  casa decimal e ausência de `-0`; mudança de unidade enquanto a consulta está
  pendente sem nova chamada.
- Concorrência e recuperação: ignorar resposta obsoleta, timeout em 10
  segundos, ignorar resposta tardia e bloquear retry duplicado.
- Componentes: carregamento, vazio, erro e parcial; nomes acessíveis e
  operação por teclado.

Mockar `fetch`, relógio e temporizadores para que os testes não dependam da
disponibilidade real das APIs.

**Playwright — E2E**

- Buscar, selecionar uma localidade e visualizar clima atual e cinco previsões.
- Exigir escolha explícita entre localidades homônimas.
- Verificar busca vazia, resultado vazio e detalhes geográficos ausentes.
- Alternar Celsius/Fahrenheit sem nova consulta e conferir unidade e
  arredondamento.
- Simular resposta parcial, falha, timeout e retry, incluindo prevenção de
  chamadas duplicadas.
- Entregar resposta obsoleta depois de iniciar outra busca e confirmar que ela
  não substitui a busca vigente.
- Validar teclado, nomes acessíveis e layouts estreitos e desktop sem rolagem
  horizontal que impeça acesso (**RNF1**, **RNF3**, **RNF6**).

## Risks & Trade-offs

- **Disponibilidade e limites da Open-Meteo:** falhas externas interrompem
  consultas. Timeout, mensagens específicas e retry são a mitigação prevista;
  cache e fallback ficam fora do escopo.
- **Cidades homônimas:** nome sozinho não identifica localidade. A seleção
  explícita e a exibição apenas dos detalhes recebidos reduzem a chance de
  consulta errada (**RF1**).
- **Fuso e virada de dia:** usar o fuso do navegador poderia deslocar datas.
  A fonte da verdade é a zona horária retornada pela API; falhar é preferível a
  presumir datas não interpretáveis (**RF3**, **RNF5**).
- **Campos ou códigos desconhecidos:** uma tabela local de códigos WMO precisa
  acompanhar códigos suportados, mas evita descrições inventadas. Dados
  ausentes ficam indisponíveis (**RF2**, **RF3**).
- **Conversão local:** evita chamadas adicionais e permite alternar durante a
  consulta, mas exige uma regra consistente de arredondamento e tratamento de
  zero negativo (**RF4**).
- **Respostas tardias:** `AbortController` reduz trabalho desnecessário, mas
  identificadores de operação são necessários para proteger o estado contra
  resultados obsoletos (**RF5**, **RNF5**).
- **Estado apenas em memória:** simplifica a primeira versão e evita persistir
  preferências sem decisão explícita; unidade e cidade não sobrevivem à sessão.
- **Sem backend:** reduz infraestrutura e é compatível com APIs públicas sem
  chave, mas mantém a aplicação dependente da disponibilidade e do
  comportamento dos serviços externos.
