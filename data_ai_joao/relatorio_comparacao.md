# Comparação das bases de cobertura vacinal

Data da comparação: 07/11/2026

## Arquivos comparados

| Base | Arquivo | Linhas | Período | Municípios |
|---|---|---:|---|---:|
| Colega | `data_ai_joao/base_completa_vacinacao.parquet` | 2.328.031 | 1994–2025 | 5.569 |
| Sua | `dados_processados/sipni_cobertura_municipios_1994_2024.rds` | 2.625.144 | 1994–2024 | 5.570 códigos |

A comparação principal foi feita em **1994–2022**, período em que as duas bases vêm do SIPNI/TabNet. A chave usada foi **código IBGE de 6 dígitos + ano + indicador**, com de-para entre vacina/dose (colega) e `imuno` (sua base).

## Resultado principal (1994–2022)

- **1.921.924 registros pareados**
- **1.894.423 idênticos = 98,57%**
- Correlação: **0,9865**
- **Excluindo a Penta:** 1.860.833 de 1.861.051 idênticos = **99,988%**
  - Apenas **41 registros** com diferença > 1 p.p.
  - Máxima diferença fora da Penta: **80,99 p.p.** (Uruçuí/PI, 2022, Hepatite B até 30 dias)

Ou seja: **fora a Penta, os dados batem praticamente registro a registro.**

## Penta: a única divergência relevante em 1994–2022

- 60.873 registros pareados
- 33.590 idênticos = **55,18%**
- 27.277 diferenças > 0,01 p.p.; 19.622 diferenças > 1 p.p.
- A divergência se concentra em **2017–2021**:

| Ano | Pareados | % idênticos | Dif. > 1 p.p. | Mediana da dif. | Máx. |
|---:|---:|---:|---:|---:|---:|
| 2012 | 5.216 | 100,0% | 0 | 0,00 | 0,00 |
| 2013 | 5.563 | 60,0% | 1.481 | 0,00 | 114,29 |
| 2014 | 5.563 | 81,2% | 430 | 0,00 | 43,27 |
| 2015 | 5.564 | 87,7% | 221 | 0,00 | 37,07 |
| 2016 | 5.565 | 96,1% | 58 | 0,00 | 54,54 |
| 2017 | 5.567 | 12,5% | 3.634 | 1,78 | 12.500,00 |
| 2018 | 5.567 | 6,7% | 4.416 | 2,90 | 141,67 |
| 2019 | 5.567 | 7,3% | 4.373 | 2,84 | 180,00 |
| 2020 | 5.567 | 16,8% | 3.286 | 1,37 | 54,48 |
| 2021 | 5.567 | 45,3% | 1.701 | 0,22 | 163,79 |
| 2022 | 5.567 | 96,3% | 22 | 0,00 | 5,88 |

A sua base reproduz o TabNet bruto; a do colega parece ter a Penta **recalculada/ajustada** nesses anos. Um caso claro de erro na fonte bruta: **Mojuí dos Campos/PA, 2017, Penta = 12.500%** na sua base (o arquivo `.xlsx` de 2017 já traz esse valor) contra **0%** no colega. Vale limpar esse outlier.

## 2023–2024: os valores não batem

Comparando com o bloco do Ministério da Saúde da sua base:

| Ano | Pareados | % idênticos | Dif. > 1 p.p. | Correlação |
|---:|---:|---:|---:|---:|
| 2023 | 89.072 | 10,88% | 67.957 | 0,895 |
| 2024 | 89.072 | 0,53% | 77.522 | 0,893 |

A correlação é alta (a tendência é parecida), mas **os valores municipais são diferentes** — não são a mesma extração/denominador. O colega também tem **2025** (116.949 registros), que não existe na sua base.

## Diferenças estruturais importantes

1. **Painel completo vs. observado**
   - O colega tem 5.569 municípios em **todos** os anos.
   - Sua base só tem os municípios com registro no ano (ex.: 2.124 em 1994, 5.568 em 2020).
   - Os 44.122 registros extras do colega em 1994–2022 são **99,9% zeros** (preenchimento do painel), principalmente 1994–1996.

2. **2023 duplicado na sua base**
   - Sua tabela empilha SIPNI 2023 + MS 2023.
   - Em 2023, BCG, DTP, Febre Amarela, Hepatite B e Varicela aparecem **duas vezes por município** (11.134 = 2 × 5.567). Escolha um dos blocos antes de analisar.

3. **Indicadores extras no colega (2023–2024)**
   - Hepatite B (até 1 dia), Hepatite B (até 2 dias), Febre Amarela 1º reforço, Varicela 2ª dose, Poliomielite 2º reforço.
   - Sua base não tem esses indicadores.

4. **Municípios só de um lado**
   - Só no colega: 110001, 110002.
   - Só na sua: 421265, 431453.
   - Provável diferença de versão da malha/códigos.

5. **2024 sem nome de município na sua base**
   - As linhas de 2024 têm `nome` e `municipio` como `NA`; o código está em `codigo_municipio`.

## Conclusão

- **1994–2022: os dados batem** (99,99% dos registros, exceto Penta).
- **Penta 2017–2021: revisar** — sua base segue o TabNet bruto; a do colega parece recalculada. Há pelo menos um outlier claro (12.500%).
- **2023–2025: não batem** — são extrações/metodologias diferentes. Não use as duas como se fossem a mesma série.
- Antes de seguir, vale decidir: (a) qual bloco de 2023 manter; (b) se o painel deve ser zero-preenchido como no colega; (c) como tratar a Penta.

## Arquivos gerados

- `data_ai_joao/base_completa_vacinacao.parquet` — arquivo do colega salvo
- `data_ai_joao/comparacao_1994_2022_por_indicador.csv` — estatísticas por vacina
- `data_ai_joao/divergencias_1994_2022.csv` — todas as diferenças > 0,01 p.p.
- `data_ai_joao/penta_por_ano_1994_2022.csv` — resumo anual da Penta
- `data_ai_joao/comparacao_2023_2024_por_indicador.csv` — comparação de 2023–2024
- `data_ai_joao/perfil_comparacao.txt` — perfil bruto das duas bases
