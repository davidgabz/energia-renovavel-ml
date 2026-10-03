# Energias renováveis e aprendizado de máquina

## Objetivo

Comparar três modelos de classificação para identificar a fonte de empreendimentos
(Solar, Eólica ou Hidráulica) e três modelos de regressão para estimar a radiação solar
em Petrolina (PE).

O [notebook](Energia_Renovavel_ML.ipynb) está executado, com análises, tabelas, matrizes
de confusão, gráficos e conclusões. Este repositório apresenta a **avaliação principal
em Python**; não inclui a atividade complementar no Orange.

## Dados utilizados

Os dois CSVs foram fornecidos com a atividade e estão incluídos sem alteração.
As origens e os parâmetros de consulta são os descritos no material do professor.

**ANEEL / SIGA:** o arquivo `aneel_classificacao_orange.csv` tem 3.876 registros.
As entradas são `potencia_kw`, `latitude` e `longitude`; a resposta é `fonte`.
Solar corresponde a UFV, Eólica a EOL e Hidráulica reúne UHE, PCH e CGH.
A data de coleta desse cadastro não foi informada nos anexos. A potência outorgada
está em kW e não representa energia gerada.

**Open-Meteo:** o arquivo `meteo_regressao_orange.csv` tem 1.001 observações horárias
de Petrolina (PE), coordenadas aproximadas -9,39 e -40,50, de **01/04/2025 a 30/06/2025**,
das 7h às 17h, no fuso `America/Recife`. As entradas são `temperatura_c`, `umidade_pct`,
`nuvens_pct`, `vento_kmh` e `hora`. A resposta é `radiacao_w_m2`, radiação solar global
horizontal média da hora anterior, em W/m². `data_hora` serve apenas para ordenar os registros.
São estimativas históricas de modelos/reanálise, não medições de painéis.

## Classificação

Não há valores ausentes. Retiramos 28 repetições completas antes da divisão,
resultando em **3.848 linhas**: 1.466 hidráulicas, 1.195 solares e 1.187 eólicas.
Essa decisão evita exemplos idênticos no treino e no teste; não comprova duplicidade de
empreendimentos, pois o CSV não possui identificador.

Usamos divisão estratificada de aproximadamente 80%/20%, com semente 42:
**3.078 linhas de treino e 770 de teste**, iguais para os três modelos.
A Regressão Logística e o kNN usam padronização aprendida apenas no treino.
O kNN usa cinco vizinhos e a Random Forest, 100 árvores. Precision, Recall e F1 usam
média **macro**, dando o mesmo peso a cada classe.

| Modelo | Accuracy | Precision macro | Recall macro | F1 macro |
|---|---:|---:|---:|---:|
| Regressão Logística | 0,8247 | 0,8268 | 0,8208 | 0,8182 |
| kNN | 0,9675 | 0,9690 | 0,9661 | 0,9673 |
| Random Forest | 0,9779 | 0,9787 | 0,9770 | 0,9778 |

A Random Forest teve o melhor resultado nesta divisão: **753 acertos em 770 exemplos**
(97,79%). A maior confusão da Regressão Logística foi prever 59 solares como eólicos.
No kNN, foram oito solares previstos como hidráulicos; na floresta, cinco solares
previstos como hidráulicos. As três matrizes completas estão no notebook.

Potência e localização não descrevem todos os aspectos de um empreendimento.
Após retirar as repetições, há 27 linhas com alguma coordenada zero, mantidas sem
correção por suposição. A consulta tem limite por sigla e os exemplos não representam
a participação das fontes na matriz energética brasileira. O resultado também não
garante desempenho igual em uma região nova.

## Regressão

Não há valores ausentes nem horários repetidos. Usamos as primeiras **800 horas para
treino** e as últimas **201 para teste**, sem embaralhar. O treino termina em 12/06/2025,
às 14h; o teste vai de 12/06/2025, às 15h, até 30/06/2025, às 17h.

Os três modelos usam a mesma divisão e as cinco entradas definidas no enunciado.
A Random Forest tem 100 árvores. A semente das árvores e florestas é 42;
não foi feita busca de parâmetros no teste. MAE e MSE menores são melhores; R² maior é melhor.

| Modelo | MAE (W/m²) | MSE ((W/m²)²) | R² |
|---|---:|---:|---:|
| Regressão Linear | 145,20 | 30.034,20 | 0,3598 |
| Árvore de Decisão | 88,79 | 15.291,16 | 0,6741 |
| Random Forest | 66,80 | 7.307,42 | 0,8442 |

A Random Forest apresentou o menor erro médio absoluto, **66,80 W/m²**.
Isso não limita o erro de cada previsão: seu maior erro absoluto foi 221,76 W/m².
A Regressão Linear produziu 19 previsões negativas; elas foram mantidas no cálculo
das métricas e discutidas como uma limitação do modelo.

O gráfico por hora mostra uma subida da radiação média até 12h e uma queda à tarde.
Isso ajuda a entender o papel da hora e a limitação de uma relação apenas linear.
O notebook também apresenta o gráfico de valores reais e previstos da Random Forest.

**Radiação não é geração elétrica:** o alvo está em W/m², não em kWh.
Faltam informações da instalação, como área dos painéis, eficiência e perdas.
O teste utiliza condições meteorológicas da própria observação; não demonstra
previsão de dias futuros sem conhecer essas entradas.

## Conclusão

Nas duas tarefas, a Random Forest teve o melhor resultado entre os três modelos
comparados. As conclusões valem para estes dados e estas divisões. O desempenho em
outras regiões e períodos precisaria de nova avaliação.

## Como executar

O notebook deve ficar na mesma pasta dos dois CSVs. As saídas já estão salvas;
para apenas consultar o trabalho, abra `Energia_Renovavel_ML.ipynb`.

Para repetir a análise localmente, instale as dependências e abra o Jupyter:

```bash
python -m pip install -r requirements.txt
python -m jupyter lab
```

Abra o notebook, reinicie o kernel e execute as células em ordem. A execução foi
verificada com **Python 3.13.5** e as versões indicadas em `requirements.txt`.
No Google Colab, abra o notebook e envie os dois CSVs para o ambiente antes de executar.

Mantenha `CONSULTAR_APIS = False` para reproduzir estes resultados. O código de coleta
do material de apoio está preservado. Com `True`, ele depende de acesso às APIs e grava
novos arquivos em `nova_coleta/`, sem substituir os dados analisados. **Não foi realizada
nova coleta para produzir os resultados apresentados.**

## Arquivos

| Arquivo | Conteúdo |
|---|---|
| `README.md` | Objetivo, dados, resultados, conclusões e execução |
| `Energia_Renovavel_ML.ipynb` | Consultas, análises e seis modelos, com saídas salvas |
| `aneel_classificacao_orange.csv` | Dados originais da classificação |
| `meteo_regressao_orange.csv` | Dados originais da regressão |
| `requirements.txt` | Bibliotecas e versões para executar |

## Fontes

Material da atividade: enunciado e notebook `Aula_APIs_Energia_Renovavel_ML.ipynb`
fornecidos pelo professor. Os nomes originais dos CSVs foram mantidos, mesmo sem o
complemento Orange neste repositório.

- [ANEEL / SIGA](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel).
- [Open-Meteo: histórico e unidades](https://open-meteo.com/en/docs/historical-weather-api).
- [scikit-learn: preparação dos dados](https://scikit-learn.org/1.8/common_pitfalls.html).
- [scikit-learn: métricas de avaliação](https://scikit-learn.org/1.8/modules/model_evaluation.html).
