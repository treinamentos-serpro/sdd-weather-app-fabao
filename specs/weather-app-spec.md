# Weather App — Especificação de Produto

## Overview

O Weather App é uma aplicação web para consultar rapidamente o clima atual de uma cidade e a previsão diária dos próximos cinco dias. A experiência deve funcionar em celulares e desktops, permitir a confirmação de cidades homônimas e apresentar temperaturas em Celsius ou Fahrenheit.

O público inclui pessoas que consultam o clima antes de sair, viajantes que planejam os próximos dias e usuários que precisam buscar uma cidade diferente da sua localização. A primeira versão não exige conta, não solicita localização automática e usa a Open-Meteo como fonte de geocodificação e dados meteorológicos.

**Objetivos do produto:**

- Permitir encontrar e confirmar uma cidade sem confundi-la com localidades de mesmo nome.
- Apresentar as condições atuais e a previsão diária de hoje mais os quatro dias seguintes.
- Tornar temperatura e datas fáceis de compreender em pt-BR, em Celsius ou Fahrenheit.

## Functional Requirements

**RF1 — Buscar e selecionar uma cidade**

O sistema deve permitir pesquisar uma cidade pelo nome, apresentar correspondências disponíveis e permitir que a pessoa selecione a localização correta. Antes de iniciar a busca, deve remover espaços em branco no início e no fim do texto. Se, após essa normalização, o campo estiver vazio, não deve consultar o serviço de geocodificação e deve informar que é necessário digitar o nome de uma cidade. Nomes válidos devem manter acentos e demais caracteres, e ser enviados como texto de busca, sem serem interpretados como marcação. Quando existirem resultados ambíguos, cada opção deve exibir contexto suficiente para distingui-la, incluindo estado ou região e país quando disponíveis; detalhes ausentes não devem ser inventados e uma localização não deve ser selecionada automaticamente.

**RF2 — Consultar as condições atuais**

Após a seleção de uma cidade, o sistema deve exibir as condições meteorológicas atuais dessa localização. No mínimo, deve apresentar temperatura e uma descrição legível da condição do tempo, associadas à cidade selecionada. Se a resposta estiver incompleta, deve exibir os campos válidos e identificar explicitamente os campos indisponíveis, sem estimar ou inventar valores. Se a resposta não puder ser interpretada, deve comunicar falha e permitir nova tentativa.

**RF3 — Consultar a previsão de cinco dias**

Após a seleção de uma cidade, o sistema deve exibir a previsão diária para cinco datas: hoje e os quatro dias seguintes, de acordo com o fuso horário da cidade retornado pela fonte meteorológica. Para cada dia, deve apresentar a data, a descrição da condição do tempo e as temperaturas mínima e máxima. Se faltarem valores para uma ou mais datas, deve manter a sequência de cinco datas e identificar os dados ausentes, sem estimar ou inventar valores. Se não for possível determinar o fuso horário ou interpretar as datas, deve comunicar falha e permitir nova tentativa.

**RF4 — Alterar a unidade de temperatura**

O sistema deve permitir alternar entre Celsius e Fahrenheit. A unidade selecionada deve ser aplicada à temperatura atual e às temperaturas mínima e máxima da previsão, com indicação visível da unidade. A seleção deve ser aplicada também a respostas de consultas ainda em andamento, sem exigir uma nova chamada à fonte meteorológica. Os valores devem ser arredondados para uma casa decimal; um valor arredondado a zero não deve ser apresentado como negativo.

**RF5 — Comunicar estados da consulta**

Durante buscas e consultas meteorológicas, o sistema deve indicar carregamento, resultados vazios e erros de modo compreensível. A interface deve aguardar no máximo 10 segundos por chamada à fonte; ao exceder esse prazo, deve encerrar o carregamento, informar que a operação demorou demais, ignorar uma eventual resposta tardia e oferecer uma ação para tentar novamente. Em caso de falha recuperável, deve oferecer uma ação para repetir a mesma operação. Uma nova busca deve prevalecer sobre buscas anteriores ainda em andamento, cujas respostas tardias devem ser ignoradas. A ação de nova tentativa deve ficar indisponível enquanto a mesma operação estiver em andamento, evitando chamadas duplicadas.

## User Stories

- **US1 — RF1 e RF2:** Como Decisor do dia a dia, quero buscar minha cidade e consultar as condições atuais para decidir como me preparar antes de sair.
- **US2 — RF3:** Como Viajante planejador, quero consultar a previsão diária de hoje e dos próximos quatro dias para organizar minhas atividades e bagagem.
- **US3 — RF1:** Como Pessoa consultando uma cidade diferente, quero comparar os detalhes geográficos das cidades encontradas e selecionar a localização correta para consultar o clima do lugar desejado.
- **US4 — RF4:** Como Pessoa consultando uma cidade diferente, quero alternar entre Celsius e Fahrenheit para compreender as temperaturas na unidade que prefiro.
- **US5 — RF5:** Como Decisor do dia a dia, quero ver quando uma busca ou consulta está carregando, sem resultados ou com erro, e poder tentar novamente após uma falha recuperável para saber como prosseguir.

