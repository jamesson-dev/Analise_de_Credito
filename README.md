# Análise de Crédito com lifelines, Kaplan-Meier e Modelo de Cox
## Este projeto realiza uma análise de sobrevivência usando o método Kaplan-Meier e o modelo de riscos proporcionais de Cox em um conjunto de dados de crédito. (Analise de Churn)

### Requisitos
#### Python 3.6 ou superior

* Pandas

* Lifelines

* Matplotlib

* Seaborn

* NumPy

Para instalar as bibliotecas necessárias, execute:

```bash
  pip install pandas lifelines matplotlib seaborn numpy

```

## Carregando os Dados
Os dados são carregados a partir de um arquivo Excel chamado Crédito.xlsx.

```python
  import pandas as pd

  # Carregar dados do Excel
  credito = pd.read_excel("caminho/para/seu/arquivo/Crédito.xlsx")
  
  print(credito.columns)
  print(credito.head())

```

## Modelo de Kaplan-Meier
### Ajustando e Plotando para Todos os Dados
Ajustamos o modelo Kaplan-Meier para todos os dados e plotamos a curva de sobrevivência.

```python
  from lifelines import KaplanMeierFitter
  import matplotlib.pyplot as plt
  
  kmf = KaplanMeierFitter()
  kmf.fit(durations=credito['Time'], event_observed=credito['Status'])
  kmf.plot_survival_function()
  plt.title('Kaplan-Meier Survival Curve - All Data')
  plt.show()

```

## Agrupamento por Score
Criamos uma variável dummy baseada na pontuação (Score) e ajustamos o modelo de Kaplan-Meier para os dados agrupados.

```python
  credito['dummy_score'] = (credito['Score'] > 100).astype(int)
  
  kmf_grouped = KaplanMeierFitter()
  kmf_grouped.fit(durations=credito['Time'], event_observed=credito['Status'], label='dummy_score')
  kmf_grouped.plot_survival_function()
  plt.title('Kaplan-Meier Survival Curve - Grouped by dummy_score')
  plt.show()

```

## Separação dos Dados em Grupos
Separamos os dados em dois grupos com base na pontuação e ajustamos o modelo de Kaplan-Meier para cada grupo.

```python
  group_above_100 = credito[credito['dummy_score'] == 1]
  group_below_or_equal_100 = credito[credito['dummy_score'] == 0]
  
  kmf_above_100 = KaplanMeierFitter()
  kmf_below_or_equal_100 = KaplanMeierFitter()
  
  kmf_above_100.fit(durations=group_above_100['Time'], event_observed=group_above_100['Status'], label='Score > 100')
  kmf_below_or_equal_100.fit(durations=group_below_or_equal_100['Time'], event_observed=group_below_or_equal_100['Status'], label='Score ≤ 100')
  
  ax = kmf_above_100.plot_survival_function()
  kmf_below_or_equal_100.plot_survival_function(ax=ax)
  
  plt.title('Kaplan-Meier Survival Curve - Grouped by Score')
  plt.show()

```

## Estimativas de Sobrevivência
Obtemos as estimativas de sobrevivência para cada grupo em pontos de tempo específicos.

```python
  timeline = [30, 60, 90, 120, 150, 180, 210, 240, 270, 300, 330, 360]
  
  survival_above_100_at_timeline = kmf_above_100.predict(timeline)
  survival_below_or_equal_100_at_timeline = kmf_below_or_equal_100.predict(timeline)
  
  survival_estimates = pd.DataFrame({
      'Time': timeline,
      'Survival Score > 100': survival_above_100_at_timeline,
      'Survival Score ≤ 100': survival_below_or_equal_100_at_timeline
  })
  
  print(survival_estimates)

```

## Modelo de Riscos Proporcionais de Cox
Ajustamos o modelo de Cox usando as variáveis Rating_b e Rating_c e exibimos o resumo do modelo.

```python
  from lifelines import CoxPHFitter
  
  cph_score = CoxPHFitter()
  cph_score.fit(credito, duration_col='Time', event_col='Status', formula='Rating_b + Rating_c')
  cph_score.print_summary()

```
