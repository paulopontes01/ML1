# Plano do Projeto de ML Imobiliario

Este plano usa como referencia o notebook `docs/Aulas - 06 - Projeto de Aprendizagem de Maquina de Ponta a Ponta.ipynb` e compara o roteiro da aula com os notebooks existentes `AnaliseImoveis_vs1.ipynb` e `analise_imoveis_2.ipynb`.

## Ambiente de execucao

O projeto sera executado no Google Colab. Este repositorio local sera usado principalmente para editar, organizar e versionar os arquivos.

Consequencias praticas:

- Nao e necessario instalar as dependencias localmente para desenvolver o notebook.
- As bibliotecas podem ser instaladas no proprio Colab quando necessario.
- A leitura dos dados pode continuar aceitando upload manual no Colab, mas o notebook final deve deixar claro quais arquivos precisam ser enviados.
- Se os CSVs estiverem no repositorio do GitHub ou no Google Drive, vale adicionar uma alternativa de leitura direta para melhorar a reproducibilidade.

## Base recomendada

Usar `analise_imoveis_2.ipynb` como notebook principal do projeto.

Motivos:

- Ja tem contexto de negocio, perguntas do projeto, bases usadas e limitacoes conhecidas.
- Ja segue uma estrutura parecida com a Aula 06.
- Ja implementa `Pipeline`, `ColumnTransformer`, validacao cruzada, `RandomizedSearchCV`, avaliacao final, scoring de imoveis ativos e persistencia com `joblib`.
- Tem resultados melhores e mais completos que `AnaliseImoveis_vs1.ipynb`.

O notebook `AnaliseImoveis_vs1.ipynb` deve ser tratado como versao exploratoria ou historico.

## Checklist do que ja existe

| Etapa da Aula 06 | Status | Evidencia no projeto | Acao necessaria |
|---|---:|---|---|
| Setup e bibliotecas | Parcial | Notebooks importam bibliotecas e usam Colab | Documentar ambiente Colab e instalacoes necessarias no proprio notebook |
| Contexto de negocio | Feito | `analise_imoveis_2.ipynb` descreve negocio, perguntas, bases e limitacoes | Refinar objetivo em uma frase mensuravel |
| Obtencao dos dados | Feito | Arquivos `Imoveis.csv`, `Contatos.csv`, `Cadastros.csv` existem no repositorio e notebooks usam upload do Colab | Explicar o upload no Colab e, opcionalmente, adicionar leitura via GitHub/Drive |
| Inspecao inicial dos dados | Feito | Notebooks exibem `head`, estatisticas, perfis e distribuicoes | Adicionar dicionario simples das colunas mais importantes |
| Criacao da tabela unificada | Feito | `df_master` junta imoveis com demanda por contatos | Validar chaves, duplicidades e perdas no merge |
| Variavel-alvo | Parcial | `comercializado = Alugado/Vendido` | Formalizar regra de alvo e tratamento de `Disponivel`, `NaN`, `Desistencia`, `Outros` |
| Separacao treino/teste | Feito | `train_test_split(..., stratify=y, random_state=42)` | Avaliar split temporal se datas forem relevantes |
| EDA e visualizacoes | Feito | Graficos de bairros, correlacoes, horarios, mapa e clusters | Selecionar graficos finais para apresentacao |
| Analise geografica | Parcial | Geocodificacao por bairro via Nominatim | Criar cache persistente ou evitar dependencia externa na execucao final |
| Engenharia de atributos | Parcial | Valor numerico, caracteristicas do imovel, demanda, cluster, categorias | Revisar risco de vazamento com `total_contatos` e datas dos contatos |
| Limpeza e imputacao | Feito | `SimpleImputer` em pipelines numerico e categorico | Documentar estrategias escolhidas |
| Categoricos e escala | Feito | `OneHotEncoder`, `StandardScaler`, `ColumnTransformer` | Manter dentro do pipeline final |
| Pipeline consolidado | Feito | `Pipeline([preprocessor, classifier])` | Nomear explicitamente o pipeline final |
| Modelagem inicial | Feito | Regressao Logistica, Random Forest e Gradient Boosting | Justificar modelo candidato principal |
| Validacao cruzada | Feito | `cross_val_score`, `cross_val_predict`, AUC por fold | Reportar media e desvio como criterio de comparacao |
| Ajuste de hiperparametros | Feito | `RandomizedSearchCV` para Regressao Logistica | Opcional: testar busca para Random Forest se houver tempo |
| Analise do melhor modelo | Parcial | AUC, matriz de confusao, coeficientes/importancias | Adicionar interpretacao dos erros e principais features |
| Avaliacao no teste | Feito | AUC final, precision, recall, F1 e matriz de confusao | Definir threshold conforme objetivo de negocio |
| Scoring de ativos | Feito | Predicao para imoveis ativos | Exportar ranking final de recomendacao |
| Persistencia do modelo | Parcial | Codigo salva `modelo_comercializacao_imoveis.pkl` | Gerar artefato real no repositorio ou documentar execucao |
| Entrega final | Parcial | Notebook tem conclusao e exportacao XLSX | Criar resumo executivo e checklist de reproducibilidade |

## Plano de execucao

### 1. Fechar o problema de negocio

Objetivo proposto:

Prever a probabilidade de um imovel ser comercializado pela Lima Imobiliaria em Sorocaba a partir de caracteristicas do imovel, historico de demanda e informacoes categoricas, para priorizar atendimento, precificacao e acao comercial.

Decisoes pendentes:

- Confirmar se o alvo positivo deve ser somente `Alugado` e `Vendido`.
- Confirmar se `Disponivel`, `Desistencia`, `Outros` e valores ausentes entram como negativos, ativos para scoring ou devem ser separados.
- Definir se o foco de negocio e maximizar `recall` dos imoveis com chance de comercializacao ou `precision` para reduzir falsos positivos.

