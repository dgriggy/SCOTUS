# Enhanced Statistical Analysis Script - Test Decision Tree Logic

## **Workflow Structure Overview**

The enhanced statistical analysis follows a **five-stage sequential workflow**:

1. **Global Summary Statistics** (no hypothesis testing)
2. **Target Normality Assessment** (preprocessing step)
3. **Two-Sample Tests** (predictor categories vs population)
4. **Within-Groups Tests** (separate analysis per binary group + Epps-Singleton distribution tests)
5. **Between-Groups Tests** (binary group comparisons)

---

## **Stage 1: Global Summary Statistics**

### **Function**: `compute_global_summary_stats()`
- **Purpose**: Generate descriptive statistics without hypothesis testing
- **Output**: Mean, SD, median, quartiles, skewness, n values, missing data counts
- **No Testing**: Purely descriptive - no p-values or significance tests

---

## **Stage 2: Target Normality Assessment**

### **Function**: `test_target_normality()` + `test_group_normality()`
- **Purpose**: Target normality check + group-specific normality testing
- **Global Logic**:
  ```
  if n > 15:
      Full normality battery (Shapiro-Wilk, D'Agostino, Kolmogorov-Smirnov)
  else:
      Shapiro-Wilk only (optimized for small samples)
  ```
- **Group-Specific Logic**: Each group tested independently for normality
- **Decision Rule**: ALL tests must pass (p > 0.05) for normality assumption
- **Test Selection**: Parametric tests used only when ALL relevant groups are normal

---

## **Stage 2b: Initial Summary Tests**

### **Function**: `perform_stage1_initial_tests()`
- **Purpose**: Initial summary testing based on variable type
- **Logic**:
  ```
  Binary Variable (2 categories):
      if target_is_normal:
          → Pairwise t-tests across samples
      else:
          → Mann-Whitney U test
          
  Categorical Variable (>2 categories):
      → Alexander-Govern test grouping by binary variable
      
  Continuous Variable:
      → Independent t-test that regression r/p values are same across treatments
  ```
- **Binary Variable Testing**: Groups by binary variable (Marks) for categorical analysis
- **Continuous Variable Enhancement**: Independent t-test for regression consistency across treatments
- **Effect Sizes**: Computed when p ≤ 0.10

---

## **Stage 3: Two-Sample Tests (Groups vs Control Population)**

### **Categorical Predictors**
For each category, test: Treatment Group (Marks=1) vs Control Group (Marks=0) within that category

```
if target_is_normal:
    → Independent t-test + Cohen's d effect size
else:
    → Wilcoxon signed-rank test (with sample adjustment) + correlation effect size (r)
```

### **Continuous Predictors**
Test correlation within each group separately + Chatterjee Xi analysis:

```
For Treatment Group (Marks=1):
  if target_is_normal:
      → Pearson correlation + r effect size
  else:
      → Spearman correlation + ρ effect size
  → Chatterjee Xi association measure
      
For Control Group (Marks=0):
  if target_is_normal:
      → Pearson correlation + r effect size
  else:
      → Spearman correlation + ρ effect size
  → Chatterjee Xi association measure
```

---

## **Stage 3b: Three-Sample Tests (Contingency Analysis)**

### **Analysis**: Contingency table analysis with Cramér's V or multi-regression for interaction effects
### **Function**: `perform_three_sample_contingency_analysis()`

#### **Categorical Predictors**
```
Create enhanced contingency table: Categories × Marks treatment
→ Include both counts AND mean target values for each cell
→ Chi-square test for independence
→ Cramér's V for association strength
→ Multi-group testing:
   → Categories within Marks=1: Median test + HSD (if p ≤ 0.10)
   → Categories within Marks=0: Median test + HSD (if p ≤ 0.10)  
   → Marks groups within each category: ANOVA/Kruskal + HSD (if p ≤ 0.10)
→ Create category × marks interaction groups
→ Tukey HSD for interaction effects (always run)
→ Pairwise comparisons within Marks groups
→ Marks aggregation analysis
→ Salience measures (categories sorted by Marks)
```

