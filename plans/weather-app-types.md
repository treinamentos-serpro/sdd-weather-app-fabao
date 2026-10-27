# Weather App — Interfaces TypeScript

Contratos de domínio do [plano técnico](./weather-app-plan.md), baseados nos
dados da Open-Meteo. São uma proposta para implementação futura em `src/types/`,
não cópias das respostas brutas da API.

## Contratos

```ts
/** Unidade de exibição; os dados permanecem em Celsius. */
export type Unit = "celsius" | "fahrenheit";

/** Distingue valores válidos de dados meteorológicos indisponíveis. */
export type WeatherValue<T> =
  | { status: "available"; value: T }
  | { status: "unavailable" };

export interface City {
  id: string; // Identificador da geocodificação convertido para string.
  name: string; // Nome da cidade.
  latitude: number; // Latitude em graus decimais.
  longitude: number; // Longitude em graus decimais.
  country?: string; // Nome do país, quando retornado.
  countryCode?: string; // Código ISO do país; origem: country_code.
  admin1?: string; // Subdivisão administrativa de primeiro nível.
  admin2?: string; // Subdivisão administrativa de segundo nível.
}

export interface CurrentWeather {
  temperatureC: WeatherValue<number>; // Temperatura atual; origem: temperature_2m.
  condition: WeatherValue<string>; // Descrição em pt-BR derivada de weather_code (WMO).
}

export interface ForecastDay {
  date: string; // Data YYYY-MM-DD no fuso da cidade; origem: daily.time.
  temperatureMinC: WeatherValue<number>; // Mínima; origem: daily.temperature_2m_min.
  temperatureMaxC: WeatherValue<number>; // Máxima; origem: daily.temperature_2m_max.
  condition: WeatherValue<string>; // Descrição derivada de daily.weather_code.
}

export interface WeatherData {
  city: City; // Cidade selecionada que originou a consulta.
  timezone: string; // Fuso IANA retornado pela API, como America/Sao_Paulo.
  current: CurrentWeather; // Condições meteorológicas atuais normalizadas.
  forecast: ForecastDay[]; // Cinco dias consecutivos em ordem cronológica.
}
```

## Normalização e validação

- Solicitar `temperature_unit=celsius`; converter para Fahrenheit apenas na
  apresentação, conforme `Unit`, sem nova chamada à API.
- Mapear os códigos WMO para descrições em `pt-BR`: a API fornece códigos,
  não textos de condição.
- Representar valores ausentes, nulos, não finitos ou códigos desconhecidos
  como `unavailable`, nunca como zero ou estimativas.
- Preservar os campos geográficos opcionais somente quando retornados pela
  geocodificação, sem deduzir detalhes ausentes.
- Validar no serviço o fuso IANA, as datas e os cinco dias consecutivos, em
  ordem e sem repetição. `string` e `ForecastDay[]` não garantem essas regras.
- Para uma data esperada sem valores, preservar o dia na previsão e marcar
  seus valores como indisponíveis. Fuso ausente/inválido e datas duplicadas
  ou ininterpretáveis invalidam a resposta, conforme o plano.

## Origem dos dados

- [Open-Meteo — Geocoding API](https://open-meteo.com/en/docs/geocoding-api):
  dados de `City`.
- [Open-Meteo — Weather Forecast API](https://open-meteo.com/en/docs):
  `current`, `daily`, `timezone` e códigos de condição WMO.