## Acceptance Criteria

Os critérios abaixo seguem o formato Dado/Quando/Então e referenciam os requisitos funcionais correspondentes.

**RF1 — Busca e seleção de cidade**

- **Dado** que o campo de busca está vazio ou contém apenas espaços em branco, **Quando** a pessoa tenta iniciar a busca, **Então** o sistema não consulta o serviço de geocodificação, informa que é necessário digitar o nome de uma cidade e mantém o campo disponível para correção.
- **Dado** que o campo de busca contém um nome de cidade precedido ou seguido por espaços em branco, **Quando** a pessoa inicia a busca, **Então** o sistema remove esses espaços e pesquisa o nome normalizado.
- **Dado** que o nome da cidade contém acentos ou outros caracteres válidos, **Quando** a pessoa inicia a busca, **Então** o sistema preserva o texto e o envia como parâmetro de busca, sem interpretar o conteúdo como marcação.
- **Dado** que a pessoa informa um nome de cidade e inicia a busca, **Quando** a busca termina com correspondências, **Então** o sistema apresenta uma opção selecionável para cada correspondência, com o nome da cidade e todos os detalhes geográficos retornados.
- **Dado** que os resultados incluem cidades com o mesmo nome, **Quando** as opções são exibidas, **Então** cada opção apresenta o estado ou região e o país quando esses dados estiverem disponíveis na correspondência.
- **Dado** que uma correspondência não contém todos os detalhes geográficos, **Quando** as opções são exibidas, **Então** o sistema mostra apenas os detalhes disponíveis e não seleciona uma localização automaticamente.
- **Dado** que a geocodificação termina com sucesso sem correspondências, **Quando** o resultado é apresentado, **Então** o sistema informa que nenhuma cidade foi encontrada, preserva o termo pesquisado e mantém disponível o controle para outra busca.
- **Dado** que uma busca anterior ainda está em andamento, **Quando** a pessoa inicia uma nova busca, **Então** a nova busca passa a ser a busca vigente e qualquer resposta tardia da busca anterior é ignorada.
- **Dado** que a pessoa seleciona uma opção de cidade, **Quando** a seleção é concluída, **Então** essa opção passa a ser a localização selecionada para a consulta meteorológica.

**RF2 — Condições atuais**

- **Dado** que uma cidade está selecionada e a consulta das condições atuais é concluída com sucesso, **Quando** os dados são exibidos, **Então** a tela apresenta o nome da cidade selecionada, a temperatura atual com sua unidade e uma descrição textual da condição do tempo.
- **Dado** que a busca contém mais de uma localização possível e a pessoa seleciona uma delas, **Quando** a consulta meteorológica termina com sucesso, **Então** a cidade e as condições atuais exibidas correspondem à localização selecionada.
- **Dado** que a resposta atual contém alguns campos válidos e outros ausentes, **Quando** os dados são exibidos, **Então** o sistema apresenta os valores válidos e identifica os campos ausentes como indisponíveis, sem substituí-los por estimativas.
- **Dado** que a resposta atual está malformada ou não contém dados que possam ser exibidos, **Quando** a consulta termina, **Então** o sistema informa que não foi possível obter as condições atuais e oferece nova tentativa.

**RF3 — Previsão de cinco dias**

- **Dado** que uma cidade está selecionada e a consulta da previsão é concluída com sucesso, **Quando** a previsão é exibida, **Então** são apresentadas exatamente cinco previsões diárias, começando na data de hoje e incluindo os quatro dias seguintes.
- **Dado** que as cinco previsões diárias são exibidas, **Quando** a pessoa consulta qualquer uma delas, **Então** cada previsão apresenta uma data, uma descrição textual da condição do tempo e as temperaturas mínima e máxima com suas unidades.
- **Dado** que as cinco previsões diárias são exibidas, **Quando** suas datas são comparadas, **Então** elas estão em ordem cronológica e não há datas repetidas.
- **Dado** que a resposta meteorológica omite valores de uma ou mais datas, **Quando** a previsão é exibida, **Então** são mantidas as cinco datas esperadas e cada dado ausente é identificado como indisponível, sem estimativas.
- **Dado** que a fonte não fornece o fuso horário da cidade ou datas interpretáveis, **Quando** a previsão é processada, **Então** o sistema não apresenta datas presumidas como corretas, informa a falha e oferece nova tentativa.
- **Dado** que a previsão é solicitada para uma cidade com fuso horário diferente do navegador, **Quando** as datas são exibidas, **Então** “hoje” e os quatro dias seguintes seguem o fuso horário da cidade retornado pela fonte, em ordem cronológica e sem duplicatas.

