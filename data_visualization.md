# Malaria Impact Indicators Trend Analysis - Complete Tutorial

## Step 1: Environment Setup and Library Import
Import all necessary Python libraries for data visualization and analysis. These libraries provide the core functionality for creating professional trend analysis plots.

```python
# Import necessary libraries
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import os
from matplotlib.ticker import MultipleLocator, FuncFormatter

# Create output directory for individual plots
output_dir = "impact_indicators_complete"
os.makedirs(output_dir, exist_ok=True)
```

## Step 2: Data Structure Definition
Define the complete dataset for all 8 Impact Indicators extracted from the Excel file. This includes baseline data, targets, and achieved values for comprehensive trend analysis.

```python
# All 8 specific Impact Indicators data extracted from Excel file with exact names
impact_indicators_data = [
    {
        'name': 'Malaria case incidence: number and rate per 1000 people per year',
        'baseline_2021_target': 298,
        'baseline_2021_achieved': 267,
        'years': [2022, 2023, 2024],
        'targets': [293, 288, 283],
        'achieved': [239, 309, 239],
        'step': 50,
        'format_type': 'number',
        'has_complete_data': True
    },
    {
        'name': 'Malaria admissions: number and rate per 10,000 persons per year',
        'baseline_2021_target': 50,
        'baseline_2021_achieved': 42,
        'years': [2022, 2023, 2024],
        'targets': [49.1, 47.9, 46.5],
        'achieved': [47, 51, 36],
        'step': 10,
        'format_type': 'number',
        'has_complete_data': True
    },
    {
        'name': 'Malaria test positivity rate',
        'baseline_2021_target': 59.3,
        'baseline_2021_achieved': 63.7,
        'years': [2022, 2023, 2024],
        'targets': [58.2, 56.7, 55.0],
        'achieved': [63.2, 66.0, 64.3],
        'step': 10,
        'format_type': 'percentage',
        'has_complete_data': True
    },
    {
        'name': 'Proportion of admissions for malaria',
        'baseline_2021_target': 37.7,
        'baseline_2021_achieved': None,  # No achieved data
        'years': [2022, 2023, 2024],
        'targets': [37.0, 36.0, 35.0],
        'achieved': [None, None, None],  # No achieved data for any year
        'step': 10,
        'format_type': 'percentage',
        'has_complete_data': False
    },
    {
        'name': 'Reported malaria cases (presumed and confirmed)',
        'baseline_2021_target': 2.396,  # Convert to millions
        'baseline_2021_achieved': 1.961,
        'years': [2022, 2023, 2024],
        'targets': [2.384, 2.372, 2.360],
        'achieved': [1.814, 2.448, 1.954],
        'step': 0.5,
        'format_type': 'millions',
        'has_complete_data': True
    },
    {
        'name': 'Inpatient malaria deaths per year: rate per 100,000 persons per year',
        'baseline_2021_target': 17.6,
        'baseline_2021_achieved': 17.0,
        'years': [2022, 2023, 2024],
        'targets': [17.2, 16.8, 16.3],
        'achieved': [18.0, 18.0, 19.0],
        'step': 5,
        'format_type': 'number',
        'has_complete_data': True
    },
    {
        'name': 'Malaria mortality: number and rate per 100,000 persons per year',
        'baseline_2021_target': 35,
        'baseline_2021_achieved': 28,
        'years': [2022, 2023, 2024],
        'targets': [34, 33, 32],
        'achieved': [33, 30, 27],
        'step': 10,
        'format_type': 'number',
        'has_complete_data': True
    },
    {
        'name': 'Proportion of inpatient deaths due to malaria',
        'baseline_2021_target': 37.7,
        'baseline_2021_achieved': None,  # No achieved data
        'years': [2022, 2023, 2024],
        'targets': [37.0, 36.0, 35.0],
        'achieved': [None, None, None],  # No achieved data for any year
        'step': 10,
        'format_type': 'percentage',
        'has_complete_data': False
    }
]
```