### 2. Preparar ambiente Colab e dados

Entregaveis:

- Celula inicial de setup para Colab, com instalacoes somente se forem necessarias.
- Orientacao explicita para upload dos arquivos `Imoveis.csv`, `Contatos.csv`, `Cadastros.csv`.
- Opcional: leitura direta via GitHub ou Google Drive para evitar upload manual a cada execucao.
- Pequeno inventario de dados: linhas, colunas, nulos, duplicidades e chaves de merge.

Dados encontrados agora:

- `Imoveis.csv`: 467 linhas e 23 colunas.
- `Contatos.csv`: 4.576 linhas e 12 colunas.
- `Cadastros.csv`: 634 linhas e 11 colunas.

### 3. Consolidar ETL

Manter:

- Extracao de codigo do imovel a partir da coluna `Imovel` em contatos.
- Agregacao de contatos por imovel.
- Conversao de valores monetarios e caracteristicas em variaveis numericas.
- Criacao de `df_master`.

Adicionar:

- Validacao de quantos contatos ficaram sem codigo de imovel.
- Validacao de quantos imoveis nao casaram com contatos.
- Checagem de duplicidade por `Referencia`.
- Separacao clara entre dados de treinamento e dados ativos para scoring.

### 4. Explorar e documentar insights

Manter:

- Distribuicao da variavel-alvo.
- Top bairros por demanda.
- Analise de horarios e dias de contato.
- Correlacao com `comercializado`.
- Clusterizacao K-Means para segmentacao.

Adicionar:

- Selecionar 3 a 5 graficos finais para apresentacao.
- Explicar o que cada grafico muda na decisao de negocio.
- Documentar limites dos dados: amostra pequena, status incompletos, dependencia de contatos e ausencia de linha temporal completa.

### 5. Tratar risco de vazamento

Ponto critico:

`total_contatos` pode ser uma feature muito util, mas tambem pode causar vazamento se os contatos foram registrados depois da comercializacao do imovel.

Acao:

- Verificar se existe data de comercializacao ou alguma forma de cortar os contatos antes do evento.
- Se nao houver, declarar a limitacao e tratar o modelo como priorizacao baseada no estado historico observado, nao como previsao temporal pura.
- Considerar uma versao alternativa do modelo sem `total_contatos` para comparar robustez.

### 6. Construir pipeline final

Manter estrutura da Aula 06:

- Separar `X` e `y`.
- Fazer split estratificado.
- Criar pipeline numerico com imputacao e escala.
- Criar pipeline categorico com imputacao e one-hot encoding.
- Unir com `ColumnTransformer`.
- Encapsular pre-processamento e classificador em um unico `Pipeline`.

Modelo base:

- Regressao Logistica otimizada como modelo final inicial, pois e interpretavel e ja teve AUC final aproximada de 0,756 no notebook.

Modelos comparativos:

- Random Forest.
- Gradient Boosting.

### 7. Validar e ajustar modelo

Metricas principais:

- `AUC-ROC`: qualidade geral de ranking.
- `Recall`: capacidade de capturar imoveis que comercializam.
- `Precision`: confiabilidade dos imoveis priorizados.
- `F1`: equilibrio entre precision e recall.
- Matriz de confusao: interpretacao dos erros.

Entregaveis:

- Resultado de validacao cruzada com media e desvio.
- Resultado final no conjunto de teste.
- Analise de threshold para escolher ponto de decisao.
- Comparacao simples entre modelo com e sem `cluster`.
- Comparacao opcional com e sem `total_contatos`.

### 8. Analisar modelo final

Entregaveis:

- Top features positivas e negativas da Regressao Logistica.
- Interpretacao de falsos positivos e falsos negativos.
- Recomendacao de uso do modelo: ranking, triagem comercial ou apoio a precificacao.
- Limites de confianca qualitativos: amostra pequena e risco de mudanca no mercado.

### 9. Gerar scoring e artefatos

Entregaveis esperados:

- `modelo_comercializacao_imoveis.pkl`.
- Planilha de ranking de imoveis ativos com probabilidade prevista.
- Planilha ou tabela com insights de negocio.
- Notebook final executado do inicio ao fim.

Observacao atual:

O codigo para salvar modelo e exportar arquivos existe em `analise_imoveis_2.ipynb`, mas esses artefatos nao estao presentes no repositorio neste momento.

### 10. Preparar entrega final

Formato sugerido:

- Notebook final: `analise_imoveis_2.ipynb`, limpo e executavel.
- Documento curto de apoio: objetivo, dados, metodologia, metricas, principais insights, limitacoes e proximos passos.
- Checklist de reproducibilidade: ambiente, arquivos necessarios, ordem de execucao e artefatos esperados.

## Lacunas prioritarias

1. Documentar explicitamente que a execucao oficial sera no Google Colab.
2. Melhorar a celula de carregamento dos dados para deixar o upload no Colab claro e robusto.
3. Formalizar a regra da variavel-alvo e separar claramente treino de scoring.
4. Avaliar vazamento de informacao em `total_contatos`.
5. Gerar os artefatos finais reais: modelo `.pkl` e planilha de ranking.
6. Adicionar interpretacao dos erros do modelo final.
7. Consolidar conclusao executiva com recomendacoes de negocio.

## Proxima acao recomendada

Transformar `analise_imoveis_2.ipynb` em notebook final para Google Colab, com secoes ajustadas ao roteiro da Aula 06, carregamento dos dados bem documentado, validacoes do ETL, comparacao de risco de vazamento e geracao dos artefatos finais.
