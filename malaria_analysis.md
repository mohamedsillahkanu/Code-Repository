## Step

```python
import pandas as pd

try:
    # Try with explicit encoding
    df = pd.read_csv("CHIRPS_Mean_2015_11.csv", encoding='utf-8')
    print("Loaded with UTF-8 encoding")
except UnicodeDecodeError:
    try:
        df = pd.read_csv("CHIRPS_Mean_2015_11.csv", encoding='latin-1')
        print("Loaded with Latin-1 encoding")
    except Exception as e:
        print(f"Encoding error: {e}")
except Exception as e:
    print(f"Other error: {e}")

if 'df' in locals():
    print(df.head(10))
```
