# Universal_Segmentation_pipeline_SAS
# Customer Segmentation Pipeline - Complete Documentation

## 📋 Project Overview

This project implements an **end-to-end customer segmentation pipeline** using SAS Model Studio that transforms raw customer data into meaningful customer segments through a modular, step-by-step process. The pipeline combines data preprocessing, dimensionality reduction, and unsupervised learning to identify distinct customer groups for targeted marketing strategies.

### **Business Objective**
- Identify natural customer segments based on purchasing behavior and characteristics
- Enable personalized marketing campaigns and customer relationship management
- Understand customer value distribution and behavioral patterns

### **Technical Approach**
- **Modular Pipeline Design**: Each processing step is encapsulated in separate nodes
- **Automated Data Quality**: Built-in missing value handling and standardization
- **Dimensionality Reduction**: PCA to handle high-dimensional encoded features
- **Intelligent Clustering**: Automatic K-selection with multiple validation metrics
- **Business Interpretation**: Automated cluster profiling and naming

---

## 🏗️ Architecture & Module Breakdown

### **Pipeline Flow**
```
Raw Data → Data Quality → Encoding → Standardization → PCA → Clustering → Segmentation
```

### **Module 1: Data Quality & High-Cardinality Filtering**
**Purpose**: Remove uninformative identifier columns that hinder analysis

```python
# Key Functionality:
- Identifies columns with >90% unique values (likely IDs)
- Removes these columns to improve model performance
- Preserves meaningful categorical and numeric features
```

**Input**: Raw customer data with potential ID columns  
**Output**: Cleaned dataset without high-cardinality identifiers

### **Module 2: Advanced Missing Value Imputation**
**Purpose**: Intelligently handle missing data using statistical methods

```python
# Key Algorithms:
- Little's MCAR Test: Determines missing data mechanism
- Predictive Imputation: Random Forest-based imputation for MAR data
- Simple Imputation: Median/mode for MCAR data with low missing %
- Mutual Information: Tests relationship between missingness and other variables
```

**Statistical Foundation**:
- **MCAR** (Missing Completely at Random): Simple imputation
- **MAR** (Missing at Random): Predictive imputation
- **MNAR** (Missing Not at Random): Requires domain knowledge

### **Module 3: Smart Feature Encoding**
**Purpose**: Transform categorical and text data into numeric representations

```python
# Encoding Strategies:
• One-Hot Encoding: For low cardinality (≤5 categories)
• Label Encoding: For medium cardinality (6-50 categories)  
• Frequency Encoding: For high cardinality (>50 categories)
• TF-IDF Vectorization: For text columns (description, comments)
```

**Text Processing**:
- Identifies text columns using pattern matching and statistical checks
- Applies TF-IDF with 100 features max, English stop words removal
- Fallback to basic text features if TF-IDF fails

### **Module 4: Feature Standardization**
**Purpose**: Ensure all numeric features are on comparable scales

```python
# Standardization Process:
- Coerces all numeric columns to float type
- Applies StandardScaler (mean=0, std=1)
- Excludes system columns (_dmIndex_, _PartInd_)
- Handles any remaining missing values with zero-fill
```

**Why Standardize?**:
- Prevents features with large scales from dominating PCA
- Improves K-means convergence and performance
- Essential for distance-based algorithms

### **Module 5: Principal Component Analysis (PCA)**
**Purpose**: Reduce dimensionality while preserving information

```python
# PCA Configuration:
- Retains 90% of variance (n_components=0.90)
- Excludes CustomerID and partition indicators
- Creates interpretable component loadings
- Generates visualization for variance explanation
```

**Component Interpretation**:
```python
# Loadings Analysis shows:
- Which original features contribute most to each PC
- Direction of correlation (positive/negative)
- Business meaning of each principal component
```

### **Module 6: Intelligent K-means Clustering**
**Purpose**: Automatically find optimal customer segments

```python
# Automatic K-Selection using 4 methods:
1. Silhouette Score (Primary): Measures cluster separation
2. Elbow Method: Finds inflection point in within-cluster variance  
3. Calinski-Harabasz: Ratio of between to within-cluster variance
4. Davies-Bouldin: Average similarity between clusters
```

**Cluster Quality Assessment**:
- **Strong**: Silhouette > 0.7
- **Reasonable**: Silhouette 0.5-0.7
- **Weak**: Silhouette 0.25-0.5
- **No Structure**: Silhouette < 0.25

---

## 🔄 Data Flow Description

### **Initial Data Input**
- **Source**: Customer transaction data with mixed data types
- **Typical Features**: CustomerID, InvoiceNo, Quantity, UnitPrice, Description, Country, etc.
- **Challenges**: High-dimensionality after encoding, mixed data types, missing values

### **Transformation Journey**
1. **Raw Data** → Remove ID columns → **Clean Data**
2. **Clean Data** → Handle missing values → **Complete Data** 
3. **Complete Data** → Encode categorical/text → **Numeric Data**
4. **Numeric Data** → Standardize features → **Scaled Data**
5. **Scaled Data** → Apply PCA → **Reduced Dimensions**
6. **Reduced Dimensions** → Cluster analysis → **Customer Segments**

### **Output Deliverables**
- **Technical**: PCA loadings, cluster assignments, quality metrics
- **Business**: Segment profiles, descriptive names, size distributions
- **Visual**: Scree plots, cluster visualizations, metric comparisons

