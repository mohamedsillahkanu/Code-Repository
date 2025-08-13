## Full Code
```python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import os
from matplotlib.ticker import MultipleLocator, FuncFormatter

# No output directory needed - displaying plots in console only

# Enhanced Impact Indicators data with unique titles and labels for each
impact_indicators_data = [
    {
        'name': 'Malaria case incidence: number and rate per 1000 people per year',
        'plot_title': 'Malaria Case Incidence Rate',
        'subtitle': 'Cases per 1,000 Population per Year',
        'y_label': 'Cases per 1,000 Population',
        'unit_description': 'case incidence rate',
        'baseline_2021_target': 298,
        'baseline_2021_achieved': 267,
        'years': [2022, 2023, 2024],
        'targets': [293, 288, 283],
        'achieved': [239, 309, 239],
        'step': 50,
        'format_type': 'number',
        'has_complete_data': True,
        'trend_goal': 'decrease'  # Lower is better
    },
    {
        'name': 'Malaria admissions: number and rate per 10,000 persons per year',
        'plot_title': 'Malaria Hospital Admissions',
        'subtitle': 'Admission Rate per 10,000 Persons per Year',
        'y_label': 'Admissions per 10,000 Persons',
        'unit_description': 'admission rate',
        'baseline_2021_target': 50,
        'baseline_2021_achieved': 42,
        'years': [2022, 2023, 2024],
        'targets': [49.1, 47.9, 46.5],
        'achieved': [47, 51, 36],
        'step': 10,
        'format_type': 'number',
        'has_complete_data': True,
        'trend_goal': 'decrease'  # Lower is better
    },
    {
        'name': 'Malaria test positivity rate',
        'plot_title': 'Malaria Test Positivity Rate',
        'subtitle': 'Percentage of Positive Test Results',
        'y_label': 'Positivity Rate (%)',
        'unit_description': 'positivity percentage',
        'baseline_2021_target': 59.3,
        'baseline_2021_achieved': 63.7,
        'years': [2022, 2023, 2024],
        'targets': [58.2, 56.7, 55.0],
        'achieved': [63.2, 66.0, 64.3],
        'step': 10,
        'format_type': 'percentage',
        'has_complete_data': True,
        'trend_goal': 'decrease'  # Lower positivity is better
    },
    {
        'name': 'Proportion of admissions for malaria',
        'plot_title': 'Proportion of Hospital Admissions Due to Malaria',
        'subtitle': 'Percentage of Total Admissions',
        'y_label': 'Proportion of Admissions (%)',
        'unit_description': 'admission proportion',
        'baseline_2021_target': 37.7,
        'baseline_2021_achieved': None,
        'years': [2022, 2023, 2024],
        'targets': [37.0, 36.0, 35.0],
        'achieved': [None, None, None],
        'step': 10,
        'format_type': 'percentage',
        'has_complete_data': False,
        'trend_goal': 'decrease'  # Lower proportion is better
    },
    {
        'name': 'Reported malaria cases (presumed and confirmed)',
        'plot_title': 'Total Reported Malaria Cases',
        'subtitle': 'Presumed and Confirmed Cases Combined',
        'y_label': 'Total Cases (Millions)',
        'unit_description': 'total cases',
        'baseline_2021_target': 2.396,
        'baseline_2021_achieved': 1.961,
        'years': [2022, 2023, 2024],
        'targets': [2.384, 2.372, 2.360],
        'achieved': [1.814, 2.448, 1.954],
        'step': 0.5,
        'format_type': 'millions',
        'has_complete_data': True,
        'trend_goal': 'decrease'  # Lower total cases is better
    },
    {
        'name': 'Inpatient malaria deaths: rate per 100,000 population',
        'plot_title': 'Inpatient Malaria Death Rate',
        'subtitle': 'Deaths per 100,000 Population',
        'y_label': 'Deaths per 100,000 Population',
        'unit_description': 'death rate',
        'baseline_2021_target': 17.6,
        'baseline_2021_achieved': 17.0,
        'years': [2022, 2023, 2024],
        'targets': [17.2, 16.8, 16.3],
        'achieved': [18.0, 18.0, 19.0],
        'step': 5,
        'format_type': 'number',
        'has_complete_data': True,
        'trend_goal': 'decrease'  # Lower death rate is better
    },
    {
        'name': 'Malaria mortality',
        'plot_title': 'Malaria Mortality Count',
        'subtitle': 'Total Number of Deaths',
        'y_label': 'Number of Deaths',
        'unit_description': 'mortality count',
        'baseline_2021_target': 35,
        'baseline_2021_achieved': 28,
        'years': [2022, 2023, 2024],
        'targets': [34, 33, 32],
        'achieved': [33, 30, 27],
        'step': 10,
        'format_type': 'number',
        'has_complete_data': True,
        'trend_goal': 'decrease'  # Lower mortality is better
    },
    {
        'name': 'Proportion of inpatient deaths due to malaria',
        'plot_title': 'Proportion of Inpatient Deaths Due to Malaria',
        'subtitle': 'Percentage of Total Inpatient Deaths',
        'y_label': 'Proportion of Deaths (%)',
        'unit_description': 'death proportion',
        'baseline_2021_target': 37.7,
        'baseline_2021_achieved': None,
        'years': [2022, 2023, 2024],
        'targets': [37.0, 36.0, 35.0],
        'achieved': [None, None, None],
        'step': 10,
        'format_type': 'percentage',
        'has_complete_data': False,
        'trend_goal': 'decrease'  # Lower proportion is better
    }
]

def calculate_variance(target, achieved):
    """Calculate variance as (Achieved - Target) / Target * 100"""
    if target == 0 or achieved is None:
        return None
    return ((achieved - target) / target) * 100

def format_millions(x, pos):
    """Format numbers in millions"""
    return f'{x:.1f}M'

def format_percentage(x, pos):
    """Format as percentage"""
    return f'{x:.0f}%'

def get_performance_status(achieved, target, trend_goal):
    """Determine if performance is good or bad based on trend goal"""
    if achieved is None or target is None:
        return 'unknown', 'gray'
    
    if trend_goal == 'decrease':
        # For indicators where lower is better
        if achieved < target:
            return 'good', 'green'  # Below target is good
        else:
            return 'poor', 'red'    # Above target is poor
    else:  # trend_goal == 'increase'
        # For indicators where higher is better
        if achieved > target:
            return 'good', 'green'  # Above target is good
        else:
            return 'poor', 'red'    # Below target is poor

def create_individual_trend_plot(indicator_data, indicator_index):
    """Create individual trend analysis plot for one indicator with unique styling"""

    # Extract data
    name = indicator_data['name']
    plot_title = indicator_data['plot_title']
    subtitle = indicator_data['subtitle']
    y_label = indicator_data['y_label']
    unit_description = indicator_data['unit_description']
    trend_goal = indicator_data['trend_goal']
    
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

    # Plot target line (always available) with enhanced styling
    ax.plot(all_years, all_targets, 'go-', label='Target', linewidth=4, markersize=12, 
            markerfacecolor='lightgreen', markeredgecolor='darkgreen', markeredgewidth=2)

    # Plot achieved line only if data is available
    if has_complete_data:
        ax.plot(all_years, all_achieved, 'bo-', label='Achieved', linewidth=4, markersize=12,
                markerfacecolor='lightblue', markeredgecolor='darkblue', markeredgewidth=2)

        # Add performance analysis arrows and variance calculations
        for i, year in enumerate(all_years):
            target_val = all_targets[i]
            achieved_val = all_achieved[i]

            # Determine arrow color based on performance vs trend goal
            status, arrow_color = get_performance_status(achieved_val, target_val, trend_goal)
            
            # Use different arrow colors for performance
            if status == 'good':
                arrow_color = 'forestgreen'
                arrow_alpha = 0.8
            elif status == 'poor':
                arrow_color = 'crimson'
                arrow_alpha = 0.8
            else:
                arrow_color = 'orange'
                arrow_alpha = 0.6

            # Draw arrow from target to achieved
            ax.annotate('', xy=(year, achieved_val), xytext=(year, target_val),
                        arrowprops=dict(arrowstyle='<->', color=arrow_color, lw=5,
                                      alpha=arrow_alpha, mutation_scale=25))

            # Add variance text with performance indicator
            variance_val = all_variances[i]
            if variance_val is not None:
                variance_text = f'+{variance_val:.1f}%' if variance_val >= 0 else f'{variance_val:.1f}%'
                mid_point = (target_val + achieved_val) / 2

                # Color code variance text based on performance
                if status == 'good':
                    variance_color = 'darkgreen'
                    variance_bg = 'lightgreen'
                elif status == 'poor':
                    variance_color = 'darkred'
                    variance_bg = 'lightcoral'
                else:
                    variance_color = 'darkorange'
                    variance_bg = 'moccasin'

                text_x_offset = 0.08
                ax.text(year + text_x_offset, mid_point, variance_text,
                        fontsize=12, color=variance_color, weight='bold',
                        bbox=dict(boxstyle="round,pad=0.4", facecolor=variance_bg, 
                                alpha=0.9, edgecolor=variance_color, linewidth=1))

            # Add achieved value with performance-based coloring
            if status == 'good':
                text_color = 'darkgreen'
                box_color = 'lightgreen'
            elif status == 'poor':
                text_color = 'darkred'
                box_color = 'lightcoral'
            else:
                text_color = 'darkorange'
                box_color = 'moccasin'

            if format_type == 'millions':
                achieved_label = f'{achieved_val:.1f}M'
            elif format_type == 'percentage':
                achieved_label = f'{achieved_val:.1f}%'
            else:
                achieved_label = f'{achieved_val:.1f}' if achieved_val != int(achieved_val) else f'{achieved_val:.0f}'

            y_range = max(all_targets + all_achieved) - min(all_targets + all_achieved)
            y_range = max(y_range, step_size)  # Ensure minimum range
            offset_direction = 1 if achieved_val > target_val else -1
            ax.text(year, achieved_val + offset_direction * y_range * 0.06,
                    achieved_label, ha='center', va='bottom' if offset_direction > 0 else 'top',
                    fontsize=12, color=text_color, weight='bold',
                    bbox=dict(boxstyle="round,pad=0.3", facecolor=box_color, alpha=0.8,
                            edgecolor=text_color, linewidth=1))
    else:
        # Enhanced "No Data Available" annotation for incomplete indicators
        ax.text(0.5, 0.5, f'ACHIEVED DATA NOT AVAILABLE\nfor {plot_title}\n\nOnly Target Trajectory Shown',
                transform=ax.transAxes, ha='center', va='center',
                fontsize=16, color='darkred', weight='bold',
                bbox=dict(boxstyle="round,pad=0.8", facecolor="mistyrose", 
                        alpha=0.9, edgecolor='darkred', linewidth=2))

    # Add target values above green points with enhanced styling
    for i, year in enumerate(all_years):
        target_val = all_targets[i]

        if format_type == 'millions':
            target_label = f'{target_val:.1f}M'
        elif format_type == 'percentage':
            target_label = f'{target_val:.1f}%'
        else:
            target_label = f'{target_val:.1f}' if target_val != int(target_val) else f'{target_val:.0f}'

        y_range = max(all_targets) - min(all_targets) if max(all_targets) != min(all_targets) else step_size
        ax.text(year, target_val + y_range * 0.06,
                target_label, ha='center', va='bottom', fontsize=12, color='darkgreen', weight='bold',
                bbox=dict(boxstyle="round,pad=0.3", facecolor="lightgreen", alpha=0.8,
                        edgecolor='darkgreen', linewidth=1))

    # Enhanced title with subtitle
    main_title = f'{plot_title}'
    full_title = f'{main_title}\n{subtitle} | Trend Analysis (2021-2024)'
    ax.set_title(full_title, fontsize=16, weight='bold', pad=25, linespacing=1.5)
    
    # Custom labels
    ax.set_xlabel('Year', fontsize=14, weight='bold')
    ax.set_ylabel(y_label, fontsize=14, weight='bold')
    
    # Enhanced legend
    legend = ax.legend(fontsize=13, loc='best', frameon=True, fancybox=True, shadow=True)
    legend.get_frame().set_facecolor('white')
    legend.get_frame().set_alpha(0.9)
    
    # Enhanced grid
    ax.grid(True, alpha=0.4, linestyle='--', linewidth=0.8)
    ax.set_facecolor('white')

    # Set x-axis
    ax.set_xticks(all_years)
    ax.tick_params(axis='both', which='major', labelsize=12)
    x_margin = 0.4
    ax.set_xlim(min(all_years) - x_margin, max(all_years) + x_margin)

    # Enhanced y-axis formatting
    if has_complete_data:
        all_values = all_targets + [x for x in all_achieved if x is not None]
        y_min = min(all_values)
        y_max = max(all_values)
    else:
        y_min = min(all_targets)
        y_max = max(all_targets)

    # Start from 0 but allow some padding
    y_max_rounded = int(np.ceil(y_max / step_size) * step_size)
    if y_max_rounded < y_max + step_size * 0.3:
        y_max_rounded += step_size

    ax.set_ylim(0, y_max_rounded)
    ax.yaxis.set_major_locator(MultipleLocator(step_size))

    # Apply formatting based on indicator type
    if format_type == 'percentage':
        ax.yaxis.set_major_formatter(FuncFormatter(format_percentage))
    elif format_type == 'millions':
        ax.yaxis.set_major_formatter(FuncFormatter(format_millions))

    # Add trend goal indicator
    trend_text = "Lower is Better" if trend_goal == 'decrease' else "Higher is Better"
    ax.text(0.02, 0.98, trend_text, transform=ax.transAxes, fontsize=11, 
            weight='bold', va='top', ha='left',
            bbox=dict(boxstyle="round,pad=0.3", facecolor="lightyellow", 
                    alpha=0.8, edgecolor='orange'))

    # Display plot in console only - no saving
    plt.tight_layout(pad=3.0)
    plt.show()
    
    # Enhanced variance summary
    print(f"\n{plot_title}")
    print(f"Description: {subtitle}")
    print("=" * 100)

    if has_complete_data:
        baseline_variance_text = f"+{baseline_variance:.1f}%" if baseline_variance >= 0 else f"{baseline_variance:.1f}%"
        baseline_status, _ = get_performance_status(baseline_achieved, baseline_target, trend_goal)
        performance_indicator = "[GOOD]" if baseline_status == 'good' else "[POOR]" if baseline_status == 'poor' else "[UNKNOWN]"
        
        print(f"2021 (Baseline): Target={baseline_target} {unit_description}, Achieved={baseline_achieved}, Variance={baseline_variance_text} {performance_indicator}")

        for i, year in enumerate(years):
            if variances[i] is not None:
                variance_text = f"+{variances[i]:.1f}%" if variances[i] >= 0 else f"{variances[i]:.1f}%"
                year_status, _ = get_performance_status(achieved[i], targets[i], trend_goal)
                performance_indicator = "[GOOD]" if year_status == 'good' else "[POOR]" if year_status == 'poor' else "[UNKNOWN]"
                status_text = "Meeting Goal" if year_status == 'good' else "Missing Target" if year_status == 'poor' else "Unknown"
                print(f"{year}: Target={targets[i]} {unit_description}, Achieved={achieved[i]}, Variance={variance_text} ({status_text}) {performance_indicator}")
    else:
        print("WARNING: ACHIEVED DATA NOT AVAILABLE - Only target trajectory shown")
        print(f"2021 (Baseline): Target={baseline_target} {unit_description}, Achieved=N/A")
        for i, year in enumerate(years):
            print(f"{year}: Target={targets[i]} {unit_description}, Achieved=N/A")

    return None  # No file saved, just displayed

# Process all 8 Impact Indicators with enhanced reporting
print("ENHANCED IMPACT INDICATORS TREND ANALYSIS")
print("="*100)
print(f"Data Source: Excel file - Complete Impact Indicators Section")
print(f"Baseline Year: 2021")
print(f"Analysis Period: 2021-2024")
print(f"Total Impact Indicators: {len(impact_indicators_data)}")
print(f"Enhanced Visualization: Unique titles, labels, and performance coding")
print(f"Displaying all plots in console - No files saved")
print("="*100)

plots_displayed = []
complete_data_count = 0
incomplete_data_count = 0

for i, indicator_data in enumerate(impact_indicators_data):
    print(f"\nProcessing Impact Indicator {i+1}/{len(impact_indicators_data)}:")
    print(f"Title: '{indicator_data['plot_title']}'")
    print(f"Description: {indicator_data['subtitle']}")

    if indicator_data['has_complete_data']:
        print("Status: Complete data available")
        complete_data_count += 1
    else:
        print("Status: Incomplete data - targets only")
        incomplete_data_count += 1

    create_individual_trend_plot(indicator_data, i)
    plots_displayed.append(indicator_data['plot_title'])
    print(f"Displayed: {indicator_data['plot_title']}")

print(f"\n" + "="*100)
print("ALL 8 ENHANCED IMPACT INDICATORS ANALYSIS COMPLETE")
print("="*100)
print(f"Total plots displayed: {len(plots_displayed)}")
print(f"Indicators with complete data: {complete_data_count}")
print(f"Indicators with targets only: {incomplete_data_count}")
print(f"All plots displayed in console - No files created")

print(f"\nSUMMARY OF ALL 8 IMPACT INDICATORS:")
for i, indicator_data in enumerate(impact_indicators_data, 1):
    status = "Complete Analysis" if indicator_data['has_complete_data'] else "Target Trajectory Only"
    trend = "Lower is Better" if indicator_data['trend_goal'] == 'decrease' else "Higher is Better"
    print(f"{i}. {status} - {trend}")
    print(f"   Title: {indicator_data['plot_title']}")
    print(f"   Description: {indicator_data['subtitle']}")

print(f"\nENHANCED FEATURES:")
print(f"- Unique titles and subtitles for each indicator")
print(f"- Custom Y-axis labels specific to each metric")
print(f"- Performance color coding (Green=Good, Red=Poor)")
print(f"- Trend goal indicators")
print(f"- Professional formatting with enhanced styling")
print(f"- Detailed variance analysis with performance assessment")

print(f"\nAll 8 Enhanced Impact Indicator plots displayed!")
print(f"Professional styling with unique customization")
print(f"Plots shown directly in console output")

```
