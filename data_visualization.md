## Full  code

```python
"""
Enhanced Trend Analysis - PNG, PDF, and HTML Output with All Variables Comparison
Creates individual PNG files, multi-page PDFs, and HTML reports with Enhanced Collapsible Navigation
Order: National → District Subplots → Individual Districts → Chiefdom Subplots
ALL PLOTS SHOW ALL VARIABLES WITH LEGENDS - NO INDIVIDUAL VARIABLE PLOTS
ENHANCED NAVIGATION WITH COLLAPSIBLE MENU AND SUBMENUS
NO TEXT LABELS ON DATA POINTS
ALL LEGENDS ON TOP RIGHT CORNER OUTSIDE THE PLOT
"""

import os
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import re
from pathlib import Path
import zipfile
from matplotlib.backends.backend_pdf import PdfPages
import base64
from io import BytesIO

def create_comprehensive_trend_analysis(output_base_dir='trend_plots/'):
    """Create comprehensive trend analysis with PNG files, PDFs, and HTML with all variables comparison"""

    print("🚀 Starting Comprehensive Trend Analysis - All Variables Comparison...")
    print("=" * 70)

    # Variables to analyze
    variables = ['crude_incidence', 'adjusted1', 'adjusted2', 'adjusted3']

    # Load data once
    print("📊 Loading data...")
    try:
        df = pd.read_excel("/content/2024_snt_data.xlsx")
        print(f"✓ Data loaded: {len(df)} rows")
        print(f"✓ Available columns: {list(df.columns)}")
    except FileNotFoundError:
        print("❌ Error: /content/2024_snt_data.xlsx not found")
        return

    # Create main output directory and subdirectories
    os.makedirs(output_base_dir, exist_ok=True)
    os.makedirs(f'{output_base_dir}/png_files/', exist_ok=True)
    os.makedirs(f'{output_base_dir}/pdf_files/', exist_ok=True)

    # Store all figures for HTML generation
    html_images = []
    all_png_files = []

    def save_and_track_figure(fig, filename, title, section_type, save_png=True):
        """Save figure as PNG and track for HTML"""
        if save_png:
            png_path = f'{output_base_dir}/png_files/{filename}.png'
            fig.savefig(png_path, dpi=300, bbox_inches='tight')
            all_png_files.append(png_path)
            print(f"   ✓ Saved PNG: {filename}.png")

        # Convert to base64 for HTML
        buffer = BytesIO()
        fig.savefig(buffer, format='png', dpi=150, bbox_inches='tight')
        buffer.seek(0)
        image_base64 = base64.b64encode(buffer.getvalue()).decode('utf-8')
        buffer.close()

        html_images.append({
            'title': title,
            'image': image_base64,
            'section': section_type,
            'filename': filename
        })
        return image_base64

    def get_variable_data(variable, df):
        """Get year columns and data for a specific variable"""
        pattern = re.compile(f'^{variable}_(\d{{4}})$')
        year_cols = [col for col in df.columns
                     if pattern.match(col) and 2021 <= int(pattern.match(col).group(1)) <= 2024]
        
        if len(year_cols) < 2:
            return None, None
            
        years = [int(re.search(r'(\d{4})', col).group(1)) for col in year_cols]
        return year_cols, years

    def calculate_global_y_limits():
        """Calculate global y-axis limits for consistent scaling across all plots"""
        print(f"   🔍 Calculating global y-axis limits for all variables...")

        all_values = []
        districts = df['FIRST_DNAM'].dropna().unique()

        for variable in variables:
            year_cols, years = get_variable_data(variable, df)
            if year_cols is None:
                continue

            # National averages
            national_avg = df[year_cols].mean(axis=0)
            all_values.extend(national_avg.dropna().values)

            # District averages
            for district in districts:
                district_data = df[df['FIRST_DNAM'] == district]
                district_avg = district_data[year_cols].mean(axis=0)
                all_values.extend(district_avg.dropna().values)

                # Chiefdom averages within this district
                chiefdoms = district_data['FIRST_CHIE'].dropna().unique()
                for chiefdom in chiefdoms:
                    chie_data = district_data[district_data['FIRST_CHIE'] == chiefdom]
                    chie_avg = chie_data[year_cols].mean(axis=0)
                    all_values.extend(chie_avg.dropna().values)

        if len(all_values) == 0:
            return 0, 10

        global_min = 0
        global_max = max(all_values) * 1.15
        
        print(f"   ✓ Global y-axis range for all variables: 0 to {global_max:.1f}")
        return global_min, global_max

    # Calculate global limits for all variables
    global_y_min, global_y_max = calculate_global_y_limits()
    districts = df['FIRST_DNAM'].dropna().unique()

    # ==========================================
    # CREATE COMPREHENSIVE PDF REPORT
    # ==========================================
    print(f"\n📄 Creating comprehensive PDF report with all variables comparison...")
    
    pdf_filename = f'{output_base_dir}/pdf_files/all_variables_trend_analysis.pdf'
    
    with PdfPages(pdf_filename) as pdf:

        # ==========================================
        # PAGE 1: NATIONAL TRENDS - ALL VARIABLES
        # ==========================================
        print(f"📊 Creating Page 1: National trends comparison...")

        fig, ax = plt.subplots(figsize=(12, 8))

        # Get first variable to establish years
        first_var_year_cols, years = get_variable_data(variables[0], df)
        if first_var_year_cols is None:
            print("❌ Error: No valid data found for any variable")
            return

        # Plot all variables on the same axis
        for variable in variables:
            year_cols, _ = get_variable_data(variable, df)
            if year_cols is None:
                continue

            national_avg = df[year_cols].mean(axis=0)
            values = national_avg.values

            if not pd.isna(values).all():
                ax.plot(years, values, marker='o', linewidth=4, markersize=8, 
                       label=variable.replace("_", " ").title())

        ax.set_title('National Malaria Incidence Trends - All Variables Comparison (2021-2024)',
                     fontsize=16, fontweight='bold', pad=50)
        
        # Place legend horizontally immediately after title
        ax.legend(bbox_to_anchor=(0.5, 1.02), loc='lower center', ncol=4,
                 frameon=True, fancybox=True, shadow=True, fontsize=11)
        
        ax.set_xlabel('Year', fontweight='bold', fontsize=14)
        ax.set_ylabel('Cases per 1000 population', fontweight='bold', fontsize=14)
        ax.grid(True, alpha=0.3)
        ax.set_xticks(years)
        ax.set_xticklabels([int(year) for year in years])
        ax.tick_params(labelsize=12)
        ax.set_ylim(global_y_min, global_y_max)

        plt.tight_layout()
        pdf.savefig(fig, bbox_inches='tight', pad_inches=0.3)
        save_and_track_figure(fig, '01_national_trends_all_variables', 
                            'National Trends - All Variables Comparison', 'national')
        plt.close(fig)

        # ==========================================
        # PAGE 2: DISTRICTS OVERVIEW - ALL VARIABLES
        # ==========================================
        print(f"🏘️ Creating Page 2: Districts overview with all variables...")

        n_districts = len(districts)
        n_cols = 4  # Fixed to 4 columns as requested
        n_rows = int(np.ceil(n_districts / n_cols))

        fig, axes = plt.subplots(n_rows, n_cols, figsize=(16, 4 * n_rows))
        fig.suptitle('District-wise Malaria Incidence Trends - All Variables Comparison (2021-2024)',
                     fontsize=18, fontweight='bold', y=0.98)

        # Add horizontal legend immediately after title
        fig.legend([plt.Line2D([0], [0], marker='o', color=f'C{i}', linewidth=2.5, markersize=5) 
                   for i in range(len(variables))],
                  [var.replace("_", " ").title() for var in variables],
                  bbox_to_anchor=(0.5, 0.93), loc='center', ncol=4,
                  frameon=True, fancybox=True, shadow=True, fontsize=11)

        # Handle different subplot configurations
        if n_districts == 1:
            axes = [axes]
        elif n_rows == 1:
            axes = axes if n_districts > 1 else [axes]
        else:
            axes = axes.flatten()

        for i, district in enumerate(districts):
            if i >= len(axes):
                break

            ax = axes[i] if isinstance(axes, list) else axes[i]
            district_data = df[df['FIRST_DNAM'] == district]

            # Plot all variables for this district
            for variable in variables:
                year_cols, _ = get_variable_data(variable, df)
                if year_cols is None:
                    continue

                district_avg = district_data[year_cols].mean(axis=0)
                values = district_avg.values

                if not pd.isna(values).all():
                    ax.plot(years, values, marker='o', linewidth=2.5, markersize=5, 
                           label=variable.replace("_", " ").title(), alpha=0.8)

            ax.set_title(f'{district}', fontsize=12, fontweight='bold')
            ax.set_ylim(global_y_min, global_y_max)
            ax.grid(True, alpha=0.3)
            ax.set_xticks(years)
            ax.set_xticklabels([int(year) for year in years])
            ax.tick_params(labelsize=10)

        # Add legend in top right corner outside the entire subplot area
        if len(districts) > 0:
            fig.legend([plt.Line2D([0], [0], marker='o', color=f'C{i}', linewidth=2.5, markersize=5) 
                       for i in range(len(variables))],
                      [var.replace("_", " ").title() for var in variables],
                      bbox_to_anchor=(0.5, 0.93), loc='center', ncol=4,
                      frameon=True, fancybox=True, shadow=True, fontsize=11)

        # Hide unused subplots
        if isinstance(axes, list) or hasattr(axes, 'flatten'):
            axes_to_check = axes if isinstance(axes, list) else axes.flatten()
            for i in range(len(districts), len(axes_to_check)):
                axes_to_check[i].set_visible(False)

        # Add common labels
        fig.text(0.5, 0.04, 'Year', ha='center', fontweight='bold', fontsize=14)
        fig.text(0.02, 0.5, 'Cases per 1000 population', va='center', rotation='vertical',
                fontweight='bold', fontsize=14)

        plt.tight_layout(rect=[0.03, 0.08, 1, 0.90])  # Adjusted for horizontal legend after title
        pdf.savefig(fig, bbox_inches='tight', pad_inches=0.8)
        save_and_track_figure(fig, '02_districts_overview_all_variables',
                            'Districts Overview - All Variables Comparison', 'districts_overview')
        plt.close(fig)

        # ==========================================
        # PAGES 3+: INDIVIDUAL DISTRICT PLOTS - ALL VARIABLES
        # ==========================================
        print(f"📈 Creating Pages 3+: Individual district plots with all variables...")

        for district_idx, district in enumerate(districts, 1):
            print(f"   Creating Page {district_idx + 2}: {district}")

            fig, ax = plt.subplots(figsize=(12, 8))

            district_data = df[df['FIRST_DNAM'] == district]

            # Plot all variables for this district
            for variable in variables:
                year_cols, _ = get_variable_data(variable, df)
                if year_cols is None:
                    continue

                district_avg = district_data[year_cols].mean(axis=0)
                values = district_avg.values

                if not pd.isna(values).all():
                    ax.plot(years, values, marker='o', linewidth=3, markersize=8, 
                           label=variable.replace("_", " ").title())

            ax.set_title(f'Malaria Incidence Trends - {district} All Variables Comparison (2021-2024)',
                       fontsize=16, fontweight='bold', pad=50)

            # Place legend horizontally immediately after title
            ax.legend(bbox_to_anchor=(0.5, 1.02), loc='lower center', ncol=4,
                     frameon=True, fancybox=True, shadow=True, fontsize=11)
            
            ax.set_xlabel('Year', fontweight='bold', fontsize=14)
            ax.set_ylabel('Cases per 1000 population', fontweight='bold', fontsize=14)
            ax.grid(True, alpha=0.3)
            ax.set_xticks(years)
            ax.set_xticklabels([int(year) for year in years])
            ax.set_ylim(global_y_min, global_y_max)

            plt.tight_layout()
            pdf.savefig(fig, bbox_inches='tight', pad_inches=0.8)

            safe_name = "".join(c for c in district if c.isalnum() or c in (' ', '-', '_')).strip().replace(' ', '_')
            save_and_track_figure(fig, f'03_{district_idx:02d}_{safe_name}_all_variables',
                                f'{district} - All Variables Comparison', 'individual_districts')
            plt.close(fig)

        # ==========================================
        # FINAL PAGES: CHIEFDOM PLOTS - ALL VARIABLES
        # ==========================================
        print(f"🏡 Creating final pages: Chiefdom plots with all variables...")

        for district_idx, district in enumerate(districts, 1):
            print(f"   Creating chiefdom plots for {district}")

            district_data = df[df['FIRST_DNAM'] == district]
            if len(district_data) == 0:
                continue

            chiefdoms = district_data['FIRST_CHIE'].dropna().unique()
            if len(chiefdoms) == 0:
                continue

            # Calculate optimal grid - ALWAYS use 4 columns
            n_cols = 4  # Fixed to 4 columns for all subplot grids
            n_rows = int(np.ceil(len(chiefdoms) / n_cols))

            fig, axes = plt.subplots(n_rows, n_cols, figsize=(18, 6 * n_rows))
            fig.suptitle(f'Chiefdom-wise Malaria Incidence Trends - {district} All Variables Comparison',
                        fontsize=16, fontweight='bold', y=0.98)

            # Add horizontal legend immediately after title
            fig.legend([plt.Line2D([0], [0], marker='o', color=f'C{i}', linewidth=2, markersize=4) 
                       for i in range(len(variables))],
                      [var.replace("_", " ").title() for var in variables],
                      bbox_to_anchor=(0.5, 0.93), loc='center', ncol=4,
                      frameon=True, fancybox=True, shadow=True, fontsize=10)

            # Handle subplot configurations
            if len(chiefdoms) == 1:
                axes = [axes]
            elif n_rows == 1:
                axes = axes if len(chiefdoms) > 1 else [axes]
            else:
                axes = axes.flatten()

            for i, chiefdom in enumerate(chiefdoms):
                if n_rows > 1:
                    ax = axes[i]
                else:
                    ax = axes[i] if isinstance(axes, (list, np.ndarray)) else axes

                chie_data = district_data[district_data['FIRST_CHIE'] == chiefdom]

                # Plot all variables for this chiefdom
                for variable in variables:
                    year_cols, _ = get_variable_data(variable, df)
                    if year_cols is None:
                        continue

                    chie_avg = chie_data[year_cols].mean(axis=0)
                    values = chie_avg.values

                    if not pd.isna(values).all():
                        ax.plot(years, values, marker='o', linewidth=2, markersize=4, 
                               label=variable.replace("_", " ").title(), alpha=0.8)

                ax.set_title(f'{chiefdom}', fontsize=11, fontweight='bold')
                ax.set_ylim(global_y_min, global_y_max)
                ax.grid(True, alpha=0.3)
                ax.set_xticks(years)
                ax.set_xticklabels([int(year) for year in years])
                ax.tick_params(labelsize=9)

            # Add legend in top right corner outside the entire subplot area
            if len(chiefdoms) > 0:
                fig.legend([plt.Line2D([0], [0], marker='o', color=f'C{i}', linewidth=2, markersize=4) 
                           for i in range(len(variables))],
                          [var.replace("_", " ").title() for var in variables],
                          bbox_to_anchor=(0.5, 0.93), loc='center', ncol=4,
                          frameon=True, fancybox=True, shadow=True, fontsize=10)

            # Hide unused subplots
            if n_rows > 1:
                for i in range(len(chiefdoms), len(axes)):
                    axes[i].set_visible(False)
            elif isinstance(axes, (list, np.ndarray)) and len(chiefdoms) < len(axes):
                for i in range(len(chiefdoms), len(axes)):
                    axes[i].set_visible(False)

            # Add common labels
            fig.text(0.5, 0.02, 'Year', ha='center', fontweight='bold', fontsize=12)
            fig.text(0.02, 0.5, 'Cases per 1000 population', va='center', rotation='vertical',
                    fontweight='bold', fontsize=12)

            plt.tight_layout(rect=[0.03, 0.03, 1, 0.90])  # Adjusted for horizontal legend after title
            pdf.savefig(fig, bbox_inches='tight', pad_inches=0.3)

            safe_district_name = "".join(c for c in district if c.isalnum() or c in (' ', '-', '_')).strip().replace(' ', '_')
            save_and_track_figure(fig, f'04_{district_idx:02d}_{safe_district_name}_chiefdoms_all_variables',
                                f'{district} - Chiefdoms All Variables Comparison', 'chiefdoms')
            plt.close(fig)

    print(f"✅ PDF created: {pdf_filename}")

    # ==========================================
    # CREATE ENHANCED HTML REPORT
    # ==========================================
    print(f"\n🌐 Creating enhanced HTML report...")

    html_filename = f'{output_base_dir}/trend_analysis_report.html'

    # Group images by section
    sections = {
        'national': [],
        'districts_overview': [],
        'individual_districts': [],
        'chiefdoms': []
    }

    for img in html_images:
        section = img.get('section', 'other')
        if section in sections:
            sections[section].append(img)

    # Enhanced HTML with improved navigation
    html_content = f"""
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Trend Analysis Report - Enhanced Navigation</title>
        <style>
            body {{
                font-family: 'Arial', 'Helvetica', sans-serif;
                margin: 0;
                padding: 20px;
                background-color: #f4f6f9;
                line-height: 1.6;
                color: #2c3e50;
            }}
            .container {{
                max-width: 1200px;
                margin: 0 auto;
                background: white;
                padding: 30px;
                border-radius: 8px;
                box-shadow: 0 2px 10px rgba(0,0,0,0.1);
                border: 1px solid #e3e8ed;
            }}
            h1 {{
                color: #1e3a8a;
                text-align: center;
                border-bottom: 3px solid #2563eb;
                padding-bottom: 20px;
                margin-bottom: 40px;
                font-size: 2.2em;
                font-weight: 600;
            }}
            h2 {{
                color: white;
                background-color: #2563eb;
                margin: 40px -30px 30px -30px;
                padding: 20px 30px;
                font-size: 1.6em;
                font-weight: 500;
                border-left: 5px solid #1d4ed8;
            }}
            h3 {{
                color: #1e3a8a;
                margin-top: 35px;
                margin-bottom: 20px;
                padding-bottom: 8px;
                border-bottom: 2px solid #e3e8ed;
                font-size: 1.3em;
                font-weight: 500;
            }}
            .image-container {{
                background-color: #fafbfc;
                border: 1px solid #e3e8ed;
                border-radius: 6px;
                margin: 25px 0;
                padding: 20px;
                text-align: center;
            }}
            .image-container img {{
                max-width: 100%;
                height: auto;
                border: 1px solid #d1d9e0;
                border-radius: 4px;
            }}
            .image-title {{
                font-weight: 600;
                margin-bottom: 15px;
                color: #1e3a8a;
                font-size: 1.1em;
                background-color: #eff6ff;
                padding: 12px;
                border-radius: 4px;
                border-left: 4px solid #2563eb;
            }}
            .toc {{
                background-color: #eff6ff;
                border: 2px solid #2563eb;
                border-radius: 6px;
                padding: 25px;
                margin-bottom: 40px;
            }}
            .toc h3 {{
                color: #1e3a8a;
                margin-top: 0;
                margin-bottom: 15px;
                border-bottom: 2px solid #bfdbfe;
                padding-bottom: 10px;
                font-size: 1.3em;
            }}
            .toc ul {{
                list-style-type: none;
                padding-left: 0;
                margin: 0;
            }}
            .toc li {{
                margin: 6px 0;
            }}
            .toc a {{
                color: #1e3a8a;
                text-decoration: none;
                padding: 8px 12px;
                display: inline-block;
                border-radius: 4px;
                transition: background-color 0.2s ease;
                font-weight: 500;
            }}
            .toc a:hover {{
                background-color: #dbeafe;
                color: #1d4ed8;
            }}
            .variable-section {{
                border-top: 2px solid #e3e8ed;
                margin-top: 50px;
                padding-top: 10px;
            }}
            .stats-box {{
                background-color: #eff6ff;
                border: 2px solid #2563eb;
                border-radius: 6px;
                padding: 20px;
                margin: 20px 0;
                text-align: center;
                color: #1e3a8a;
            }}
            .scaling-notice {{
                background-color: #fef3c7;
                border: 2px solid #f59e0b;
                border-radius: 6px;
                padding: 20px;
                margin: 20px 0;
                color: #92400e;
                font-weight: 500;
            }}

            /* Enhanced Navigation Menu Styles */
            .section-nav {{
                position: fixed;
                top: 20px;
                right: 20px;
                background: white;
                border: 2px solid #2563eb;
                border-radius: 8px;
                max-width: 280px;
                min-width: 50px;
                z-index: 1000;
                box-shadow: 0 4px 20px rgba(0,0,0,0.15);
                transition: all 0.3s ease;
                overflow: hidden;
            }}

            .nav-header {{
                background: linear-gradient(135deg, #2563eb, #1d4ed8);
                color: white;
                padding: 12px 15px;
                cursor: pointer;
                display: flex;
                justify-content: space-between;
                align-items: center;
                font-weight: 600;
                font-size: 14px;
                user-select: none;
            }}

            .nav-toggle {{
                font-size: 18px;
                transition: transform 0.3s ease;
                line-height: 1;
            }}

            .nav-toggle.expanded {{
                transform: rotate(180deg);
            }}

            .nav-content {{
                max-height: 0;
                overflow: hidden;
                transition: max-height 0.3s ease;
                background: white;
            }}

            .nav-content.expanded {{
                max-height: 500px;
                padding: 10px 0;
            }}

            .section-nav.collapsed {{
                width: 50px;
            }}

            .section-nav.collapsed .nav-header {{
                justify-content: center;
            }}

            .section-nav.collapsed .nav-title {{
                display: none;
            }}

            .nav-main-item {{
                display: block;
                color: #2563eb;
                text-decoration: none;
                padding: 8px 15px;
                font-size: 13px;
                font-weight: 600;
                border-bottom: 1px solid #f1f5f9;
                transition: all 0.2s ease;
                cursor: pointer;
                user-select: none;
            }}

            .nav-main-item:hover {{
                background-color: #eff6ff;
                color: #1d4ed8;
            }}

            .nav-main-item.active {{
                background-color: #dbeafe;
                color: #1d4ed8;
                border-left: 4px solid #2563eb;
            }}

            /* Mobile responsiveness */
            @media (max-width: 768px) {{
                .section-nav {{
                    top: 10px;
                    right: 10px;
                    max-width: 250px;
                }}

                .container {{
                    padding: 15px;
                    margin: 0 10px;
                }}
            }}

            /* Scroll progress bar */
            .scroll-progress {{
                position: fixed;
                top: 0;
                left: 0;
                width: 0%;
                height: 3px;
                background: linear-gradient(90deg, #2563eb, #60a5fa);
                z-index: 1001;
                transition: width 0.3s ease;
            }}
        </style>
    </head>
    <body>
        <div class="scroll-progress" id="scrollProgress"></div>

        <div class="section-nav" id="sectionNav">
            <div class="nav-header" onclick="toggleNav()">
                <span class="nav-title">Navigation</span>
                <span class="nav-toggle" id="navToggle">▼</span>
            </div>
            <div class="nav-content" id="navContent">
                <a href="#national" class="nav-main-item" onclick="scrollToSection('national')">🌍 National Analysis</a>
                <a href="#districts_overview" class="nav-main-item" onclick="scrollToSection('districts_overview')">🏘️ Districts Overview</a>
                <a href="#individual_districts" class="nav-main-item" onclick="scrollToSection('individual_districts')">📊 Individual Districts</a>
                <a href="#chiefdoms" class="nav-main-item" onclick="scrollToSection('chiefdoms')">🏡 Chiefdoms Analysis</a>
            </div>
        </div>

        <div class="container">
            <h1>All Variables Malaria Trend Analysis<br><small>Comprehensive Comparative Report (2021-2024)</small></h1>

            <div class="scaling-notice">
                <strong>📊 All Variables Comparison:</strong> Every plot shows all variables (Crude Incidence, Adjusted 1, Adjusted 2, Adjusted 3) 
                with 4-column horizontal legends positioned immediately after each title. Consistent Y-axis scaling enables accurate visual comparison across all geographic areas.
            </div>

            <div class="toc">
                <h3>Table of Contents</h3>
                <ul>
                    <li><a href="#national">🌍 National Trends</a></li>
                    <li><a href="#districts_overview">🏘️ Districts Overview</a></li>
                    <li><a href="#individual_districts">📊 Individual Districts</a></li>
                    <li><a href="#chiefdoms">🏡 Chiefdoms Analysis</a></li>
                </ul>
            </div>
    """

    # Add sections
    section_titles = {
        'national': '🌍 National Trends - All Variables',
        'districts_overview': '🏘️ Districts Overview - All Variables', 
        'individual_districts': '📊 Individual Districts - All Variables',
        'chiefdoms': '🏡 Chiefdoms Analysis - All Variables'
    }

    for section_key, section_title in section_titles.items():
        if section_key in sections and sections[section_key]:
            html_content += f'''
            <div id="{section_key}" class="variable-section">
                <h2>{section_title}</h2>
            '''
            
            for img in sections[section_key]:
                html_content += f'''
                <div class="image-container">
                    <div class="image-title">{img['title']}</div>
                    <img src="data:image/png;base64,{img['image']}" alt="{img['title']}">
                </div>
                '''
            
            html_content += '</div>\n'

    html_content += '''
            <div class="stats-box" style="margin-top: 50px;">
                <h3>Analysis Features:</h3>
                <p><strong>All Variables Comparison:</strong> Every plot shows all 4 variables with legends</p>
                <p><strong>Legend Position:</strong> 4-column horizontal format immediately after titles</p>
                <p><strong>Consistent Scaling:</strong> Same Y-axis range for accurate comparison</p>
                <p><strong>Enhanced Navigation:</strong> Smooth scrolling and progress tracking</p>
            </div>
        </div>

        <script>
            let navExpanded = true;

            function toggleNav() {
                const nav = document.getElementById('sectionNav');
                const content = document.getElementById('navContent');
                const toggle = document.getElementById('navToggle');

                navExpanded = !navExpanded;

                if (navExpanded) {
                    nav.classList.remove('collapsed');
                    content.classList.add('expanded');
                    toggle.classList.add('expanded');
                } else {
                    nav.classList.add('collapsed');
                    content.classList.remove('expanded');
                    toggle.classList.remove('expanded');
                }
            }

            function scrollToSection(sectionId) {
                const target = document.getElementById(sectionId);
                if (target) {
                    target.scrollIntoView({
                        behavior: 'smooth',
                        block: 'start'
                    });
                    updateActiveNav(sectionId);
                }
            }

            function updateActiveNav(sectionId) {
                document.querySelectorAll('.nav-main-item').forEach(item => {
                    item.classList.remove('active');
                });
                
                const activeItem = document.querySelector(`a[href="#${sectionId}"]`);
                if (activeItem) {
                    activeItem.classList.add('active');
                }
            }

            function updateScrollProgress() {
                const scrollTop = window.pageYOffset;
                const docHeight = document.body.scrollHeight - window.innerHeight;
                const scrollPercent = (scrollTop / docHeight) * 100;
                document.getElementById('scrollProgress').style.width = scrollPercent + '%';
            }

            function setupIntersectionObserver() {
                const sections = document.querySelectorAll('.variable-section');
                
                const observer = new IntersectionObserver((entries) => {
                    entries.forEach(entry => {
                        if (entry.isIntersecting) {
                            updateActiveNav(entry.target.id);
                        }
                    });
                }, {
                    threshold: 0.1,
                    rootMargin: '-20% 0px -70% 0px'
                });

                sections.forEach(section => observer.observe(section));
            }

            window.addEventListener('scroll', updateScrollProgress);
            document.addEventListener('DOMContentLoaded', () => {
                setupIntersectionObserver();
                updateScrollProgress();
            });

            document.querySelectorAll('a[href^="#"]').forEach(anchor => {
                anchor.addEventListener('click', function (e) {
                    e.preventDefault();
                    const targetId = this.getAttribute('href').substring(1);
                    scrollToSection(targetId);
                });
            });
        </script>
    </body>
    </html>
    '''

    # Save HTML file
    with open(html_filename, 'w', encoding='utf-8') as f:
        f.write(html_content)

    print(f"✅ Enhanced HTML report created: {html_filename}")

    # ==========================================
    # CREATE FILE INDEX
    # ==========================================
    print(f"\n📋 Creating comprehensive file index...")

    index_content = f"""# All Variables Comparative Trend Analysis Report

Generated on: {pd.Timestamp.now().strftime('%Y-%m-%d %H:%M:%S')}
```