## Step 3: Utility Functions for Analysis
Create helper functions for variance calculations and formatting to ensure consistent and accurate trend analysis across all indicators.

```python
def calculate_variance(target, achieved):
    """Calculate variance as (Achieved - Target) / Target * 100"""
    if target == 0 or achieved is None:
        return None
    return ((achieved - target) / target) * 100

def format_millions(x, pos):
    """Format numbers in millions"""
    return f'{x:.0f}M'

def format_percentage(x, pos):
    """Format as percentage"""
    return f'{x:.0f}%'
```

## Step 4: Individual Trend Plot Creation Function
Develop the main visualization function that creates professional, publication-ready trend analysis plots for each impact indicator.

```python
def create_individual_trend_plot(indicator_data, indicator_index):
    """Create individual trend analysis plot for one indicator and save as PNG"""

    # Extract data
    name = indicator_data['name']
    baseline_target = indicator_data['baseline_2021_target']
    baseline_achieved = indicator_data['baseline_2021_achieved']
    years = indicator_data['years']
    targets = indicator_data['targets']
    achieved = indicator_data['achieved']
    step_size = indicator_data['step']
    format_type = indicator_data['format_type']
    has_complete_data = indicator_data['has_complete_data']

    # Create individual plot with proper margins
    fig, ax = plt.subplots(figsize=(14, 10))

    # Plot baseline (2021) and subsequent years
    all_years = [2021] + years
    all_targets = [baseline_target] + targets

    # Handle missing achieved data
    if has_complete_data:
        all_achieved = [baseline_achieved] + achieved
        # Calculate variances for complete data
        baseline_variance = calculate_variance(baseline_target, baseline_achieved)
        variances = [calculate_variance(t, a) for t, a in zip(targets, achieved)]
        all_variances = [baseline_variance] + variances
    else:
        # For indicators with no achieved data, create placeholder
        all_achieved = [None] * len(all_years)
        all_variances = [None] * len(all_years)

    # Plot target line (always available)
    ax.plot(all_years, all_targets, 'go-', label='Target', linewidth=3, markersize=10)

    # Plot achieved line only if data is available
    if has_complete_data:
        ax.plot(all_years, all_achieved, 'bo-', label='Achieved', linewidth=3, markersize=10)
```

## Step 5: Visual Enhancement and Annotations
Add visual elements including variance arrows, color-coded annotations, and professional formatting to make the plots informative and presentation-ready.

```python
        # Add arrows and variance calculations
        for i, year in enumerate(all_years):
            target_val = all_targets[i]
            achieved_val = all_achieved[i]

            # Determine arrow color based on direction
            if achieved_val < target_val:
                arrow_color = 'gold'  # Yellow for reduction
            else:
                arrow_color = 'red'   # Red for increase

            # Draw arrow from target to achieved
            ax.annotate('', xy=(year, achieved_val), xytext=(year, target_val),
                        arrowprops=dict(arrowstyle='<->', color=arrow_color, lw=4,
                                      alpha=1.0, mutation_scale=20))

            # Add variance text
            variance_val = all_variances[i]
            if variance_val is not None:
                variance_text = f'+{variance_val:.1f}%' if variance_val >= 0 else f'{variance_val:.1f}%'
                mid_point = (target_val + achieved_val) / 2

                text_x_offset = 0.08
                ax.text(year + text_x_offset, mid_point, variance_text,
                        fontsize=11, color='black', weight='bold',
                        bbox=dict(boxstyle="round,pad=0.3", facecolor="white", alpha=0.9, edgecolor='black'))

            # Add achieved value - RED TEXT if above target, BLUE if below
            text_color = 'red' if achieved_val > target_val else 'darkblue'
            box_color = 'lightcoral' if achieved_val > target_val else 'lightblue'

            if format_type == 'millions':
                achieved_label = f'{achieved_val:.0f}M'
            elif format_type == 'percentage':
                achieved_label = f'{achieved_val:.1f}%'
            else:
                achieved_label = f'{achieved_val:.0f}'

            y_range = max(all_targets) - min(all_targets)
            offset_direction = 1 if achieved_val > target_val else -1
            ax.text(year, achieved_val + offset_direction * y_range * 0.04,
                    achieved_label, ha='center', va='bottom' if offset_direction > 0 else 'top',
                    fontsize=11, color=text_color, weight='bold',
                    bbox=dict(boxstyle="round,pad=0.2", facecolor=box_color, alpha=0.7))
    else:
        # Add "No Data Available" annotation for incomplete indicators
        ax.text(0.5, 0.5, 'ACHIEVED DATA NOT AVAILABLE\nOnly Target Values Shown',
                transform=ax.transAxes, ha='center', va='center',
                fontsize=14, color='red', weight='bold',
                bbox=dict(boxstyle="round,pad=0.5", facecolor="lightyellow", alpha=0.8, edgecolor='red'))
```

