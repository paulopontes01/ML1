# Guia de Estudo - Projeto ML Imoveis

Este guia explica o notebook `analise_imoveis_multialvo.ipynb` seção por seção. A versão atual trabalha com dois alvos: `sucesso_comercial` e `volume_de_contatos`.

## Visao geral do projeto

O projeto cria uma base consolidada de imóveis a partir de três arquivos:

- `Imoveis.csv`: base principal, com características, localização, preço e status dos imóveis.
- `Contatos.csv`: base de leads, usada para medir o volume de contatos por imóvel.
- `Cadastros.csv`: base complementar, usada para enriquecer imóveis com dados de cadastro quando existe referência.

A unidade de análise é:

```text
1 linha = 1 imóvel
```

A versão final usa dois alvos:

| Alvo | Tipo | O que responde |
|---|---|---|
| `sucesso_comercial` | Classificação | O imóvel tende a ser alugado/vendido? |
| `volume_de_contatos` | Regressão | Quantos contatos o imóvel tende a receber? |

## Abertura - Contexto de negocio

### O que esta seção faz

Apresenta o problema antes da parte técnica. A imobiliária tem dados de imóveis, contatos e cadastros, e o objetivo é transformar esses dados em apoio à decisão comercial.

### Por que isso é importante

Projetos de machine learning não começam pelo modelo. Primeiro é preciso definir:

- Qual decisão será apoiada.
- Qual dado está disponível.
- Qual resultado queremos prever.
- Como o resultado será usado pela empresa.

Neste projeto, o uso prático é priorizar imóveis ativos.

### Justificativa dos dois alvos

`sucesso_comercial` mede o desfecho de negócio: se o imóvel foi alugado ou vendido.

`volume_de_contatos` mede a intensidade da demanda: quantos leads o imóvel tende a gerar.

Esses dois alvos são complementares. Um imóvel pode ter muitos contatos e não fechar, ou pode ter poucos contatos e fechar rapidamente. Por isso, faz sentido analisar resultado comercial e volume de demanda separadamente.

## 1. Instalação e Setup do Google Colab

### O que esta seção faz

Define o Google Colab como ambiente de execução e deixa uma célula opcional para instalar bibliotecas.

### Justificativa

O projeto será executado no Colab, então não depende do ambiente local. Isso facilita a reprodução pelo professor.

### Pontos importantes

- A instalação fica comentada para não rodar sem necessidade.
- `openpyxl` é usado para exportar Excel.
- `joblib` é usado para salvar os modelos.

## 2. Importação das bibliotecas

### O que esta seção faz

Importa bibliotecas de manipulação, visualização, modelagem e persistência.

Principais bibliotecas:

- `pandas`: manipulação de tabelas.
- `numpy`: operações numéricas.
- `matplotlib`: gráficos.
- `scikit-learn`: modelos, pipelines, métricas e pré-processamento.
- `joblib`: salvar e carregar modelos `.pkl`.

### Justificativa

O projeto é tabular. `pandas` e `scikit-learn` são adequados para esse tipo de problema porque permitem limpar dados, criar pipelines e treinar modelos clássicos de machine learning.

## 3. Carregamento dos dados

### O que esta seção faz

Carrega os arquivos:

- `Imoveis.csv`.
- `Contatos.csv`.
- `Cadastros.csv`.

Se os arquivos não estiverem no ambiente do Colab, o notebook solicita upload.

### Justificativa

O upload manual no Colab é simples para a entrega. O código também funciona se os arquivos já estiverem no diretório.

### Ponto técnico

Os CSVs usam separador `;` e codificação `latin-1`, comuns em exportações de sistemas brasileiros.

## 4. Inspeção inicial das bases

### O que esta seção faz

Mostra:

- Linhas e colunas.
- Tipos de dados.
- Nulos.
- Percentual de nulos.
- Valores únicos.
- Amostras das bases.
- Distribuições importantes.

### Justificativa

Antes de modelar, precisamos entender a qualidade dos dados. Essa etapa identifica colunas vazias, constantes, chaves problemáticas e campos que precisam de conversão.

### Exemplos relevantes

Em `Contatos.csv`, algumas colunas têm pouco valor para modelagem:

- `Importante` é constante.
- `E-mail Aberto?` é constante.
- `Cidade`, `Destinatário` e `Data Abertura` estão vazias.