#### **Continuous Predictors**
```
Multi-regression model: measurement ~ var + marks + var:marks
→ Test main effects and interaction
→ R² change analysis
→ Create interaction groups (quartiles × marks)
→ Tukey HSD for interaction effects
```

#### **Key Outputs**
- **Enhanced Contingency Analysis**: 
  - Count table (frequencies) + Mean table (target variable means)
  - Chi-square test, Cramér's V, pairwise differences
  - Detailed cell information (n, mean, std for each category × marks combination)
- **Multi-Group Testing**:
  - **Within-Marks Tests**: Median test for categories within each Marks group
  - **Cross-Marks Tests**: ANOVA/Kruskal-Wallis for Marks groups within each category
  - **HSD Follow-ups**: Tukey HSD when main test p ≤ 0.10
- **Marks Aggregation**: Differences of target variable by Marks treatment  
- **Salience Measures**: All groups sorted by Marks values
- **Interaction HSD**: Tukey HSD for interaction effects (both variable types)
  - **Categorical**: Category × marks combinations (if chi-square significant)
  - **Continuous**: Quartile × marks combinations

#### **Multi-Group Testing Logic (with Group-Specific Normality)**

**Test Set 1: Categories within Marks=1**
```
Extract all categories where Marks=1
→ Test normality for each category group
→ Mood's median test across categories
→ IF p ≤ 0.10: Tukey HSD for pairwise category comparisons
```

**Test Set 2: Categories within Marks=0**
```
Extract all categories where Marks=0
→ Test normality for each category group
→ Mood's median test across categories  
→ IF p ≤ 0.10: Tukey HSD for pairwise category comparisons
```

**Test Set 3: Cross-Marks within Categories**
```
For each category:
  Extract Marks=0 vs Marks=1 groups
  → Test normality for each marks group
  → IF all_groups_normal: ANOVA
  → ELSE: Kruskal-Wallis test
  → IF p ≤ 0.10: Tukey HSD for Marks group comparisons
```

#### **Association Strength Interpretation (Cramér's V)**
- **< 0.1**: Negligible association
- **0.1-0.3**: Weak association  
- **0.3-0.5**: Moderate association
- **> 0.5**: Strong association

---

## **Stage 4: Within-Groups Tests**

### **Analysis**: Separate analysis for each binary group (e.g., Marks=0, Marks=1)
### **Function**: `perform_statistical_test()` called for each subgroup

### **Enhanced**: Epps-Singleton Within-Groups Tests
### **Function**: `perform_epps_singleton_within_groups()` - Enhanced logic based on category count

### **Enhanced**: Chatterjee Xi Within-Groups Analysis (for all variable types)
### **Function**: `perform_chatterjee_xi_within_groups()` - measures association within each Marks group

#### **Enhanced Epps-Singleton Test Logic**
```
If category variable has 2 values:
    → Run Epps-Singleton tests on all groups of treatment and category variable
    → Test all treatment × category combinations
    
If category variable has >2 grouped values:
    → Run Chatterjee Xi analysis on all groups instead
    → More appropriate for multiple group comparisons
```
- **Purpose**: Detect differences in location, scale, and shape between distributions
- **Advantage**: More sensitive than location-only tests (like Mann-Whitney)
- **Effect Size**: √(statistic / total_n)
- **Threshold**: Effect size computed when p ≤ 0.10
- **Enhancement**: Now applied to all treatment × category combinations when appropriate

#### **Binary Predictors (2 categories)**
```
if both_groups_normal:
    → Independent t-test + Cohen's d
elif is_ordered:
    → Point Biserial Correlation + r_pb
else:
    → Wilcoxon signed-rank test (with sample size adjustment) + correlation r
```

#### **Multiple Categories (>2)**
```
if all_groups_normal AND equal_variances:
    → One-way ANOVA + eta-squared (η²)
elif is_ordered:
    → Page's Trend Test + correlation r
else:
    → Kruskal-Wallis test + correlation r
```

