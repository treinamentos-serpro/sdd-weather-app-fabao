# Discovery — Weather App

## Contexto

O produto é uma aplicação web de previsão do tempo. Usuários precisam consultar rapidamente as condições climáticas de uma cidade e a previsão para os próximos dias, em dispositivos móveis ou desktop. O briefing prioriza busca por cidade, clima atual, previsão de cinco dias e escolha entre Celsius e Fahrenheit.

## Requisitos Funcionais

- **RF1 — Buscar cidades:** permitir que o usuário pesquise uma cidade pelo nome e selecione a localização desejada.
- **RF2 — Consultar clima atual:** exibir as condições atuais para a cidade selecionada.
- **RF3 — Consultar previsão:** exibir a previsão do tempo para cinco dias.
- **RF4 — Alterar unidade:** permitir alternar entre Celsius e Fahrenheit e atualizar os valores exibidos.

## Requisitos Não-Funcionais

- **RNF1 — Responsividade:** a interface deve funcionar em dispositivos móveis e em telas desktop.
- **RNF2 — Usabilidade:** busca e consulta do clima devem ser diretas e compreensíveis, com informações organizadas para leitura rápida.
- **RNF3 — Acessibilidade:** controles devem ser utilizáveis por teclado e possuir nomes acessíveis; conteúdo e estados devem ser apresentados semanticamente.
- **RNF4 — Desempenho:** buscas e consultas devem responder em tempo adequado para uma experiência fluida.
- **RNF5 — Resiliência:** indisponibilidade da rede ou da fonte de dados não deve deixar a interface sem feedback; erros devem ser comunicados de forma clara.
- **RNF6 — Legibilidade:** valores, unidades e datas devem permanecer claros em diferentes tamanhos de tela.

## Riscos

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| A fonte de dados pode ficar indisponível ou limitar requisições. | Média | Alto: usuários podem não conseguir consultar as condições. | Tratar erros e limites, apresentar mensagem clara e oferecer nova tentativa. |
| Nomes de cidades podem ser ambíguos ou corresponder a localizações incorretas. | Alta | Médio: previsão exibida para o lugar errado. | Apresentar informações de desambiguação, como estado e país, antes da seleção. |
| A experiência pode variar entre dispositivos e tamanhos de tela. | Média | Médio: conteúdo ou controles podem ficar difíceis de usar. | Projetar mobile-first e verificar a interface em viewports móveis e desktop. |
| Falhas de rede podem interromper buscas e consultas. | Média | Médio: usuário fica sem resultado ou sem saber o que ocorreu. | Exibir estados de carregamento e erro, com ação para tentar novamente. |
| Conversões ou rótulos de unidade podem estar incorretos. | Baixa | Alto: usuário pode tomar decisões com valores enganosos. | Centralizar conversões e cobri-las com testes, incluindo valores limites. |

## Perguntas em Aberto (Open Questions)

1. Qual fonte fornecerá geocodificação e dados meteorológicos? Há custos ou necessidade de chave? **Impacto:** determina integração, custos e requisitos de implantação.
2. O período de cinco dias inclui o dia atual ou começa no dia seguinte? **Impacto:** afeta datas, quantidade de previsões futuras e apresentação.
3. A previsão deve ser diária, horária ou ambas? **Impacto:** define os dados necessários e a densidade da interface.
4. A aplicação deve solicitar localização automática do usuário? **Impacto:** envolve permissões do navegador, privacidade e fluxo inicial.
5. Qual deve ser a unidade inicial, Celsius ou Fahrenheit? **Impacto:** define os valores apresentados na primeira consulta.
6. Quais idiomas devem ser suportados? **Impacto:** afeta textos, formatos de data e estratégia de localização.
7. É necessário manter dados disponíveis offline ou armazenar cache? **Impacto:** altera complexidade, atualização e comportamento sem rede.
8. Usuários precisam salvar cidades favoritas ou consultar histórico? **Impacto:** pode exigir persistência e ampliar o escopo do produto.

## Suposições (Assumptions)

- O produto será uma aplicação web acessível em navegadores modernos.
- O uso principal é uma consulta rápida, sem necessidade de conta ou autenticação.
- Usuários terão acesso à internet na maior parte do tempo.
- A busca será feita pelo nome da cidade; não se presume geolocalização automática.
- Celsius e Fahrenheit são as unidades de temperatura relevantes para a primeira versão.
- O briefing não define fonte de dados, periodicidade de atualização nem armazenamento; esses pontos permanecem em aberto até validação.

## Personas

### Decisor do dia a dia

- **Objetivo principal:** consultar rapidamente o clima atual para decidir como se vestir ou se precisa levar guarda-chuva.
- **Contexto de uso:** principalmente mobile, em intervalos curtos antes de sair de casa ou durante o dia.
- **Métrica de sucesso:** consegue encontrar e entender as condições atuais da cidade em menos de um minuto, sem ajuda.

### Viajante planejador

- **Objetivo principal:** avaliar a previsão dos próximos cinco dias para organizar atividades e bagagem.
- **Contexto de uso:** mobile e desktop; tende a consultar com antecedência, com mais tempo para comparar os dias.
- **Métrica de sucesso:** consegue verificar as condições previstas para cada dia do período e tomar uma decisão de planejamento.

### Pessoa consultando uma cidade diferente

- **Objetivo principal:** buscar uma cidade específica, confirmar que selecionou a localização correta e consultar o clima na unidade que prefere.
- **Contexto de uso:** mobile ou desktop, frequentemente ao comparar cidades ou em contexto de viagem.
- **Métrica de sucesso:** identifica a cidade correta entre possíveis homônimas e compreende os valores exibidos sem precisar converter a temperatura mentalmente.

### Decisões

- Fonte de dados: Open-Meteo (sem API key)
- "5 dias" = hoje + 4 dias
- Unidade padrão: Celsius
- Sem autenticação e sem persistência de servidor
- Idioma da UI: pt-BR