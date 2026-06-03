# Guia de Estudo - Projeto ML Imoveis Sucesso Comercial

Este guia explica o notebook `analise_imoveis_sucesso_comercial.ipynb` seção por seção. Esta versão do projeto tem apenas uma variável-alvo: `sucesso_comercial`.

## Visao geral do projeto

O projeto cria uma base consolidada de imóveis a partir de três arquivos:

- `Imoveis.csv`: base principal, com características, localização, preço e status dos imóveis.
- `Contatos.csv`: base de leads, usada como indicador de demanda e possível variável explicativa.
- `Cadastros.csv`: base complementar, usada para enriquecer imóveis com dados de cadastro quando existe referência.

A unidade de análise é:

```text
1 linha = 1 imóvel
```

A variável-alvo é:

| Alvo | Tipo | O que responde |
|---|---|---|
| `sucesso_comercial` | Classificação | O imóvel tende a ser alugado/vendido? |

## Objetivo de negócio

O objetivo é gerar um ranking de imóveis ativos por probabilidade de sucesso comercial.

Em termos práticos, o projeto ajuda a responder:

- Quais imóveis ativos devem ser priorizados pela equipe comercial?
- Quais características estão associadas a aluguel ou venda?
- A quantidade de contatos ajuda a explicar o sucesso comercial?
- O uso de contatos pode gerar risco de vazamento de informação?

## Abertura - Contexto de negócio

### O que esta seção faz

Apresenta a empresa, o problema e o objetivo antes da parte técnica.

### Por que isso é importante

Um projeto de machine learning precisa estar ligado a uma decisão real. Neste caso, a decisão é priorizar imóveis com maior chance de aluguel ou venda.

### Justificativa da variável-alvo

`sucesso_comercial` mede diretamente o resultado de interesse para a imobiliária.

Regra:

```text
1 = Alugado ou Vendido
0 = Desistência ou Outros
```

Imóveis `Disponível` ou com status vazio não entram como negativos, porque ainda podem ser comercializados.

## 1. Instalação e Setup do Google Colab

### O que esta seção faz

Define o Google Colab como ambiente de execução e deixa uma célula opcional para instalar bibliotecas.

### Justificativa

O projeto será executado no Colab. Isso facilita a reprodução pelo professor e evita dependência do ambiente local.

## 2. Importação das bibliotecas

### O que esta seção faz

Importa bibliotecas para:

- Manipulação de dados.
- Visualização.
- Pré-processamento.
- Modelagem.
- Avaliação.
- Persistência do modelo.

Principais bibliotecas:

- `pandas`: manipulação de tabelas.
- `numpy`: operações numéricas.
- `matplotlib`: gráficos.
- `scikit-learn`: pipelines, modelos e métricas.
- `joblib`: salvar o modelo em `.pkl`.

## 3. Carregamento dos dados

### O que esta seção faz

Carrega:

- `Imoveis.csv`.
- `Contatos.csv`.
- `Cadastros.csv`.

Se os arquivos não estiverem no ambiente, o notebook solicita upload no Colab.

### Ponto técnico

Os CSVs são lidos com:

```text
encoding="latin-1"
sep=";"
```

Isso é comum em arquivos exportados por sistemas brasileiros.

## 4. Inspeção inicial das bases

### O que esta seção faz

Mostra:

- Linhas e colunas.
- Tipos de dados.
- Nulos.
- Percentual de nulos.
- Valores únicos.
- Primeiras linhas.
- Distribuições importantes.

### Por que é importante

Antes de modelar, precisamos entender a qualidade dos dados.

Essa etapa ajuda a identificar:

- Colunas vazias.
- Colunas constantes.
- Dados em formato textual.
- Possíveis problemas de chave.
- Campos que precisam de conversão.

### Exemplo de descoberta

Em `Contatos.csv`, algumas colunas têm pouco valor:

- `Importante` é constante.
- `E-mail Aberto?` é constante.
- `Cidade`, `Destinatário` e `Data Abertura` estão vazias.

