# csstrike_data

Análise exploratória de estatísticas de jogadores profissionais de CS:GO, com foco na cena brasileira. O projeto limpa o dataset, produz análises comparativas e exporta tabelas prontas para dashboards em Power BI.

## Sumário

- [Objetivo](#objetivo)
- [Dados](#dados)
- [Arquitetura do pipeline](#arquitetura-do-pipeline)
- [Notebooks](#notebooks)
- [Saídas para Power BI](#saídas-para-power-bi)
- [Modelo de machine learning](#modelo-de-machine-learning)
- [Como executar](#como-executar)
- [Estrutura do repositório](#estrutura-do-repositório)

## Objetivo

Responder, com base em dados, como o Brasil se posiciona no cenário mundial de CS:GO: volume de jogadores, relação entre idade e desempenho, melhores jogadores e times, e perfil de armas, comparando sempre com a média global.

## Dados

Fonte: `csgo_players.csv`, com 811 jogadores e 41 colunas de estatísticas de carreira (kills, deaths, headshot, dano por round, opening duels, kills por arma, rating HLTV, entre outras).

Problemas tratados na limpeza:

- Colunas de percentual (`headshot_percentage`, `team_win_percent_after_first_kill`, `first_kill_in_won_rounds`) chegavam como texto com `%` e foram convertidas para número.
- `kill_to_death_diff` vinha inválida e foi recalculada como `total_kills - total_deaths`.
- `current_team` possui valores nulos (jogadores sem time); foi criado o indicador `has_team`.

Colunas derivadas: `is_brazilian`, `age_group` (`<=20`, `21-23`, `24-26`, `27-29`, `30+`) e a participação de cada arma nas kills (`rifle_share`, `sniper_share`, `smg_share`, `pistol_share`, `grenade_share`, `other_share`).

## Arquitetura do pipeline

```
csgo_players.csv  ->  exploration.ipynb  ->  brazil_analysis.ipynb  ->  powerbi/*.csv  ->  Power BI
   (dado bruto)       (EDA global)           (recorte BR, KPIs, ML)     (camada de consumo)   (dashboard)
```

Princípios adotados:

- **Dado bruto imutável**: o CSV original não é alterado; toda limpeza ocorre nos notebooks, de forma reproduzível.
- **Camada de consumo separada**: o Power BI lê apenas os arquivos de `powerbi/`, já limpos e agregados.
- **Tabelas em formato longo para o radar**: `player_radar.csv` normaliza cada métrica de 0 a 100 (min-max sobre os 811 jogadores), permitindo comparar qualquer jogador com justiça.
- **Filtros de amostra mínima**: países com pelo menos 10 jogadores e times com pelo menos 5 (ou 3 brasileiros, no recorte BR), para evitar distorção por amostras pequenas.
- **Células independentes**: as etapas de exportação e de machine learning recarregam e limpam o CSV se necessário.

## Notebooks

| Notebook                | Conteúdo                                                                                           |
| ----------------------- | -------------------------------------------------------------------------------------------------- |
| `exploration.ipynb`     | Qualidade dos dados, limpeza, distribuições, correlações, comparação por país e time, perfil por arma, idade vs. desempenho, Brasil vs. mundo, top 5 times. |
| `brazil_analysis.ipynb` | Panorama da cena brasileira, idade vs. desempenho, melhores jogadores e times BR, melhor brasileiro vs. melhor do mundo, KPIs, exportações para Power BI e modelo Naive Bayes. |

## Saídas para Power BI

Arquivos gerados em `powerbi/` (CSV em UTF-8 com BOM):

| Arquivo                  | Conteúdo                                              |
| ------------------------ | ----------------------------------------------------- |
| `players_full.csv`       | Dataset limpo de todos os jogadores                   |
| `players_brazil.csv`     | Subconjunto de jogadores brasileiros                  |
| `country_stats.csv`      | Estatísticas agregadas por país                       |
| `team_stats_world.csv`   | Estatísticas agregadas por time (mundo)               |
| `team_stats_brazil.csv`  | Estatísticas agregadas por time com brasileiros       |
| `age_group_compare.csv`  | Métricas por faixa etária, Brasil e mundo             |
| `kpis_brazil.csv`        | KPIs do Brasil comparados ao mundo                    |
| `best_player_compare.csv`| Melhor brasileiro vs. melhor jogador do mundo         |
| `best_team_compare.csv`  | Melhor time brasileiro vs. melhor time do mundo       |
| `player_radar.csv`       | Métricas normalizadas em formato longo para o radar   |

## Modelo de machine learning

Classificador Gaussian Naive Bayes (scikit-learn) que prevê o papel do jogador:

- **Classe**: `AWPer` quando as kills de sniper superam as de rifle; caso contrário, `Rifler`.
- **Previsores**: idade, país, time e métricas de desempenho (kills e mortes por round, K/D, headshot, dano por round, opening kill rating, assistências).
- **Divisão**: 75% treino e 25% teste, estratificada, `random_state=42`.
- **Resultado no conjunto de teste**: acurácia de 97,5%.

A classe `AWPer` representa cerca de 17% dos jogadores. Como o rótulo é derivado de `sniper_kills` e `rifle_kills`, que se relacionam com o perfil de armas, o resultado indica a consistência do perfil e não capacidade preditiva sobre dados externos.

## Como executar

Pré-requisitos: Python 3.10+ e Jupyter.

```bash
python -m venv .venv
source .venv/bin/activate
pip install pandas numpy matplotlib scikit-learn jupyter
jupyter notebook
```

Execute `exploration.ipynb` e depois `brazil_analysis.ipynb`. A última célula de exportação recria a pasta `powerbi/`.

## Estrutura do repositório

```
csgo_players.csv       dataset bruto
exploration.ipynb      análise exploratória global
brazil_analysis.ipynb  análise da cena brasileira e modelo
powerbi/               tabelas de consumo do dashboard
```