Por isso, elas não entram nos modelos.

## 5. Funções auxiliares de limpeza

### O que esta seção faz

Cria funções para padronizar e transformar os dados:

- Converter moeda para número.
- Extrair código do imóvel.
- Extrair dormitórios, suítes, banheiros e garagens.
- Simplificar tipo de imóvel.
- Converter datas de contato.
- Criar `OneHotEncoder` compatível com diferentes versões do scikit-learn.

### Justificativa

Os dados originais vêm em formato textual. Modelos precisam de dados estruturados. As funções transformam textos como `R$ 350.000,00` e `3 dormitórios | 2 banheiros` em variáveis numéricas.

## 6. ETL dos imóveis

### O que esta seção faz

Transforma `Imoveis.csv` em uma base modelável.

Principais transformações:

- `Referência` vira `Referencia_num`.
- Valores monetários viram colunas numéricas.
- Características textuais viram números.
- `Tipo` vira `tipo_simples`.
- Textos são padronizados.

### Por que criar `tipo_simples`

A coluna `Tipo` tem muitas categorias específicas. Com poucos dados, muitas categorias podem gerar ruído. `tipo_simples` agrupa em classes mais estáveis:

- Casa.
- Apartamento.
- Terreno/Lote.
- Comercial.
- Outro.

## 7. ETL dos contatos

### O que esta seção faz

Agrega contatos por imóvel.

Principais variáveis criadas:

- `total_contatos`.
- Quantidade de nomes únicos.
- Primeira e última data de contato.
- Hora e dia mais frequentes.
- Origem provável do lead.
- Quantidade de contatos por tipo.

### Justificativa

`Contatos.csv` tem uma linha por contato. Como o `df_master` tem uma linha por imóvel, precisamos resumir os contatos por código de imóvel.

### Importância de `total_contatos`

`total_contatos` é o alvo da regressão de volume. Ele mede a intensidade de demanda do imóvel.

No modelo de sucesso comercial, `total_contatos` pode entrar como feature em uma versão específica, mas precisa de cuidado por risco de vazamento.

## 8. ETL dos cadastros

### O que esta seção faz

Agrega `Cadastros.csv` por referência de imóvel.

Variáveis criadas:

- Se existe cadastro relacionado.
- Quantidade de cadastros por referência.
- Perfil de proprietário.
- Perfil de cliente.
- Captação principal.
- Idade do cadastro.

### Justificativa

A base de cadastros pode enriquecer o contexto comercial do imóvel. Porém, poucas linhas têm `Referência` preenchida, então essa base é complementar.

## 9. Criação do `df_master`

### O que esta seção faz

Junta imóveis, contatos agregados e cadastros agregados.

Resultado:

```text
df_master = uma linha por imóvel
```

### Por que preencher contagens com zero

Quando um imóvel não tem contato, o merge gera nulo. Nesse caso, nulo significa ausência de contato encontrado, então faz sentido preencher com `0`.

## 10. Criação dos dois alvos

### `sucesso_comercial`

Regra:

```text
1 = Alugado ou Vendido
0 = Desistência ou Outros
NaN = Disponível ou vazio
```

Justificativa:

Imóveis disponíveis ou sem status claro não são fracasso. Eles entram no scoring, não no treino do alvo de sucesso.

### `volume_de_contatos`

Regra:

```text
volume_de_contatos = total_contatos
```

Também é criada a versão:

```text
log_volume_de_contatos = log1p(total_contatos)
```

Justificativa:

O volume de contatos é assimétrico. Poucos imóveis concentram muitos contatos. `log1p` reduz o impacto desses extremos durante o treino.

## 11. EDA orientada aos alvos

### O que esta seção faz

Analisa os dados com foco nos dois alvos.

Gráficos e tabelas principais:

- Distribuição de disponibilidade.
- Distribuição de `total_contatos`.
- Média de contatos por tipo de imóvel.
- Top bairros por volume de contatos.
- Relação entre faixas de contatos e sucesso comercial.

### Justificativa

A EDA mostra se existe sinal nos dados e ajuda a interpretar o comportamento dos alvos.

Exemplo de pergunta:

```text
Imóveis com mais contatos também têm maior taxa de sucesso comercial?
```

## 12. Features e pipelines

