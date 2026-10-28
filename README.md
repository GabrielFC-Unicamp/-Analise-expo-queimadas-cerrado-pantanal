# Dois biomas, dois regimes de fogo

**Por que o Pantanal e o Cerrado queimam de formas diferentes**

Análise exploratória que cruza focos de calor (INPE) com dados meteorológicos (INMET) para explicar as diferenças no comportamento das queimadas entre o Pantanal e o Cerrado.

![Python](https://img.shields.io/badge/Python-3.x-blue) ![Colab](https://img.shields.io/badge/Google-Colab-orange) ![Status](https://img.shields.io/badge/status-em%20andamento-yellow)

> Trabalho em dupla · Unicamp · Entrega: **[definir: 13/11 ou 27/11]**

---

## Sumário

1. [Pergunta central](#1-pergunta-central)
2. [Escopo e decisões](#2-escopo-e-decisões)
3. [Status do projeto](#3-status-do-projeto)
4. [Fontes de dados](#4-fontes-de-dados)
5. [Metodologia](#5-metodologia)
6. [Dados do INMET: o que o notebook 02 faz](#6-dados-do-inmet-o-que-o-notebook-02-faz)
7. [Como reproduzir](#7-como-reproduzir)
8. [Estrutura do repositório](#8-estrutura-do-repositório)
9. [Fluxo de trabalho em dupla](#9-fluxo-de-trabalho-em-dupla)
10. [Limitações conhecidas](#10-limitações-conhecidas)
11. [Diário de decisões](#11-diário-de-decisões)
12. [Autores e referências](#12-autores-e-referências)

---

## 1. Pergunta central

> O que explica a diferença no comportamento das queimadas entre o Pantanal e o Cerrado: volume, época do ano e relação com seca, umidade e temperatura?

Subperguntas (cada uma vira uma seção da história):

1. **Volume:** quantos focos cada bioma teve por ano? (comparar também normalizado por área)
2. **Sazonalidade:** em que meses cada bioma queima? Os picos coincidem?
3. **Anos atípicos:** o que foi diferente em 2020 (Pantanal) e nos demais anos do período?
4. **Clima:** como dias sem chuva, umidade e temperatura se relacionam com os focos em cada bioma?
5. **Intensidade:** a potência do fogo (FRP) é semelhante nos dois biomas?
6. **Limitações:** o que o clima não explica (causas humanas, cobertura de estações).

## 2. Escopo e decisões

| Item | Decisão |
|---|---|
| Tipo de trabalho | Análise exploratória (EDA) com narrativa, sem modelagem preditiva |
| Biomas | Pantanal e Cerrado |
| Período | **2020 a 2023** (o planejamento inicial incluía 2024, que ficou fora para os dois conjuntos de dados) |
| Ferramentas | Python (pandas, geopandas, matplotlib/seaborn), Google Colab, GitHub |
| Bioma de cada foco | O do próprio foco (campo `Bioma` do INPE), não o da estação |
| Raio de associação foco-estação | 100 km como principal; 50 km como teste de sensibilidade |

## 3. Status do projeto

| Etapa | Notebook | Responsável | Status |
|---|---|---|---|
| Coleta e limpeza INPE | `01_coleta_limpeza_inpe` | [colega] | [a fazer] |
| Coleta e limpeza INMET | `02_coleta_limpeza_inmet` | Gabriel | ✅ pronto |
| Junção espacial foco × estação | `03_juncao_espacial` | Ambos | a fazer |
| EDA Pantanal | `04_eda_pantanal` | [definir] | a fazer |
| EDA Cerrado | `05_eda_cerrado` | [definir] | a fazer |
| Comparação e narrativa | `06_comparacao_e_historia` | Ambos | a fazer |

## 4. Fontes de dados

### Focos de calor: INPE (BDQueimadas)
- Portal: https://terrabrasilis.dpi.inpe.br/queimadas/bdqueimadas/
- Filtros: bioma = Pantanal e Cerrado · período = 01/01/2020 a 31/12/2023
- Satélite de referência: **[definir, para a série ser comparável ao longo dos anos]**
- Colunas desejadas: data, latitude, longitude, bioma, estado, município, dias sem chuva, risco de fogo, precipitação, FRP
- Horário em UTC (por isso o INMET também é tratado em UTC)

### Clima: INMET
- Portal de dados históricos: https://portal.inmet.gov.br/dadoshistoricos
- Arquivos `.zip` anuais (2020–2023), um CSV por estação automática (~570–590 estações por ano), medições **horárias** em UTC
- Variáveis usadas: precipitação, temperatura do ar, umidade relativa, rajada de vento

### Limites dos biomas: IBGE
- Versão 2025, escala 1:250.000, arquivo `dmRgNtTrBmBr_v2025_vetor_e250K.zip`
- Camada de **domínios** (`dmRgNtTrBmBr_v2025_dominios_vetor_e250K.shp`), coluna `nm_bm`

> Os dados brutos **não** são versionados no Git. Eles ficam no Google Drive (ver [Como reproduzir](#7-como-reproduzir)).

## 5. Metodologia

1. **Coleta:** download dos dados com os filtros registrados acima.
2. **Limpeza:** INMET (cabeçalho com metadados, separador `;`, decimal com vírgula, `-9999` como ausente) e INPE (duplicatas, datas e coordenadas inválidas).
3. **Agregação diária do INMET:** do horário para uma linha por estação × dia.
4. **Seleção de estações:** apenas as que ficam dentro ou a até 100 km do Pantanal ou do Cerrado.
5. **Junção espaço-temporal:** cada foco recebe os dados da estação mais próxima **que tenha dado naquele dia**, dentro do raio escolhido.
6. **EDA por bioma:** sazonalidade, anos de pico, relação com dias sem chuva, umidade e temperatura, mapas, correlações.
7. **Comparação e narrativa:** onde os biomas se parecem, onde divergem e por quê.

## 6. Dados do INMET: o que o notebook 02 faz

O notebook `02_coleta_limpeza_inmet.ipynb` transforma os dados horários de ~590 estações em uma tabela **diária** e seleciona as estações relevantes.

### Etapas

| Seção | O que faz |
|---|---|
| 1. Configuração | Monta o Google Drive, define parâmetros (anos, biomas, margem) e caminhos |
| 2. Carga | Copia os `.zip` anuais do Drive para o disco local do Colab (muito mais rápido) |
| 3. Funções | Lê, limpa e agrega **uma** estação por vez |
| 4. Processamento | Roda todas as estações ano a ano, com checkpoint no Drive |
| 5. Consolidação | Junta os anos e confere a qualidade (duplicatas, nulos, faixas, cobertura) |
| 6. Dias sem chuva | Calcula a sequência de dias secos consecutivos por estação |
| 7. Diário nacional | Salva o resultado de todas as estações |
| 8. Seleção por bioma | Baixa os limites do IBGE, calcula a distância de cada estação a cada bioma e seleciona |
| 9. Resumo | Resultados de referência e notas para este README |

### Regras de limpeza (decisões que impactam os resultados)

- `-9999` vira `NaN`. **Não** se preenche com 0, pois chuva 0 mm é diferente de "sem dado".
- Horário tratado em **UTC**, o mesmo padrão do BDQueimadas.
- Valores impossíveis viram `NaN`: umidade fora de 0–100, temperatura fora de −10 a 50 °C, chuva negativa.
- Um dia só entra se tiver **pelo menos 18 de 24 horas** de temperatura.
- `Dias_sem_chuva`: dia com **≥ 1 mm** conta como chuvoso. Dia sem medição de chuva e lacunas de datas **reiniciam** a contagem, então o valor é uma contagem conservadora de dias secos confirmados. No início de 2020 a contagem parte de zero.

### Saídas (em `datasets_inmet/processed/` no Drive)

| Arquivo | Conteúdo |
|---|---|
| `inmet_diario_AAAA.parquet` | Checkpoint por ano |
| `inmet_diario_final.parquet` | Diário de todas as estações (583.649 linhas, 590 estações) |
| `estacoes.csv` | Código, UF, nome e coordenadas de cada estação |
| `estacoes_pantanal_cerrado.csv` | Estações selecionadas, com distância até cada bioma |
| `inmet_diario_pantanal_cerrado.parquet` | Diário só das estações selecionadas (entrada da junção) |

Colunas do diário: `Data`, `Precip_total`, `Temp_max`, `Temp_min`, `Temp_med`, `Umid_min`, `Umid_med`, `Vento_max`, `N_horas`, `Codigo`, `UF`, `Estacao`, `Latitude`, `Longitude`, `Dias_sem_chuva`.

### Resultados de referência (2020–2023)

| Item | Valor |
|---|---|
| Estações nos zips (2020 / 2021 / 2022 / 2023) | 589 / 588 / 567 / 567 |
| Estações no diário (2020 / 2021 / 2022 / 2023) | 558 / 451 / 526 / 547 |
| Estações selecionadas (margem de 100 km) | **277** de 590 |
| Dentro do Cerrado / dentro do Pantanal | **169 / 4** |
| Estações dentro do Pantanal | A717 Nhumirim, A723 Porto Murtinho, A724 Corumbá, A941 Cáceres |
| Linhas do diário selecionado | 277.891 |

### Cobertura espacial das estações

Porcentagem da **área** do bioma que fica a até X km de uma estação:

| Distância | Pantanal | Cerrado |
|---|---|---|
| até 25 km | 4,9% | 16,1% |
| até 50 km | 18,3% | 53,8% |
| até 100 km | 63,0% | 97,1% |
| até 150 km | 93,4% | 99,9% |
| Distância mediana até a estação mais próxima | 86 km | 47 km |

A cobertura do Pantanal é bem pior que a do Cerrado. Isso precisa ser considerado na interpretação, pois o clima associado a um foco do Pantanal vem, em média, de uma estação mais distante.

## 7. Como reproduzir

1. Baixe os `.zip` anuais do INMET e coloque em `MyDrive/datasets_inmet/` (nome com o ano, ex.: `2022.zip`).
2. Abra `notebooks/02_coleta_limpeza_inmet.ipynb` no Google Colab e execute as seções em ordem. Parâmetros no topo: `ANOS`, `BIOMAS_ALVO`, `MARGEM_KM`.
3. Se a sessão cair: rode a Seção 1 e vá direto para a Seção 8. Na Seção 4, anos já processados são pulados.
4. Para reprocessar um ano (por exemplo, depois de mudar uma função), apague o parquet daquele ano.
5. Para incluir outro ano: adicione-o em `ANOS`, coloque o `.zip` na pasta e rode de novo. Apenas o ano novo será processado.

## 8. Estrutura do repositório

```
queimadas-pantanal-cerrado/
├── README.md
├── .gitignore
├── data/                  # ignorado pelo Git (os dados ficam no Drive)
├── docs/
│   └── decisoes.md        # diário de decisões detalhado
├── notebooks/
│   ├── 01_coleta_limpeza_inpe.ipynb
│   ├── 02_coleta_limpeza_inmet.ipynb
│   ├── 03_juncao_espacial.ipynb
│   ├── 04_eda_pantanal.ipynb
│   ├── 05_eda_cerrado.ipynb
│   └── 06_comparacao_e_historia.ipynb
└── src/                   # funções reutilizáveis (opcional)
```

## 9. Fluxo de trabalho em dupla

- Um notebook por etapa, com um dono por notebook, para evitar conflitos de merge em `.ipynb`.
- Uma branch por tarefa (`feat/limpeza-inmet`, `feat/eda-pantanal`...) e pull request revisado pelo outro antes de ir para a `main`. Ninguém dá push direto na `main`.
- Antes de commitar notebooks pesados: *Edit → Clear all outputs* (mantendo apenas as saídas dos gráficos finais, se necessário).
- Commits curtos e descritivos, por exemplo: `inmet: agrega horário em diário e trata -9999`.

## 10. Limitações conhecidas

- **Causalidade:** o clima explica parte do risco, mas muitos incêndios têm origem humana (manejo agrícola, desmatamento). A análise mostra associação, não causa.
- **Poucas estações no Pantanal:** só 4 estações dentro do bioma e 63% da área a até 100 km de uma estação. O raio de 100 km é o principal, com 50 km como teste de sensibilidade.
- **Estações com poucos dados:** cerca de 40% das estações-ano têm menos de 300 dias com dados, e 2021 teve 137 estações sem dados suficientes (arquivos praticamente vazios no próprio INMET). Na junção usa-se a estação mais próxima **com dado no dia**.
- **Dias sem chuva:** contagem conservadora, que reinicia após dia sem medição e parte de zero no começo da série.
- **Focos de calor não são área queimada:** o número depende do satélite e da hora da passagem.
- **Áreas diferentes:** o Cerrado é muito maior que o Pantanal, então comparações de contagem exigem normalização por área.
- **Bioma do foco:** a análise usa o bioma informado pelo INPE no foco, não o da estação.

## 11. Diário de decisões

| Data | Decisão | Motivo |
|---|---|---|
| 20/10/2026 | Comparar Pantanal e Cerrado | Biomas vizinhos com regimes de fogo contrastantes |
| 20/10/2026 | Apenas EDA com narrativa | Escopo do trabalho |
| 20/10/2026 | Dados brutos fora do Git | Arquivos pesados; reprodução documentada |
| 27/10/2026 | Dias com ≥ 18 h de temperatura | Evita médias diárias de dias com poucas medições |
| 27/10/2026 | `-9999` vira `NaN`, sem preencher com 0 | Chuva 0 mm é diferente de sem dado |
| 27/10/2026 | Estações a até 100 km dos biomas | Compensa a baixa densidade de estações, principalmente no Pantanal |
| 27/10/2026 | Limites dos biomas do IBGE 2025 (1:250.000) | Fonte oficial |
| 28/10/2026 | Período final: 2020 a 2023 | Alinha com o INMET já processado; 2024 fica fora |



## 12. Autores e referências

**Autores**
- Gabriel Fernandes Carvalho ([@carvalhoo-gb](https://github.com/carvalhoo-gb))
- Matheus Souza Bueno de Camargo ([@Matheusdev29](https://github.com/Matheusdev29))
**Referências**
- INPE, BDQueimadas: https://terrabrasilis.dpi.inpe.br/queimadas/bdqueimadas/
- INMET, Dados Históricos: https://portal.inmet.gov.br/dadoshistoricos
- IBGE, Biomas e Sistema Costeiro-Marinho do Brasil (1:250.000)
