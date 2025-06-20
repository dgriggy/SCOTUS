# Statistical Test Decision Logic and Function Reference

## Overview
This document describes the comprehensive statistical analysis workflow implemented in `enhanced_datasort.py`, including decision trees for test selection, input requirements, and functional descriptions.

## Analysis Workflow Structure

### Stage 1: Data Preparation and Initial Assessment
1. **Data Loading and Cleaning**
2. **Variable Type Detection** (continuous vs categorical)
3. **Normality Assessment** (overall and group-specific)
4. **Initial Summary Statistics**

### Stage 2: Multi-Stage Statistical Testing
1. **Stage 1 Initial Tests** - Summary testing across variable types
2. **Overall Analysis** - Treatment vs control comparisons
3. **Subgroup Analysis** - Within-group testing
4. **Three-Sample Analysis** - Multi-group contingency analysis
5. **Between-Groups Analysis** - Variance-based cross-group testing

---

## Test Selection Decision Trees

### Primary Variable Type Classification

```
Input Variable Analysis
├── Continuous (numeric, >10 unique values)
│   ├── Correlation Analysis (Pearson/Spearman)
│   ├── Regression Consistency Testing
│   └── Group Comparison Tests
└── Categorical (≤10 unique values)
    ├── Binary (2 levels) → Chi-square, Fisher's Exact
    ├── Ordinal (3+ ordered levels) → Alexander-Govern, Kruskal-Wallis
    └── Nominal (3+ unordered levels) → Chi-square, Cramér's V
```

### Normality-Based Test Selection

```
Target Variable Normality Assessment
├── Normal Distribution (all normality tests p > 0.05)
│   ├── Parametric Tests
│   │   ├── t-tests (2 groups)
│   │   ├── ANOVA (3+ groups)
│   │   └── Pearson correlation
│   └── Equal Variances Check (Levene's test)
└── Non-Normal Distribution
    ├── Non-Parametric Tests
    │   ├── Mann-Whitney U (2 groups)
    │   ├── Kruskal-Wallis (3+ groups)
    │   └── Spearman correlation
    └── Rank-Based Methods
```

---

## Stage-by-Stage Test Logic

### Stage 1 Initial Tests
**Purpose**: Comprehensive summary testing across all variable types
**Function**: `perform_stage1_initial_tests()`

#### Continuous Variables
- **Correlation Analysis**
  - *Normal data*: Pearson correlation
  - *Non-normal data*: Spearman rank correlation
  - *Inputs*: predictor variable, measurement variable
  - *Output*: correlation coefficient (r/ρ), p-value, effect size

- **Regression Consistency Testing**
  - *Test*: Fisher z-test for correlation differences
  - *Purpose*: Compare correlation strength across binary groups
  - *Inputs*: correlations from treatment/control groups
  - *Output*: z-statistic, p-value for difference

#### Categorical Variables (>2 levels)
- **Alexander-Govern Test**
  - *Purpose*: Robust test for mean differences with unequal variances
  - *When used*: Group-specific normality testing shows mixed normal/non-normal
  - *Inputs*: measurement variable grouped by categories within binary groups
  - *Output*: test statistic, p-value, partial η²

- **Mixed Parametric/Non-Parametric Logic**
  ```
  Group Normality Assessment
  ├── All groups normal → ANOVA
  ├── All groups non-normal → Kruskal-Wallis
  └── Mixed normality → Alexander-Govern
  ```

### Overall Analysis
**Purpose**: Treatment vs control comparisons within categorical levels
**Function**: `perform_overall_analysis()`

#### Two-Sample Comparisons
- **Continuous Predictor**
  - *Treatment Group*: Spearman correlation (predictor vs measurement)
  - *Control Group*: Spearman correlation (predictor vs measurement)
  - *Comparison*: Effect size difference between groups

- **Categorical Predictor**
  - *Within each category*: Treatment vs Control comparison
  - *Test selection*: Based on normality and sample size
    - Normal data + equal variances → t-test
    - Normal data + unequal variances → Welch's t-test
    - Non-normal data → Mann-Whitney U test

### Subgroup Analysis
**Purpose**: Within-group testing across predictor categories
**Function**: `perform_subgroup_analysis()`

#### Within Binary Groups (Marks=0, Marks=1)
- **Continuous Predictor**
  - *Test*: Spearman correlation within each binary group
  - *Purpose*: Assess predictor-measurement relationship stability

- **Categorical Predictor**
  - *Normal data*: One-way ANOVA
  - *Non-normal data*: Kruskal-Wallis test
  - *Purpose*: Test for differences across categories within each binary group

#### Chatterjee's ξ (Xi) Analysis
- **Purpose**: Non-parametric association measure for any variable type
- **Function**: `chatterjee_xi_test()`
- **Advantages**: 
  - No distributional assumptions
  - Captures non-linear relationships
  - Asymmetric dependency measure
- **Outputs**: ξ statistic, p-value, association interpretation
- **Note**: Association strength measures removed (cannot be reliably computed)

### Three-Sample Analysis
**Purpose**: Multi-group contingency and interaction analysis
**Function**: `perform_three_sample_analysis()`

#### Median Tests by Groups
- **Chi-square Tests**
  - *Within Marks=0*: Test measurement variable across predictor categories
  - *Within Marks=1*: Test measurement variable across predictor categories
  - *Combined/Overall*: Test across all groups
  - *Effect Size*: Cramér's V for association strength