### O que esta seção faz

Define listas de features para cada modelo.

Principais grupos:

- `FEATURES_BASE_NUM`: variáveis numéricas do imóvel/cadastro.
- `FEATURES_BASE_CAT`: variáveis categóricas do imóvel/cadastro.
- `FEATURES_CONTATOS_NUM`: variáveis derivadas dos contatos.
- `FEATURES_CONTATOS_CAT`: variáveis categóricas de contatos.
- `FEATURES_SUCESSO_COM_CONTATOS`.
- `FEATURES_SUCESSO_SEM_CONTATOS`.
- `FEATURES_VOLUME`.

### Justificativa

Cada modelo usa features diferentes.

Para prever `volume_de_contatos`, não usamos `total_contatos` nem variáveis derivadas dos contatos, porque o alvo é o próprio volume. Usar essas variáveis seria vazamento direto.

Para prever `sucesso_comercial`, treinamos duas versões:

- Com contatos.
- Sem contatos.

Isso permite avaliar o impacto e o risco de usar informações de demanda.

### Pipeline numérico

Usa:

- `SimpleImputer(strategy="median")`.
- `StandardScaler`.

A mediana é robusta a outliers. A escala ajuda modelos lineares.

### Pipeline categórico

Usa:

- `SimpleImputer(strategy="most_frequent")`.
- `OneHotEncoder(handle_unknown="ignore")`.

O one-hot transforma categorias em colunas numéricas. `handle_unknown="ignore"` evita erro quando aparece uma categoria nova no teste ou scoring.

## 13. Modelo 1 - Sucesso comercial

### O que esta seção faz

Treina modelos para prever se o imóvel teve sucesso comercial.

Modelos testados:

- `LogisticRegression`.
- `RandomForestClassifier`.
- `GradientBoostingClassifier`.

### Por que comparar com e sem contatos

`total_contatos` pode explicar sucesso comercial, mas também pode representar vazamento se os contatos aconteceram depois do desfecho.

Por isso, o notebook compara:

- Modelo com contatos.
- Modelo sem contatos.

### Métricas usadas

- `AUC-ROC`: capacidade de ranquear positivos acima de negativos.
- `Precision`: dos previstos como sucesso, quantos estavam corretos.
- `Recall`: dos sucessos reais, quantos o modelo encontrou.
- `F1`: equilíbrio entre precision e recall.

## 14. Modelo 2 - Volume de contatos

### O que esta seção faz

Treina modelos de regressão para prever a quantidade de contatos.

Modelos testados:

- `DummyRegressor`.
- `Ridge`.
- `RandomForestRegressor`.
- `GradientBoostingRegressor`.

### Por que usar `log1p(total_contatos)`

O volume de contatos é concentrado: muitos imóveis têm poucos contatos e poucos imóveis têm muitos contatos.

`log1p` reduz o impacto dos valores extremos.

Exemplo:

```text
log1p(0) = 0
log1p(10) ≈ 2.40
log1p(100) ≈ 4.61
```

### O que é `MAE`

`MAE` significa erro médio absoluto.

Ele mede, em média, quanto o modelo erra em unidades do alvo. Neste caso, contatos.

Exemplo:

```text
Valor real: 10 contatos
Previsto:    7 contatos
Erro:        3 contatos
```

Se `MAE = 4`, o modelo erra em média 4 contatos por imóvel.

### O que é `RMSE`

`RMSE` é a raiz do erro quadrático médio.

Ele penaliza mais erros grandes. Se o modelo erra muito em poucos imóveis, o `RMSE` sobe bastante.

### O que é `R2`

`R2` mede quanto da variação do alvo o modelo explica.

Interpretação simples:

- Próximo de `1`: melhor.
- Próximo de `0`: pouco poder explicativo.
- Negativo: pior que uma referência simples.

### O que é `DummyRegressor`

`DummyRegressor` é um modelo simples de referência. Ele não aprende padrões de bairro, preço ou tipo.

No projeto usamos:

```text
DummyRegressor(strategy="median")
```

Isso significa prever sempre a mediana de `total_contatos`.

### Por que o `DummyRegressor` não é usado no ranking final

Na primeira execução, todos os `volume_contatos_previsto` ficaram zero porque a mediana de contatos era zero. Isso aconteceu porque muitos imóveis não têm contato associado.