#### **Continuous Predictors**
```
if target_is_normal:
    → Pearson correlation + r
else:
    → Spearman correlation + ρ
```

---

## **Stage 5: Between-Groups Tests (Variance-Based + Interaction Effects)**

### **Analysis**: Variance-conditional testing + Chatterjee Xi interaction effects
### **Function**: `perform_variance_based_across_groups_tests()` for categorical predictors
### **New**: `compute_chatterjee_xi_interaction_effect()` for association differences

#### **Decision Logic (Separate Analysis per Marks Group)**
```
For each binary group (Marks=0, Marks=1) separately:

Step 1: Test variance equality (Levene's test) within that binary group
Step 2: Choose appropriate test for that specific group:

if variances_equal (p > 0.05):
    → Anderson-Ksamp test (k-sample Anderson-Darling)
    → Effect Size: Standardized test statistic
    → Tests: Categories within this specific Marks group
    
else (variances_unequal):
    → Alexander-Govern test (robust to heteroscedasticity)
    → Tests: Categories within this specific Marks group
    → Effect Size: Partial η²
```

#### **Separate Marks Analysis Structure (ENHANCED)**
- **Marks=0 Analysis**: Independent variance testing and follow-up test for control group categories
- **Marks=1 Analysis**: Independent variance testing and follow-up test for treatment group categories
- **Individual Test Selection**: Each Marks group can use different tests based on its own variance characteristics
- **Detailed Output**: Sample sizes, test statistics, and interpretations provided separately for each binary group

#### **Anderson-Ksamp Test Structure (ENHANCED)**
- **Purpose**: Compare distributions of categories within a specific Marks group
- **Advantage**: More powerful than multiple pairwise comparisons for k-sample testing within binary groups
- **Test Statistic**: Based on empirical distribution functions
- **Effect Size**: Standardized test statistic measure per binary group

#### **Alexander-Govern Test Structure (ENHANCED)**
- **Purpose**: Robust mean comparisons of categories within a specific Marks group
- **Advantage**: Handles unequal variances within binary groups
- **Effect Size**: Partial η² per binary group
- **Sample-Specific**: Each Marks group analyzed independently

### **Chatterjee Xi Interaction Effects (NEW)**
#### **Analysis**: Difference in association measures between sample and control data
```
Compute Xi for each group:
  → Xi_control = Chatterjee(ξ) for Marks=0 group
  → Xi_sample = Chatterjee(ξ) for Marks=1 group
  
Interaction Effect:
  → Δξ = Xi_sample - Xi_control
  → Effect magnitude: Negligible/Small/Moderate/Large
  → Effect direction: Sample vs Control stronger
```

#### **Interaction Effect Classification**
- **|Δξ| < 0.05**: Negligible interaction
- **0.05 ≤ |Δξ| < 0.15**: Small interaction
- **0.15 ≤ |Δξ| < 0.35**: Moderate interaction  
- **|Δξ| ≥ 0.35**: Large interaction

### **Continuous Predictor Fallback**
For continuous predictors, uses simple binary comparison + Xi interaction:
```
if target_is_normal:
    → Independent t-test + Cohen's d
else:
    → Wilcoxon signed-rank test (with sample adjustment) + correlation r
→ Chatterjee Xi interaction effect analysis
```

---

## **Key Test Selection Functions**

### **Sample Size Adjustment**: `shrink_to_equal_samples()`
- **Purpose**: Enable Wilcoxon signed-rank test for unequal sample sizes
- **Method**: Randomly sample from larger group to match smaller group
- **Fallback**: Mann-Whitney U test only if Wilcoxon fails

### **Effect Size Computation**: `compute_correlation_effect_size()`
- **Wilcoxon**: r = |Z| / √n
- **Mann-Whitney**: r = |Z| / √(n₁ + n₂)
- **Kruskal-Wallis**: r = √(H / (n-1))
- **Page's Trend**: r = |Z| / √n

