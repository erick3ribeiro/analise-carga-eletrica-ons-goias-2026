# Modelagem de Dados — Análise da Carga Elétrica ONS | Goiás 2026

## 1. Visão geral

O modelo semântico do projeto foi desenvolvido no Microsoft Power BI para analisar registros de carga elétrica do estado de Goiás, obtidos da API de carga verificada do Operador Nacional do Sistema Elétrico (ONS).

A estrutura identificada contém a tabela **`Carga ONS`**, com os campos provenientes da consulta Power Query e colunas calculadas destinadas à organização temporal dos registros e à análise por período do dia.

## 2. Origem e preparação dos dados

A consulta `Carga ONS` utiliza o endpoint de carga verificada do ONS, com os parâmetros de área de carga `GO` e intervalo de 01/09/2026 a 30/09/2026. No Power Query, os registros JSON são convertidos em tabela, expandidos, tipados e recebem a coluna `DataHora_Goias`, calculada pela subtração de três horas da referência `din_referenciautc`.

O detalhamento das transformações e o código M estão registrados em [`POWER_QUERY.md`](POWER_QUERY.md).

## 3. Estrutura da tabela `Carga ONS`

### 3.1. Campos da consulta Power Query

| Campo | Tipo definido no Power Query | Descrição técnica |
|---|---|---|
| `cod_areacarga` | Texto | Código da área de carga consultada (`GO`). |
| `din_atualizacao` | Data/hora | Data e hora de atualização informadas na fonte. |
| `dat_referencia` | Data | Data de referência do registro. |
| `din_referenciautc` | Data/hora | Referência temporal em UTC. |
| `val_cargaglobal` | Número | Valor de carga global informado pela fonte. |
| `val_cargaglobalcons` | Número | Campo de carga global de consumo informado pela fonte. |
| `val_cargaglobalsmmgd` | Número | Campo de carga global identificado na fonte como `val_cargaglobalsmmgd`. |
| `val_cargasupervisionada` | Número | Campo de carga supervisionada. |
| `val_carganaosupervisionada` | Número | Campo de carga não supervisionada. |
| `val_cargammgd` | Número | Campo identificado na fonte como `val_cargammgd`. |
| `val_consistencia` | Número | Campo de consistência informado pela fonte. |
| `DataHora_Goias` | Data/hora | Referência temporal ajustada para UTC−3. |

**Nota:** a interpretação operacional detalhada dos campos específicos da API deve seguir o dicionário oficial de dados do ONS.

### 3.2. Colunas calculadas no modelo

| Coluna | Finalidade |
|---|---|
| `Data_Goias` | Extrair a data da referência local para agrupamentos e filtros diários. |
| `Periodo_Dia` | Classificar os registros em madrugada, manhã, tarde e noite. |
| `Ordem_Periodo` | Estabelecer a sequência numérica dos períodos do dia. |

## 4. Expressões DAX das colunas calculadas

### 4.1. `Data_Goias`

```dax
Data_Goias =
DATE(
    YEAR('Carga ONS'[DataHora_Goias]),
    MONTH('Carga ONS'[DataHora_Goias]),
    DAY('Carga ONS'[DataHora_Goias])
)
```

**Finalidade:** obter a data correspondente ao registro no horário de Goiás, sem considerar as horas, os minutos e os segundos. Essa coluna possibilita segmentações e agrupamentos diários no relatório.

### 4.2. `Periodo_Dia`

```dax
Periodo_Dia =
VAR Hora = HOUR('Carga ONS'[DataHora_Goias])
RETURN
SWITCH(
    TRUE(),
    Hora < 6, "Madrugada",
    Hora < 12, "Manhã",
    Hora < 18, "Tarde",
    "Noite"
)
```

**Finalidade:** classificar cada registro de acordo com a hora local, permitindo análises comparativas entre períodos do dia.

### 4.3. `Ordem_Periodo`

```dax
Ordem_Periodo =
VAR Hora = HOUR('Carga ONS'[DataHora_Goias])
RETURN
SWITCH(
    TRUE(),
    Hora < 6, 1,
    Hora < 12, 2,
    Hora < 18, 3,
    4
)
```

**Finalidade:** atribuir uma ordem numérica às categorias de período do dia, permitindo sua apresentação em sequência cronológica.

## 5. Regras de classificação temporal

| Período | Horário local | Ordem |
|---|---|---:|
| Madrugada | 00h00–05h59 | 1 |
| Manhã | 06h00–11h59 | 2 |
| Tarde | 12h00–17h59 | 3 |
| Noite | 18h00–23h59 | 4 |

A classificação utiliza `DataHora_Goias` como referência comum às três colunas calculadas. Para ordenar os rótulos no Power BI, `Periodo_Dia` pode ser classificada por `Ordem_Periodo`.

## 6. Aplicação no relatório

A estrutura temporal do modelo sustenta as segmentações por data e as análises de carga por horário e período do dia. A coluna `Data_Goias` é destinada aos filtros diários, enquanto `Periodo_Dia` e `Ordem_Periodo` apoiam os gráficos comparativos dos períodos de madrugada, manhã, tarde e noite.

Os cálculos de indicadores, como carga máxima, mínima, média, fator de carga e variações em intervalos de 30 minutos, são tratados na documentação específica das medidas DAX.

## 7. Organização técnica

O projeto separa a preparação dos registros na consulta Power Query da organização analítica no modelo semântico. O Power Query realiza a conexão com a fonte, a estruturação e a tipagem dos dados, além do ajuste da referência temporal. As colunas calculadas DAX complementam o modelo com atributos para classificação, ordenação e análise dos registros no Power BI.

**Documentos relacionados:** [`POWER_QUERY.md`](POWER_QUERY.md) · [`MEDIDAS_DAX_revisado.md`](MEDIDAS_DAX_revisado.md)
