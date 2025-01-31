## Stratified Sampling

https://colab.research.google.com/drive/12AzNCDb7oPWR3WQJVPh9kN9V01MplmZb?usp=sharing

<br>

```{python}
pip install pandas scikit-learn

import pandas as pd
from sklearn.model_selection import train_test_split
```

```{python}
df = pd.read_csv('fraud_original.csv')

X = df.drop(columns=['isFraud'])
y = df['isFraud']

_, sampled = train_test_split(df, test_size=0.2, stratify=y, random_state=42)
```
```{python}
# Original vs sampled by ratio

print(df['isFraud'].value_counts(normalize=True))
print(sampled['isFraud'].value_counts(normalize=True))
```

```{python}
# Original vs sampled by length

print(df['isFraud'].value_counts()) in j

print(sampled['isFraud'].value_counts())
```

```{python}
sampled.to_csv('fraud_sampled.csv', index=True)
files.download('fraud_sampled.csv')
```
