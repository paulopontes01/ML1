# Plano do Projeto de ML Imobiliario

Este documento complementa o plano original e reflete a versão atual do notebook `analise_imoveis_multialvo.ipynb`.

Ambiente oficial de execução: Google Colab.

## Ideia central

O projeto passa a trabalhar com dois objetivos:

| Tarefa | Variável-alvo | Tipo de problema | Pergunta de negócio |
|---|---|---|---|
| Resultado comercial | `sucesso_comercial` | Classificação binária | Quais imóveis tendem a virar aluguel ou venda? |
| Volume de contatos | `volume_de_contatos` | Regressão | Quantos contatos um imóvel tende a receber? |

## Dataset mestre

Unidade de análise:

```text
1 linha = 1 imóvel
```

Bases usadas:

- `Imoveis.csv`: base central.
- `Contatos.csv`: base para calcular volume de contatos e features de demanda.
- `Cadastros.csv`: enriquecimento parcial por referência.

## Alvos

### `sucesso_comercial`

```text
1 = Alugado ou Vendido
0 = Desistência ou Outros
NaN = Disponível ou vazio
```

Imóveis disponíveis ou sem status claro ficam fora do treino desse alvo e podem ser usados no scoring.

### `volume_de_contatos`

```text
volume_de_contatos = total_contatos
```

Para treino da regressão, usar:

```text
log_volume_de_contatos = log1p(total_contatos)
```

## Features por modelo

### Sucesso comercial

Duas versões:

- Com dados de contatos.
- Sem dados de contatos.

Motivo: medir o impacto de `total_contatos` e controlar risco de vazamento.

### Volume de contatos

Usar apenas features base de imóvel/cadastro.

Não usar `total_contatos` nem variáveis derivadas de contatos como entrada, pois o alvo é o próprio volume.

## Modelos

### Classificação de sucesso comercial

Modelos candidatos:

- `LogisticRegression`.
- `RandomForestClassifier`.
- `GradientBoostingClassifier`.

Métricas:

- `AUC-ROC`.
- `Precision`.
- `Recall`.
- `F1`.

### Regressão de volume de contatos

Modelos candidatos:

- `DummyRegressor` como baseline.
- `Ridge`.
- `RandomForestRegressor`.
- `GradientBoostingRegressor`.

Métricas:

- `MAE`.
- `RMSE`.
- `R2`.

O `DummyRegressor` não deve ser escolhido como modelo final de ranking, pois pode prever zero para todos os imóveis quando a mediana de contatos for zero.

## Scoring final

Gerar ranking de imóveis ativos com:

- `prob_sucesso_comercial`.
- `volume_contatos_previsto`.
- `score_volume_contatos`.
- `score_priorizacao`.

Fórmula atual:

```text
score_priorizacao = 0.7 * prob_sucesso_comercial + 0.3 * score_volume_contatos
```

## Artefatos esperados

- `modelo_sucesso_comercial.pkl`.
- `modelo_volume_contatos.pkl`.
- `ranking_imoveis_ativos_multialvo.xlsx`.

## Estrutura atual do notebook

1. Contexto de negócio.
2. Setup Colab.
3. Carregamento dos três CSVs.
4. Inspeção inicial.
5. Funções auxiliares.
6. ETL de imóveis.
7. ETL de contatos.
8. ETL de cadastros.
9. Criação do `df_master`.
10. Criação dos dois alvos.
11. EDA orientada aos alvos.
12. Features e pipelines.
13. Modelo de sucesso comercial.
14. Modelo de volume de contatos.
15. Análise dos melhores modelos.
16. Scoring final dos imóveis ativos.
17. Persistência e exportação.
18. Conclusão executiva.