## 5. Funções auxiliares de limpeza

### O que esta seção faz

Cria funções para transformar os dados brutos:

- Converter moeda em número.
- Extrair código do imóvel.
- Extrair dormitórios, suítes, banheiros e garagens.
- Simplificar tipo de imóvel.
- Converter datas de contato.
- Criar `OneHotEncoder` compatível com versões diferentes do scikit-learn.

### Justificativa

Os dados vêm em texto e precisam virar variáveis estruturadas para o modelo.

Exemplos:

```text
R$ 350.000,00 -> 350000.0
3 dormitórios | 2 banheiros -> dormitorios=3, banheiros=2
Cód. - 584 Casa para Locação -> cod_imovel=584
```

## 6. ETL dos imóveis

### O que esta seção faz

Transforma `Imoveis.csv` em uma base modelável.

Principais transformações:

- `Referência` vira `Referencia_num`.
- Valores monetários viram números.
- Características textuais viram colunas numéricas.
- `Tipo` vira `tipo_simples`.
- Textos são padronizados.

### Por que criar `tipo_simples`

A coluna `Tipo` tem muitas categorias específicas. Com poucos dados, categorias demais podem gerar ruído.

`tipo_simples` agrupa imóveis em categorias mais estáveis:

- Casa.
- Apartamento.
- Terreno/Lote.
- Comercial.
- Outro.

## 7. ETL dos contatos

### O que esta seção faz

Agrega contatos por imóvel.

Variáveis criadas:

- `total_contatos`.
- Quantidade de nomes únicos.
- Primeira e última data de contato.
- Hora e dia mais frequentes.
- Origem provável do lead.
- Quantidade de contatos por tipo.

### Justificativa

`Contatos.csv` tem uma linha por contato. O modelo precisa de uma linha por imóvel. Por isso, os contatos são agregados por código de imóvel.

### Cuidado importante

`total_contatos` pode ajudar a explicar sucesso comercial, mas pode representar vazamento se os contatos ocorreram depois do aluguel ou venda.

Por isso, o notebook compara:

- Modelo com contatos.
- Modelo sem contatos.

## 8. ETL dos cadastros

### O que esta seção faz

Agrega `Cadastros.csv` por referência.

Variáveis criadas:

- Cadastro relacionado.
- Quantidade de cadastros por referência.
- Perfil de proprietário.
- Perfil de cliente.
- Captação principal.
- Idade do cadastro.

### Limitação

Poucas linhas de `Cadastros.csv` têm `Referência` preenchida. Portanto, essa base é complementar.

## 9. Criação do `df_master`

### O que esta seção faz

Junta:

- Imóveis tratados.
- Contatos agregados.
- Cadastros agregados.

Resultado:

```text
df_master = uma linha por imóvel
```

### Por que preencher contagens com zero

Quando um imóvel não tem contato, o merge gera nulo. Nesse caso, nulo significa ausência de contato encontrado. Por isso, as contagens são preenchidas com `0`.

## 10. Criação da variável-alvo

### Regra de `sucesso_comercial`

```text
1 = Alugado ou Vendido
0 = Desistência ou Outros
NaN = Disponível ou vazio
```

### Justificativa

`Alugado` e `Vendido` são tratados como sucesso porque representam fechamento comercial.

`Desistência` e `Outros` entram como negativos porque são desfechos conhecidos sem sucesso comercial.

`Disponível` e vazio não entram no treino porque não são necessariamente fracasso.

## 10.1 Análise do desbalanceamento de classes

### O que esta seção faz

Verifica a proporção entre:

```text
0 = Não sucesso
1 = Sucesso
```

### Por que é importante

Se uma classe for muito maior que a outra, o modelo pode parecer bom apenas prevendo a classe majoritária.

Exemplo:

```text
90% = Não sucesso
10% = Sucesso
```