Esse comportamento é válido como baseline, mas não é útil para ranking, pois não diferencia imóveis.

Ajuste feito:

- `DummyRegressor` continua aparecendo na tabela de métricas.
- Ele serve como comparação mínima.
- O modelo final de volume é escolhido apenas entre modelos reais: `Ridge`, `RandomForestRegressor` e `GradientBoostingRegressor`.

## 15. Análise dos melhores modelos

### O que esta seção faz

Mostra coeficientes ou importâncias de features dos modelos finais.

### Justificativa

Além de medir desempenho, precisamos interpretar quais variáveis influenciam as previsões.

- Regressão logística: coeficientes.
- Random Forest/Gradient Boosting: importância de features.

## 16. Scoring final dos imóveis ativos

### O que esta seção faz

Aplica os modelos nos imóveis ativos e gera o ranking final.

Colunas principais:

- `prob_sucesso_comercial`.
- `volume_contatos_previsto`.
- `score_volume_contatos`.
- `score_priorizacao`.

### Fórmula do score

```text
score_priorizacao = 0.7 * prob_sucesso_comercial + 0.3 * score_volume_contatos
```

### Justificativa

`prob_sucesso_comercial` recebe peso maior porque o objetivo principal é fechar aluguel ou venda.

`score_volume_contatos` entra como apoio, porque imóveis que tendem a gerar mais leads também podem merecer atenção comercial.

### O que é `score_volume_contatos`

É o volume previsto normalizado entre `0` e `1`.

Exemplo:

```text
volume_contatos_previsto = 20
maior volume previsto = 40
score_volume_contatos = 20 / 40 = 0.5
```

Essa normalização permite combinar volume com probabilidade no mesmo score.

## 17. Persistência dos modelos e exportação

### O que esta seção faz

Salva:

- `modelo_sucesso_comercial.pkl`.
- `modelo_volume_contatos.pkl`.
- `ranking_imoveis_ativos_multialvo.xlsx`.

### Justificativa

Persistir modelos permite reutilizar o resultado sem treinar tudo novamente. Exportar o ranking em Excel facilita apresentação e uso prático.

## 18. Conclusão executiva

### O que esta seção faz

Lista os pontos que devem ser preenchidos após rodar o notebook.

Uma boa conclusão responde:

- Qual modelo foi melhor?
- O resultado é confiável?
- Quais variáveis mais importam?
- Quais limitações existem?
- Como a imobiliária usaria isso?

## Principais riscos do projeto

### Vazamento de informação

O principal risco é usar `total_contatos` para prever sucesso comercial.

Se os contatos foram registrados depois do imóvel ser alugado/vendido, o modelo estaria usando informação do futuro.

Mitigação:

- Treinar sucesso comercial com contatos.
- Treinar sucesso comercial sem contatos.
- Comparar os resultados.
- Documentar a limitação.

### Poucos dados

A base tem poucos registros para alguns desfechos, principalmente `Vendido`.

Por isso, `sucesso_comercial` junta `Alugado` e `Vendido`.

### Cadastro parcial

`Cadastros.csv` tem poucas referências preenchidas. Portanto, é enriquecimento parcial.

### Linha temporal incompleta

Não temos data de fechamento comercial. Por isso, o projeto é mais analítico e de priorização do que uma previsão temporal pura.

## Glossário rápido

### Modelo

Algoritmo treinado para aprender padrões e fazer previsões.

### Feature

Variável de entrada usada pelo modelo.

Exemplos: `Valor_num`, `Bairro`, `tipo_simples`, `dormitorios`.

### Alvo

Variável que queremos prever.

No projeto: `sucesso_comercial` e `volume_de_contatos`.

### Classificação

Modelo que prevê uma classe, como `0` ou `1`.

### Regressão

Modelo que prevê um número, como quantidade de contatos.

### Baseline

Referência simples para comparar modelos reais.

### Mediana

Valor central de uma lista ordenada.

Exemplo:

```text
0, 0, 0, 1, 5
Mediana = 0
```

### Pipeline

Sequência de etapas executadas juntas: pré-processamento e modelo.

### ColumnTransformer

Ferramenta que aplica tratamentos diferentes para colunas numéricas e categóricas.

### Vazamento de informação

