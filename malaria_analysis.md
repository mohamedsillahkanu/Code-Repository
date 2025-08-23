## Method

![NMCP lOGO](NMCP.png)


## Step

```python
import pandas as pd

# Direct raw CSV link from GitHub
url = "https://raw.githubusercontent.com/mohamedsillahkanu/Code-Repository/main/CHIRPS_Mean_2015_11.csv"

# Read the CSV
df = pd.read_csv(url)

# Show first 10 rows in HTML format
html_table = df.head(10).to_html(index=False)

# Save to an HTML file
with open("output.html", "w", encoding="utf-8") as f:
    f.write(html_table)

print("HTML table saved as 'output.html'")
```
