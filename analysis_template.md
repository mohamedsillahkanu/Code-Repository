# Malaria Data Analysis Template

## Overview
This template provides a comprehensive framework for analyzing malaria surveillance data in Sierra Leone.

## Data Requirements

### Required Columns
- **Date**: Date of data collection (YYYY-MM-DD format)
- **District**: Administrative district name
- **Facility**: Health facility name
- **Tested Cases**: Number of suspected cases tested
- **Confirmed Cases**: Number of confirmed malaria cases
- **Age Group**: Patient age categories (Under 5, 5-14, 15+)
- **Gender**: Patient gender (Male, Female)

### Data Quality Checks
> **Important**: Ensure all dates are valid and in correct format
> Check for missing values in critical columns
> Validate that confirmed cases ≤ tested cases
> Verify district and facility names are standardized

##STEP## Data Processing and Analysis Steps

###SUBSTEP### 1. Data Loading and Preprocessing

```python
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
```

####SUB#### Basic Data Exploration
- Check data shape and structure
- Identify missing values
- Review data types and formats
- Generate summary statistics

####SUB#### Data Validation
- Ensure date ranges are realistic
- Validate numerical constraints
- Check for duplicate records
- Verify categorical values

###SUBSTEP### 2. Data Cleaning and Validation

```python
# Data cleaning function
def clean_malaria_data(df):
    """
    Clean and validate malaria surveillance data
    
    Parameters:
    df (DataFrame): Raw malaria data
    
    Returns:
    DataFrame: Cleaned data
    """
    # Remove duplicate records
    df = df.drop_duplicates()
    
    # Handle missing values
    df['tested_cases'] = df['tested_cases'].fillna(0)
    df['confirmed_cases'] = df['confirmed_cases'].fillna(0)
    
    # Validate data consistency
    df = df[df['confirmed_cases'] <= df['tested_cases']]
    
    # Standardize district names
    district_mapping = {
        'Western Area Urban': 'Western Urban',
        'Western Area Rural': 'Western Rural',
        'Bo District': 'Bo',
        'Kenema District': 'Kenema'
    }
    df['district'] = df['district'].replace(district_mapping)
    
    return df

# Apply cleaning function
cleaned_df = clean_malaria_data(df)
print(f"Data cleaned: {len(df)} -> {len(cleaned_df)} records")
```

####SUB#### Missing Value Treatment
- Identify patterns in missing data
- Apply appropriate imputation methods
- Document assumptions and decisions

####SUB#### Outlier Detection
- Use statistical methods (IQR, Z-score)
- Visual inspection with box plots
- Domain knowledge validation

###SUBSTEP### 3. Key Performance Indicators Calculation

```python
def calculate_kpis(df):
    """Calculate key performance indicators"""
    
    kpis = {
        'total_tested': df['tested_cases'].sum(),
        'total_confirmed': df['confirmed_cases'].sum(),
        'overall_confirmation_rate': (df['confirmed_cases'].sum() / df['tested_cases'].sum()) * 100,
        'districts_analyzed': df['district'].nunique(),
        'facilities_analyzed': df['facility'].nunique(),
        'date_range': f"{df['date'].min().strftime('%Y-%m-%d')} to {df['date'].max().strftime('%Y-%m-%d')}"
    }
    
    return kpis

# Calculate KPIs
kpis = calculate_kpis(cleaned_df)
for key, value in kpis.items():
    print(f"{key.replace('_', ' ').title()}: {value}")
```

####SUB#### Descriptive Statistics
- Calculate central tendencies
- Measure variability
- Generate frequency distributions

####SUB#### Inferential Statistics
- Hypothesis testing
- Confidence intervals
- Correlation analysis

##STEP_END##

##STEP## Data Visualization and Analysis

###SUBSTEP### Time Series Analysis

```python
# Monthly trend analysis
monthly_trends = df.groupby(pd.Grouper(key='date', freq='M'))['confirmation_rate'].mean()

plt.figure(figsize=(12, 6))
plt.plot(monthly_trends.index, monthly_trends.values, marker='o')
plt.title('Monthly Malaria Confirmation Rate Trends')
plt.xlabel('Month')
plt.ylabel('Confirmation Rate (%)')
plt.xticks(rotation=45)
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

####SUB#### Time Series Components
- Trend analysis
- Seasonal decomposition
- Moving averages
- Autocorrelation analysis

####SUB#### Forecasting Methods
- Linear trend projection
- Seasonal ARIMA models
- Exponential smoothing

###SUBSTEP### Geographic Analysis

```python
# District comparison
district_rates = df.groupby('district')['confirmation_rate'].mean().sort_values(ascending=False)

plt.figure(figsize=(10, 6))
bars = plt.bar(range(len(district_rates)), district_rates.values)
plt.title('Malaria Confirmation Rate by District')
plt.xlabel('District')
plt.ylabel('Confirmation Rate (%)')
plt.xticks(range(len(district_rates)), district_rates.index, rotation=45, ha='right')

# Add value labels on bars
for i, bar in enumerate(bars):
    plt.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 0.5, 
             f'{district_rates.values[i]:.1f}%', 
             ha='center', va='bottom')

plt.tight_layout()
plt.show()
```

####SUB#### Spatial Analysis
- Choropleth mapping
- Hot spot identification
- Spatial clustering
- Geographic correlations

####SUB#### District Performance
- Ranking methodologies
- Confidence intervals
- Statistical significance testing

###SUBSTEP### Demographic Analysis

```python
# Age group analysis
age_analysis = df.groupby(['age_group', 'district']).agg({
    'confirmed_cases': 'sum',
    'tested_cases': 'sum'
}).reset_index()

