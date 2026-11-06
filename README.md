# Dois biomas, dois regimes de fogo

**Por que o Pantanal e o Cerrado queimam de formas diferentes**

Análise exploratória que cruza focos de calor (INPE) com dados meteorológicos (INMET) para explicar as diferenças no comportamento das queimadas entre o Pantanal e o Cerrado.

![Python](https://img.shields.io/badge/Python-3.x-blue) ![Colab](https://img.shields.io/badge/Google-Colab-orange) ![Status](https://img.shields.io/badge/status-em%20andamento-yellow)

> Trabalho em trio · Unicamp · Entrega: **27/11/2026**

---

## Sumário

1. [Pergunta central](#1-pergunta-central)
2. [Escopo e decisões](#2-escopo-e-decisões)
3. [Status e divisão de tarefas](#3-status-e-divisão-de-tarefas)
4. [Cronograma](#4-cronograma)
5. [Fontes de dados](#5-fontes-de-dados)
6. [Metodologia](#6-metodologia)
7. [Dados do INPE: o que o notebook 01 faz](#7-dados-do-inpe-o-que-o-notebook-01-faz)
8. [Dados do INMET: o que o notebook 02 faz](#8-dados-do-inmet-o-que-o-notebook-02-faz)
9. [Como reproduzir](#9-como-reproduzir)
10. [Estrutura do repositório](#10-estrutura-do-repositório)
11. [Fluxo de trabalho em trio](#11-fluxo-de-trabalho-em-trio)
12. [Limitações conhecidas](#12-limitações-conhecidas)
13. [Diário de decisões](#13-diário-de-decisões)
14. [Autores e referências](#14-autores-e-referências)

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
| Período | **2020 a 2023** (01/01/2020 a 31/12/2023) |
| Satélite dos focos | **AQUA_M-T** (satélite de referência do INPE), para a série ser comparável entre os anos |
| Ferramentas | Python (pandas, geopandas, matplotlib/seaborn), Google Colab, GitHub |
| Datas | Tudo em **UTC**, guardado **sem fuso** (focos e INMET usam o mesmo padrão) |
| Bioma de cada foco | O do próprio foco (campo `Bioma` do INPE), não o da estação |
| Raio de associação foco-estação | 100 km como principal; 50 km como teste de sensibilidade |

## 3. Status e divisão de tarefas

| Etapa | Notebook | Responsável | Status |
|---|---|---|---|
| Coleta e limpeza INPE | `01_coleta_limpeza_inpe` | [definir] | em ajustes finais |
| Coleta e limpeza INMET | `02_coleta_limpeza_inmet` | Gabriel | ✅ pronto |
| Junção espacial foco × estação | `03_juncao_espacial` | Gabriel | a fazer (próximo) |
| EDA Pantanal | `04_eda_pantanal` | [definir] | a fazer |
| EDA Cerrado | `05_eda_cerrado` | [definir] | a fazer |
| Comparação e narrativa | `06_comparacao_e_historia` | Todos | a fazer |

As subperguntas 1, 2, 3 e 5 (volume, sazonalidade, anos atípicos e FRP) usam **só os focos do INPE** e podem começar antes da junção. A subpergunta 4 (clima × fogo) depende do notebook 03.

Os dois EDAs (04 e 05) seguem o **mesmo modelo de notebook**, trocando apenas o bioma, para a comparação final ser direta.

## 4. Cronograma

| Período | Meta |
|---|---|
| até 09/11 | Notebook 01 corrigido, README e repositório no ar, notebook 03 escrito |
| 10/11 a 13/11 | Junção (03) rodada e validada, raio decidido; EDA só com INPE (subperguntas 1, 2, 3 e 5) |
| 14/11 a 20/11 | Clima × fogo por bioma (subpergunta 4) e comparação Pantanal × Cerrado |
| 21/11 a 24/11 | Narrativa, resultados no README e revisão cruzada |
| 25/11 e 26/11 | Folga: rodar tudo do zero, conferir e polir |
| **27/11** | **Entrega** |

## 5. Fontes de dados

### Focos de calor: INPE (BDQueimadas)
- Portal: https://terrabrasilis.dpi.inpe.br/queimadas/bdqueimadas/
- Exportação: um CSV por ano, todos os satélites e biomas (o filtro é feito no notebook 01)
- Filtros aplicados no notebook: satélite = **AQUA_M-T** · bioma = Pantanal e Cerrado · período = 01/01/2020 a 31/12/2023
- Colunas usadas: data e hora, latitude, longitude, bioma, estado, município, satélite, FRP, dias sem chuva, precipitação, risco de fogo
- Horário em UTC (por isso o INMET também é tratado em UTC)

### Clima: INMET
- Portal de dados históricos: https://portal.inmet.gov.br/dadoshistoricos
- Arquivos `.zip` anuais (2020–2023), um CSV por estação automática (~570–590 estações por ano), medições **horárias** em UTC
- Variáveis usadas: precipitação, temperatura do ar, umidade relativa, rajada de vento

### Limites dos biomas: IBGE
- Versão 2025, escala 1:250.000, arquivo `dmRgNtTrBmBr_v2025_vetor_e250K.zip`
- Camada de **domínios** (`dmRgNtTrBmBr_v2025_dominios_vetor_e250K.shp`), coluna `nm_bm`

> Os dados brutos **não** são versionados no Git. Eles ficam no Google Drive (ver [Como reproduzir](#9-como-reproduzir)).

## 6. Metodologia

1. **Coleta:** download dos dados com os filtros registrados acima.
2. **Limpeza do INPE:** filtro de satélite e bioma já na leitura, datas em UTC, valores sentinela (`-999`) viram `NaN`, remoção de duplicados e de registros fora do período.
3. **Limpeza e agregação diária do INMET:** do horário para uma linha por estação × dia, com `-9999` como ausente.
4. **Seleção de estações:** apenas as que ficam dentro ou a até 100 km do Pantanal ou do Cerrado.
5. **Junção espaço-temporal**, que gera duas tabelas:
   - **foco → estação:** cada foco recebe os dados da estação mais próxima **que tenha dado naquele dia**, dentro do raio escolhido;
   - **painel estação × dia:** número de focos em volta de cada estação a cada dia, **incluindo os dias com zero foco**. É o que permite comparar dias com e sem fogo (subpergunta 4).
6. **EDA por bioma:** volume, sazonalidade, anos de pico, intensidade (FRP), relação com dias sem chuva, umidade e temperatura, mapas e correlações. A normalização por área usa a área calculada dos polígonos do IBGE em projeção de área equivalente.
7. **Comparação e narrativa:** onde os biomas se parecem, onde divergem e por quê.

## 7. Dados do INPE: o que o notebook 01 faz

O notebook `01_coleta_limpeza_inpe.ipynb` transforma os CSVs anuais do BDQueimadas em uma tabela única de focos do Pantanal e do Cerrado.

### Etapas

1. **Leitura econômica:** cada CSV anual é lido só com as colunas necessárias e filtrado **logo na leitura**. A base completa (todos os satélites e todos os biomas) tem cerca de 18,8 milhões de registros de 16 satélites; filtrar antes de juntar evita estourar a memória do Colab.
2. **Filtros:** satélite = `AQUA_M-T` (antes, mostra a contagem de todos os satélites para registrar a decisão) e bioma ∈ {Pantanal, Cerrado}.
3. **Padronização:** remove espaços e o caractere invisível (BOM) dos textos e datas.
4. **Datas:** `DataHora` em UTC, sem fuso; `Data` é o dia. Registros com data inválida são descartados, e o recorte é exatamente 01/01/2020 a 31/12/2023 (o arquivo de 2023 traz alguns focos de 01/01/2024, removidos).
5. **Valores ausentes:** `-999` vira `NaN`. **Não** se preenche com 0.
6. **Duplicados** removidos, com contagem impressa. Coordenadas em `float64` para não perder precisão na junção espacial.

### Saída

`focos_pantanal_cerrado_2020_2023.parquet` (em `datasets_queimadas/` no Drive), com as colunas:
`DataHora`, `Data`, `Latitude`, `Longitude`, `Bioma`, `Estado`, `Municipio`, `Satelite`, `FRP`, `DiaSemChuva`, `Precipitacao`, `RiscoFogo`.

> `DiaSemChuva`, `Precipitacao` e `RiscoFogo` são **estimativas do próprio INPE**, não medições de estação. Na junção, as variáveis do INMET entram com nomes diferentes. O `RiscoFogo` é uma previsão do INPE baseada em clima, então serve como comparação, não como explicação independente do fogo.

### Resultados de referência (focos por ano e bioma)

| Ano | Cerrado | Pantanal |
|---|---|---|
| 2020 | 63.819 | 22.116 |
| 2021 | 62.727 | 8.258 |
| 2022 | 56.884 | 1.637 |
| 2023 | 50.713 | 6.580 |

Os totais anuais do **Pantanal** coincidem com os divulgados em boletins do governo com dados do INPE, o que valida o satélite de referência, o filtro de bioma e o período.

## 8. Dados do INMET: o que o notebook 02 faz

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

## 9. Como reproduzir

**Focos (INPE)**
1. Exporte do BDQueimadas um CSV por ano (2020 a 2023, todos os satélites e biomas) e coloque em `MyDrive/datasets_queimadas/` (nome no padrão `bdqueimadas_AAAA-01-01_AAAA+1-01-01.csv`).
2. Abra `notebooks/01_coleta_limpeza_inpe.ipynb` no Google Colab e execute em ordem. A saída é o parquet de focos descrito na seção 7.

**Clima (INMET)**
1. Baixe os `.zip` anuais do INMET e coloque em `MyDrive/datasets_inmet/` (nome com o ano, ex.: `2022.zip`).
2. Abra `notebooks/02_coleta_limpeza_inmet.ipynb` no Google Colab e execute as seções em ordem. Parâmetros no topo: `ANOS`, `BIOMAS_ALVO`, `MARGEM_KM`.
3. Se a sessão cair: rode a Seção 1 e vá direto para a Seção 8. Na Seção 4, anos já processados são pulados.
4. Para reprocessar um ano (por exemplo, depois de mudar uma função), apague o parquet daquele ano.
5. Para incluir outro ano: adicione-o em `ANOS`, coloque o `.zip` na pasta e rode de novo. Apenas o ano novo será processado.

**Junção e análises:** notebooks `03` a `06`, em ordem (instruções serão adicionadas conforme ficarem prontos).

## 10. Estrutura do repositório

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

## 11. Fluxo de trabalho em trio

- Um notebook por etapa, com um dono por notebook, para evitar conflitos de merge em `.ipynb`.
- Uma branch por tarefa (`feat/limpeza-inpe`, `feat/eda-pantanal`...) e pull request revisado por **pelo menos uma outra pessoa** antes de ir para a `main`. Ninguém dá push direto na `main`.
- Antes de commitar notebooks pesados: *Edit → Clear all outputs* (mantendo apenas as saídas dos gráficos finais, se necessário).
- Commits curtos e descritivos, por exemplo: `inmet: agrega horário em diário e trata -9999`.
- Decisões que mudam o recorte (biomas, período, satélite, raio) são combinadas entre os três e registradas no diário de decisões antes de alterar qualquer notebook.

## 12. Limitações conhecidas

- **Causalidade:** o clima explica parte do risco, mas muitos incêndios têm origem humana (manejo agrícola, desmatamento). A análise mostra associação, não causa.
- **Poucas estações no Pantanal:** só 4 estações dentro do bioma e 63% da área a até 100 km de uma estação. O raio de 100 km é o principal, com 50 km como teste de sensibilidade.
- **Estações com poucos dados:** cerca de 40% das estações-ano têm menos de 300 dias com dados, e 2021 teve 137 estações sem dados suficientes (arquivos praticamente vazios no próprio INMET). Na junção usa-se a estação mais próxima **com dado no dia**.
- **Dias sem chuva:** contagem conservadora, que reinicia após dia sem medição e parte de zero no começo da série.
- **Focos de calor não são área queimada:** o número depende do satélite e da hora da passagem. Usamos só o satélite de referência (AQUA_M-T), que vê o território em poucas passagens por dia; ele serve para comparar anos e biomas, não para contar todos os incêndios.
- **Duas fontes de clima:** as variáveis do INPE (`DiaSemChuva`, `Precipitacao`, `RiscoFogo`) são estimativas do próprio INPE; as do INMET são medições de estação. Elas não são misturadas.
- **Áreas diferentes:** o Cerrado é muito maior que o Pantanal, então comparações de contagem exigem normalização por área.
- **Bioma do foco:** a análise usa o bioma informado pelo INPE no foco, não o da estação.

## 13. Diário de decisões

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
| 06/11/2026 | Satélite de referência: AQUA_M-T | Série comparável entre os anos; é o satélite de referência do INPE |
| 06/11/2026 | Filtrar satélite e bioma na leitura dos CSVs | A base completa tem ~18,8 milhões de registros; evita estourar a memória |
| 06/11/2026 | Datas dos focos em UTC, sem fuso | Mesmo padrão do INMET; evita erro na junção |
| 06/11/2026 | Recorte exato de 01/01/2020 a 31/12/2023 | O arquivo de 2023 traz focos de 01/01/2024 |
| 06/11/2026 | Variáveis de clima do INPE separadas das do INMET | São estimativas do INPE, não medições de estação |
| 06/11/2026 | Junção gera duas tabelas (foco → estação e painel estação × dia, com zeros) | A subpergunta 4 precisa de dias sem fogo para comparar |
| 06/11/2026 | Equipe de três pessoas e entrega em 27/11 | Cronograma reorganizado |

## 14. Autores e referências

**Autores**
- Gabriel Fernandes Carvalho ([@carvalhoo-gb](https://github.com/carvalhoo-gb))
- Matheus Souza Bueno de Camargo ([@Matheusdev29](https://github.com/Matheusdev29))
- Maria Luiza Gomes Tavares ([@malutavarexx-tech](https://github.com/malutavarexx-tech))

**Referências**
- INPE, BDQueimadas: https://terrabrasilis.dpi.inpe.br/queimadas/bdqueimadas/
- INMET, Dados Históricos: https://portal.inmet.gov.br/dadoshistoricos
- IBGE, Biomas e Sistema Costeiro-Marinho do Brasil (1:250.000): https://www.ibge.gov.br/geociencias/cartas-e-mapas/informacoes-ambientais/15842-biomas.html
- MMA, boletim de combate aos incêndios no Pantanal e na Amazônia (focos por ano, dados do INPE): https://www.gov.br/mma/pt-br/composicao/secd/boletins-combate-aos-incendios/boletim-pantanal-e-amazonia-30-de-agosto.pdf