### **Effect Size Threshold**
- **Computed when**: p-value ≤ 0.10 (twice the alpha level)
- **Rationale**: Include borderline significant effects for completeness

---

## **Page's Trend Test Implementation**

### **When Used**: Ordered categorical predictors with >2 categories
### **Logic**: Tests for monotonic trend across ordered categories
### **Statistic**: L = Σ(i × R_i) where i is category order, R_i is rank sum
### **Effect Size**: r = |Z| / √n

---

## **Test Decision Matrix**

| **Data Type** | **Normality** | **Categories** | **Ordered** | **Test Selection** | **Effect Size** |
|---------------|---------------|----------------|-------------|-------------------|-----------------|
| Categorical | Normal | 2 | Any | Independent t-test | Cohen's d |
| Categorical | Non-normal | 2 | Yes | Point Biserial | r_pb |
| Categorical | Non-normal | 2 | No | Wilcoxon signed-rank | r |
| Categorical | Normal | >2 | Any | One-way ANOVA | η² |
| Categorical | Non-normal | >2 | Yes | Page's Trend Test | r |
| Categorical | Non-normal | >2 | No | Kruskal-Wallis | r |
| Continuous | Normal | N/A | N/A | Pearson correlation | r |
| Continuous | Non-normal | N/A | N/A | Spearman correlation | ρ |

## **Additional Specialized Tests**

| **Analysis Phase** | **Data Type** | **Test** | **Purpose** | **Effect Size** |
|-------------------|---------------|----------|-------------|-----------------|
| Stage 2 | Continuous | Independent t-test | Regression r/p consistency across treatments | t-statistic |
| Within-Groups | Categorical (2 values) | Epps-Singleton 2-sample | Distribution differences on all treatment × category combinations | √(stat/n) |
| Within-Groups | Categorical (>2 values) | Chatterjee Xi | Association analysis on all groups | ξ (0 to 1) |
| Between-Groups | Categorical (Equal Var) | Anderson-Ksamp test | k-sample distribution comparison per Marks group | Standardized statistic |
| Between-Groups | Categorical (Unequal Var) | Alexander-Govern test | Robust mean comparisons per Marks group | Partial η² |
| Between-Groups | All Types | Chatterjee Xi Interaction | Association difference between Marks groups | Δξ (-1 to 1) |

---

## **Normality Testing Hierarchy**

### **Target Variable Assessment (Single Test)**
1. **Large samples (n > 15)**: 
   - Shapiro-Wilk (most reliable)
   - D'Agostino's normality test
   - Kolmogorov-Smirnov test
2. **Small samples (n ≤ 15)**: 
   - Shapiro-Wilk only

### **Decision Rule**: ALL tests must pass (p > 0.05) for normality

---

## **Implementation Notes**

### **Reproducibility**
- Fixed random seed (42) for sample size adjustments
- Consistent test ordering and parameter settings

### **Error Handling**
- Graceful fallbacks for insufficient data
- Clear error messages for diagnostic purposes
- Minimum sample size requirements enforced

### **Output Structure**
- Standardized result dictionaries with test metadata
- Effect sizes included only when appropriate (p ≤ 0.10)
- Sample size information (original and adjusted) reported

---

## **Key Advantages of This Implementation**

1. **Single Normality Decision**: One assessment drives all test selections
2. **Sample Size Optimization**: Tests chosen based on sample size constraints  
3. **Comprehensive Non-Parametric Coverage**: Wilcoxon preferred over Mann-Whitney
4. **Ordered Category Support**: Page's trend test for ordinal data
5. **Effect Size Consistency**: Correlation-based measures for non-parametric tests
6. **Threshold-Based Effect Sizes**: Only computed when statistically meaningful
7. **Sample Adjustment**: Enables paired tests even with unequal sample sizes

This decision tree ensures consistent, statistically appropriate test selection across all analysis phases while maintaining the flexibility to handle diverse data types and structures.