## Step 6: Chart Formatting and Styling
Apply professional formatting including axis labels, grid lines, legends, and appropriate scaling for publication-quality output.

```python
    # Add target values above green points
    for i, year in enumerate(all_years):
        target_val = all_targets[i]

        if format_type == 'millions':
            target_label = f'{target_val:.0f}M'
        elif format_type == 'percentage':
            target_label = f'{target_val:.1f}%'
        else:
            target_label = f'{target_val:.0f}'

        y_range = max(all_targets) - min(all_targets) if max(all_targets) != min(all_targets) else 10
        ax.text(year, target_val + y_range * 0.04,
                target_label, ha='center', va='bottom', fontsize=11, color='darkgreen', weight='bold',
                bbox=dict(boxstyle="round,pad=0.2", facecolor="lightgreen", alpha=0.7))

    # Formatting and title
    ax.set_title(f'{name}\nTrend Analysis (2021-2024)', fontsize=14, weight='bold', pad=20)
    ax.set_xlabel('Year', fontsize=12, weight='bold')
    ax.set_ylabel('Value', fontsize=12, weight='bold')
    ax.legend(fontsize=12)
    ax.grid(True, alpha=0.3)

    # Set x-axis
    ax.set_xticks(all_years)
    ax.tick_params(axis='both', which='major', labelsize=11)
    x_margin = 0.3
    ax.set_xlim(min(all_years) - x_margin, max(all_years) + x_margin)

    # Set y-axis starting from 0 with specified step sizes
    if has_complete_data:
        y_max = max(all_targets + [x for x in all_achieved if x is not None])
    else:
        y_max = max(all_targets)

    y_max_rounded = int(np.ceil(y_max / step_size) * step_size)
    if y_max_rounded < y_max + step_size * 0.2:
        y_max_rounded += step_size

    ax.set_ylim(0, y_max_rounded)
    ax.yaxis.set_major_locator(MultipleLocator(step_size))

    # Apply formatting based on indicator type
    if format_type == 'percentage':
        ax.yaxis.set_major_formatter(FuncFormatter(format_percentage))
    elif format_type == 'millions':
        ax.yaxis.set_major_formatter(FuncFormatter(format_millions))
```

## Step 7: File Export and Variance Reporting
Save high-quality PNG files and generate detailed variance analysis reports for each indicator to support decision-making.

