# Catálogo de Dados

## Convenções

- Preços e valores financeiros: `DECIMAL`, em reais (R$).
- Datas: `DATE` no calendário civil brasileiro.
- Chave natural de cotação: `data_pregao + ticker`.
- `NAO_INFORMADO`: referência setorial não encontrada.

## `bronze_cotahist_raw`

Contexto: cópia auditável do COTAHIST. Linhagem: arquivo TXT da B3 para Bronze, sem alteração do conteúdo.

| Campo | Tipo | Descrição/domínio |
|---|---|---|
| raw_line | string | Linha original; largura esperada conforme layout COTAHIST |
| source_file | string | Caminho/nome do arquivo de origem |
| ingestion_ts | timestamp | Momento UTC da ingestão |
| record_hash | string | SHA-256 da linha para rastreabilidade |

## `bronze_setores_raw`

Contexto: referência setorial carregada pelo aluno a partir da classificação pública da B3.

| Campo | Tipo | Descrição/domínio |
|---|---|---|
| ticker | string | Código de negociação em maiúsculas |
| empresa | string | Razão/nome da companhia |
| setor | string | Setor econômico ou `NAO_INFORMADO` |
| subsetor | string | Subsetor econômico |
| segmento | string | Segmento econômico |
| ingestion_ts | timestamp | Momento da ingestão |

## `silver_cotacoes`

Contexto: cotações diárias parseadas, tipadas e deduplicadas. Linhagem: `bronze_cotahist_raw`; preços inteiros da origem divididos por 100.

| Campo | Tipo | Descrição/domínio |
|---|---|---|
| data_pregao | date | Data válida de pregão |
| cod_bdi | string | Código BDI da B3 |
| ticker | string | Código de negociação não vazio |
| tipo_mercado | int | Código do tipo de mercado; MVP mantém mercado à vista |
| nome_resumido | string | Nome abreviado do emissor |
| especificacao | string | Especificação do papel |
| prazo_termo | string | Prazo para mercado a termo, quando aplicável |
| moeda | string | Moeda de referência, normalmente `R$` |
| preco_abertura | decimal(18,2) | >= 0 |
| preco_maximo | decimal(18,2) | >= preço mínimo |
| preco_minimo | decimal(18,2) | >= 0 |
| preco_medio | decimal(18,2) | >= 0 |
| preco_fechamento | decimal(18,2) | > 0 |
| preco_melhor_compra | decimal(18,2) | >= 0 |
| preco_melhor_venda | decimal(18,2) | >= 0 |
| numero_negocios | long | >= 0 |
| quantidade_titulos | long | >= 0 |
| volume_financeiro | decimal(24,2) | >= 0 |
| fator_cotacao | long | > 0 |
| isin | string | Identificador ISIN quando informado |
| source_file | string | Arquivo de origem |
| ingestion_ts | timestamp | Momento da carga Bronze |

## `silver_empresas`

| Campo | Tipo | Descrição/domínio |
|---|---|---|
| ticker | string | Chave única |
| empresa | string | Nome da companhia |
| setor | string | Classificação setorial padronizada |
| subsetor | string | Classificação subsetorial |
| segmento | string | Classificação por segmento |

## `dim_ativo`

Contexto: dimensão descritiva dos ativos. Linhagem: último registro de `silver_cotacoes`, enriquecido por `silver_empresas`.

| Campo | Tipo | Descrição/domínio |
|---|---|---|
| ticker | string | Chave da dimensão |
| nome_resumido | string | Nome da companhia na cotação |
| especificacao | string | Tipo/especificação do ativo |
| isin | string | Código ISIN |
| empresa | string | Nome da referência setorial |
| setor | string | Setor ou `NAO_INFORMADO` |
| subsetor | string | Subsetor ou `NAO_INFORMADO` |
| segmento | string | Segmento ou `NAO_INFORMADO` |

## `fato_cotacao_diaria`

Contexto: fato central para análise. Linhagem: `silver_cotacoes`, com retornos e médias calculados por janela temporal.

| Campo | Tipo | Descrição/domínio |
|---|---|---|
| data_pregao, ticker | date, string | Chave composta |
| preco_abertura/maximo/minimo/fechamento | decimal | OHLC em reais |
| numero_negocios | long | Número de negócios no pregão |
| quantidade_titulos | long | Quantidade negociada |
| volume_financeiro | decimal | Volume em reais |
| retorno_diario | double | `fechamento / fechamento_anterior - 1` |
| mm_50 | double | Média móvel de até 50 pregões |
| mm_200 | double | Média móvel de até 200 pregões |

## `agg_indicadores_ativos`

| Campo | Tipo | Descrição/domínio |
|---|---|---|
| ticker | string | Um registro por ativo |
| primeiro/ultimo_fechamento | double | Preços extremos da janela temporal |
| retorno_12m | double | Retorno simples da janela |
| volatilidade_anual | double | Desvio-padrão diário × raiz de 252 |
| volume_medio_diario | double | Média de volume financeiro |
| numero_pregoes | long | Cobertura temporal |
| ultimo_mm_50/ultimo_mm_200 | double | Médias móveis no último pregão |
| tendencia_50/200 | int | 1 quando fechamento > respectiva média |

## `agg_desempenho_setor`

| Campo | Tipo | Descrição/domínio |
|---|---|---|
| setor | string | Um registro por setor |
| quantidade_ativos | long | Ativos elegíveis |
| volume_medio_diario | double | Soma dos volumes médios dos ativos |
| retorno_mediano_12m | double | Mediana robusta do retorno |
| volatilidade_mediana | double | Mediana da volatilidade anual |
| pct_acima_mm200 | double | Participação dos ativos em tendência positiva |

## `ranking_ativos_analise`

| Campo | Tipo | Descrição/domínio |
|---|---|---|
| ticker, setor | string | Identificação |
| retorno_12m | double | Retorno observado |
| volatilidade_anual | double | Risco histórico |
| volume_medio_diario | double | Liquidez histórica |
| tendencia_200 | int | Sinal de tendência |
| score_retorno/tendencia/liquidez/risco | double | Percentis de 0 a 1 |
| score_final | double | Soma ponderada de 0 a 100 |
| posicao | int | Ordem decrescente do score |

