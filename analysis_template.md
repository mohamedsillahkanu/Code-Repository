```python
# Step 1: Import required libraries
import matplotlib.pyplot as plt
import pandas as pd
```
```python
# Step 2: Create dummy data
data = {
    'Category': ['A', 'B', 'C', 'D', 'E'],
    'Value': [23, 45, 12, 67, 34]
}
df = pd.DataFrame(data)
print("Dummy Data:")
print(df)
```

```python
# Step 3: Create the bar chart
plt.figure(figsize=(8, 5))
plt.bar(df['Category'], df['Value'], color='skyblue', edgecolor='black')
plt.title('Sample Bar Chart', fontsize=14)
plt.xlabel('Category')
plt.ylabel('Value')
plt.grid(axis='y', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
```

```python
# Step 4 (Optional): Save the chart as an image
# plt.savefig('bar_chart.png', dpi=300)
```
