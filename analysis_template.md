### 1. Data Loading and Preprocessing

\`\`\`python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime

# Load the dataset
df = pd.read_csv('malaria_data.csv')

# Convert date column to datetime
df['date'] = pd.to_datetime(df['date'])

# Calculate confirmation rate
df['confirmation_rate'] = (df['confirmed_cases'] / df['tested_cases']) * 100
\`\`\`