Um modelo que sempre prevê `Não sucesso` teria 90% de acurácia, mas seria inútil para identificar imóveis com chance de fechamento.

### O que essa seção justifica

- Uso de `stratify=y`.
- Uso de `class_weight="balanced"`.
- Uso de métricas como `precision`, `recall`, `F1` e `AUC-ROC`.
- Cuidado ao interpretar acurácia.

## 11. EDA orientada ao alvo

### O que esta seção faz

Analisa os dados pensando no alvo `sucesso_comercial`.

Análises principais:

- Distribuição de disponibilidade.
- Distribuição de contatos.
- Média de contatos por tipo de imóvel.
- Top bairros por volume de contatos.
- Relação entre faixas de contatos e taxa de sucesso.

### Justificativa

A EDA ajuda a verificar se existe sinal nos dados antes de modelar.

Exemplo:

```text
Imóveis com mais contatos têm maior taxa de sucesso comercial?
```

## 12. Features e pipelines

### O que esta seção faz

Define as variáveis usadas nos modelos.

Grupos principais:

- `FEATURES_BASE_NUM`.
- `FEATURES_BASE_CAT`.
- `FEATURES_CONTATOS_NUM`.
- `FEATURES_CONTATOS_CAT`.
- `FEATURES_SUCESSO_COM_CONTATOS`.
- `FEATURES_SUCESSO_SEM_CONTATOS`.

### Por que separar features com e sem contatos

O objetivo é medir o impacto dos contatos e avaliar risco de vazamento.

Se o modelo com contatos for muito melhor, isso pode indicar que a demanda explica sucesso, mas também exige cautela temporal.

### Pipeline numérico

Usa:

- `SimpleImputer(strategy="median")`.
- `StandardScaler`.

### Pipeline categórico

Usa:

- `SimpleImputer(strategy="most_frequent")`.
- `OneHotEncoder(handle_unknown="ignore")`.

### Por que usar `Pipeline`

Garante que treino, teste e scoring recebam o mesmo pré-processamento.

### Por que usar `ColumnTransformer`

Permite tratar colunas numéricas e categóricas de formas diferentes dentro do mesmo pipeline.

## 13. Modelo 1 - Sucesso comercial

### O que esta seção faz

Treina modelos para prever `sucesso_comercial`.

Modelos testados:

- `LogisticRegression`.
- `RandomForestClassifier`.
- `GradientBoostingClassifier`.

### Justificativa dos modelos

`LogisticRegression` é simples e interpretável.

`RandomForestClassifier` captura relações não lineares.

`GradientBoostingClassifier` também captura padrões complexos e costuma funcionar bem com dados tabulares.

### Métricas usadas

- `AUC-ROC`.
- `Precision`.
- `Recall`.
- `F1`.

## 13.1 Avaliação gráfica do modelo final

### Curva ROC

Mostra a capacidade do modelo de separar imóveis de sucesso e não sucesso em diferentes thresholds.

Interpretação:

```text
Curva perto da diagonal = modelo próximo do aleatório
Curva distante da diagonal = melhor separação
```

### AUC-ROC

Área abaixo da Curva ROC.

Referência:

```text
0.50 = aleatório
0.70 = razoável
0.80 = bom
0.90 = excelente
```

### Matriz de confusão

Mostra:

- Verdadeiros negativos.
- Falsos positivos.
- Falsos negativos.
- Verdadeiros positivos.

### Interpretação de negócio

Falso positivo:

```text
Priorizar imóvel que talvez não feche.
```

Falso negativo:

```text
Deixar de priorizar imóvel que poderia fechar.
```

## 13.2 Curva Precision-Recall

### O que esta seção faz

Mostra o trade-off entre `precision` e `recall`.

### Precision

Dos imóveis previstos como sucesso, quantos realmente foram sucesso.

### Recall

Dos imóveis que realmente foram sucesso, quantos o modelo encontrou.

### Threshold

É o corte de probabilidade usado para transformar probabilidade em classe.

Exemplo:

```text
threshold = 0.50
probabilidade >= 0.50 -> Sucesso
```

### Por que testar thresholds

Threshold menor aumenta a quantidade de imóveis priorizados e tende a aumentar recall.

Threshold maior deixa o modelo mais seletivo e tende a aumentar precision.

## 14. Análise do melhor modelo

### O que esta seção faz

Mostra coeficientes ou importâncias das features do modelo final.

### Justificativa

Não basta medir desempenho. Precisamos entender quais variáveis influenciam a previsão.

Dependendo do modelo:

- Regressão logística usa coeficientes.
- Random Forest e Gradient Boosting usam importância de features.

## 15. Scoring final dos imóveis ativos

### O que esta seção faz

Aplica o modelo final nos imóveis ativos.

Resultado:

- `prob_sucesso_comercial`.
- `score_priorizacao`.

Nesta versão, o score é igual à probabilidade de sucesso comercial:

```text
score_priorizacao = prob_sucesso_comercial
```

### Justificativa

Como o notebook tem apenas um alvo, o ranking final deve refletir diretamente a probabilidade prevista de sucesso comercial.

## 15.1 Resumo quantitativo para apresentação

### O que esta seção faz

Resume os resultados em formato útil para apresentação:

- Modelo final escolhido.
- Versão com ou sem contatos.
- AUC-ROC.
- Precision.
- Recall.
- F1.
- Top 10 imóveis ativos.
- Resumo do ranking.

### Por que é importante

Traduz os resultados técnicos para uma leitura de negócio.

## 16. Persistência do modelo e exportação

### O que esta seção faz

Salva:

- `modelo_sucesso_comercial.pkl`.
- `ranking_imoveis_sucesso_comercial.xlsx`.

### Justificativa

O `.pkl` permite reutilizar o modelo sem treinar novamente.

O Excel permite apresentar e compartilhar o ranking.

## 17. Conclusão executiva

### O que esta seção faz

Lista os pontos finais do projeto:

- Melhor modelo.
- Diferença entre modelo com e sem contatos.
- Principais variáveis.
- Ranking de imóveis ativos.
- Limitações.
- Próximos passos.

## Principais riscos do projeto

### Vazamento de informação

O principal risco é usar `total_contatos` para prever sucesso comercial se os contatos ocorreram depois do desfecho.

Mitigação:

- Comparar modelo com contatos e sem contatos.
- Documentar a limitação.

### Poucos dados

`Vendido` tem poucos exemplos. Por isso, `Alugado` e `Vendido` foram agrupados como sucesso comercial.

### Cadastro parcial

`Cadastros.csv` tem poucas referências preenchidas. Ele é usado apenas como enriquecimento parcial.

### Linha temporal incompleta

Não há data de fechamento comercial. Por isso, o modelo deve ser interpretado como apoio à priorização, não como previsão temporal perfeita.

## Glossário rápido

### Modelo

Algoritmo treinado para aprender padrões e fazer previsões.

### Feature

Variável de entrada usada pelo modelo.

### Alvo

Variável que queremos prever.

### Classificação

Problema em que o modelo prevê uma classe, como `0` ou `1`.

### Pipeline

Sequência de etapas executadas juntas: pré-processamento e modelo.

### ColumnTransformer

Ferramenta que aplica tratamentos diferentes para colunas numéricas e categóricas.

### OneHotEncoder

Transforma categorias em colunas numéricas.

### StandardScaler

Padroniza variáveis numéricas para ficarem em escala comparável.

### AUC-ROC

Mede a capacidade do modelo de ranquear positivos acima de negativos.

### Precision

Dos previstos como positivos, quantos estavam corretos.

### Recall

Dos positivos reais, quantos o modelo encontrou.

### F1

Média harmônica entre precision e recall.

### Threshold

Corte de probabilidade usado para transformar probabilidade em classe.

### Vazamento de informação

Uso de informação que não estaria disponível no momento real da previsão.

### Scoring