Uso de informação que não estaria disponível no momento real da previsão.

### Scoring

Aplicação dos modelos treinados em imóveis ativos para gerar previsões e ranking.

## Perguntas que o professor pode fazer

### 1. Qual é a diferença entre `sucesso_comercial` e `volume_de_contatos`?

`sucesso_comercial` mede desfecho de negócio: alugado ou vendido. `volume_de_contatos` mede interesse do mercado: quantos leads o imóvel tende a gerar.

### 2. Por que não transformar volume em classe?

Porque isso perde informação. Um imóvel com 11 contatos e outro com 100 contatos seriam tratados apenas como mesma classe, embora tenham comportamentos muito diferentes.

### 3. Por que `Disponível` e vazio não entram como negativos em sucesso comercial?

Porque não são necessariamente fracasso. Podem ser imóveis ainda ativos ou sem atualização de status.

### 4. Por que `Desistência` e `Outros` entram como negativos?

Porque são desfechos conhecidos que não representam aluguel ou venda.

### 5. Por que juntar `Alugado` e `Vendido`?

Porque ambos representam sucesso comercial. Separar os dois deixaria poucos exemplos, principalmente para `Vendido`.

### 6. Por que comparar sucesso com e sem contatos?

Porque contatos podem explicar sucesso, mas também podem conter informação posterior ao desfecho. A comparação mostra o impacto dessa variável.

### 7. O que é vazamento de informação?

É usar no treino uma informação que não estaria disponível no momento real da previsão.

### 8. Por que usar `Pipeline`?

Para garantir que treino, teste e scoring recebam o mesmo pré-processamento.

### 9. Por que usar `ColumnTransformer`?

Porque variáveis numéricas e categóricas precisam de tratamentos diferentes.

### 10. O que é `OneHotEncoder`?

Transforma categorias em colunas numéricas. Exemplo: cada bairro vira uma coluna indicadora.

### 11. Por que usar `StandardScaler`?

Para colocar variáveis numéricas em escalas comparáveis, o que ajuda modelos lineares.

### 12. O que é `MAE`?

Erro médio absoluto. Mede, em média, quantos contatos o modelo erra por imóvel.

### 13. O que é `RMSE`?

Métrica de erro que penaliza mais os erros grandes.

### 14. O que é `R2`?

Mede quanto da variação do alvo o modelo consegue explicar.

### 15. O que é `DummyRegressor`?

Um modelo simples de referência que prevê uma regra fixa, como sempre a mediana.

### 16. Por que o `DummyRegressor` previa zero?

Porque a mediana de `total_contatos` era zero, já que muitos imóveis não tinham contatos associados.

### 17. Por que não usar o `DummyRegressor` no ranking?

Porque ele pode prever o mesmo valor para todos e não ajuda a diferenciar prioridades.

### 18. Isso é manipular o resultado?

Não. O `DummyRegressor` continua sendo reportado como baseline. Apenas não é usado como modelo operacional para ranking.

### 19. Por que usar `log1p(total_contatos)`?

Porque a distribuição de contatos é concentrada em poucos imóveis. `log1p` reduz o impacto de valores extremos.

### 20. Como o ranking final é calculado?

Combina 70% da probabilidade de sucesso comercial com 30% do score normalizado de volume previsto de contatos.

### 21. Por que o sucesso comercial tem peso maior?

Porque o objetivo principal da imobiliária é fechar aluguel ou venda.

### 22. Por que exportar Excel?

Porque facilita a apresentação e o uso do ranking fora do notebook.

### 23. Qual é a maior limitação do projeto?

A ausência de data de fechamento comercial. Isso limita a interpretação temporal do modelo.

### 24. O projeto é preditivo ou analítico?

Ele tem modelos preditivos, mas deve ser interpretado principalmente como apoio analítico à priorização comercial.

### 25. Por que `Cadastros.csv` não é base principal?

Porque poucas linhas têm referência preenchida. A base principal precisa ser uma linha por imóvel, então `Imoveis.csv` é central.

### 26. Por que contatos sem código não entram no `df_master`?

Porque não é possível associá-los com segurança a um imóvel específico.

### 27. O que poderia melhorar o projeto?

Adicionar data de fechamento, aumentar a base histórica, validar o ranking com a equipe comercial e acompanhar novos imóveis e contatos.
