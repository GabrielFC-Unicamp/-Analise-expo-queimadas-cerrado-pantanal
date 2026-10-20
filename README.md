# Dois biomas, dois regimes de fogo

**Por que o Pantanal e o Cerrado queimam de formas diferentes (2020–2024)**

Análise exploratória de dados que cruza focos de calor (INPE) com dados meteorológicos (INMET) para explicar as diferenças no comportamento das queimadas entre o Pantanal e o Cerrado.

> Trabalho em dupla · Unicamp · Entrega: **[definir: 13/11 ou 27/11]**

---

## 1. Pergunta central

> O que explica a diferença no comportamento das queimadas entre o Pantanal e o Cerrado: volume, época do ano e relação com seca, umidade e temperatura?

### Subperguntas (cada uma vira uma seção da história)

1. **Volume:** quantos focos cada bioma teve por ano? (comparar também de forma normalizada, por área, e não só o número bruto)
2. **Sazonalidade:** em que meses cada bioma queima? Os picos coincidem?
3. **Anos atípicos:** o que foi diferente em 2020 (Pantanal) e em 2024 (ambos)?
4. **Clima:** como dias sem chuva, umidade e temperatura se relacionam com os focos em cada bioma?
5. **Intensidade:** a potência do fogo (FRP) é semelhante nos dois biomas?
6. **Limitações:** o que o clima não explica (causas humanas, cobertura de estações).

## 2. Escopo

| Item | Decisão |
|---|---|
| Tipo de trabalho | Análise exploratória (EDA) com narrativa, sem modelagem preditiva |
| Biomas | Pantanal e Cerrado |
| Período | 01/01/2020 a 31/12/2024 |
| Ferramentas | Python (pandas, matplotlib/seaborn, geopandas ou scikit-learn para junção espacial), Google Colab |
| Versionamento | GitHub (código e documentação; dados brutos fora do repositório) |

## 3. Fontes de dados

### Focos de calor: INPE (BDQueimadas)
- Portal: https://terrabrasilis.dpi.inpe.br/queimadas/bdqueimadas/
- Filtros: bioma = Pantanal e Cerrado · período = 01/01/2020 a 31/12/2024
- Satélite de referência: **[definir, ex.: satélite de referência AQUA_M-T para série histórica comparável]**
- Colunas desejadas: data, latitude, longitude, bioma, estado, município, dias sem chuva, risco de fogo, precipitação, FRP

### Clima: INMET
- Portal de dados históricos: https://portal.inmet.gov.br/dadoshistoricos (arquivos `.zip` anuais com um CSV por estação)
- Alternativa pré-tratada: METBRA25Y (conferir se cobre 2020–2024)
- Variáveis de interesse: temperatura, umidade relativa, precipitação
- Seleção: apenas estações localizadas dentro ou próximas aos limites dos dois biomas

> Os dados brutos **não** são versionados. Para reproduzir, siga os filtros acima e salve os arquivos em `data/raw/`.

## 4. Metodologia

1. **Coleta:** download dos dois conjuntos com os filtros registrados neste README.
2. **Limpeza:** INMET vem com metadados no cabeçalho, separador `;`, decimal com vírgula e valores sentinela (`-9999`) no lugar de dados ausentes. INPE: checar duplicatas, datas e coordenadas inválidas.
3. **Junção espaço-temporal:** associar cada foco à estação meteorológica mais próxima (distância haversine, `BallTree` ou `geopandas.sjoin_nearest`), respeitando um raio máximo de **[definir: 50 ou 100 km]**, e agregar por estação × dia (ou semana/mês).
4. **Análise exploratória por bioma:** sazonalidade, anos de pico, relação com dias sem chuva, umidade e temperatura, mapas de calor, correlações.
5. **Comparação e narrativa:** onde os biomas se parecem, onde divergem e por quê.

## 5. Estrutura do repositório

```
queimadas-pantanal-cerrado/
├── README.md
├── .gitignore
├── data/                  # ignorado pelo Git
│   ├── raw/
│   └── processed/
├── docs/
│   └── decisoes.md        # diário de decisões
├── notebooks/
│   ├── 01_coleta_limpeza_inpe.ipynb
│   ├── 02_coleta_limpeza_inmet.ipynb
│   ├── 03_juncao_espacial.ipynb
│   ├── 04_eda_pantanal.ipynb
│   ├── 05_eda_cerrado.ipynb
│   └── 06_comparacao_e_historia.ipynb
└── src/                   # funções reutilizáveis (opcional)
```

## 6. Divisão de tarefas

| Etapa | Responsável |
|---|---|
| Coleta e limpeza INPE | [colega] |
| Coleta e limpeza INMET | Gabriel |
| Junção espacial | Ambos |
| EDA Pantanal | [definir] |
| EDA Cerrado | [definir] |
| Comparação e narrativa final | Ambos |

## 7. Fluxo de trabalho com Git

- Um notebook por etapa, com um dono por notebook, para evitar conflitos de merge em `.ipynb`.
- Uma branch por tarefa (`feat/limpeza-inmet`, `feat/eda-pantanal`...) e pull request revisado pelo outro antes de ir para a `main`.
- Ninguém dá push direto na `main`.
- Antes de commitar notebooks pesados: *Edit → Clear all outputs* (mantendo apenas as saídas dos gráficos finais, se necessário).
- Mensagens de commit curtas e descritivas, por exemplo: `limpeza: trata -9999 e vírgula decimal no INMET`.

## 8. Cronograma

| Etapa | Prazo |
|---|---|
| Decisões e repositório | até 22/10 |
| Download e limpeza | até 28/10 |
| Junção espacial | até 02/11 |
| EDA por bioma | até 08/11 |
| História, revisão e entrega | até **[13/11 ou 27/11]** |

> Se o prazo for 27/11, estique cada etapa a partir da junção espacial (ver `docs/decisoes.md`).

## 9. Limitações conhecidas

- O clima explica parte do risco, mas muitos incêndios têm **origem humana** (manejo agrícola, desmatamento). A análise mostra associação, não causalidade.
- O Pantanal tem **poucas estações do INMET**; parte dos focos fica longe de qualquer estação. Por isso o raio máximo de associação é registrado e discutido.
- Focos de calor não são área queimada: um foco indica detecção por satélite, e o número depende do satélite e da passagem.
- Cerrado e Pantanal têm áreas muito diferentes, então comparações diretas de contagem exigem normalização.

## 10. Diário de decisões (resumo)

| Data | Decisão | Motivo |
|---|---|---|
| 20/10/2026 | Comparar Pantanal e Cerrado | Biomas vizinhos com regimes de fogo contrastantes; fácil de contar |
| 20/10/2026 | Período 2020–2024 | Inclui o pico de 2020 no Pantanal e o ano crítico de 2024 |
| 20/10/2026 | Apenas EDA com narrativa | Escopo do trabalho |
| 20/10/2026 | Dados brutos fora do Git | Arquivos pesados; reprodução via filtros documentados |

Detalhes e novas decisões ficam em [`docs/decisoes.md`](docs/decisoes.md).

## 11. Autores

- Gabriel Fernandes Carvalho ([@carvalhoo-gb](https://github.com/carvalhoo-gb))
- [Nome do colega]

## 12. Referências

- INPE, BDQueimadas: https://terrabrasilis.dpi.inpe.br/queimadas/bdqueimadas/
- INMET, Dados Históricos: https://portal.inmet.gov.br/dadoshistoricos
- METBRA25Y (dados meteorológicos do Brasil, pré-tratados)