```python
    # Save individual plot
    safe_name = name.replace(':', '').replace('/', '_').replace('(', '').replace(')', '').replace(' ', '_')[:60]
    filename = f"impact_{indicator_index+1:02d}_{safe_name}.png"
    filepath = os.path.join(output_dir, filename)

    plt.tight_layout(pad=2.0)
    fig.savefig(filepath, dpi=300, bbox_inches='tight', pad_inches=0.3,
                facecolor='white', edgecolor='none')

    plt.show()
    plt.close(fig)

    # Print variance summary
    print(f"\n{name}")
    print("=" * 120)

    if has_complete_data:
        baseline_variance_text = f"+{baseline_variance:.1f}%" if baseline_variance >= 0 else f"{baseline_variance:.1f}%"
        print(f"2021 (Baseline): Target={baseline_target}, Achieved={baseline_achieved}, Variance={baseline_variance_text}")

        for i, year in enumerate(years):
            if variances[i] is not None:
                variance_text = f"+{variances[i]:.1f}%" if variances[i] >= 0 else f"{variances[i]:.1f}%"
                status = "Above target" if variances[i] > 0 else "Below target"
                print(f"{year}: Target={targets[i]}, Achieved={achieved[i]}, Variance={variance_text} ({status})")
    else:
        print("⚠️  ACHIEVED DATA NOT AVAILABLE - Only target values shown")
        print(f"2021 (Baseline): Target={baseline_target}, Achieved=N/A")
        for i, year in enumerate(years):
            print(f"{year}: Target={targets[i]}, Achieved=N/A")

    return filepath
```

## Step 8: Batch Processing and Analysis Execution
Process all 8 Impact Indicators systematically, generating comprehensive trend analysis plots and tracking data completeness across indicators.

```python
# Process all 8 Impact Indicators
print("ALL 8 IMPACT INDICATORS TREND ANALYSIS")
print("="*120)
print(f"Data Source: Excel file - Complete Impact Indicators Section")
print(f"Baseline Year: 2021")
print(f"Analysis Period: 2021-2024")
print(f"Total Impact Indicators: {len(impact_indicators_data)}")
print("="*120)

saved_files = []
complete_data_count = 0
incomplete_data_count = 0

for i, indicator_data in enumerate(impact_indicators_data):
    print(f"\nProcessing Impact Indicator {i+1}/{len(impact_indicators_data)}:")
    print(f"'{indicator_data['name']}'")

    if indicator_data['has_complete_data']:
        print("✓ Complete data available")
        complete_data_count += 1
    else:
        print("⚠️  Incomplete data - targets only")
        incomplete_data_count += 1

    filepath = create_individual_trend_plot(indicator_data, i)
    saved_files.append(filepath)
    print(f"✓ Saved: {os.path.basename(filepath)}")
```

## Step 9: Summary Report and Data Quality Assessment
Generate comprehensive summary reports categorizing indicators by data completeness and providing actionable insights for program management.

```python
print(f"\n" + "="*120)
print("ALL 8 IMPACT INDICATORS ANALYSIS COMPLETE")
print("="*120)
print(f"Total plots created: {len(saved_files)}")
print(f"Indicators with complete data: {complete_data_count}")
print(f"Indicators with targets only: {incomplete_data_count}")
print(f"Output directory: {output_dir}")

print("\nAll 8 Impact Indicators processed:")
for i, indicator_data in enumerate(impact_indicators_data, 1):
    status = "✓ Complete" if indicator_data['has_complete_data'] else "⚠️  Targets Only"
    print(f"{i}. {status} - {indicator_data['name']}")

print("\nData Status by Indicator:")
print("COMPLETE DATA (6 indicators):")
complete_indicators = [ind for ind in impact_indicators_data if ind['has_complete_data']]
for i, ind in enumerate(complete_indicators, 1):
    print(f"  {i}. {ind['name']}")

print("\nTARGETS ONLY (2 indicators):")
incomplete_indicators = [ind for ind in impact_indicators_data if not ind['has_complete_data']]
for i, ind in enumerate(incomplete_indicators, 1):
    print(f"  {i}. {ind['name']}")

print(f"\n✓ All 8 Impact Indicator trend analysis plots saved!")
print(f"✓ Resolution: 300 DPI (presentation quality)")
print(f"✓ Format: PNG with white background")
print(f"✓ Location: ./{output_dir}/ directory")

print(f"\n🎯 COMPLETE IMPACT INDICATORS SERIES!")
print(f"📊 All 8 indicators from Excel Impact section")
print(f"📈 6 with full trend analysis, 2 with target trajectories")
print(f"🔢 Professional formatting with exact indicator names")
print(f"💾 Individual high-quality PNG files ready for use")
```