---

## 🧮 Detailed Algorithm Explanations

### **Little's MCAR Test**
```python
def littles_mcar_test(df):
    # Tests if missing data is Missing Completely At Random
    # Uses chi-square test on observed data patterns
    # Returns: chi2 statistic, degrees of freedom, p-value
```

**Interpretation**:
- **p > 0.05**: Data is MCAR → Simple imputation appropriate
- **p ≤ 0.05**: Data is not MCAR → Predictive imputation needed

### **PCA Variance Retention**
```python
pca = PCA(n_components=0.90)  # Keep components explaining 90% variance
```

**Why 90%?**:
- Balances dimensionality reduction with information preservation
- Typically captures most meaningful patterns while eliminating noise
- Adjustable based on specific use case requirements

### **K-means with Automatic K-selection**
```python
# Tests K from 2 to min(10, n_samples-1)
for k in range(K_MIN, K_MAX + 1):
    kmeans = KMeans(n_clusters=k)
    # Evaluate multiple metrics for each K
```

**K-selection Logic**:
- **Primary**: Maximize silhouette score
- **Validation**: Check consistency across other metrics
- **Business**: Consider segment interpretability and actionability

---

## ⚙️ Configuration & Parameters

### **Adjustable Parameters**

| Module | Parameter | Default | Description |
|--------|-----------|---------|-------------|
| Data Quality | `high_cardinality_threshold` | 0.9 | Remove cols with >90% unique values |
| Missing Values | `low_missing_threshold` | 1.0 | Use simple imputation for <1% missing |
| Encoding | `max_tfidf_features` | 100 | Maximum features for text vectorization |
| PCA | `variance_threshold` | 0.90 | Retain components explaining 90% variance |
| Clustering | `K_MIN`, `K_MAX` | 2, 10 | Range of cluster numbers to test |

### **System Configuration**
```python
# Excluded columns (never processed):
excluded_cols = ['CustomerID', '_PartInd_']

# Random seeds for reproducibility:
random_state = 42

# Quality thresholds:
mean_tol = 1e-6    # Standardization tolerance for means
std_tol = 1e-1     # Standardization tolerance for std dev
```

---

## 📊 Results Interpretation Guide

### **PCA Results**
```python
# Example Output:
"PC_1 (Variance: 25.4%):"
"   - total_purchase_amount: 0.8743 (positively correlated)"
"   - num_orders: 0.8122 (positively correlated)"
```

**Interpretation**: PC_1 represents "Overall Customer Value/Activity"

### **Clustering Results**
```python
# Cluster Profile Example:
"Cluster 0 (Size: 1,500, 30.0%)"
"Higher than average:"
"   ✅ total_purchase_amount: 450.20 vs 220.50 (+104.2%)"
"   ✅ avg_order_value: 85.30 vs 45.20 (+88.7%)"
"Lower than average:"
"   ❌ days_since_last_purchase: 5.20 vs 45.80 (-88.6%)"
```

**Business Interpretation**: "High-Value Active Customers"

### **Quality Metrics**
- **Silhouette Score**: 0.0-1.0 (higher = better separated clusters)
- **Calinski-Harabasz**: Higher values indicate better defined clusters
- **Davies-Bouldin**: Lower values indicate better separation

---

## 🎯 Business Applications

### **Segment-Based Strategies**
1. **High-Value Customers**: Premium services, loyalty programs
2. **At-Risk Customers**: Reactivation campaigns, special offers  
3. **New Customers**: Onboarding sequences, education content
4. **Price-Sensitive**: Discount promotions, value messaging

### **Actionable Insights**
- **Resource Allocation**: Focus on high-value segments
- **Product Development**: Tailor offerings to segment needs
- **Marketing Efficiency**: Targeted campaigns based on segment characteristics
- **Customer Retention**: Identify at-risk segments for intervention

---

## 🔧 Usage Instructions

### **Pipeline Execution**
1. **Load Data**: Import customer data into SAS Model Studio
2. **Configure Nodes**: Adjust parameters as needed for your data
3. **Run Pipeline**: Execute sequentially from Data Quality to Clustering
4. **Review Results**: Analyze segment profiles and business implications

### **Customization Points**
- **Threshold Adjustment**: Modify cardinality and missing value thresholds
- **Segment Naming**: Customize the automatic naming logic for your business context
- **K-range**: Adjust the range of clusters tested based on expected segment count

### **Monitoring & Validation**
- Check model quality metrics (silhouette scores)
- Validate segment business relevance with domain experts
- Monitor segment stability over time with new data

---

## 📈 Performance Considerations

### **Computational Efficiency**
- **PCA**: Handles high-dimensional data efficiently
- **K-means**: Scales well with large datasets
- **Automatic K-selection**: Tests multiple values but reuses fitted models

### **Scalability**
- Suitable for datasets with thousands to hundreds of thousands of customers
- Memory efficient through incremental processing in nodes
- Parallelizable components for very large datasets

### **Maintenance**
- Regular validation with new data
- Periodic retraining as customer behavior evolves
- Monitoring of segment drift and emergence of new patterns

This modular pipeline provides a robust, interpretable, and actionable framework for customer segmentation that can be adapted to various business contexts and data characteristics.