**RF4 — Unidade de temperatura**

- **Dado** que as temperaturas estão sendo exibidas em Celsius, **Quando** a pessoa seleciona Fahrenheit, **Então** a temperatura atual e todas as temperaturas mínimas e máximas da previsão são exibidas em Fahrenheit, com a unidade indicada junto aos valores.
- **Dado** que as temperaturas estão sendo exibidas em Fahrenheit, **Quando** a pessoa seleciona Celsius, **Então** a temperatura atual e todas as temperaturas mínimas e máximas da previsão são exibidas em Celsius, com a unidade indicada junto aos valores.
- **Dado** que a pessoa inicia uma consulta sem ter selecionado outra unidade na sessão, **Quando** as temperaturas são exibidas, **Então** a unidade apresentada é Celsius.
- **Dado** que uma consulta está em andamento, **Quando** a pessoa altera a unidade, **Então** os dados da resposta são apresentados na unidade atualmente selecionada, sem uma nova chamada à fonte.
- **Dado** que uma conversão resulta em valor decimal ou próximo de zero, **Quando** a temperatura é exibida, **Então** ela é arredondada para uma casa decimal, acompanhada da unidade correta e sem sinal negativo quando o valor arredondado for zero.

**RF5 — Estados e recuperação**

- **Dado** que uma busca de cidade ou consulta meteorológica foi iniciada e ainda não foi concluída, **Quando** a interface aguarda a resposta, **Então** apresenta um indicador de carregamento para a operação em andamento.
- **Dado** que uma busca de cidade é concluída sem correspondências, **Quando** o resultado é apresentado, **Então** a interface informa que nenhuma cidade foi encontrada e mantém disponível o controle para iniciar outra busca.
- **Dado** que uma busca ou consulta meteorológica falha por indisponibilidade da rede ou da fonte de dados, **Quando** a falha é apresentada, **Então** a interface informa que a operação não foi concluída e oferece uma ação para tentar novamente.
- **Dado** que uma operação falhou e a ação de nova tentativa está disponível, **Quando** a pessoa aciona essa ação, **Então** a mesma operação é solicitada novamente e a interface apresenta o estado de carregamento seguido do resultado ou da falha da nova tentativa.
- **Dado** que uma chamada à fonte não responde em até 10 segundos, **Quando** o prazo é excedido, **Então** o sistema encerra o estado de carregamento, informa que a operação demorou demais e oferece nova tentativa.
- **Dado** que a tentativa de uma operação está em andamento, **Quando** a pessoa aciona novamente o controle de busca ou de nova tentativa, **Então** o controle não inicia uma chamada duplicada para a mesma operação.
- **Dado** que uma busca ou consulta falha após já haver dados de outra cidade ou busca, **Quando** o erro é exibido, **Então** os dados anteriores não são apresentados como resultado da operação que falhou e permanecem associados somente à localização a que pertencem.

## Non-Functional Requirements

**RNF1 — Responsividade:** os fluxos de busca, seleção e consulta devem estar disponíveis em viewports móveis e desktop, sem exigir rolagem horizontal para acessar os controles ou ler os dados principais. Textos longos e ampliação do conteúdo não devem ocultar controles ou impedir o acesso às informações.

**RNF2 — Usabilidade:** busca, seleção de cidade, clima atual e previsão devem estar organizados de modo que a cidade consultada, as datas e as unidades sejam identificáveis sem ambiguidade.

**RNF3 — Acessibilidade:** controles devem ter nomes acessíveis e funcionar por teclado. Resultados, carregamento, vazio e erro devem ser apresentados com semântica apropriada e conteúdo compreensível por tecnologias assistivas.

**RNF4 — Desempenho percebido:** uma busca ou consulta deve dar feedback visual de carregamento assim que a operação começar; a interface não deve parecer inativa enquanto aguarda a rede. A interface deve aguardar no máximo 10 segundos por chamada à fonte, após os quais deve comunicar timeout e permitir nova tentativa.

**RNF5 — Resiliência:** indisponibilidade da rede ou da fonte de dados não deve deixar a interface sem feedback nem apresentar dados como se fossem atuais quando a consulta falhar. Respostas atrasadas de operações substituídas e dados incompletos não devem ser apresentados como resultados completos ou associados à cidade errada.

**RNF6 — Legibilidade:** datas, temperaturas e unidades devem permanecer legíveis em telas móveis e desktop, sem sobreposição ou corte que impeça sua compreensão.

**RNF7 — Privacidade e acesso:** a aplicação não deve exigir autenticação nem solicitar localização automática do dispositivo nesta versão.

