## SMC map code
```python
import geopandas as gpd
import pandas as pd  
import matplotlib.pyplot as plt
import requests

files = ["Chiefdom2021.shp", "Chiefdom2021.shx", "Chiefdom2021.dbf"]
base_url = "https://github.com/mohamedsillahkanu/Code-Repository/raw/9faef1963141e18344df17a4310a7a07100679c4/"


for f in files:
    r = requests.get(base_url + f)
    with open(f, "wb") as out:
        out.write(r.content)

# ===== 1. Load shapefile =====

gdf = gpd.read_file("Chiefdom2021.shp")

# ===== 2. Load Excel data =====
excel_path = "https://github.com/mohamedsillahkanu/Code-Repository/raw/cc8ca054a0ad965678e90f2214d9d99b4f4f7cba/rainfall_max_percentages_by_location.xlsx"
df = pd.read_excel(excel_path)

df2 = pd.read_excel("https://github.com/mohamedsillahkanu/Code-Repository/raw/2aa33791c3b0ede446e73a0c49669282aaf8da53/scenario_with_irs_smc_06_20_2025.xlsx")
df2.to_excel("maximum window percent plotting data_08_13_2025.xls")

# ===== 3. Merge with shapefile =====
merged = gdf.merge(df, on=['FIRST_DNAM', 'FIRST_CHIE'], how='left', validate='1:1')
merged = merged.merge(df2, on=['FIRST_DNAM', 'FIRST_CHIE'], how='left', validate='1:1')

# ===== 4. Recode 6,7,8 to month names =====
def recode_month(val):
    if pd.isnull(val):
        return val
    elif val == 6:
        return "June"
    elif val == 7:
        return "July"
    elif val == 8:
        return "August"
    else:
        return val

merged["5_month_start"] = merged["max_5_month_start"].apply(recode_month)

# ===== 5. Apply "No SMC" if smc == "No SMC" =====
merged["Category"] = merged.apply(
    lambda row: "No SMC" if row["smc"] == "No SMC" else row["5_month_start"],
    axis=1
)

# ===== 6. Count categories =====
cat_counts = merged["Category"].value_counts().to_dict()

# ===== 7. Assign colors =====
colors = {
    "June": "#1f77b4",    # blue
    "July": "#ff7f0e",    # orange
    "August": "#2ca02c",  # green
    "No SMC": "white"
}
merged["color"] = merged["Category"].map(colors).fillna("#D9D9D9")  # gray for other values

# ===== 8. Dissolve boundaries by FIRST_DNAM =====
boundaries = merged.dissolve(by="FIRST_DNAM", as_index=False)

# ===== 9. Plot =====
fig, ax = plt.subplots(1, 1, figsize=(5, 5))

# Plot polygons with colors
merged.plot(color=merged["color"], edgecolor="black", linewidth=0.5, ax=ax)

# Add FIRST_DNAM boundaries
boundaries.boundary.plot(ax=ax, edgecolor="black", linewidth=1.2)

# Custom legend with counts
for cat in ["June", "July", "August", "No SMC"]:
    if cat in cat_counts:
        ax.plot([], [], color=colors[cat], marker='s', linestyle='',
                markersize=10, markeredgecolor='black',
                label=f"{cat} ({cat_counts[cat]})")

ax.legend(title="5-month start", loc='center left', bbox_to_anchor=(1, 0.5), frameon=True, edgecolor='black')

ax.set_title("5-month window start month with SMC status", fontsize=14)
ax.axis("off")

plt.tight_layout()
plt.savefig("5_month_window_SMC_map_counts.png", dpi=300, bbox_inches='tight')
plt.show()
```
