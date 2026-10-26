# Tratamento e preparação dos dados — Power Query

**Projeto:** Análise da Carga Elétrica ONS — Goiás 2026  
**Consulta:** `Carga ONS`  
**Ferramenta:** Power Query (linguagem M), no modelo semântico do Microsoft Power BI

## 1. Objetivo

Documentar o processo de obtenção, estruturação e tipagem dos registros de carga elétrica verificada do Operador Nacional do Sistema Elétrico (ONS), utilizados na análise de Goiás em setembro de 2026.

## 2. Fonte e recorte dos dados

- **Fonte:** API de carga verificada do ONS.
- **Endpoint:** `https://apicarga.ons.org.br/prd/cargaverificada`
- **Parâmetros:** `dat_inicio=2026-09-01`, `dat_fim=2026-09-30`, `cod_areacarga=GO`.
- **Formato de resposta:** JSON.
- **Atualização:** o intervalo está fixado na URL da consulta; a atualização do conjunto de dados consulta novamente o mesmo período, não avança automaticamente para outros meses.

## 3. Etapas aplicadas

| Etapa | Operação em M | Finalidade |
|---|---|---|
| `Origem` | `Web.Contents` e `Json.Document` | Solicita os registros ao endpoint e interpreta a resposta JSON. |
| `Convertido em tabela` | `Table.FromList` | Converte a lista de registros em uma tabela com a coluna inicial `Column1`. |
| `Expandido Column1` | `Table.ExpandRecordColumn` | Expande os 11 atributos presentes nos registros JSON em colunas individuais. |
| `Tipo de coluna alterado` | `Table.TransformColumnTypes` | Define tipos de texto, data, data/hora e número para os campos extraídos. |
| `Personalização adicionada` | `Table.AddColumn` e `Table.TransformColumnTypes` | Cria `DataHora_Goias` a partir de `din_referenciautc`, subtraindo três horas, e define o tipo data/hora. |

## 4. Campos extraídos da API

| Campo | Tipo definido no Power Query |
|---|---|
| `cod_areacarga` | Texto |
| `din_atualizacao` | Data/hora |
| `dat_referencia` | Data |
| `din_referenciautc` | Data/hora |
| `val_cargaglobal` | Número |
| `val_cargaglobalcons` | Número |
| `val_cargaglobalsmmgd` | Número |
| `val_cargasupervisionada` | Número |
| `val_carganaosupervisionada` | Número |
| `val_cargammgd` | Número |
| `val_consistencia` | Número |

**Campo adicional criado na consulta:** `DataHora_Goias` (data/hora).

A descrição de negócio, as unidades e a interpretação de cada variável devem ser validadas com a documentação oficial da fonte; os nomes e tipos acima são os efetivamente presentes na consulta.

## 5. Tratamento temporal

A coluna `DataHora_Goias` é calculada pela expressão:

```powerquery
DateTime.From([din_referenciautc]) - #duration(0, 3, 0, 0)
```

A transformação aplica **deslocamento fixo de UTC−3** à data/hora de referência. Esse procedimento é adequado ao recorte temporal documentado para Goiás, mas não constitui uma conversão dinâmica de fuso horário com regras históricas de horário de verão.

## 6. Código M da consulta

O script abaixo reproduz as etapas fornecidas a partir do Editor do Power Query, com a URL representada como texto simples (sem a formatação de link aplicada na mensagem de origem).

```powerquery
let
    Origem = Json.Document(
        Web.Contents("https://apicarga.ons.org.br/prd/cargaverificada?dat_inicio=2026-09-01&dat_fim=2026-09-30&cod_areacarga=GO")
    ),
    #"Convertido em tabela" = Table.FromList(Origem, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Expandido Column1" = Table.ExpandRecordColumn(
        #"Convertido em tabela", "Column1",
        {"cod_areacarga", "din_atualizacao", "dat_referencia", "din_referenciautc", "val_cargaglobal", "val_cargaglobalcons", "val_cargaglobalsmmgd", "val_cargasupervisionada", "val_carganaosupervisionada", "val_cargammgd", "val_consistencia"},
        {"cod_areacarga", "din_atualizacao", "dat_referencia", "din_referenciautc", "val_cargaglobal", "val_cargaglobalcons", "val_cargaglobalsmmgd", "val_cargasupervisionada", "val_carganaosupervisionada", "val_cargammgd", "val_consistencia"}
    ),
    #"Tipo de coluna alterado" = Table.TransformColumnTypes(
        #"Expandido Column1",
        {{"cod_areacarga", type text}, {"din_atualizacao", type datetime}, {"dat_referencia", type date}, {"din_referenciautc", type datetime}, {"val_cargaglobal", type number}, {"val_cargaglobalcons", type number}, {"val_cargaglobalsmmgd", type number}, {"val_cargasupervisionada", type number}, {"val_carganaosupervisionada", type number}, {"val_cargammgd", type number}, {"val_consistencia", type number}}
    ),
    #"Personalização adicionada" = Table.TransformColumnTypes(
        Table.AddColumn(
            #"Tipo de coluna alterado", "DataHora_Goias",
            each DateTime.From([din_referenciautc]) - #duration(0, 3, 0, 0)
        ),
        {{"DataHora_Goias", type datetime}}
    )
in
    #"Personalização adicionada"
```

## 7. Organização do tratamento de dados

O Power Query foi utilizado para realizar a conexão com a API do ONS, estruturar os registros recebidos em formato JSON, definir os tipos das variáveis e adequar a referência temporal ao horário de Goiás (UTC−3).

A preparação dos dados foi organizada de forma a disponibilizar uma base estruturada para a construção dos indicadores e das visualizações no Power BI.

As colunas auxiliares de classificação temporal e as medidas analíticas integram a camada de modelagem do projeto, sendo utilizadas na segmentação dos dados, na organização dos períodos do dia e no cálculo dos indicadores apresentados no dashboard.

## 8. Referência

Operador Nacional do Sistema Elétrico (ONS): https://dados.ons.org.br/
