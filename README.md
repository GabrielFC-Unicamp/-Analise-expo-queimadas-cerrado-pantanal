# Dois biomas, dois regimes de fogo
**Por que o Pantanal e o Cerrado queimam de formas diferentes**

Análise exploratória que cruza focos de calor (INPE) com dados meteorológicos (INMET) para explicar as diferenças no comportamento das queimadas entre o Pantanal e o Cerrado.

![Python](https://img.shields.io/badge/Python-3.x-blue) ![Google Colab](https://img.shields.io/badge/Google-Colab-orange) ![status](https://img.shields.io/badge/status-concluído-brightgreen)

Trabalho em trio · Unicamp · Entrega: 27/11/2026

## Sumário
1. Pergunta central
2. Escopo e decisões
3. Status e divisão de tarefas
4. Fontes de dados
5. Metodologia
6. Notebooks: o que cada um faz
7. Resultados principais
8. Como reproduzir
9. Estrutura do repositório
10. Fluxo de trabalho em trio
11. Limitações conhecidas
12. Diário de decisões
13. Autores e referências

## 1. Pergunta central
O que explica a diferença no comportamento das queimadas entre o Pantanal e o Cerrado: volume, época do ano e relação com seca, umidade e temperatura?

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
| Período | 2020 a 2023 (01/01/2020 a 31/12/2023) |
| Satélite dos focos | AQUA_M-T (satélite de referência do INPE), para a série ser comparável entre os anos |
| Ferramentas | Python (pandas, geopandas, matplotlib/seaborn), Google Colab, GitHub |
| Datas | Tudo em UTC, guardado sem fuso (focos e INMET usam o mesmo padrão) |
| Bioma de cada foco | O do próprio foco (campo Bioma do INPE), não o da estação |
| Raio de associação foco-estação | 100 km como principal; 50 km como teste de sensibilidade |

## 3. Status e divisão de tarefas
| Etapa | Notebook | Responsável | Status |
|---|---|---|---|
| Coleta e limpeza INPE | `01_coleta_limpeza_inpe` | [definir] | ✅ pronto |
| Coleta e limpeza INMET | `02_coleta_limpeza_inmet` | Gabriel | ✅ pronto |
| Junção espacial foco × estação | `03_juncao_espacial` | Maria Luiza | ✅ pronto |
| EDA Pantanal | `04_eda_pantanal` | [definir] | ✅ pronto |
| EDA Cerrado | `05_eda_cerrado` | [definir] | ✅ pronto |
| Comparação e narrativa | `06_comparacao_e_historia` | Todos | ✅ pronto (revisão cruzada pendente) |

Os dois EDAs (04 e 05) são o mesmo notebook, trocando apenas a variável `BIOMA`, para a comparação final ser direta.

## 4. Fontes de dados
**Focos de calor: INPE (BDQueimadas)**
- Portal: https://terrabrasilis.dpi.inpe.br/queimadas/bdqueimadas/
- Exportação: um CSV por ano, todos os satélites e biomas (o filtro é feito no notebook 01)
- Filtros aplicados no notebook: satélite = AQUA_M-T · bioma = Pantanal e Cerrado · período = 01/01/2020 a 31/12/2023
- Colunas usadas: data e hora, latitude, longitude, bioma, estado, município, satélite, FRP, dias sem chuva, precipitação, risco de fogo
- Horário em UTC (por isso o INMET também é tratado em UTC)

**Clima: INMET**
- Portal de dados históricos: https://portal.inmet.gov.br/dadoshistoricos
- Arquivos .zip anuais (2020–2023), um CSV por estação automática (~570–590 estações por ano), medições horárias em UTC
- Variáveis usadas: precipitação, temperatura do ar, umidade relativa, rajada de vento

**Limites dos biomas: IBGE**
- Versão 2025, escala 1:250.000, arquivo `dmRgNtTrBmBr_v2025_vetor_e250K.zip`
- Camada de domínios (`dmRgNtTrBmBr_v2025_dominios_vetor_e250K.shp`), coluna `nm_bm`

Os dados brutos **não** são versionados no Git. Eles ficam no Google Drive (ver Como reproduzir).

## 5. Metodologia
1. **Coleta:** download dos dados com os filtros registrados acima.
2. **Limpeza do INPE:** filtro de satélite e bioma já na leitura, datas em UTC, valores sentinela (-999) viram NaN, remoção de duplicados e de registros fora do período.
3. **Limpeza e agregação diária do INMET:** do horário para uma linha por estação × dia, com -9999 como ausente.
4. **Seleção de estações:** apenas as que ficam dentro ou a até 100 km do Pantanal ou do Cerrado.
5. **Junção espaço-temporal**, que gera duas tabelas:
   - *foco → estação:* cada foco recebe os dados da estação mais próxima que tenha dado naquele dia, dentro do raio escolhido;
   - *painel estação × dia:* número de focos em volta de cada estação a cada dia, incluindo os dias com zero foco. É o que permite comparar dias com e sem fogo (subpergunta 4).
6. **EDA por bioma:** volume, sazonalidade, anos de pico, intensidade (FRP), relação com dias sem chuva, umidade e temperatura, mapas e correlações. A normalização por área usa a área calculada dos polígonos do IBGE em projeção de área equivalente.
7. **Comparação e narrativa:** onde os biomas se parecem, onde divergem e por quê.

## 6. Notebooks: o que cada um faz
**01 · Coleta e limpeza INPE.** Lê os CSVs anuais do BDQueimadas, filtra satélite (AQUA_M-T) e biomas (Pantanal e Cerrado) já na leitura, padroniza datas em UTC, trata -999 como NaN, remove duplicados e recorta 01/01/2020 a 31/12/2023. Saída: `focos_pantanal_cerrado_2020_2023.parquet`.

**02 · Coleta e limpeza INMET.** Transforma os dados horários de ~590 estações em uma tabela diária (mínimo de 18 h de temperatura por dia, -9999 vira NaN, valores impossíveis viram NaN), calcula dias sem chuva e seleciona as estações a até 100 km dos biomas. Saídas: `inmet_diario_pantanal_cerrado.parquet`, `estacoes_pantanal_cerrado.csv`.

**03 · Junção espacial.** Para cada foco, acha a estação INMET mais próxima que tenha dado naquele dia (distância em km, BallTree com métrica haversine). Saídas:
- `focos_com_clima.parquet`: todos os focos, com as colunas do INMET (prefixo `INMET_`), a distância até a estação e as flags `Tem_clima` (≤100 km) e `Dentro_raio_teste` (≤50 km);
- `painel_estacao_dia.parquet`: uma linha por estação × dia com o número de focos em volta (100 km e 50 km, por bioma do foco), incluindo zeros.

**04 e 05 · EDA Pantanal / Cerrado.** Mesmo modelo, trocando `BIOMA`: volume por ano, sazonalidade (mês × ano), anos atípicos, FRP, clima × fogo (dias com e sem foco, probabilidade por faixa de seca e umidade, correlação de Spearman a 100 km e 50 km), mapa e cobertura. Para o clima, usa as estações a até 100 km do bioma.

**06 · Comparação e narrativa.** Gráficos lado a lado dos dois biomas, tabela-resumo, narrativa e sugestão de gráficos para a apresentação. Salva as figuras em `MyDrive/figuras/`.

## 7. Resultados principais
*(números do período 2020–2023, satélite AQUA_M-T; confira com as saídas do notebook 06)*

| | Pantanal | Cerrado |
|---|---|---|
| Focos no período | 38.591 | 234.145 |
| Focos por 1.000 km² por ano (média) | ~64 | ~29 |
| Ano de pico | 2020 (22.116 focos, 57% do total) | 2020 (63.819 focos, 27% do total) |
| Variação entre anos (desvio/média) | 0,91 | 0,10 |
| Meses de pico | Set, Ago, Out | Set, Ago, Out |
| FRP mediano (MW) | ~48 | ~31 |
| Focos com estação INMET a ≤100 km no dia | ~31% | ~79% |

- **Volume:** o Cerrado tem muito mais focos em número, mas o Pantanal tem densidade mais de duas vezes maior por área.
- **Época:** os dois queimam na estação seca, com cerca de dois terços dos focos entre agosto e outubro.
- **Regime:** o Cerrado tem fogo alto e constante; o Pantanal tem anos extremos (2020) e anos quase sem fogo (2022).
- **Clima:** nos dois biomas, os dias com foco são mais secos, mais quentes e menos úmidos; a chance de foco sobe com os dias sem chuva e cai com a umidade. As correlações são moderadas (cerca de 0,1 a 0,3 em módulo) e consistentes entre 100 km e 50 km.
- **Intensidade:** o FRP mediano é maior no Pantanal.

## 8. Como reproduzir
**Focos (INPE)**
1. Exporte do BDQueimadas um CSV por ano (2020 a 2023, todos os satélites e biomas) e coloque em `MyDrive/datasets_queimadas/` (nome no padrão `bdqueimadas_AAAA-01-01_AAAA+1-01-01.csv`).
2. Abra `notebooks/01_coleta_limpeza_inpe.ipynb` no Google Colab e execute em ordem.

**Clima (INMET)**
1. Baixe os .zip anuais do INMET e coloque em `MyDrive/datasets_inmet/` (nome com o ano, ex.: `2022.zip`).
2. Abra `notebooks/02_coleta_limpeza_inmet.ipynb` e execute as seções em ordem. Parâmetros no topo: `ANOS`, `BIOMAS_ALVO`, `MARGEM_KM`.
3. Se a sessão cair: rode a Seção 1 e vá direto para a Seção 8. Para reprocessar um ano, apague o parquet daquele ano.

**Junção e análises**
1. Rode `notebooks/03_juncao_espacial.ipynb` (precisa dos parquets dos notebooks 01 e 02). Gera `focos_com_clima.parquet` e `painel_estacao_dia.parquet` em `MyDrive/datasets_queimadas/`.
2. Rode `04_eda_pantanal.ipynb` e `05_eda_cerrado.ipynb` (mesmo modelo; só muda `BIOMA`).
3. Rode `06_comparacao_e_historia.ipynb`. As figuras ficam em `MyDrive/figuras/`.

## 9. Estrutura do repositório
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

## 10. Fluxo de trabalho em trio
- Um notebook por etapa, com um dono por notebook, para evitar conflitos de merge em `.ipynb`.
- Uma branch por tarefa (`feat/limpeza-inpe`, `feat/eda-pantanal`...) e pull request revisado por pelo menos uma outra pessoa antes de ir para a `main`. Ninguém dá push direto na `main`.
- Antes de commitar notebooks pesados: *Edit → Clear all outputs* (mantendo apenas as saídas dos gráficos finais, se necessário).
- Commits curtos e descritivos, por exemplo: `inmet: agrega horário em diário e trata -9999`.
- Decisões que mudam o recorte (biomas, período, satélite, raio) são combinadas entre os três e registradas no diário de decisões antes de alterar qualquer notebook.

## 11. Limitações conhecidas
- **Causalidade:** o clima explica parte do risco, mas muitos incêndios têm origem humana (manejo agrícola, desmatamento). A análise mostra associação, não causa.
- **Poucas estações no Pantanal:** só 4 estações dentro do bioma e 63% da área a até 100 km de uma estação. Só cerca de 31% dos focos do Pantanal têm estação a até 100 km no dia (cerca de 79% no Cerrado), então os resultados de clima do Pantanal valem sobretudo para a parte do bioma próxima às estações. O raio de 100 km é o principal, com 50 km como teste de sensibilidade.
- **Estações com poucos dados:** cerca de 40% das estações-ano têm menos de 300 dias com dados, e 2021 teve 137 estações sem dados suficientes (arquivos praticamente vazios no próprio INMET). Na junção usa-se a estação mais próxima com dado no dia.
- **Dias sem chuva:** contagem conservadora, que reinicia após dia sem medição e parte de zero no começo da série.
- **Focos de calor não são área queimada:** o número depende do satélite e da hora da passagem. Usamos só o satélite de referência (AQUA_M-T), que serve para comparar anos e biomas, não para contar todos os incêndios.
- **Duas fontes de clima:** as variáveis do INPE (`DiaSemChuva`, `Precipitacao`, `RiscoFogo`) são estimativas do próprio INPE; as do INMET são medições de estação. Elas não são misturadas.
- **Áreas diferentes:** o Cerrado é muito maior que o Pantanal, então comparações de contagem exigem normalização por área.
- **Bioma do foco:** a análise usa o bioma informado pelo INPE no foco, não o da estação.

## 12. Diário de decisões
| Data | Decisão | Motivo |
|---|---|---|
| 20/10/2026 | Comparar Pantanal e Cerrado | Biomas vizinhos com regimes de fogo contrastantes |
| 20/10/2026 | Apenas EDA com narrativa | Escopo do trabalho |
| 20/10/2026 | Dados brutos fora do Git | Arquivos pesados; reprodução documentada |
| 27/10/2026 | Dias com ≥ 18 h de temperatura | Evita médias diárias de dias com poucas medições |
| 27/10/2026 | -9999 vira NaN, sem preencher com 0 | Chuva 0 mm é diferente de sem dado |
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
| 20/11/2026 | Painel de cada bioma usa as estações a até 100 km dele (não só as de dentro) | Só 4 estações ficam dentro do Pantanal; poucas para analisar clima |
| 20/11/2026 | Estação de cada foco = a mais próxima com dado no dia, mantendo todos os focos e marcando os que ficam a ≤100 km e ≤50 km | Permite o teste de sensibilidade sem perder focos |

## 13. Autores e referências
**Autores**
- Gabriel Fernandes Carvalho ([@carvalhoo-gb](https://github.com/carvalhoo-gb))
- Matheus Souza Bueno de Camargo ([@Matheusdev29](https://github.com/Matheusdev29))
- Maria Luiza Gomes Tavares ([@malutavarexx-tech](https://github.com/malutavarexx-tech))

**Referências**
- INPE, BDQueimadas: https://terrabrasilis.dpi.inpe.br/queimadas/bdqueimadas/
- INMET, Dados Históricos: https://portal.inmet.gov.br/dadoshistoricos
- IBGE, Biomas e Sistema Costeiro-Marinho do Brasil (1:250.000): https://www.ibge.gov.br/geociencias/cartas-e-mapas/informacoes-ambientais/15842-biomas.html
- MMA, boletim de combate aos incêndios no Pantanal e na Amazônia (focos por ano, dados do INPE): https://www.gov.br/mma/pt-br/composicao/secd/boletins-combate-aos-incendios/boletim-pantanal-e-amazonia-30-de-agosto.pdf