age_analysis['confirmation_rate'] = (age_analysis['confirmed_cases'] / age_analysis['tested_cases']) * 100

# Create pivot table for heatmap
pivot_table = age_analysis.pivot(index='district', columns='age_group', values='confirmation_rate')

plt.figure(figsize=(10, 8))
sns.heatmap(pivot_table, annot=True, cmap='YlOrRd', fmt='.1f')
plt.title('Confirmation Rate by District and Age Group')
plt.tight_layout()
plt.show()
```

####SUB#### Age Stratification
- Under-5 mortality patterns
- Adult vs pediatric cases
- Age-specific incidence rates

####SUB#### Gender Analysis
- Male vs female case patterns
- Pregnancy-related malaria
- Gender-specific risk factors

##STEP_END##

##STEP## Advanced Analytics and Machine Learning

###SUBSTEP### Predictive Modeling

```python
# Machine Learning for Trend Prediction
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_absolute_error, r2_score

def predict_malaria_trends(df):
    """
    Use machine learning to predict malaria trends
    """
    # Feature engineering
    df['month'] = df['date'].dt.month
    df['year'] = df['date'].dt.year
    df['quarter'] = df['date'].dt.quarter
    df['is_rainy_season'] = df['month'].isin([5, 6, 7, 8, 9, 10])
    
    # Prepare features
    feature_cols = ['month', 'year', 'quarter', 'is_rainy_season', 'tested_cases']
    X = df[feature_cols]
    y = df['confirmed_cases']
    
    # Split data
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )
    
    # Train model
    model = RandomForestRegressor(n_estimators=100, random_state=42)
    model.fit(X_train, y_train)
    
    # Make predictions
    y_pred = model.predict(X_test)
    
    # Evaluate model
    mae = mean_absolute_error(y_test, y_pred)
    r2 = r2_score(y_test, y_pred)
    
    print(f"Model Performance:")
    print(f"Mean Absolute Error: {mae:.2f}")
    print(f"R² Score: {r2:.3f}")
    
    # Feature importance
    importance_df = pd.DataFrame({
        'feature': feature_cols,
        'importance': model.feature_importances_
    }).sort_values('importance', ascending=False)
    
    return model, importance_df

# Run prediction model
model, feature_importance = predict_malaria_trends(cleaned_df)
print("Feature Importance:")
print(feature_importance)
```

####SUB#### Model Selection
- Compare different algorithms
- Cross-validation techniques
- Hyperparameter tuning
- Model performance evaluation

####SUB#### Feature Engineering
- Create temporal features
- Seasonal indicators
- Lagged variables
- Interaction terms

###SUBSTEP### Risk Assessment and Classification

```python
# Risk classification based on confirmation rates
def classify_risk_level(confirmation_rate):
    """Classify districts by malaria risk level"""
    if confirmation_rate < 10:
        return 'Low Risk'
    elif confirmation_rate < 25:
        return 'Medium Risk'
    elif confirmation_rate < 40:
        return 'High Risk'
    else:
        return 'Very High Risk'

# Apply risk classification
district_risk = df.groupby('district')['confirmation_rate'].mean().reset_index()
district_risk['risk_level'] = district_risk['confirmation_rate'].apply(classify_risk_level)

print("District Risk Classification:")
print(district_risk.sort_values('confirmation_rate', ascending=False))
```

####SUB#### Risk Stratification
- Define risk thresholds
- Validate classification criteria
- Monitor risk level changes
- Alert system development

####SUB#### Intervention Prioritization
- Resource allocation algorithms
- Cost-effectiveness analysis
- Impact modeling

##STEP_END##

## Key Performance Indicators (KPIs)

| **Indicator** | **Formula** | **Target** | **Interpretation** |
|---------------|-------------|------------|-------------------|
| Overall Confirmation Rate | (Total confirmed / Total tested) × 100 | 10-40% | Higher rates may indicate outbreak |
| Monthly Incidence Rate | Confirmed cases / Population × 1,000 | <5 per 1,000 | Seasonal variation expected |
| Test Positivity Rate | Positive tests / Total tests × 100 | 10-40% | Quality of case management |
| District Performance | District rate vs National average | ±20% | Identifies priority areas |

## Best Practices

1. **Data Validation**: Always validate data quality before analysis
2. **Standardization**: Use consistent naming conventions for districts and facilities
3. **Documentation**: Document all analysis steps and assumptions
4. **Reproducibility**: Ensure analysis can be reproduced with same data
5. **Version Control**: Track changes to analysis code and data

### Quality Assurance Checklist

- [ ] Data completeness check (missing values < 5%)
- [ ] Data consistency validation (confirmed ≤ tested)
- [ ] Outlier detection and handling
- [ ] Statistical significance testing
- [ ] Cross-validation of results
- [ ] Peer review of analysis code

## Contact Information

For technical support or questions about this template:

**Organization**: Informatics Consultancy Firm Sierra Leone (ICF-SL)  
**Motto**: *A local firm with international standards*  
**Email**: support@icf-sl.org  
**Website**: www.icf-sl.org

---

*This template is designed to support the National Malaria Control Programme (NMCP) in Sierra Leone with standardized data analysis procedures.*