#### Multi-Regression Analysis (Continuous Predictors)
- **Model**: `measurement ~ predictor + binary + predictor*binary`
- **Purpose**: Test for interaction effects and main effects
- **Outputs**:
  - R², Adjusted R², F-statistic
  - Coefficient estimates and p-values
  - Model comparison statistics

### Between-Groups Analysis
**Purpose**: Variance-based cross-group comparisons
**Function**: `perform_between_groups_tests()`

#### Variance Equality Testing
- **Levene's Test**: Test for equal variances across groups
- **Decision Logic**:
  ```
  Variance Equality Check
  ├── Equal variances (p > 0.05) → ANOVA
  └── Unequal variances (p ≤ 0.05) → Alexander-Govern Test
  ```

#### Continuous Variable Between-Groups
- **Test**: Mann-Whitney U test (Marks=0 vs Marks=1)
- **Purpose**: Compare measurement variable across binary groups
- **Effect Size**: Rank-biserial correlation (r)

---

## Key Statistical Functions

### Normality Testing
**Function**: `test_normality()`
**Tests Applied**:
1. **Shapiro-Wilk Test** (n ≤ 5000): Most powerful for small-medium samples
2. **D'Agostino's Test** (n ≥ 8): Good for larger samples
3. **Kolmogorov-Smirnov Test** (n ≥ 3): Distribution shape comparison
4. **Skewness Assessment**: Measure of asymmetry

**Decision Rule**: Conservative approach - all tests must indicate normality (p > 0.05)

### Group-Specific Normality
**Function**: `test_target_normality()`
**Purpose**: Assess normality within each categorical group
**Logic**:
```
For categorical variables:
├── Test normality within each category
├── Test normality within each binary group  
├── Overall normality = all groups normal
└── Enable mixed parametric/non-parametric testing
```

### Enhanced Output Formatting
**Function**: `format_statistical_output()`
**Features**:
- **Significance Notation**: *** (p<0.001), ** (p<0.01), * (p<0.05), † (p<0.10)
- **Effect Size Reporting**: Contextual effect size measures (r, d, η², V)
- **Sample Size**: Always reported with test statistics
- **Named Tests**: Clear test identification in output

### Content Validation
**Function**: `has_meaningful_content()`
**Purpose**: Prevent empty section headers in output
**Criteria**:
- Requires actual statistical results (test_type + statistic/p_value)
- Validates nested test structures
- Ensures only substantive content generates headers

---

## Input Requirements and Data Structure

### Required DataFrame Columns
- **Predictor Variable**: Continuous or categorical grouping variable
- **Measurement Variable**: Continuous outcome variable being analyzed
- **Binary Variable**: Treatment/control grouping (typically "Marks" with values 0/1)

### Variable Type Detection Logic
```python
def detect_variable_type(series):
    if pd.api.types.is_numeric_dtype(series) and series.nunique() > 10:
        return 'continuous'
    else:
        return 'categorical'
```

### Missing Data Handling
- **Listwise deletion**: Removes rows with missing values in any analysis variable
- **Sample size reporting**: Always reports effective n after deletion
- **Data shape tracking**: Reports original vs analysis sample sizes

---

## Output Structure

### Summary Report Sections
1. **Stage 1: Comprehensive Summary Statistics**
2. **Detailed Statistical Test Results**
   - Stage 1 Initial Tests
   - Overall Analysis  
   - Subgroup Analysis
   - Three Sample Analysis
   - Between Groups Analysis
3. **Complete Console Output** (all processing steps)
4. **Descriptive Statistics Summary**

### Test Result Dictionary Structure
```python
test_result = {
    'test_type': 'test_name',
    'statistic': float,
    'p_value': float, 
    'test_description': 'formatted_description',
    'effect_size': float,
    'effect_type': 'effect_measure_name',
    'total_n': int,
    'parametric': bool,
    'formatted_output': 'enhanced_formatted_string'
}
```

---

## Error Handling and Edge Cases

### Sample Size Requirements
- **Minimum n=3**: Required for most statistical tests
- **Normality testing**: Different minimum requirements per test
- **Fisher's exact test**: Used for small cell counts in contingency tables

### Data Quality Checks
- **Variance check**: Handles zero-variance groups
- **Perfect correlation**: Handles r=1.0 cases in regression
- **Insufficient data**: Graceful fallback with informative messages

### Robust Test Selection
- **Mixed normality**: Alexander-Govern test for unequal variances
- **Small samples**: Exact tests when appropriate
- **Non-parametric fallbacks**: Spearman when Pearson assumptions violated

---

## Performance Considerations

### Computational Efficiency
- **Vectorized operations**: Uses pandas/numpy optimizations
- **Cached normality tests**: Avoids redundant calculations
- **Parallel test execution**: Independent tests can run concurrently

### Memory Management
- **Incremental results**: Builds result dictionary progressively
- **Selective copying**: Only copies necessary data subsets
- **Garbage collection**: Clears large intermediate objects

---

## Usage Examples

### Basic Analysis Call
```python
results = analyze_enhanced_statistical_relationships(
    data=df,
    predictor_var='salience',  # categorical: 0,1,2
    measurement_vars=['Authentic'], 
    binary_var='Marks',  # 0=control, 1=treatment
    var_type='categorical'
)
```

### Continuous Variable Analysis
```python
results = analyze_enhanced_statistical_relationships(
    data=df,
    predictor_var='Tone',  # continuous measure
    measurement_vars=['Authentic'],
    binary_var='Marks',
    var_type='continuous'
)
```

This comprehensive workflow provides robust statistical analysis across different variable types while maintaining appropriate test selection based on data characteristics and distributional assumptions.