**RNF8 — Localização:** os textos da interface devem estar em pt-BR. Datas e números devem seguir convenções adequadas ao idioma e à unidade selecionada.

## Edge Cases

- Input vazio ou composto apenas por espaços: não consultar a geocodificação, solicitar o nome de uma cidade e manter o campo disponível.
- Nome inexistente ou geocodificação sem resultados: informar que nenhuma cidade foi encontrada, preservar a busca e permitir nova tentativa com outro termo.
- Nome com acentos ou caracteres especiais válidos: preservar o texto e tratá-lo como dado, não como marcação.
- Resultados homônimos ou sem todos os detalhes geográficos: exibir os detalhes retornados, não inventar informações e não selecionar automaticamente uma opção ambígua.
- Falha de rede, indisponibilidade, erro ou timeout de geocodificação ou consulta meteorológica: apresentar erro específico da operação, encerrar o carregamento e oferecer nova tentativa; o limite de espera é 10 segundos.
- Resposta parcial das condições atuais ou previsão: mostrar valores válidos e marcar os ausentes como indisponíveis, sem estimativas; manter cinco datas na previsão. Se datas ou dados não puderem ser interpretados, comunicar erro e permitir nova tentativa.
- Busca mais recente iniciada antes de uma busca anterior terminar: ignorar a resposta antiga e apresentar somente os resultados da busca vigente.
- Unidade alterada durante ou após consulta: aplicar a unidade selecionada aos dados exibidos ou recebidos sem repetir a consulta; arredondar a uma casa decimal e evitar zero negativo.
- Cidade em fuso horário diferente do navegador: determinar “hoje” e os próximos quatro dias pelo fuso horário da cidade retornado pela fonte; sem fuso ou datas válidos, não presumir datas e comunicar falha.
- Viewport estreita, texto longo ou ampliação: permitir leitura e acesso aos controles sem corte impeditivo nem rolagem horizontal.
- Nova tentativa acionada enquanto a mesma operação está em andamento: impedir chamadas duplicadas e manter um único estado de carregamento.

## Assumptions

- O produto é uma aplicação web para navegadores modernos e o uso principal é uma consulta rápida.
- A Open-Meteo será usada para geocodificação e dados meteorológicos, conforme decisão registrada no discovery.
- “Cinco dias” significa hoje mais os quatro dias seguintes.
- A unidade inicial é Celsius.
- A interface da primeira versão é em pt-BR.
- A busca é feita pelo nome da cidade; não há localização automática.
- Não há autenticação nem persistência em servidor.
- A pessoa terá conexão com a internet na maior parte do tempo; não há requisito confirmado de funcionamento offline.

## Risks

| Risco | Impacto | Mitigação de produto |
| --- | --- | --- |
| Indisponibilidade, limite de requisições ou alteração na fonte de dados | Consultas podem falhar ou não ser concluídas. | Comunicar falhas claramente e oferecer nova tentativa; monitorar a disponibilidade da fonte antes da entrega. |
| Cidades homônimas ou resultados geográficos insuficientes | A pessoa pode consultar a localização errada. | Exibir estado ou região e país quando disponíveis antes da seleção. |
| Diferenças entre dispositivos e tamanhos de tela | Controles ou dados podem ficar difíceis de usar. | Validar os fluxos em tamanhos móveis e desktop e garantir leitura sem rolagem horizontal. |
| Falhas de rede | Buscas ou consultas podem parar sem resultado. | Exibir carregamento, erro e ação de nova tentativa. |
| Conversão ou rótulo de unidade incorreto | A pessoa pode interpretar temperaturas de forma errada. | Exibir a unidade junto aos valores e validar consistência da conversão em todas as temperaturas apresentadas. |
| Diferenças de fuso horário entre localização, navegador e fonte | Datas da previsão podem ser deslocadas ou duplicadas. | Usar o fuso horário da cidade retornado pela fonte e validar que as cinco datas estejam em ordem e sem duplicatas. |

## Out of Scope

- Solicitação de localização automática ou uso de GPS do dispositivo.
- Cadastro, login ou qualquer funcionalidade dependente de conta.
- Salvar cidades favoritas, histórico de consultas ou preferências entre sessões.
- Disponibilidade offline ou cache garantido de previsões.
- Alertas meteorológicos ou notificações.
- Previsões horárias; a previsão desta versão é diária.
- Suporte a idiomas além de pt-BR.
- Outras unidades além de Celsius e Fahrenheit.

## Open Questions

- A unidade escolhida deve persistir entre consultas ou sessões, apesar de não haver persistência de servidor definida?
- Quais detalhes adicionais, além de temperatura e condição, devem aparecer no clima atual ou na previsão, como sensação térmica, precipitação e vento?
- Há requisitos de disponibilidade ou acessibilidade além dos descritos nesta especificação?