# Medidas DAX — Análise da Carga ONS | Goiás 2026

Documentação técnica das medidas do relatório Power BI **Análise da Carga ONS — Goiás 2026**.

> **Escopo da documentação:** este documento apresenta as expressões DAX disponíveis para os indicadores do relatório. As medidas ainda não verificadas diretamente no modelo estão identificadas para validação no arquivo `.pbix`. Os valores exibidos nos cartões representam resultados de cálculo, não substituem as definições das medidas.

## Estrutura do modelo

- **Tabela:** `'Carga ONS'`
- **Carga elétrica:** `'Carga ONS'[val_cargaglobal]` (MW)
- **Data e hora local:** `'Carga ONS'[DataHora_Goias]`
- **Granularidade da análise:** intervalos de 30 minutos

## Expressões DAX documentadas

### Fator de Carga (%)

```dax
Fator de Carga (%) =
DIVIDE(
    [Carga Média (MW)],
    [Carga Máxima (MW)],
    0
) * 100
```

**Finalidade:** representa a relação percentual entre a carga média e a carga máxima no contexto de filtros aplicado. A medida depende de `[Carga Média (MW)]` e `[Carga Máxima (MW)]`.

### Horário de Ponta

```dax
Horário de Ponta =
VAR MaiorCarga = MAX('Carga ONS'[val_cargaglobal])
VAR MomentoPonta =
    CALCULATE(
        MIN('Carga ONS'[DataHora_Goias]),
        KEEPFILTERS('Carga ONS'[val_cargaglobal] = MaiorCarga)
    )
RETURN
    FORMAT(MomentoPonta, "HH:mm")
```

**Finalidade:** identifica o horário correspondente ao maior valor de carga no contexto filtrado. Em caso de empate, utiliza o primeiro registro temporal encontrado.

### Horário de Carga Mínima

```dax
Horário de Carga Mínima =
VAR MenorCarga = MIN('Carga ONS'[val_cargaglobal])
VAR MomentoMinimo =
    CALCULATE(
        MIN('Carga ONS'[DataHora_Goias]),
        KEEPFILTERS('Carga ONS'[val_cargaglobal] = MenorCarga)
    )
RETURN
    IF(
        ISBLANK(MomentoMinimo),
        BLANK(),
        FORMAT(MomentoMinimo, "HH:mm")
    )
```

**Finalidade:** identifica o horário correspondente ao menor valor de carga no contexto filtrado. Em caso de empate, utiliza o primeiro registro temporal encontrado.

### Maior Elevação 30min (%)

```dax
Maior Elevação 30min (%) =
MAXX(
    VALUES('Carga ONS'[DataHora_Goias]),
    [Variação de Carga 30min (%)]
)
```

**Finalidade:** retorna a maior variação percentual calculada entre intervalos de 30 minutos, respeitando o contexto de filtros. Depende da medida `[Variação de Carga 30min (%)]`.

### Maior Redução 30min (%)

```dax
Maior Redução 30min (%) =
MINX(
    VALUES('Carga ONS'[DataHora_Goias]),
    [Variação de Carga 30min (%)]
)
```

**Finalidade:** retorna a menor variação percentual calculada entre intervalos de 30 minutos, respeitando o contexto de filtros. Depende da medida `[Variação de Carga 30min (%)]`.

## Medidas com expressão a validar

As medidas abaixo integram os indicadores do relatório. Suas expressões devem ser verificadas diretamente no modelo do Power BI antes da consolidação da documentação técnica.

| Medida | Finalidade | Situação |
|---|---|---|
| `Carga Máxima (MW)` | Maior carga elétrica no contexto filtrado | Conferir expressão DAX |
| `Carga Mínima (MW)` | Menor carga elétrica no contexto filtrado | Conferir expressão DAX |
| `Carga Média (MW)` | Carga elétrica média no contexto filtrado | Conferir expressão DAX |
| `Amplitude de Carga (MW)` | Diferença entre os extremos de carga | Conferir expressão DAX |
| `Variação de Carga 30min (%)` | Variação percentual entre medições consecutivas | Conferir expressão DAX |

## Indicadores exibidos no dashboard de referência

| Indicador | Valor exibido |
|---|---:|
| Carga máxima | 3,13 mil MW |
| Carga mínima | 2,28 mil MW |
| Carga média | 2,73 mil MW |
| Fator de carga | 87,19% |
| Amplitude de carga | 849,21 MW |
| Maior elevação em 30 min | 3,88% |
| Maior redução em 30 min | -3,55% |
| Horário de ponta | 19:30 |
| Horário de carga mínima | 04:00 |

> Os resultados acima são os valores apresentados na captura do dashboard e podem variar conforme os filtros selecionados. A precisão dos cálculos deve ser conferida com as medidas e os dados do arquivo Power BI original.

**Fonte de dados:** [Operador Nacional do Sistema Elétrico — ONS](https://dados.ons.org.br/).
