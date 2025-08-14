## SMC
```python
import geopandas as gpd
import pandas as pd  
import matplotlib.pyplot as plt
import requests
import time
import os

# Add headers to avoid connection issues
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
}

# Shapefile component files
files = ["Chiefdom2021.shp", "Chiefdom2021.shx", "Chiefdom2021.dbf"]
base_url = "https://github.com/mohamedsillahkanu/Code-Repository/raw/9faef1963141e18344df17a4310a7a07100679c4/"

# Download files with error handling
for f in files:
    if os.path.exists(f):
        print(f"✓ {f} already exists, skipping download")
        continue
    
    try:
        print(f"Downloading {f}...")
        r = requests.get(base_url + f, headers=headers, timeout=30)
        r.raise_for_status()
        with open(f, "wb") as out:
            out.write(r.content)
        print(f"✓ Downloaded {f}")
        time.sleep(1)  # Small delay between downloads
    except Exception as e:
        print(f"✗ Failed to download {f}: {e}")
        print(f"  Please download manually from: {base_url + f}")

# ===== 1. Load shapefile =====
try:
    gdf = gpd.read_file("Chiefdom2021.shp")
    print(f"✓ Loaded shapefile with {len(gdf)} features")
    print(f"Columns: {gdf.columns.tolist()}")
except Exception as e:
    print(f"✗ Failed to load shapefile: {e}")
    exit()

# ===== 2. Load Excel data =====
try:
    excel_path = "https://github.com/mohamedsillahkanu/Code-Repository/raw/cc8ca054a0ad965678e90f2214d9d99b4f4f7cba/rainfall_max_percentages_by_location.xlsx"
    df = pd.read_excel(excel_path)
    print(f"✓ Loaded first Excel file with {len(df)} rows")
    print(f"Columns: {df.columns.tolist()}")
except Exception as e:
    print(f"✗ Failed to load first Excel file: {e}")
    exit()

try:
    df2 = pd.read_excel("https://github.com/mohamedsillahkanu/Code-Repository/raw/2aa33791c3b0ede446e73a0c49669282aaf8da53/scenario_with_irs_smc_06_20_2025.xlsx")
    print(f"✓ Loaded second Excel file with {len(df2)} rows")
    print(f"Columns: {df2.columns.tolist()}")
except Exception as e:
    print(f"✗ Failed to load second Excel file: {e}")
    exit()

# Check for required columns
required_cols = ['FIRST_DNAM', 'FIRST_CHIE']
for dataset_name, dataset in [("Shapefile", gdf), ("Excel 1", df), ("Excel 2", df2)]:
    missing_cols = [col for col in required_cols if col not in dataset.columns]
    if missing_cols:
        print(f"⚠ Warning: {dataset_name} missing columns: {missing_cols}")

# ===== 3. Merge with shapefile =====
try:
    # Remove validate parameter to avoid 1:1 merge errors
    merged = gdf.merge(df, on=['FIRST_DNAM', 'FIRST_CHIE'], how='left')
    print(f"✓ First merge completed: {len(merged)} rows")
    
    merged = merged.merge(df2, on=['FIRST_DNAM', 'FIRST_CHIE'], how='left')
    print(f"✓ Second merge completed: {len(merged)} rows")
    
    # Check for required columns after merge
    required_analysis_cols = ['max_5_month_start', 'smc']
    missing_analysis_cols = [col for col in required_analysis_cols if col not in merged.columns]
    if missing_analysis_cols:
        print(f"⚠ Warning: Missing required columns for analysis: {missing_analysis_cols}")
        print(f"Available columns: {merged.columns.tolist()}")
        
except Exception as e:
    print(f"✗ Merge failed: {e}")
    exit()

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

try:
    merged["5_month_start"] = merged["max_5_month_start"].apply(recode_month)
    print("✓ Month recoding completed")
except KeyError as e:
    print(f"✗ Column not found for month recoding: {e}")
    exit()

# ===== 5. Apply "No SMC" if smc == "No SMC" =====
try:
    merged["Category"] = merged.apply(
        lambda row: "No SMC" if row["smc"] == "No SMC" else row["5_month_start"],
        axis=1
    )
    print("✓ Category assignment completed")
except KeyError as e:
    print(f"✗ Column not found for category assignment: {e}")
    exit()

# ===== 6. Count categories =====
cat_counts = merged["Category"].value_counts().to_dict()
print(f"Category counts: {cat_counts}")

# ===== 7. Assign colors =====
colors = {
    "June": "#1f77b4",    # blue
    "July": "#ff7f0e",    # orange
    "August": "#2ca02c",  # green
    "No SMC": "white"
}
merged["color"] = merged["Category"].map(colors).fillna("#D9D9D9")  # gray for other values

# ===== 8. Dissolve boundaries by FIRST_DNAM =====
try:
    boundaries = merged.dissolve(by="FIRST_DNAM", as_index=False)
    print("✓ Boundary dissolution completed")
except Exception as e:
    print(f"⚠ Warning: Boundary dissolution failed: {e}")
    print("Continuing without dissolved boundaries...")
    boundaries = None

# ===== 9. Plot =====
try:
    fig, ax = plt.subplots(1, 1, figsize=(10, 8))
    
    # Plot polygons with colors
    merged.plot(color=merged["color"], edgecolor="black", linewidth=0.5, ax=ax)
    
    # Add FIRST_DNAM boundaries if available
    if boundaries is not None:
        boundaries.boundary.plot(ax=ax, edgecolor="black", linewidth=1.2)
    
    # Custom legend with counts
    legend_elements = []
    for cat in ["June", "July", "August", "No SMC"]:
        if cat in cat_counts:
            ax.plot([], [], color=colors[cat], marker='s', linestyle='',
                    markersize=10, markeredgecolor='black',
                    label=f"{cat} ({cat_counts[cat]})")
    
    ax.legend(title="5-month start", loc='center left', bbox_to_anchor=(1, 0.5), 
              frameon=True, edgecolor='black')
    ax.set_title("5-month window start month with SMC status", fontsize=14)
    ax.axis("off")
    
    plt.tight_layout()
    
    # Save with error handling
    try:
        plt.savefig("5_month_window_SMC_map_counts.png", dpi=300, bbox_inches='tight')
        print("✓ Map saved as '5_month_window_SMC_map_counts.png'")
    except Exception as e:
        print(f"⚠ Warning: Could not save map: {e}")
    
    plt.show()
    print("✓ Map creation completed successfully!")
    
except Exception as e:
    print(f"✗ Plotting failed: {e}")
    import traceback
    traceback.print_exc()

```
