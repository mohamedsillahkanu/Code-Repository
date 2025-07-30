## Step 1: Data Loading and Preprocessing

```python
#import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime
```
## Step2: Load the dataset
```python
df = pd.read_csv('malaria_data.csv')
```
## Step 3: Convert date column to datetime
```python
df['date'] = pd.to_datetime(df['date'])
```
# Step 4: Calculate confirmation rate
```python
df['confirmation_rate'] = (df['confirmed_cases'] / df['tested_cases']) * 100
```