Aplicação do modelo treinado em imóveis ativos para gerar ranking.

## Perguntas que o professor pode fazer

### 1. Qual é o objetivo do projeto?

Prever a probabilidade de sucesso comercial dos imóveis e gerar um ranking de imóveis ativos para priorização.

### 2. Qual é a variável-alvo?

`sucesso_comercial`, onde `1` representa `Alugado` ou `Vendido`, e `0` representa `Desistência` ou `Outros`.

### 3. Por que `Disponível` e vazio não entram como negativos?

Porque não são necessariamente fracasso. Podem ser imóveis ainda ativos ou sem atualização.

### 4. Por que juntar `Alugado` e `Vendido`?

Porque ambos representam sucesso comercial e `Vendido` tem poucos exemplos para ser modelado separadamente.

### 5. Por que usar os contatos?

Porque contatos indicam demanda. Mas eles são usados com cautela porque podem representar informação posterior ao desfecho.

### 6. Por que comparar modelo com contatos e sem contatos?

Para medir o impacto dos contatos e avaliar risco de vazamento.

### 7. O que é vazamento de informação?

É usar uma informação que não estaria disponível no momento real da previsão.

### 8. Por que analisar desbalanceamento de classes?

Porque uma classe muito maior pode tornar a acurácia enganosa.

### 9. Por que não usar só acurácia?

Porque o modelo pode acertar a classe majoritária e ainda ser ruim para encontrar imóveis de sucesso.

### 10. O que é `stratify=y`?

É uma forma de manter proporção parecida das classes no treino e no teste.

### 11. O que é `class_weight="balanced"`?

É um ajuste que dá mais peso à classe minoritária durante o treino.

### 12. Por que usar `Pipeline`?

Para garantir o mesmo pré-processamento em treino, teste e scoring.

### 13. Por que usar `ColumnTransformer`?

Porque variáveis numéricas e categóricas precisam de tratamentos diferentes.

### 14. O que é Curva ROC?

É um gráfico que mostra a capacidade do modelo de separar sucesso e não sucesso em diferentes thresholds.

### 15. O que é AUC-ROC?

É a área abaixo da Curva ROC. Quanto mais perto de 1, melhor.

### 16. O que é matriz de confusão?

É uma tabela que mostra acertos e erros do modelo.

### 17. O que é falso positivo?

Quando o modelo prevê sucesso, mas o imóvel não teve sucesso.

### 18. O que é falso negativo?

Quando o modelo prevê não sucesso, mas o imóvel teve sucesso.

### 19. O que é Precision?

Dos imóveis previstos como sucesso, quantos realmente foram sucesso.

### 20. O que é Recall?

Dos imóveis que realmente foram sucesso, quantos o modelo encontrou.

### 21. O que é F1?

Uma métrica que equilibra precision e recall.

### 22. O que é threshold?

É o corte de probabilidade usado para classificar um imóvel como sucesso ou não sucesso.

### 23. Por que testar thresholds diferentes?

Porque thresholds diferentes mudam o equilíbrio entre precision e recall.

### 24. Como o ranking final é calculado?

Nesta versão, o ranking é ordenado por `prob_sucesso_comercial`.

### 25. O que significa `prob_sucesso_comercial`?

É a probabilidade prevista pelo modelo de o imóvel ter sucesso comercial.

### 26. Por que exportar `.pkl`?

Para salvar o pipeline completo e reutilizar o modelo sem treinar novamente.

### 27. Por que exportar Excel?

Para compartilhar o ranking de imóveis ativos de forma simples.

### 28. Qual é a maior limitação do projeto?

A ausência de data de fechamento comercial.

### 29. O projeto é preditivo ou analítico?

É preditivo, mas deve ser interpretado como apoio analítico à priorização comercial.

### 30. O que poderia melhorar o projeto?

Adicionar data de fechamento, aumentar a base histórica, validar o ranking com a equipe comercial e acompanhar novos imóveis.
