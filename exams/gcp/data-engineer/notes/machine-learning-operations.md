# Machine Learning Operations - GCP Professional Data Engineer

## Overview

Comprehensive ML integration for data engineers covering BigQuery ML, Vertex AI, AutoML, and MLOps practices. This guide focuses on data engineering aspects of ML pipelines, model deployment, and production ML workflows on GCP.

**Key Focus Areas**:
- BigQuery ML for SQL-based machine learning (primary tool for data engineers)
- Vertex AI integration for advanced ML workflows
- Data preparation and feature engineering for ML
- ML pipeline orchestration and automation
- Model serving patterns (batch vs real-time)
- MLOps practices for production systems

**Exam Weight**: ~15-20% of Professional Data Engineer exam questions involve ML integration, particularly BigQuery ML usage and ML pipeline design.

## Key Topics

1. **BigQuery ML** - SQL-based ML (linear/logistic regression, DNN, K-means, time series, AutoML)
2. **Vertex AI for Data Engineers** - AutoML, custom training, pipelines, feature store
3. **Data Preparation for ML** - Feature engineering, preprocessing, transformations
4. **ML Pipeline Integration** - Vertex AI Pipelines, Kubeflow, Dataflow ML
5. **Model Serving** - Vertex AI endpoints, BigQuery predictions, batch vs real-time
6. **MLOps Patterns** - Versioning, monitoring, retraining, A/B testing
7. **Pre-built ML APIs** - Vision, NLP, Translation for data enrichment

## BigQuery ML

### Model Types and Use Cases

BigQuery ML enables data engineers to create and execute ML models using SQL. No Python/TensorFlow knowledge required for basic models.

| Model Type | Use Case | Training Data | Output |
|------------|----------|---------------|--------|
| **LINEAR_REG** | Numeric prediction (revenue, sales) | Historical numeric values | Continuous prediction |
| **LOGISTIC_REG** | Binary/multiclass classification | Labeled categorical data | Class probability |
| **DNN_REGRESSOR** | Complex numeric patterns | Large datasets with non-linear patterns | Continuous prediction |
| **DNN_CLASSIFIER** | Complex classification | Large labeled datasets | Class probability |
| **KMEANS** | Customer segmentation, anomaly detection | Unlabeled numeric features | Cluster assignment |
| **MATRIX_FACTORIZATION** | Product recommendations | User-item interaction matrix | Item scores |
| **ARIMA_PLUS** | Time series forecasting | Historical time series data | Future values + intervals |
| **BOOSTED_TREE_REGRESSOR** | High-accuracy numeric prediction | Tabular data with mixed types | Continuous prediction |
| **BOOSTED_TREE_CLASSIFIER** | High-accuracy classification | Tabular data with mixed types | Class probability |
| **AUTOML_REGRESSOR** | Automatic model selection for regression | Any tabular data | Continuous prediction |
| **AUTOML_CLASSIFIER** | Automatic model selection for classification | Any tabular data | Class probability |
| **PCA** | Dimensionality reduction | High-dimensional features | Principal components |
| **TENSORFLOW** | Custom TensorFlow models | Any data format | Custom output |

### Linear Regression

**Use Case**: Predict continuous numeric values (sales, revenue, prices, demand).

**Complete Example**: Sales forecasting
```sql
-- Step 1: Create training dataset with feature engineering
CREATE OR REPLACE TABLE `project.dataset.sales_training` AS
SELECT
  product_id,
  category,
  -- Temporal features
  EXTRACT(MONTH FROM sale_date) AS month,
  EXTRACT(DAYOFWEEK FROM sale_date) AS day_of_week,
  EXTRACT(QUARTER FROM sale_date) AS quarter,
  -- Lag features
  AVG(daily_sales) OVER (
    PARTITION BY product_id
    ORDER BY sale_date
    ROWS BETWEEN 7 PRECEDING AND 1 PRECEDING
  ) AS avg_sales_last_7_days,
  AVG(daily_sales) OVER (
    PARTITION BY product_id
    ORDER BY sale_date
    ROWS BETWEEN 30 PRECEDING AND 1 PRECEDING
  ) AS avg_sales_last_30_days,
  -- Target variable
  daily_sales,
  -- Split column for train/test
  CASE
    WHEN RAND() < 0.8 THEN 'TRAIN'
    ELSE 'TEST'
  END AS split
FROM `project.dataset.raw_sales`
WHERE sale_date >= '2022-01-01';

-- Step 2: Create linear regression model
CREATE OR REPLACE MODEL `project.dataset.sales_forecast_model`
OPTIONS(
  model_type='LINEAR_REG',
  input_label_cols=['daily_sales'],
  l2_reg=0.1,  -- Ridge regularization
  max_iterations=50,
  learn_rate_strategy='line_search',
  early_stop=TRUE,
  min_rel_progress=0.01,
  data_split_method='SEQ',  -- Sequential split for time series
  data_split_eval_fraction=0.2,
  data_split_col='split'
) AS
SELECT
  category,
  month,
  day_of_week,
  quarter,
  avg_sales_last_7_days,
  avg_sales_last_30_days,
  daily_sales
FROM `project.dataset.sales_training`
WHERE split = 'TRAIN';

-- Step 3: Evaluate model performance
SELECT
  mean_absolute_error,
  mean_squared_error,
  mean_squared_log_error,
  median_absolute_error,
  r2_score,
  explained_variance
FROM ML.EVALUATE(MODEL `project.dataset.sales_forecast_model`,
  (
    SELECT
      category,
      month,
      day_of_week,
      quarter,
      avg_sales_last_7_days,
      avg_sales_last_30_days,
      daily_sales
    FROM `project.dataset.sales_training`
    WHERE split = 'TEST'
  )
);

-- Step 4: Get feature importance
SELECT
  feature,
  importance
FROM ML.FEATURE_IMPORTANCE(MODEL `project.dataset.sales_forecast_model`)
ORDER BY importance DESC;

-- Step 5: Make predictions
SELECT
  product_id,
  category,
  predicted_daily_sales,
  daily_sales AS actual_sales,
  ABS(predicted_daily_sales - daily_sales) AS prediction_error
FROM ML.PREDICT(MODEL `project.dataset.sales_forecast_model`,
  (
    SELECT
      product_id,
      category,
      month,
      day_of_week,
      quarter,
      avg_sales_last_7_days,
      avg_sales_last_30_days,
      daily_sales
    FROM `project.dataset.sales_training`
    WHERE split = 'TEST'
  )
)
ORDER BY prediction_error DESC
LIMIT 100;
```

**Key Options**:
- `l1_reg`: Lasso regularization (feature selection)
- `l2_reg`: Ridge regularization (prevent overfitting)
- `learn_rate_strategy`: 'line_search' or 'constant'
- `early_stop`: Stop when no improvement
- `data_split_method`: 'AUTO_SPLIT', 'SEQ', 'RANDOM', 'NO_SPLIT', 'CUSTOM'

### Logistic Regression

**Use Case**: Binary or multiclass classification (churn, fraud, category prediction).

**Complete Example**: Customer churn prediction
```sql
-- Step 1: Create features with data preparation
CREATE OR REPLACE TABLE `project.dataset.churn_features` AS
SELECT
  customer_id,
  -- Customer demographics
  age,
  income_bracket,
  region,
  -- Behavioral features
  tenure_months,
  monthly_spend,
  support_tickets_count,
  product_count,
  -- Engagement features
  days_since_last_login,
  page_views_last_30_days,
  -- Derived features
  CASE
    WHEN monthly_spend = 0 THEN 'zero_spend'
    WHEN monthly_spend < 50 THEN 'low_spend'
    WHEN monthly_spend < 200 THEN 'medium_spend'
    ELSE 'high_spend'
  END AS spend_category,
  monthly_spend / NULLIF(tenure_months, 0) AS avg_monthly_spend_per_tenure,
  support_tickets_count / NULLIF(tenure_months, 0) AS tickets_per_month,
  -- Label (target variable)
  churned AS label
FROM `project.dataset.customer_data`
WHERE customer_id IS NOT NULL;

-- Step 2: Create logistic regression model
CREATE OR REPLACE MODEL `project.dataset.churn_prediction_model`
OPTIONS(
  model_type='LOGISTIC_REG',
  input_label_cols=['label'],
  auto_class_weights=TRUE,  -- Handle class imbalance
  l2_reg=0.1,
  max_iterations=20,
  enable_global_explain=TRUE,  -- For model explainability
  data_split_method='AUTO_SPLIT',
  data_split_eval_fraction=0.2
) AS
SELECT
  age,
  income_bracket,
  region,
  tenure_months,
  monthly_spend,
  support_tickets_count,
  product_count,
  days_since_last_login,
  page_views_last_30_days,
  spend_category,
  avg_monthly_spend_per_tenure,
  tickets_per_month,
  label
FROM `project.dataset.churn_features`;

-- Step 3: Comprehensive evaluation
SELECT
  precision,
  recall,
  accuracy,
  f1_score,
  log_loss,
  roc_auc
FROM ML.EVALUATE(MODEL `project.dataset.churn_prediction_model`);

-- Step 4: Confusion matrix
SELECT
  *
FROM ML.CONFUSION_MATRIX(MODEL `project.dataset.churn_prediction_model`);

-- Step 5: ROC curve
SELECT
  *
FROM ML.ROC_CURVE(MODEL `project.dataset.churn_prediction_model`);

-- Step 6: Predictions with probability scores
SELECT
  customer_id,
  predicted_label,
  predicted_label_probs[OFFSET(0)].prob AS probability_no_churn,
  predicted_label_probs[OFFSET(1)].prob AS probability_churn,
  -- Custom threshold (0.3 instead of 0.5 for high-risk identification)
  CASE
    WHEN predicted_label_probs[OFFSET(1)].prob >= 0.3 THEN TRUE
    ELSE FALSE
  END AS high_churn_risk
FROM ML.PREDICT(MODEL `project.dataset.churn_prediction_model`,
  (
    SELECT * FROM `project.dataset.churn_features`
    WHERE label IS NULL  -- Score new customers without labels
  )
)
ORDER BY probability_churn DESC;

-- Step 7: Explain predictions for individual customers
SELECT
  customer_id,
  feature,
  attribution
FROM ML.EXPLAIN_PREDICT(
  MODEL `project.dataset.churn_prediction_model`,
  (
    SELECT * FROM `project.dataset.churn_features`
    WHERE customer_id = 'CUST-12345'
  ),
  STRUCT(3 AS top_k_features)
);
```

**Multiclass Classification Example**:
```sql
-- Product category prediction
CREATE OR REPLACE MODEL `project.dataset.category_classifier`
OPTIONS(
  model_type='LOGISTIC_REG',
  input_label_cols=['category'],  -- Multiple classes: electronics, clothing, home, etc.
  auto_class_weights=TRUE,
  max_iterations=30
) AS
SELECT
  product_title,
  product_description,
  price,
  brand,
  category  -- Target: electronics, clothing, home, toys, sports
FROM `project.dataset.product_catalog`;
```

### Deep Neural Networks (DNN)

**Use Case**: Complex non-linear patterns, large datasets, feature interactions.

**Complete Example**: Advanced fraud detection
```sql
-- DNN Classifier for fraud detection with complex patterns
CREATE OR REPLACE MODEL `project.dataset.fraud_detection_dnn`
OPTIONS(
  model_type='DNN_CLASSIFIER',
  input_label_cols=['is_fraud'],
  hidden_units=[128, 64, 32],  -- Three hidden layers
  activation_fn='RELU',
  batch_size=64,
  dropout=0.2,
  optimizer='ADAM',
  learn_rate=0.001,
  max_iterations=50,
  early_stop=TRUE,
  min_rel_progress=0.01,
  auto_class_weights=TRUE
) AS
SELECT
  -- Transaction features
  transaction_amount,
  transaction_hour,
  transaction_day_of_week,
  merchant_category,
  -- Customer features
  customer_age,
  account_age_days,
  customer_credit_score,
  -- Behavioral features
  transactions_last_24h,
  avg_transaction_amount_30d,
  max_transaction_amount_30d,
  unique_merchants_30d,
  -- Geographic features
  distance_from_home_km,
  is_international,
  -- Device features
  device_type,
  is_new_device,
  -- Target
  is_fraud
FROM `project.dataset.transaction_features`
WHERE transaction_date >= '2022-01-01';

-- Evaluate DNN model
SELECT
  precision,
  recall,
  accuracy,
  f1_score,
  roc_auc
FROM ML.EVALUATE(MODEL `project.dataset.fraud_detection_dnn`);

-- Real-time fraud scoring
SELECT
  transaction_id,
  predicted_is_fraud,
  predicted_is_fraud_probs[OFFSET(1)].prob AS fraud_probability,
  CASE
    WHEN predicted_is_fraud_probs[OFFSET(1)].prob >= 0.9 THEN 'BLOCK'
    WHEN predicted_is_fraud_probs[OFFSET(1)].prob >= 0.5 THEN 'REVIEW'
    ELSE 'APPROVE'
  END AS recommendation
FROM ML.PREDICT(MODEL `project.dataset.fraud_detection_dnn`,
  (SELECT * FROM `project.dataset.incoming_transactions`)
);
```

**DNN Regressor Example**:
```sql
-- Complex price prediction with feature interactions
CREATE OR REPLACE MODEL `project.dataset.price_prediction_dnn`
OPTIONS(
  model_type='DNN_REGRESSOR',
  input_label_cols=['price'],
  hidden_units=[256, 128, 64, 32],  -- Four hidden layers for complex patterns
  activation_fn='RELU',
  batch_size=128,
  dropout=0.3,  -- Higher dropout for regularization
  optimizer='ADAM',
  learn_rate=0.0005,
  max_iterations=100
) AS
SELECT
  square_feet,
  bedrooms,
  bathrooms,
  year_built,
  lot_size,
  garage_spaces,
  neighborhood_id,
  school_rating,
  crime_rate,
  walkability_score,
  distance_to_downtown,
  recent_renovations,
  has_pool,
  has_fireplace,
  price
FROM `project.dataset.real_estate_listings`;
```

### K-means Clustering

**Use Case**: Customer segmentation, anomaly detection, grouping similar items.

**Complete Example**: Customer segmentation
```sql
-- Step 1: Prepare features for clustering
CREATE OR REPLACE TABLE `project.dataset.customer_clustering_features` AS
SELECT
  customer_id,
  -- Normalize features manually or let BigQuery ML auto-normalize
  total_purchases,
  avg_purchase_value,
  purchase_frequency,
  days_since_last_purchase,
  product_category_diversity,
  discount_usage_rate,
  email_open_rate,
  support_interactions
FROM `project.dataset.customer_metrics`
WHERE total_purchases > 0;

-- Step 2: Create K-means model
CREATE OR REPLACE MODEL `project.dataset.customer_segments`
OPTIONS(
  model_type='KMEANS',
  num_clusters=5,  -- Or use HPARAM_TUNING for automatic selection
  kmeans_init_method='KMEANS++',  -- Better initialization
  standardize_features=TRUE,  -- Auto-normalize features
  max_iterations=50,
  early_stop=TRUE,
  min_rel_progress=0.01
) AS
SELECT
  total_purchases,
  avg_purchase_value,
  purchase_frequency,
  days_since_last_purchase,
  product_category_diversity,
  discount_usage_rate,
  email_open_rate,
  support_interactions
FROM `project.dataset.customer_clustering_features`;

-- Step 3: Evaluate clustering quality
SELECT
  davies_bouldin_index,  -- Lower is better (cluster separation)
  mean_squared_distance  -- Lower is better (within-cluster variance)
FROM ML.EVALUATE(MODEL `project.dataset.customer_segments`);

-- Step 4: Assign customers to clusters
SELECT
  customer_id,
  CENTROID_ID AS cluster_id,
  NEAREST_CENTROIDS_DISTANCE[OFFSET(0)].DISTANCE AS distance_to_centroid
FROM ML.PREDICT(MODEL `project.dataset.customer_segments`,
  (SELECT * FROM `project.dataset.customer_clustering_features`)
);

-- Step 5: Analyze cluster characteristics
SELECT
  CENTROID_ID AS cluster_id,
  feature,
  numerical_value AS avg_value
FROM ML.CENTROIDS(MODEL `project.dataset.customer_segments`)
ORDER BY CENTROID_ID, feature;

-- Step 6: Business interpretation of clusters
WITH cluster_assignments AS (
  SELECT
    cf.customer_id,
    cf.total_purchases,
    cf.avg_purchase_value,
    cf.purchase_frequency,
    p.CENTROID_ID AS cluster_id
  FROM `project.dataset.customer_clustering_features` cf
  JOIN ML.PREDICT(MODEL `project.dataset.customer_segments`,
    (SELECT * FROM `project.dataset.customer_clustering_features`)
  ) p ON cf.customer_id = p.customer_id
)
SELECT
  cluster_id,
  COUNT(*) AS customer_count,
  AVG(total_purchases) AS avg_purchases,
  AVG(avg_purchase_value) AS avg_value,
  AVG(purchase_frequency) AS avg_frequency,
  -- Label clusters based on characteristics
  CASE
    WHEN AVG(avg_purchase_value) > 500 THEN 'High-Value'
    WHEN AVG(purchase_frequency) > 10 THEN 'Frequent-Buyers'
    WHEN AVG(total_purchases) < 3 THEN 'New-Customers'
    ELSE 'Regular-Customers'
  END AS segment_label
FROM cluster_assignments
GROUP BY cluster_id
ORDER BY customer_count DESC;
```

**Automatic Cluster Selection**:
```sql
-- Let BigQuery ML find optimal number of clusters
CREATE OR REPLACE MODEL `project.dataset.customer_segments_auto`
OPTIONS(
  model_type='KMEANS',
  num_clusters=HPARAM_RANGE(3, 10),  -- Test 3 to 10 clusters
  kmeans_init_method='KMEANS++',
  standardize_features=TRUE
) AS
SELECT * FROM `project.dataset.customer_clustering_features`;
```

### Time Series Forecasting (ARIMA_PLUS)

**Use Case**: Sales forecasting, demand prediction, trend analysis.

**Complete Example**: Multi-product demand forecasting
```sql
-- Step 1: Prepare time series data
CREATE OR REPLACE TABLE `project.dataset.product_time_series` AS
SELECT
  DATE(order_timestamp) AS date,
  product_id,
  SUM(quantity) AS daily_demand,
  AVG(unit_price) AS avg_price
FROM `project.dataset.orders`
WHERE order_timestamp >= '2022-01-01'
GROUP BY date, product_id
ORDER BY product_id, date;

-- Step 2: Create ARIMA_PLUS model
CREATE OR REPLACE MODEL `project.dataset.demand_forecast_model`
OPTIONS(
  model_type='ARIMA_PLUS',
  time_series_timestamp_col='date',
  time_series_data_col='daily_demand',
  time_series_id_col='product_id',  -- Separate model per product
  holiday_region='US',  -- Account for US holidays
  auto_arima=TRUE,  -- Automatic parameter selection
  data_frequency='DAILY',
  adjust_step_changes=TRUE,  -- Detect level shifts
  enable_auto_arima_seasonality=TRUE
) AS
SELECT
  date,
  product_id,
  daily_demand
FROM `project.dataset.product_time_series`;

-- Step 3: Evaluate forecast accuracy
SELECT
  product_id,
  mean_absolute_error,
  mean_squared_error,
  symmetric_mean_absolute_percentage_error AS smape,
  root_mean_squared_error AS rmse
FROM ML.EVALUATE(MODEL `project.dataset.demand_forecast_model`);

-- Step 4: Generate forecasts for next 30 days
SELECT
  forecast_timestamp AS date,
  time_series_id AS product_id,
  forecast_value AS predicted_demand,
  prediction_interval_lower_bound AS lower_bound,
  prediction_interval_upper_bound AS upper_bound,
  confidence_level
FROM ML.FORECAST(MODEL `project.dataset.demand_forecast_model`,
  STRUCT(30 AS horizon,  -- Forecast 30 days ahead
         0.95 AS confidence_level)  -- 95% confidence intervals
)
ORDER BY product_id, date;

-- Step 5: Detect anomalies in historical data
SELECT
  history_timestamp AS date,
  time_series_id AS product_id,
  history_value AS actual_demand,
  is_anomaly,
  anomaly_probability,
  lower_bound,
  upper_bound
FROM ML.DETECT_ANOMALIES(MODEL `project.dataset.demand_forecast_model`,
  STRUCT(0.95 AS anomaly_prob_threshold)
)
WHERE is_anomaly = TRUE
ORDER BY product_id, date;

-- Step 6: Explain time series components
SELECT
  time_series_id AS product_id,
  time_series_type,
  trend,
  seasonal_periods,
  has_drift,
  has_holiday_effect
FROM ML.ARIMA_EVALUATE(MODEL `project.dataset.demand_forecast_model`);
```

**Single Time Series Example** (no time_series_id_col):
```sql
-- Overall company revenue forecasting
CREATE OR REPLACE MODEL `project.dataset.revenue_forecast`
OPTIONS(
  model_type='ARIMA_PLUS',
  time_series_timestamp_col='month',
  time_series_data_col='revenue',
  data_frequency='MONTHLY',
  holiday_region='US'
) AS
SELECT
  DATE_TRUNC(order_date, MONTH) AS month,
  SUM(order_total) AS revenue
FROM `project.dataset.orders`
GROUP BY month
ORDER BY month;

-- Forecast next 12 months
SELECT
  forecast_timestamp AS month,
  forecast_value AS predicted_revenue,
  prediction_interval_lower_bound,
  prediction_interval_upper_bound
FROM ML.FORECAST(MODEL `project.dataset.revenue_forecast`,
  STRUCT(12 AS horizon)
);
```

### AutoML Tables

**Use Case**: Automatic model selection and hyperparameter tuning for best performance.

**Complete Example**: Customer lifetime value prediction
```sql
-- AutoML automatically selects best algorithm and hyperparameters
CREATE OR REPLACE MODEL `project.dataset.clv_automl_model`
OPTIONS(
  model_type='AUTOML_REGRESSOR',
  input_label_cols=['customer_lifetime_value'],
  optimization_objective='MINIMIZE_MAE',  -- Or MINIMIZE_RMSE
  budget_hours=2,  -- Training time budget
  data_split_method='AUTO_SPLIT',
  data_split_eval_fraction=0.2
) AS
SELECT
  -- Customer demographics
  age,
  gender,
  location,
  income_level,
  -- Purchase behavior
  first_purchase_date,
  total_orders,
  avg_order_value,
  total_spend,
  purchase_frequency,
  product_categories_purchased,
  -- Engagement metrics
  email_open_rate,
  email_click_rate,
  app_usage_days,
  referrals_made,
  -- Target variable
  customer_lifetime_value
FROM `project.dataset.customer_features`
WHERE customer_lifetime_value IS NOT NULL;

-- AutoML for classification
CREATE OR REPLACE MODEL `project.dataset.churn_automl_model`
OPTIONS(
  model_type='AUTOML_CLASSIFIER',
  input_label_cols=['churned'],
  optimization_objective='MAXIMIZE_AU_ROC',  -- Or MAXIMIZE_AU_PRC, MAXIMIZE_PRECISION, MAXIMIZE_RECALL
  budget_hours=3,
  auto_class_weights=TRUE
) AS
SELECT * FROM `project.dataset.customer_churn_features`;

-- View AutoML trial results
SELECT
  trial_id,
  training_loss,
  evaluation_loss,
  duration_ms
FROM ML.TRIAL_INFO(MODEL `project.dataset.clv_automl_model`)
ORDER BY evaluation_loss ASC
LIMIT 10;
```

### Feature Engineering in BigQuery ML

**Complete Feature Engineering Example**:
```sql
-- Comprehensive feature engineering for prediction model
CREATE OR REPLACE TABLE `project.dataset.engineered_features` AS
SELECT
  customer_id,
  order_date,

  -- 1. TEMPORAL FEATURES
  EXTRACT(YEAR FROM order_date) AS year,
  EXTRACT(MONTH FROM order_date) AS month,
  EXTRACT(QUARTER FROM order_date) AS quarter,
  EXTRACT(DAYOFWEEK FROM order_date) AS day_of_week,
  EXTRACT(DAYOFYEAR FROM order_date) AS day_of_year,
  EXTRACT(WEEK FROM order_date) AS week_of_year,
  CASE
    WHEN EXTRACT(DAYOFWEEK FROM order_date) IN (1, 7) THEN TRUE
    ELSE FALSE
  END AS is_weekend,

  -- 2. CATEGORICAL ENCODING (BigQuery ML auto one-hot encodes STRING types)
  product_category,  -- Will be one-hot encoded
  customer_segment,  -- Will be one-hot encoded

  -- 3. BUCKETIZING (continuous to categorical)
  CASE
    WHEN age < 25 THEN '18-24'
    WHEN age < 35 THEN '25-34'
    WHEN age < 45 THEN '35-44'
    WHEN age < 55 THEN '45-54'
    ELSE '55+'
  END AS age_bucket,

  -- 4. FEATURE CROSSES (interaction features)
  ML.FEATURE_CROSS(STRUCT(product_category, customer_segment)) AS category_segment_cross,
  ML.FEATURE_CROSS(STRUCT(age_bucket, income_bracket)) AS age_income_cross,

  -- 5. POLYNOMIAL FEATURES
  order_value AS order_value_1,
  POW(order_value, 2) AS order_value_squared,
  POW(order_value, 3) AS order_value_cubed,

  -- 6. MATHEMATICAL TRANSFORMATIONS
  LOG(order_value + 1) AS log_order_value,  -- +1 to handle zeros
  SQRT(order_value) AS sqrt_order_value,

  -- 7. AGGREGATED FEATURES (window functions)
  AVG(order_value) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) AS avg_order_value_last_7_orders,

  SUM(order_value) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS cumulative_spend,

  COUNT(*) OVER (
    PARTITION BY customer_id
    ORDER BY order_date
    ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
  ) AS orders_last_30_days,

  -- 8. LAG FEATURES (previous values)
  LAG(order_value, 1) OVER (
    PARTITION BY customer_id ORDER BY order_date
  ) AS previous_order_value,

  LAG(order_date, 1) OVER (
    PARTITION BY customer_id ORDER BY order_date
  ) AS previous_order_date,

  -- 9. TIME SINCE FEATURES
  DATE_DIFF(
    order_date,
    LAG(order_date, 1) OVER (PARTITION BY customer_id ORDER BY order_date),
    DAY
  ) AS days_since_last_order,

  -- 10. RATIO FEATURES
  order_value / NULLIF(
    AVG(order_value) OVER (PARTITION BY customer_id),
    0
  ) AS order_value_vs_customer_avg,

  -- 11. TEXT FEATURES (for text columns)
  ML.NGRAMS(SPLIT(product_description, ' '), [1, 2], ' ') AS product_ngrams,

  -- 12. NORMALIZE/STANDARDIZE (BigQuery ML does this automatically, but manual option)
  (order_value - AVG(order_value) OVER ()) /
    NULLIF(STDDEV(order_value) OVER (), 0) AS order_value_standardized,

  -- Target variable
  will_purchase_next_30_days AS label

FROM `project.dataset.raw_transactions`;

-- Using ML.BUCKETIZE for automated bucketizing
SELECT
  customer_id,
  ML.BUCKETIZE(age, [25, 35, 45, 55]) AS age_bucket,
  ML.BUCKETIZE(income, [30000, 50000, 75000, 100000]) AS income_bucket,
  ML.QUANTILE_BUCKETIZE(purchase_frequency, 4) AS frequency_quartile
FROM `project.dataset.customers`;

-- Using ML.MIN_MAX_SCALER for normalization
SELECT
  customer_id,
  ML.MIN_MAX_SCALER(order_value) OVER() AS order_value_normalized,
  ML.STANDARD_SCALER(order_value) OVER() AS order_value_standardized
FROM `project.dataset.orders`;
```

**Feature Transform Functions**:
```sql
-- ML.FEATURE_CROSS: Create interaction features
SELECT
  ML.FEATURE_CROSS(STRUCT(state, product_category)) AS state_category,
  ML.FEATURE_CROSS(STRUCT(age_group, gender, income_bracket)) AS demographic_cross
FROM `project.dataset.customers`;

-- ML.POLYNOMIAL_EXPAND: Create polynomial features
SELECT
  customer_id,
  ML.POLYNOMIAL_EXPAND(STRUCT(feature1, feature2, feature3), 2) AS poly_features
FROM `project.dataset.customer_features`;

-- ML.NGRAMS: Create n-grams from text
SELECT
  ML.NGRAMS(SPLIT(review_text, ' '), [1, 2, 3], ' ') AS text_features
FROM `project.dataset.product_reviews`;

-- ML.QUANTILE_BUCKETIZE: Create equal-sized buckets
SELECT
  ML.QUANTILE_BUCKETIZE(revenue, 10) AS revenue_decile  -- 10 equal buckets
FROM `project.dataset.customers`;
```

### Model Export and Import

**Export models** for deployment outside BigQuery:
```sql
-- Export to Cloud Storage
EXPORT MODEL `project.dataset.churn_prediction_model`
OPTIONS(
  uri='gs://my-bucket/models/churn_model/*'
);

-- Export as TensorFlow SavedModel format
EXPORT MODEL `project.dataset.dnn_model`
OPTIONS(
  uri='gs://my-bucket/models/dnn_tf_model/*'
);
```

**Import models** into BigQuery:
```sql
-- Import TensorFlow model
CREATE OR REPLACE MODEL `project.dataset.imported_tf_model`
OPTIONS(
  model_type='TENSORFLOW',
  model_path='gs://my-bucket/tensorflow_saved_model/*'
);

-- Import ONNX model
CREATE OR REPLACE MODEL `project.dataset.imported_onnx_model`
OPTIONS(
  model_type='ONNX',
  model_path='gs://my-bucket/onnx_model/model.onnx'
);
```

### BigQuery ML Best Practices

1. **Data Splitting**:
   - Use `data_split_method='AUTO_SPLIT'` for random 80/20 split
   - Use `data_split_method='SEQ'` for time series data
   - Use `data_split_col` for custom train/test splits

2. **Feature Engineering**:
   - Let BigQuery ML auto one-hot encode STRING columns
   - Use `ML.FEATURE_CROSS` for interaction features
   - Create temporal features (year, month, day of week)
   - Use window functions for aggregated features

3. **Class Imbalance**:
   - Set `auto_class_weights=TRUE` for classification
   - Manually set `class_weight` for custom balancing
   - Use precision/recall metrics instead of accuracy

4. **Model Evaluation**:
   - Use `ML.EVALUATE` on held-out test set
   - Check multiple metrics (precision, recall, F1, AUC)
   - Use `ML.ROC_CURVE` and `ML.CONFUSION_MATRIX` for classification
   - Use `ML.FEATURE_IMPORTANCE` to understand model

5. **Hyperparameter Tuning**:
   - Use `HPARAM_RANGE` and `HPARAM_CANDIDATES` for tuning
   - Check `ML.TRIAL_INFO` for trial results
   - Balance budget_hours vs accuracy needs

6. **Model Versioning**:
   - Use dataset naming conventions: `model_v1`, `model_v2`
   - Include date in model names: `model_20240115`
   - Use separate datasets for dev/staging/prod models

7. **Performance Optimization**:
   - Filter data appropriately before training
   - Use `WHERE` clauses to exclude nulls
   - Partition training data for faster queries
   - Set reasonable `max_iterations` to avoid long training

8. **Cost Control**:
   - BigQuery ML charges for bytes processed in SELECT query
   - Use clustered/partitioned tables for training data
   - Limit AutoML `budget_hours` to control costs
   - Export models for repeated predictions outside BigQuery

## Vertex AI for Data Engineers

### Vertex AI Overview

Vertex AI is GCP's unified ML platform for training, deploying, and managing ML models. Data engineers use it for advanced ML workflows beyond BigQuery ML capabilities.

**When to use Vertex AI vs BigQuery ML**:
- **BigQuery ML**: SQL-based, simple models, data already in BigQuery, quick prototyping
- **Vertex AI**: Complex models, custom algorithms, need for production deployment infrastructure, PyTorch/TensorFlow models

### Vertex AI AutoML

AutoML provides automatic model selection, feature engineering, and hyperparameter tuning without code.

**Complete Example**: AutoML Tables for structured data
```python
from google.cloud import aiplatform

# Initialize Vertex AI
aiplatform.init(
    project='my-project',
    location='us-central1',
    staging_bucket='gs://my-bucket'
)

# Create dataset from BigQuery
dataset = aiplatform.TabularDataset.create(
    display_name='customer_churn_dataset',
    bq_source='bq://my-project.my_dataset.customer_features',
    labels={'env': 'prod'}
)

# Create AutoML training job
job = aiplatform.AutoMLTabularTrainingJob(
    display_name='churn_automl_job',
    optimization_prediction_type='classification',
    optimization_objective='maximize-au-roc',
    column_specs={
        'customer_id': 'auto',  # Exclude from training
        'churned': 'target',
        'age': 'numeric',
        'income': 'numeric',
        'tenure_months': 'numeric',
        'region': 'categorical'
    }
)

# Train model
model = job.run(
    dataset=dataset,
    target_column='churned',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    budget_milli_node_hours=3000,  # 3 hours
    model_display_name='churn_automl_model',
    disable_early_stopping=False
)

# Deploy model to endpoint
endpoint = model.deploy(
    deployed_model_display_name='churn_model_v1',
    machine_type='n1-standard-4',
    min_replica_count=1,
    max_replica_count=5,
    traffic_split={'0': 100}
)

# Make predictions
predictions = endpoint.predict(instances=[
    {
        'age': 35,
        'income': 75000,
        'tenure_months': 24,
        'region': 'west'
    }
])

print(predictions)
```

**AutoML for Images**:
```python
# Create image dataset
image_dataset = aiplatform.ImageDataset.create(
    display_name='product_classification_dataset',
    gcs_source='gs://my-bucket/images_metadata.csv',
    import_schema_uri=aiplatform.schema.dataset.ioformat.image.single_label_classification
)

# Train AutoML Vision model
image_job = aiplatform.AutoMLImageTrainingJob(
    display_name='product_classifier',
    prediction_type='classification',
    model_type='CLOUD',  # or 'MOBILE_TF_LOW_LATENCY_1'
    base_model=None
)

image_model = image_job.run(
    dataset=image_dataset,
    model_display_name='product_classifier_v1',
    training_fraction_split=0.8,
    budget_milli_node_hours=8000
)
```

### Custom Training with Vertex AI

**Complete Example**: Custom TensorFlow training
```python
from google.cloud import aiplatform
from google.cloud import bigquery

# Step 1: Prepare training script (trainer/task.py)
training_script_content = '''
import tensorflow as tf
from google.cloud import bigquery
import pandas as pd
import argparse
import os

def create_model(input_dim):
    model = tf.keras.Sequential([
        tf.keras.layers.Dense(128, activation='relu', input_shape=(input_dim,)),
        tf.keras.layers.Dropout(0.3),
        tf.keras.layers.Dense(64, activation='relu'),
        tf.keras.layers.Dropout(0.2),
        tf.keras.layers.Dense(32, activation='relu'),
        tf.keras.layers.Dense(1, activation='sigmoid')
    ])
    model.compile(
        optimizer='adam',
        loss='binary_crossentropy',
        metrics=['accuracy', 'AUC']
    )
    return model

def load_data_from_bigquery(project, dataset, table):
    client = bigquery.Client(project=project)
    query = f"""
        SELECT * FROM `{project}.{dataset}.{table}`
    """
    df = client.query(query).to_dataframe()
    return df

def main(args):
    # Load data
    df = load_data_from_bigquery(args.project, args.dataset, args.table)

    # Prepare features
    X = df.drop(['label', 'customer_id'], axis=1)
    y = df['label']

    # Split data
    from sklearn.model_selection import train_test_split
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.2, random_state=42
    )

    # Create and train model
    model = create_model(X_train.shape[1])

    model.fit(
        X_train, y_train,
        epochs=args.epochs,
        batch_size=args.batch_size,
        validation_data=(X_test, y_test),
        callbacks=[
            tf.keras.callbacks.EarlyStopping(patience=5),
            tf.keras.callbacks.ModelCheckpoint(
                os.path.join(args.model_dir, 'model'),
                save_best_only=True
            )
        ]
    )

    # Export model
    model.save(os.path.join(args.model_dir, 'final_model'))

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--project', type=str, required=True)
    parser.add_argument('--dataset', type=str, required=True)
    parser.add_argument('--table', type=str, required=True)
    parser.add_argument('--epochs', type=int, default=50)
    parser.add_argument('--batch_size', type=int, default=64)
    parser.add_argument('--model_dir', type=str, required=True)
    args = parser.parse_args()
    main(args)
'''

# Step 2: Create and run custom training job
job = aiplatform.CustomTrainingJob(
    display_name='custom_churn_training',
    script_path='trainer/task.py',
    container_uri='us-docker.pkg.dev/vertex-ai/training/tf-cpu.2-12:latest',
    requirements=['google-cloud-bigquery', 'pandas', 'scikit-learn'],
    model_serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/tf2-cpu.2-12:latest'
)

model = job.run(
    model_display_name='custom_churn_model',
    args=[
        '--project', 'my-project',
        '--dataset', 'my_dataset',
        '--table', 'customer_features',
        '--epochs', '50',
        '--batch_size', '64'
    ],
    replica_count=1,
    machine_type='n1-standard-8',
    accelerator_type='NVIDIA_TESLA_T4',
    accelerator_count=1,
    base_output_dir='gs://my-bucket/training_output'
)
```

**Hyperparameter Tuning**:
```python
from google.cloud.aiplatform import hyperparameter_tuning as hpt

# Define hyperparameter tuning job
job = aiplatform.CustomTrainingJob(
    display_name='churn_hptuning',
    script_path='trainer/task.py',
    container_uri='us-docker.pkg.dev/vertex-ai/training/tf-cpu.2-12:latest'
)

# Define hyperparameter search space
hpt_job = job.run(
    model_display_name='churn_model_tuned',
    args=['--project', 'my-project', '--dataset', 'my_dataset', '--table', 'features'],
    replica_count=1,
    machine_type='n1-standard-4',
    hyperparameter_tuning_job_spec=hpt.HyperparameterTuningJobSpec(
        metric_spec={'auc': 'maximize'},
        parameter_spec={
            'learning_rate': hpt.DoubleParameterSpec(min=0.0001, max=0.1, scale='log'),
            'batch_size': hpt.DiscreteParameterSpec(values=[32, 64, 128], scale='linear'),
            'hidden_units': hpt.DiscreteParameterSpec(values=[64, 128, 256], scale='linear')
        },
        max_trial_count=20,
        parallel_trial_count=4
    )
)
```

### Vertex AI Feature Store

Feature Store provides centralized feature management, serving, and monitoring for ML.

**Complete Example**: Feature store setup
```python
from google.cloud import aiplatform

# Create feature store
feature_store = aiplatform.Featurestore.create(
    featurestore_id='customer_features_store',
    online_store_fixed_node_count=1,  # For online serving
    labels={'team': 'data-engineering'}
)

# Create entity type (e.g., customer)
customer_entity = feature_store.create_entity_type(
    entity_type_id='customer',
    description='Customer features'
)

# Create features
customer_entity.create_feature(
    feature_id='total_purchases',
    value_type='INT64',
    description='Total number of purchases'
)

customer_entity.create_feature(
    feature_id='avg_order_value',
    value_type='DOUBLE',
    description='Average order value'
)

customer_entity.create_feature(
    feature_id='last_purchase_date',
    value_type='STRING',
    description='Date of last purchase'
)

# Ingest features from BigQuery
customer_entity.ingest_from_bq(
    feature_ids=['total_purchases', 'avg_order_value', 'last_purchase_date'],
    feature_time='event_timestamp',
    bq_source_uri='bq://my-project.my_dataset.customer_features',
    entity_id_field='customer_id',
    worker_count=5
)

# Read features for training (batch)
training_data = customer_entity.batch_serve_to_bq(
    bq_destination_output_uri='bq://my-project.my_dataset.training_features',
    serving_feature_ids=['total_purchases', 'avg_order_value', 'last_purchase_date'],
    read_instances_uri='bq://my-project.my_dataset.customer_ids'
)

# Read features for online prediction
features = customer_entity.read(
    entity_ids=['CUST-12345']
)
```

### Model Deployment and Serving

**Deploy model to endpoint**:
```python
# Create endpoint
endpoint = aiplatform.Endpoint.create(
    display_name='churn_prediction_endpoint',
    description='Production churn prediction service'
)

# Deploy model
deployed_model = model.deploy(
    endpoint=endpoint,
    deployed_model_display_name='churn_v1',
    machine_type='n1-standard-4',
    min_replica_count=2,  # For high availability
    max_replica_count=10,  # Auto-scaling
    traffic_percentage=100,
    accelerator_type='NVIDIA_TESLA_T4',  # Optional GPU
    accelerator_count=1,
    service_account='model-serving@my-project.iam.gserviceaccount.com',
    enable_access_logging=True
)

# Make online predictions
instances = [
    {'age': 35, 'income': 75000, 'tenure_months': 24},
    {'age': 42, 'income': 95000, 'tenure_months': 48}
]

predictions = endpoint.predict(instances=instances)
print(predictions.predictions)

# Update traffic split for A/B testing
endpoint.update(
    traffic_split={
        'deployed_model_v1': 80,  # 80% traffic to v1
        'deployed_model_v2': 20   # 20% traffic to v2
    }
)
```

**Batch predictions**:
```python
# Create batch prediction job
batch_prediction_job = model.batch_predict(
    job_display_name='daily_churn_scoring',
    gcs_source='gs://my-bucket/customers_to_score.csv',
    gcs_destination_prefix='gs://my-bucket/predictions/',
    instances_format='csv',
    predictions_format='jsonl',
    machine_type='n1-standard-4',
    starting_replica_count=5,
    max_replica_count=10
)

# Or predict from BigQuery
batch_prediction_job = model.batch_predict(
    job_display_name='bq_churn_scoring',
    bigquery_source='bq://my-project.my_dataset.customers',
    bigquery_destination_prefix='bq://my-project.predictions_dataset',
    instances_format='bigquery',
    predictions_format='bigquery'
)

# Wait for completion
batch_prediction_job.wait()
print(f"Job state: {batch_prediction_job.state}")
```

### Pre-trained APIs for Data Enrichment

**Vision API** for image analysis in data pipelines:
```python
from google.cloud import vision
import pandas as pd

def analyze_product_images(image_uris):
    client = vision.ImageAnnotatorClient()
    results = []

    for uri in image_uris:
        image = vision.Image()
        image.source.image_uri = uri

        # Detect labels
        response = client.label_detection(image=image)
        labels = [label.description for label in response.label_annotations]

        # Detect objects
        objects_response = client.object_localization(image=image)
        objects = [obj.name for obj in objects_response.localized_object_annotations]

        # Detect text (OCR)
        text_response = client.text_detection(image=image)
        text = text_response.text_annotations[0].description if text_response.text_annotations else ''

        results.append({
            'image_uri': uri,
            'labels': labels,
            'objects': objects,
            'text': text
        })

    return pd.DataFrame(results)

# Use in data pipeline
image_data = analyze_product_images(['gs://my-bucket/product1.jpg', 'gs://my-bucket/product2.jpg'])
```

**Natural Language API** for text analysis:
```python
from google.cloud import language_v1

def analyze_customer_reviews(reviews):
    client = language_v1.LanguageServiceClient()
    results = []

    for review in reviews:
        document = language_v1.Document(
            content=review,
            type_=language_v1.Document.Type.PLAIN_TEXT
        )

        # Sentiment analysis
        sentiment = client.analyze_sentiment(request={'document': document}).document_sentiment

        # Entity extraction
        entities = client.analyze_entities(request={'document': document}).entities
        entity_list = [(e.name, e.type_.name) for e in entities]

        # Content classification
        categories = client.classify_text(request={'document': document}).categories
        category_list = [c.name for c in categories]

        results.append({
            'review': review,
            'sentiment_score': sentiment.score,
            'sentiment_magnitude': sentiment.magnitude,
            'entities': entity_list,
            'categories': category_list
        })

    return pd.DataFrame(results)

# Integrate with BigQuery
from google.cloud import bigquery

def enrich_reviews_in_bigquery(project, dataset):
    bq_client = bigquery.Client(project=project)

    # Read reviews
    query = f"SELECT review_id, review_text FROM `{project}.{dataset}.reviews` WHERE sentiment_score IS NULL LIMIT 1000"
    reviews_df = bq_client.query(query).to_dataframe()

    # Analyze sentiment
    enriched_df = analyze_customer_reviews(reviews_df['review_text'].tolist())
    enriched_df['review_id'] = reviews_df['review_id'].tolist()

    # Write back to BigQuery
    table_id = f'{project}.{dataset}.enriched_reviews'
    job = bq_client.load_table_from_dataframe(enriched_df, table_id)
    job.result()
```

## Data Preparation for ML

### TensorFlow Transform (TFT) for Feature Engineering

TFT enables consistent feature transformations between training and serving.

**Complete Example**: Feature preprocessing pipeline
```python
import tensorflow as tf
import tensorflow_transform as tft
from tensorflow_transform.beam import impl as beam_impl
from tensorflow_transform.tf_metadata import dataset_metadata, schema_utils
import apache_beam as beam

# Define feature spec
RAW_DATA_FEATURE_SPEC = {
    'age': tf.io.FixedLenFeature([], tf.int64),
    'income': tf.io.FixedLenFeature([], tf.float32),
    'category': tf.io.FixedLenFeature([], tf.string),
    'product_description': tf.io.FixedLenFeature([], tf.string),
    'label': tf.io.FixedLenFeature([], tf.int64)
}

# Define preprocessing function
def preprocessing_fn(inputs):
    outputs = {}

    # Scale numerical features
    outputs['age_scaled'] = tft.scale_to_z_score(inputs['age'])
    outputs['income_normalized'] = tft.scale_to_0_1(inputs['income'])

    # Bucketize
    outputs['age_bucket'] = tft.bucketize(
        inputs['age'],
        num_buckets=5
    )

    # Vocabulary for categorical
    outputs['category_idx'] = tft.compute_and_apply_vocabulary(
        inputs['category'],
        top_k=100,
        num_oov_buckets=1
    )

    # Text features (bag of words)
    outputs['product_bow'] = tft.compute_and_apply_vocabulary(
        tf.strings.split(inputs['product_description']),
        top_k=1000,
        num_oov_buckets=1
    )

    # Pass through label
    outputs['label'] = inputs['label']

    return outputs

# Create Beam pipeline
def transform_data(input_path, output_path, temp_dir):
    with beam.Pipeline() as pipeline:
        with beam_impl.Context(temp_dir=temp_dir):
            # Read raw data
            raw_data = (
                pipeline
                | 'ReadData' >> beam.io.ReadFromBigQuery(
                    query='SELECT * FROM `project.dataset.raw_features`',
                    use_standard_sql=True
                )
            )

            # Create dataset metadata
            raw_data_metadata = dataset_metadata.DatasetMetadata(
                schema_utils.schema_from_feature_spec(RAW_DATA_FEATURE_SPEC)
            )

            # Transform data
            transformed_dataset, transform_fn = (
                (raw_data, raw_data_metadata)
                | 'AnalyzeAndTransform' >> beam_impl.AnalyzeAndTransformDataset(
                    preprocessing_fn
                )
            )

            transformed_data, transformed_metadata = transformed_dataset

            # Write transformed data
            _ = (
                transformed_data
                | 'WriteTransformedData' >> beam.io.WriteToTFRecord(
                    output_path,
                    coder=tft.coders.ExampleProtoCoder(transformed_metadata.schema)
                )
            )

            # Write transform function for serving
            _ = (
                transform_fn
                | 'WriteTransformFn' >> tft.beam.WriteTransformFn(temp_dir)
            )

# Usage
transform_data(
    input_path='bq://project.dataset.raw_features',
    output_path='gs://bucket/transformed_data/data',
    temp_dir='gs://bucket/temp'
)
```

### Dataflow for ML Feature Engineering

**Complete Example**: Real-time feature engineering pipeline
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions
from google.cloud import bigquery
import json

class FeatureEngineeringFn(beam.DoFn):
    def process(self, element):
        # Extract features from raw event
        customer_id = element['customer_id']
        event_time = element['event_timestamp']
        event_type = element['event_type']
        value = element.get('value', 0)

        # Engineer features
        features = {
            'customer_id': customer_id,
            'event_time': event_time,
            'event_type': event_type,
            'value': value,
            # Derived features
            'hour_of_day': int(event_time.split('T')[1].split(':')[0]),
            'is_weekend': self._is_weekend(event_time),
            'value_category': self._categorize_value(value),
            'log_value': self._safe_log(value)
        }

        yield features

    def _is_weekend(self, timestamp):
        # Simplified weekend check
        from datetime import datetime
        dt = datetime.fromisoformat(timestamp.replace('Z', '+00:00'))
        return dt.weekday() >= 5

    def _categorize_value(self, value):
        if value < 50:
            return 'low'
        elif value < 200:
            return 'medium'
        else:
            return 'high'

    def _safe_log(self, value):
        import math
        return math.log(value + 1)

class AggregateFeaturesFn(beam.DoFn):
    def process(self, element):
        customer_id, events = element
        # Aggregate features over window
        features = {
            'customer_id': customer_id,
            'event_count': len(events),
            'total_value': sum(e['value'] for e in events),
            'avg_value': sum(e['value'] for e in events) / len(events) if events else 0,
            'unique_event_types': len(set(e['event_type'] for e in events))
        }
        yield features

def run_feature_pipeline(project, subscription, output_table):
    options = PipelineOptions(
        streaming=True,
        project=project,
        region='us-central1',
        temp_location='gs://my-bucket/temp'
    )

    with beam.Pipeline(options=options) as pipeline:
        # Read from Pub/Sub
        raw_events = (
            pipeline
            | 'ReadPubSub' >> beam.io.ReadFromPubSub(
                subscription=f'projects/{project}/subscriptions/{subscription}'
            )
            | 'ParseJSON' >> beam.Map(lambda x: json.loads(x.decode('utf-8')))
        )

        # Engineer features
        engineered = (
            raw_events
            | 'EngineerFeatures' >> beam.ParDo(FeatureEngineeringFn())
        )

        # Aggregate features over sliding window
        aggregated = (
            engineered
            | 'AddTimestamps' >> beam.Map(
                lambda x: beam.window.TimestampedValue(x, x['event_time'])
            )
            | 'WindowInto' >> beam.WindowInto(
                beam.window.SlidingWindows(size=3600, period=300)  # 1hr window, 5min slide
            )
            | 'GroupByCustomer' >> beam.GroupBy('customer_id')
            | 'AggregateFeatures' >> beam.ParDo(AggregateFeaturesFn())
        )

        # Write to BigQuery
        _ = (
            aggregated
            | 'WriteToBigQuery' >> beam.io.WriteToBigQuery(
                table=output_table,
                schema='customer_id:STRING,event_count:INTEGER,total_value:FLOAT,avg_value:FLOAT,unique_event_types:INTEGER',
                write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND
            )
        )

# Run pipeline
run_feature_pipeline(
    project='my-project',
    subscription='raw-events-sub',
    output_table='my-project:ml_features.customer_aggregates'
)
```

## ML Pipelines

### Vertex AI Pipelines

Vertex AI Pipelines orchestrate end-to-end ML workflows using Kubeflow Pipelines (KFP).

**Complete Example**: Production ML pipeline
```python
from kfp.v2 import dsl
from kfp.v2.dsl import component, pipeline, Input, Output, Dataset, Model, Metrics
from google.cloud import aiplatform

# Component 1: Extract data from BigQuery
@component(
    base_image='python:3.9',
    packages_to_install=['google-cloud-bigquery', 'pandas', 'pyarrow']
)
def extract_data(
    project_id: str,
    dataset_id: str,
    table_id: str,
    output_dataset: Output[Dataset]
):
    from google.cloud import bigquery
    import pandas as pd

    client = bigquery.Client(project=project_id)
    query = f"""
        SELECT * FROM `{project_id}.{dataset_id}.{table_id}`
        WHERE date >= DATE_SUB(CURRENT_DATE(), INTERVAL 365 DAY)
    """

    df = client.query(query).to_dataframe()
    df.to_csv(output_dataset.path, index=False)

    print(f"Extracted {len(df)} rows")

# Component 2: Feature engineering
@component(
    base_image='python:3.9',
    packages_to_install=['pandas', 'scikit-learn']
)
def engineer_features(
    input_dataset: Input[Dataset],
    output_dataset: Output[Dataset]
):
    import pandas as pd
    from sklearn.preprocessing import StandardScaler, LabelEncoder
    import pickle

    # Read data
    df = pd.read_csv(input_dataset.path)

    # Feature engineering
    df['age_group'] = pd.cut(df['age'], bins=[0, 25, 35, 45, 55, 100], labels=['18-25', '25-35', '35-45', '45-55', '55+'])
    df['tenure_years'] = df['tenure_months'] / 12
    df['spend_per_tenure'] = df['total_spend'] / (df['tenure_months'] + 1)

    # Encode categoricals
    le = LabelEncoder()
    for col in ['region', 'product_category']:
        df[col + '_encoded'] = le.fit_transform(df[col])

    # Save engineered features
    df.to_csv(output_dataset.path, index=False)

    print(f"Engineered features for {len(df)} rows")

# Component 3: Train model
@component(
    base_image='python:3.9',
    packages_to_install=['pandas', 'scikit-learn', 'joblib']
)
def train_model(
    input_dataset: Input[Dataset],
    output_model: Output[Model],
    metrics: Output[Metrics]
):
    import pandas as pd
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, roc_auc_score
    import joblib

    # Read data
    df = pd.read_csv(input_dataset.path)

    # Prepare features and target
    feature_cols = [c for c in df.columns if c not in ['customer_id', 'label', 'date']]
    X = df[feature_cols]
    y = df['label']

    # Split data
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

    # Train model
    model = RandomForestClassifier(n_estimators=100, max_depth=10, random_state=42)
    model.fit(X_train, y_train)

    # Evaluate
    y_pred = model.predict(X_test)
    y_proba = model.predict_proba(X_test)[:, 1]

    accuracy = accuracy_score(y_test, y_pred)
    precision = precision_score(y_test, y_pred)
    recall = recall_score(y_test, y_pred)
    f1 = f1_score(y_test, y_pred)
    auc = roc_auc_score(y_test, y_proba)

    # Log metrics
    metrics.log_metric('accuracy', accuracy)
    metrics.log_metric('precision', precision)
    metrics.log_metric('recall', recall)
    metrics.log_metric('f1_score', f1)
    metrics.log_metric('auc', auc)

    # Save model
    joblib.dump(model, output_model.path)

    print(f"Model trained. AUC: {auc:.4f}, F1: {f1:.4f}")

# Component 4: Model validation
@component(
    base_image='python:3.9',
    packages_to_install=['pandas', 'scikit-learn', 'joblib']
)
def validate_model(
    model: Input[Model],
    validation_dataset: Input[Dataset],
    metrics: Output[Metrics],
    deployment_decision: Output[str]
):
    import pandas as pd
    import joblib
    from sklearn.metrics import roc_auc_score

    # Load model
    clf = joblib.load(model.path)

    # Load validation data
    df = pd.read_csv(validation_dataset.path)
    X = df.drop(['customer_id', 'label', 'date'], axis=1, errors='ignore')
    y = df['label']

    # Validate
    y_proba = clf.predict_proba(X)[:, 1]
    auc = roc_auc_score(y, y_proba)

    metrics.log_metric('validation_auc', auc)

    # Deployment decision
    if auc >= 0.85:
        deployment_decision = 'DEPLOY'
    else:
        deployment_decision = 'REJECT'

    print(f"Validation AUC: {auc:.4f}, Decision: {deployment_decision}")

# Component 5: Deploy model
@component(
    base_image='python:3.9',
    packages_to_install=['google-cloud-aiplatform']
)
def deploy_model(
    model: Input[Model],
    project_id: str,
    region: str,
    deployment_decision: str
):
    if deployment_decision != 'DEPLOY':
        print("Model deployment skipped based on validation")
        return

    from google.cloud import aiplatform

    aiplatform.init(project=project_id, location=region)

    # Upload model to Vertex AI
    uploaded_model = aiplatform.Model.upload(
        display_name='churn_model_pipeline',
        artifact_uri=model.uri,
        serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/sklearn-cpu.1-0:latest'
    )

    # Deploy to endpoint
    endpoint = aiplatform.Endpoint.create(display_name='churn_endpoint_pipeline')
    uploaded_model.deploy(
        endpoint=endpoint,
        machine_type='n1-standard-4',
        min_replica_count=1,
        max_replica_count=5
    )

    print(f"Model deployed to endpoint: {endpoint.resource_name}")

# Define pipeline
@pipeline(
    name='customer-churn-ml-pipeline',
    description='End-to-end pipeline for churn prediction',
    pipeline_root='gs://my-bucket/pipeline_root'
)
def churn_prediction_pipeline(
    project_id: str,
    dataset_id: str,
    table_id: str,
    region: str = 'us-central1'
):
    # Step 1: Extract data
    extract_task = extract_data(
        project_id=project_id,
        dataset_id=dataset_id,
        table_id=table_id
    )

    # Step 2: Engineer features
    engineer_task = engineer_features(
        input_dataset=extract_task.outputs['output_dataset']
    )

    # Step 3: Train model
    train_task = train_model(
        input_dataset=engineer_task.outputs['output_dataset']
    )

    # Step 4: Validate model
    validate_task = validate_model(
        model=train_task.outputs['output_model'],
        validation_dataset=engineer_task.outputs['output_dataset']
    )

    # Step 5: Deploy model (conditional)
    deploy_task = deploy_model(
        model=train_task.outputs['output_model'],
        project_id=project_id,
        region=region,
        deployment_decision=validate_task.outputs['deployment_decision']
    )

# Compile and run pipeline
from kfp.v2 import compiler

compiler.Compiler().compile(
    pipeline_func=churn_prediction_pipeline,
    package_path='churn_pipeline.json'
)

# Submit pipeline job
aiplatform.init(project='my-project', location='us-central1')

job = aiplatform.PipelineJob(
    display_name='churn-pipeline-run',
    template_path='churn_pipeline.json',
    parameter_values={
        'project_id': 'my-project',
        'dataset_id': 'ml_data',
        'table_id': 'customer_features',
        'region': 'us-central1'
    }
)

job.run(service_account='pipeline-sa@my-project.iam.gserviceaccount.com')
```

### Cloud Composer for ML Orchestration

**Complete Example**: DAG for scheduled model retraining
```python
from airflow import DAG
from airflow.providers.google.cloud.operators.bigquery import BigQueryInsertJobOperator
from airflow.providers.google.cloud.operators.vertex_ai import (
    CreateCustomTrainingJobOperator,
    CreateBatchPredictionJobOperator
)
from airflow.operators.python import PythonOperator
from airflow.utils.dates import days_ago
from datetime import timedelta

default_args = {
    'owner': 'data-engineering',
    'depends_on_past': False,
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 2,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'ml_model_retraining',
    default_args=default_args,
    description='Weekly model retraining pipeline',
    schedule_interval='0 2 * * 1',  # Every Monday at 2 AM
    start_date=days_ago(1),
    catchup=False,
    tags=['ml', 'retraining']
)

# Task 1: Prepare training data
prepare_data = BigQueryInsertJobOperator(
    task_id='prepare_training_data',
    configuration={
        'query': {
            'query': '''
                CREATE OR REPLACE TABLE `my-project.ml_data.training_features` AS
                SELECT
                    customer_id,
                    age,
                    income,
                    tenure_months,
                    total_spend,
                    churned AS label
                FROM `my-project.analytics.customers`
                WHERE last_updated >= DATE_SUB(CURRENT_DATE(), INTERVAL 90 DAY)
            ''',
            'useLegacySql': False
        }
    },
    dag=dag
)

# Task 2: Train BigQuery ML model
train_bqml_model = BigQueryInsertJobOperator(
    task_id='train_bqml_model',
    configuration={
        'query': {
            'query': '''
                CREATE OR REPLACE MODEL `my-project.ml_models.churn_model_{{ ds_nodash }}`
                OPTIONS(
                    model_type='LOGISTIC_REG',
                    input_label_cols=['label'],
                    auto_class_weights=TRUE
                ) AS
                SELECT * FROM `my-project.ml_data.training_features`
            ''',
            'useLegacySql': False
        }
    },
    dag=dag
)

# Task 3: Evaluate model
evaluate_model = BigQueryInsertJobOperator(
    task_id='evaluate_model',
    configuration={
        'query': {
            'query': '''
                CREATE OR REPLACE TABLE `my-project.ml_models.model_metrics_{{ ds_nodash }}` AS
                SELECT
                    '{{ ds }}' AS training_date,
                    precision,
                    recall,
                    accuracy,
                    f1_score,
                    roc_auc
                FROM ML.EVALUATE(MODEL `my-project.ml_models.churn_model_{{ ds_nodash }}`)
            ''',
            'useLegacySql': False
        }
    },
    dag=dag
)

# Task 4: Generate predictions
generate_predictions = BigQueryInsertJobOperator(
    task_id='generate_predictions',
    configuration={
        'query': {
            'query': '''
                CREATE OR REPLACE TABLE `my-project.predictions.churn_scores_{{ ds_nodash }}` AS
                SELECT
                    customer_id,
                    predicted_label,
                    predicted_label_probs[OFFSET(1)].prob AS churn_probability,
                    '{{ ds }}' AS prediction_date
                FROM ML.PREDICT(
                    MODEL `my-project.ml_models.churn_model_{{ ds_nodash }}`,
                    (SELECT * FROM `my-project.analytics.active_customers`)
                )
            ''',
            'useLegacySql': False
        }
    },
    dag=dag
)

# Task 5: Export model (optional)
def export_model_fn(**context):
    from google.cloud import bigquery
    client = bigquery.Client()

    execution_date = context['ds_nodash']
    export_query = f'''
        EXPORT MODEL `my-project.ml_models.churn_model_{execution_date}`
        OPTIONS(
            uri='gs://my-bucket/models/churn_model_{execution_date}/*'
        )
    '''

    client.query(export_query).result()
    print(f"Model exported to GCS")

export_model = PythonOperator(
    task_id='export_model',
    python_callable=export_model_fn,
    provide_context=True,
    dag=dag
)

# Define task dependencies
prepare_data >> train_bqml_model >> evaluate_model >> [generate_predictions, export_model]
```

### Integration with Data Pipelines

**Pattern 1**: BigQuery + BigQuery ML + Scheduled Queries
```sql
-- Create scheduled query for daily predictions
CREATE OR REPLACE PROCEDURE `project.dataset.daily_churn_scoring`()
BEGIN
    -- Refresh features
    CREATE OR REPLACE TABLE `project.dataset.latest_features` AS
    SELECT
        customer_id,
        age,
        tenure_months,
        total_spend,
        days_since_last_purchase
    FROM `project.dataset.customer_metrics`;

    -- Generate predictions
    CREATE OR REPLACE TABLE `project.dataset.churn_predictions` AS
    SELECT
        customer_id,
        predicted_label_probs[OFFSET(1)].prob AS churn_risk,
        CURRENT_TIMESTAMP() AS scored_at
    FROM ML.PREDICT(
        MODEL `project.dataset.churn_model`,
        (SELECT * FROM `project.dataset.latest_features`)
    );

    -- Send high-risk customers to action table
    CREATE OR REPLACE TABLE `project.dataset.high_risk_customers` AS
    SELECT *
    FROM `project.dataset.churn_predictions`
    WHERE churn_risk >= 0.7;
END;

-- Schedule via BigQuery Transfer Service (UI or API)
```

**Pattern 2**: Pub/Sub → Dataflow → BigQuery ML
```python
# Dataflow pipeline that triggers ML predictions
def run_realtime_ml_pipeline():
    with beam.Pipeline() as p:
        # Read from Pub/Sub
        events = (
            p
            | 'ReadPubSub' >> beam.io.ReadFromPubSub(subscription='projects/my-project/subscriptions/events')
            | 'ParseJSON' >> beam.Map(json.loads)
        )

        # Feature engineering
        features = (
            events
            | 'EngineerFeatures' >> beam.ParDo(FeatureEngineeringFn())
        )

        # Write features to BigQuery
        _ = (
            features
            | 'WriteToBQ' >> beam.io.WriteToBigQuery(
                'my-project:dataset.realtime_features',
                write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND
            )
        )

        # Batch predictions (via separate scheduled query on BigQuery ML)
```

**Pattern 3**: Dataflow → Feature Store → Vertex AI Endpoint
```python
# Real-time prediction pipeline
def run_realtime_prediction_pipeline():
    with beam.Pipeline() as p:
        predictions = (
            p
            | 'ReadPubSub' >> beam.io.ReadFromPubSub(subscription='...')
            | 'ParseJSON' >> beam.Map(json.loads)
            | 'EngineerFeatures' >> beam.ParDo(FeatureEngineeringFn())
            | 'PredictWithVertexAI' >> beam.ParDo(VertexAIPredictionFn(
                endpoint_id='projects/123/locations/us-central1/endpoints/456'
            ))
            | 'WriteResults' >> beam.io.WriteToBigQuery('dataset.predictions')
        )
```

## MLOps Practices

### Model Versioning

**Complete Example**: Model registry and versioning
```python
from google.cloud import aiplatform

# Initialize
aiplatform.init(project='my-project', location='us-central1')

# Upload model with version information
model = aiplatform.Model.upload(
    display_name='churn_prediction_model',
    artifact_uri='gs://my-bucket/models/churn_v2/',
    serving_container_image_uri='us-docker.pkg.dev/vertex-ai/prediction/sklearn-cpu.1-0:latest',
    labels={
        'model_type': 'classification',
        'version': 'v2.0',
        'training_date': '2024-01-15',
        'framework': 'sklearn',
        'dataset_version': 'v1.2'
    },
    version_aliases=['production', 'latest'],
    version_description='Improved feature engineering with better precision'
)

# List all versions of a model
models = aiplatform.Model.list(
    filter='display_name="churn_prediction_model"',
    order_by='create_time desc'
)

for m in models:
    print(f"Version: {m.version_id}, Labels: {m.labels}, Created: {m.create_time}")

# Get specific version
model_v1 = aiplatform.Model(
    model_name='projects/123/locations/us-central1/models/456@1'  # @1 for version 1
)

# Compare model versions in BigQuery
comparison_query = '''
WITH model_v1_metrics AS (
  SELECT 'v1' AS version, precision, recall, f1_score, roc_auc
  FROM ML.EVALUATE(MODEL `project.dataset.churn_model_v1`)
),
model_v2_metrics AS (
  SELECT 'v2' AS version, precision, recall, f1_score, roc_auc
  FROM ML.EVALUATE(MODEL `project.dataset.churn_model_v2`)
)
SELECT * FROM model_v1_metrics
UNION ALL
SELECT * FROM model_v2_metrics
ORDER BY version;
'''
```

### Model Monitoring

**Complete Example**: Vertex AI Model Monitoring setup
```python
from google.cloud import aiplatform
from google.cloud.aiplatform import model_monitoring

# Create monitoring job
monitoring_job = aiplatform.ModelDeploymentMonitoringJob.create(
    display_name='churn_model_monitoring',
    endpoint=endpoint,
    logging_sampling_strategy=model_monitoring.SamplingStrategy(
        random_sample_config=model_monitoring.RandomSampleConfig(
            sample_rate=0.5  # Monitor 50% of predictions
        )
    ),
    schedule_config=model_monitoring.ScheduleConfig(
        monitor_interval=3600  # Check every hour
    ),
    alert_config=model_monitoring.AlertConfig(
        email_alert_config=model_monitoring.EmailAlertConfig(
            user_emails=['data-eng@company.com']
        )
    ),
    model_monitoring_objective_configs=[
        model_monitoring.ObjectiveConfig(
            training_dataset=model_monitoring.Dataset(
                gcs_source=model_monitoring.GcsSource(
                    uris=['gs://my-bucket/training_data.csv']
                ),
                target_field='label'
            ),
            training_prediction_skew_detection_config=model_monitoring.SkewDetectionConfig(
                skew_thresholds={
                    'age': 0.3,
                    'income': 0.3,
                    'tenure_months': 0.3
                },
                attribution_score_skew_thresholds={
                    'age': 0.3,
                    'income': 0.3
                }
            ),
            prediction_drift_detection_config=model_monitoring.DriftDetectionConfig(
                drift_thresholds={
                    'age': 0.3,
                    'income': 0.3,
                    'tenure_months': 0.3
                }
            )
        )
    ]
)

print(f"Monitoring job created: {monitoring_job.resource_name}")
```

**BigQuery ML Monitoring**:
```sql
-- Monitor prediction distribution over time
CREATE OR REPLACE TABLE `project.dataset.prediction_monitoring` AS
SELECT
  DATE(scored_at) AS prediction_date,
  COUNT(*) AS total_predictions,
  AVG(churn_probability) AS avg_churn_prob,
  STDDEV(churn_probability) AS stddev_churn_prob,
  COUNTIF(churn_probability >= 0.7) AS high_risk_count,
  COUNTIF(churn_probability >= 0.7) / COUNT(*) AS high_risk_rate
FROM `project.dataset.churn_predictions`
GROUP BY prediction_date
ORDER BY prediction_date DESC;

-- Detect data drift by comparing feature distributions
CREATE OR REPLACE TABLE `project.dataset.feature_drift_detection` AS
WITH training_stats AS (
  SELECT
    AVG(age) AS avg_age,
    STDDEV(age) AS stddev_age,
    AVG(tenure_months) AS avg_tenure,
    STDDEV(tenure_months) AS stddev_tenure
  FROM `project.dataset.training_data`
),
recent_stats AS (
  SELECT
    AVG(age) AS avg_age,
    STDDEV(age) AS stddev_age,
    AVG(tenure_months) AS avg_tenure,
    STDDEV(tenure_months) AS stddev_tenure
  FROM `project.dataset.customer_features`
  WHERE updated_date >= CURRENT_DATE() - 7
)
SELECT
  'age' AS feature,
  t.avg_age AS training_mean,
  r.avg_age AS recent_mean,
  ABS(r.avg_age - t.avg_age) / t.stddev_age AS drift_zscore,
  CASE WHEN ABS(r.avg_age - t.avg_age) / t.stddev_age > 2 THEN TRUE ELSE FALSE END AS drift_detected
FROM training_stats t, recent_stats r
UNION ALL
SELECT
  'tenure_months' AS feature,
  t.avg_tenure AS training_mean,
  r.avg_tenure AS recent_mean,
  ABS(r.avg_tenure - t.avg_tenure) / t.stddev_tenure AS drift_zscore,
  CASE WHEN ABS(r.avg_tenure - t.avg_tenure) / t.stddev_tenure > 2 THEN TRUE ELSE FALSE END AS drift_detected
FROM training_stats t, recent_stats r;

-- Monitor model performance over time (if ground truth available)
CREATE OR REPLACE TABLE `project.dataset.model_performance_tracking` AS
SELECT
  DATE_TRUNC(prediction_date, WEEK) AS week,
  COUNT(*) AS total_predictions,
  AVG(CAST(predicted_churn AS INT64)) AS avg_predicted_churn,
  AVG(CAST(actual_churn AS INT64)) AS avg_actual_churn,
  -- Calculate precision and recall
  SUM(CASE WHEN predicted_churn AND actual_churn THEN 1 ELSE 0 END) / NULLIF(SUM(CAST(predicted_churn AS INT64)), 0) AS precision,
  SUM(CASE WHEN predicted_churn AND actual_churn THEN 1 ELSE 0 END) / NULLIF(SUM(CAST(actual_churn AS INT64)), 0) AS recall
FROM `project.dataset.predictions_with_labels`
GROUP BY week
ORDER BY week DESC;
```

### Continuous Training

**Complete Example**: Automated retraining pipeline
```python
# Cloud Function triggered by Pub/Sub for model retraining
import functions_framework
from google.cloud import aiplatform, bigquery
from datetime import datetime

@functions_framework.cloud_event
def trigger_retraining(cloud_event):
    """Triggered by Pub/Sub message indicating model drift or performance degradation"""

    data = cloud_event.data
    trigger_reason = data.get('reason', 'scheduled')

    print(f"Retraining triggered: {trigger_reason}")

    # Check if retraining is needed
    if should_retrain():
        # Submit Vertex AI Pipeline job
        aiplatform.init(project='my-project', location='us-central1')

        job = aiplatform.PipelineJob(
            display_name=f'retraining_{datetime.now().strftime("%Y%m%d_%H%M%S")}',
            template_path='gs://my-bucket/pipelines/training_pipeline.json',
            parameter_values={
                'project_id': 'my-project',
                'dataset_id': 'ml_data',
                'training_date': datetime.now().isoformat()
            },
            labels={
                'trigger': trigger_reason,
                'automated': 'true'
            }
        )

        job.run(service_account='training-sa@my-project.iam.gserviceaccount.com')

        print(f"Retraining job submitted: {job.resource_name}")
    else:
        print("Retraining not needed based on current metrics")

def should_retrain():
    """Check if retraining is needed based on performance metrics"""
    client = bigquery.Client()

    # Query recent model performance
    query = '''
        SELECT
          precision,
          recall,
          f1_score
        FROM `my-project.ml_models.model_performance_tracking`
        ORDER BY week DESC
        LIMIT 1
    '''

    result = client.query(query).result()
    metrics = list(result)[0]

    # Retraining criteria
    if metrics.f1_score < 0.75:  # F1 score dropped below threshold
        return True
    if metrics.precision < 0.70 or metrics.recall < 0.70:  # Either metric too low
        return True

    return False
```

**Scheduler-based retraining** (Cloud Scheduler → Pub/Sub → Cloud Function):
```bash
# Create Cloud Scheduler job
gcloud scheduler jobs create pubsub weekly-model-retraining \
  --schedule="0 2 * * 1" \
  --time-zone="America/New_York" \
  --topic=model-retraining-trigger \
  --message-body='{"reason": "scheduled_weekly"}' \
  --description="Weekly automated model retraining"

# Create Pub/Sub topic
gcloud pubsub topics create model-retraining-trigger

# Deploy Cloud Function
gcloud functions deploy trigger-retraining \
  --gen2 \
  --runtime=python311 \
  --region=us-central1 \
  --source=. \
  --entry-point=trigger_retraining \
  --trigger-topic=model-retraining-trigger \
  --service-account=training-trigger-sa@my-project.iam.gserviceaccount.com
```

### A/B Testing and Model Rollout

**Complete Example**: Traffic splitting for A/B testing
```python
from google.cloud import aiplatform

# Deploy model v1 and v2 to same endpoint
endpoint = aiplatform.Endpoint('projects/123/locations/us-central1/endpoints/456')

# Deploy v2 with 20% traffic (canary deployment)
model_v2 = aiplatform.Model('projects/123/locations/us-central1/models/789')
deployed_model_v2 = model_v2.deploy(
    endpoint=endpoint,
    deployed_model_display_name='churn_model_v2',
    machine_type='n1-standard-4',
    min_replica_count=1,
    max_replica_count=5,
    traffic_percentage=20,  # 20% traffic to v2
    traffic_split={
        'deployed_model_v1_id': 80,  # 80% to v1
        'deployed_model_v2_id': 20   # 20% to v2
    }
)

# Monitor performance of both versions
# ... collect metrics ...

# If v2 performs better, gradually increase traffic
endpoint.update(
    traffic_split={
        'deployed_model_v1_id': 50,  # Reduce v1 to 50%
        'deployed_model_v2_id': 50   # Increase v2 to 50%
    }
)

# Eventually route all traffic to v2
endpoint.update(
    traffic_split={
        'deployed_model_v1_id': 0,
        'deployed_model_v2_id': 100
    }
)

# Rollback if issues detected
endpoint.update(
    traffic_split={
        'deployed_model_v1_id': 100,  # Revert to v1
        'deployed_model_v2_id': 0
    }
)
```

**A/B testing analysis**:
```sql
-- Compare model performance during A/B test
WITH predictions_with_model_version AS (
  SELECT
    customer_id,
    prediction,
    actual_outcome,
    model_version,
    prediction_timestamp
  FROM `project.dataset.ab_test_predictions`
  WHERE prediction_timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)
)
SELECT
  model_version,
  COUNT(*) AS total_predictions,
  AVG(CAST(prediction AS FLOAT64)) AS avg_predicted_prob,
  -- Accuracy (if ground truth available)
  SUM(CASE WHEN (prediction >= 0.5) = actual_outcome THEN 1 ELSE 0 END) / COUNT(*) AS accuracy,
  -- Precision
  SUM(CASE WHEN prediction >= 0.5 AND actual_outcome THEN 1 ELSE 0 END) /
    NULLIF(SUM(CASE WHEN prediction >= 0.5 THEN 1 ELSE 0 END), 0) AS precision,
  -- Recall
  SUM(CASE WHEN prediction >= 0.5 AND actual_outcome THEN 1 ELSE 0 END) /
    NULLIF(SUM(CAST(actual_outcome AS INT64)), 0) AS recall,
  -- Business metric: conversion rate for high-risk customers
  SUM(CASE WHEN prediction >= 0.7 AND actual_outcome THEN 1 ELSE 0 END) /
    NULLIF(SUM(CASE WHEN prediction >= 0.7 THEN 1 ELSE 0 END), 0) AS high_risk_precision
FROM predictions_with_model_version
GROUP BY model_version
ORDER BY model_version;
```

### Model Serving Patterns

**Batch vs Real-time Decision Matrix**:

| Pattern | Latency | Throughput | Use Case | GCP Service |
|---------|---------|------------|----------|-------------|
| **Batch** | Minutes to hours | Very high (millions/hour) | Daily scoring, reports | BigQuery ML, Vertex AI Batch |
| **Real-time** | <100ms | Medium (thousands/sec) | User-facing predictions | Vertex AI Endpoints |
| **Near real-time** | Seconds | High | Stream processing | Dataflow + BigQuery ML |
| **Micro-batch** | Minutes | High | Periodic updates | Scheduled queries |

**Batch prediction example**:
```python
# Vertex AI batch prediction
from google.cloud import aiplatform

model = aiplatform.Model('projects/123/locations/us-central1/models/456')

batch_job = model.batch_predict(
    job_display_name='daily_customer_scoring',
    bigquery_source='bq://my-project.dataset.customers_to_score',
    bigquery_destination_prefix='bq://my-project.predictions',
    instances_format='bigquery',
    predictions_format='bigquery',
    machine_type='n1-standard-4',
    starting_replica_count=10,
    max_replica_count=20
)

batch_job.wait()
```

**Real-time serving example**:
```python
# Low-latency prediction service
from google.cloud import aiplatform
from flask import Flask, request, jsonify

app = Flask(__name__)

# Initialize endpoint
endpoint = aiplatform.Endpoint('projects/123/locations/us-central1/endpoints/456')

@app.route('/predict', methods=['POST'])
def predict():
    data = request.json

    # Prepare instance
    instance = {
        'age': data['age'],
        'income': data['income'],
        'tenure_months': data['tenure_months']
    }

    # Get prediction with low latency
    prediction = endpoint.predict(instances=[instance])

    return jsonify({
        'customer_id': data['customer_id'],
        'churn_probability': prediction.predictions[0]['probability'],
        'recommendation': 'contact' if prediction.predictions[0]['probability'] > 0.7 else 'monitor'
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

## Complete ML Scenarios for Data Engineers

### Scenario 1: Customer Churn Prediction (Batch)

**Requirements**: Daily batch scoring of 10 million customers for churn risk.

**Solution Architecture**:
```
BigQuery (data warehouse) → BigQuery ML (training/prediction) →
Scheduled queries (automation) → Looker/Data Studio (visualization)
```

**Complete Implementation**:
```sql
-- Step 1: Create training features table
CREATE OR REPLACE TABLE `project.ml_data.churn_training_features` AS
SELECT
  customer_id,
  -- Demographics
  age,
  gender,
  region,
  -- Behavioral features
  tenure_months,
  total_spend_12m,
  avg_monthly_spend,
  transaction_count_30d,
  transaction_count_90d,
  days_since_last_transaction,
  -- Engagement features
  support_ticket_count,
  login_count_30d,
  email_open_rate,
  -- Product usage
  product_count,
  active_subscriptions,
  -- Target
  churned_90d AS label
FROM `project.analytics.customer_features`
WHERE feature_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 365 DAY);

-- Step 2: Train model
CREATE OR REPLACE MODEL `project.ml_models.churn_prediction_v1`
OPTIONS(
  model_type='BOOSTED_TREE_CLASSIFIER',  -- High accuracy for tabular data
  input_label_cols=['label'],
  auto_class_weights=TRUE,
  max_iterations=50,
  enable_global_explain=TRUE
) AS
SELECT * EXCEPT(customer_id) FROM `project.ml_data.churn_training_features`;

-- Step 3: Evaluate model
CREATE OR REPLACE TABLE `project.ml_models.churn_model_metrics` AS
SELECT
  CURRENT_DATE() AS evaluation_date,
  'v1' AS model_version,
  precision,
  recall,
  accuracy,
  f1_score,
  roc_auc
FROM ML.EVALUATE(MODEL `project.ml_models.churn_prediction_v1`);

-- Step 4: Create scheduled prediction procedure
CREATE OR REPLACE PROCEDURE `project.ml_data.daily_churn_scoring`()
BEGIN
  -- Generate predictions for all active customers
  CREATE OR REPLACE TABLE `project.predictions.churn_scores` AS
  SELECT
    customer_id,
    predicted_label AS predicted_churn,
    predicted_label_probs[OFFSET(1)].prob AS churn_probability,
    CASE
      WHEN predicted_label_probs[OFFSET(1)].prob >= 0.8 THEN 'CRITICAL'
      WHEN predicted_label_probs[OFFSET(1)].prob >= 0.5 THEN 'HIGH'
      WHEN predicted_label_probs[OFFSET(1)].prob >= 0.3 THEN 'MEDIUM'
      ELSE 'LOW'
    END AS risk_category,
    CURRENT_TIMESTAMP() AS scored_at
  FROM ML.PREDICT(
    MODEL `project.ml_models.churn_prediction_v1`,
    (SELECT * FROM `project.analytics.active_customers`)
  );

  -- Create action list for marketing team
  CREATE OR REPLACE TABLE `project.actions.high_risk_customers` AS
  SELECT
    c.customer_id,
    c.customer_name,
    c.email,
    cs.churn_probability,
    cs.risk_category,
    c.total_lifetime_value,
    c.last_contact_date
  FROM `project.predictions.churn_scores` cs
  JOIN `project.analytics.customers` c USING (customer_id)
  WHERE cs.risk_category IN ('CRITICAL', 'HIGH')
  ORDER BY cs.churn_probability DESC;
END;

-- Schedule via Cloud Scheduler or BigQuery Transfer Service
```

**Key Decisions**:
- Use BigQuery ML (not Vertex AI) because data is already in BigQuery and batch processing is sufficient
- Use BOOSTED_TREE_CLASSIFIER for best accuracy on tabular data
- Schedule with BigQuery scheduled queries for simplicity

### Scenario 2: Real-time Fraud Detection

**Requirements**: Detect fraudulent transactions in <200ms for online payments.

**Solution Architecture**:
```
Payment API → Pub/Sub (buffer) → Dataflow (feature engineering) →
Vertex AI Endpoint (real-time prediction) → Bigtable (fast lookup) →
BigQuery (analytics/retraining)
```

**Complete Implementation**:
```python
# Dataflow pipeline for real-time fraud detection
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions
from google.cloud import aiplatform, bigtable
import json

class EnrichTransactionFn(beam.DoFn):
    def process(self, transaction):
        # Enrich transaction with historical features from Bigtable
        customer_id = transaction['customer_id']

        # Fetch customer history (cached in Bigtable)
        # ... Bigtable lookup ...

        features = {
            'transaction_id': transaction['transaction_id'],
            'amount': transaction['amount'],
            'merchant_category': transaction['merchant_category'],
            'hour_of_day': int(transaction['timestamp'].split('T')[1].split(':')[0]),
            # Historical features
            'avg_transaction_30d': 150.25,  # From Bigtable
            'transaction_count_24h': 3,  # From Bigtable
            'time_since_last_transaction_mins': 120  # Calculated
        }

        yield features

class PredictFraudFn(beam.DoFn):
    def __init__(self, endpoint_id):
        self.endpoint_id = endpoint_id
        self.endpoint = None

    def setup(self):
        # Initialize endpoint connection once per worker
        self.endpoint = aiplatform.Endpoint(self.endpoint_id)

    def process(self, features):
        # Get prediction
        prediction = self.endpoint.predict(instances=[features])
        fraud_probability = prediction.predictions[0]['probability']

        result = {
            'transaction_id': features['transaction_id'],
            'fraud_probability': fraud_probability,
            'decision': 'BLOCK' if fraud_probability > 0.9 else 'ALLOW' if fraud_probability < 0.3 else 'REVIEW',
            'timestamp': features['timestamp']
        }

        yield result

class WriteToBigtableFn(beam.DoFn):
    def process(self, result):
        # Write decision to Bigtable for fast lookup
        # ... Bigtable write ...
        yield result

def run_fraud_detection_pipeline():
    options = PipelineOptions(
        streaming=True,
        project='my-project',
        region='us-central1'
    )

    with beam.Pipeline(options=options) as p:
        predictions = (
            p
            | 'ReadPubSub' >> beam.io.ReadFromPubSub(
                subscription='projects/my-project/subscriptions/transactions'
            )
            | 'ParseJSON' >> beam.Map(lambda x: json.loads(x.decode('utf-8')))
            | 'EnrichFeatures' >> beam.ParDo(EnrichTransactionFn())
            | 'PredictFraud' >> beam.ParDo(PredictFraudFn(
                endpoint_id='projects/123/locations/us-central1/endpoints/456'
            ))
            | 'WriteToBigtable' >> beam.ParDo(WriteToBigtableFn())
            | 'WriteToBigQuery' >> beam.io.WriteToBigQuery(
                table='my-project:fraud.predictions',
                schema='transaction_id:STRING,fraud_probability:FLOAT,decision:STRING,timestamp:TIMESTAMP',
                write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND
            )
        )

# Run pipeline
run_fraud_detection_pipeline()
```

**Key Decisions**:
- Use Vertex AI Endpoint (not BigQuery ML) for low-latency real-time predictions
- Use Bigtable for fast feature lookup and result storage
- Use Dataflow for stream processing and feature engineering
- Store predictions in BigQuery for model retraining

### Scenario 3: Product Recommendation System

**Requirements**: Generate personalized product recommendations for 5 million users.

**Solution Architecture**:
```
BigQuery (user interactions) → BigQuery ML (matrix factorization) →
Cloud Functions (serving API) → Memorystore (caching) → Application
```

**Complete Implementation**:
```sql
-- Step 1: Prepare user-item interaction matrix
CREATE OR REPLACE TABLE `project.ml_data.user_product_interactions` AS
SELECT
  user_id,
  product_id,
  -- Implicit feedback: higher rating = more interaction
  SUM(view_count) * 1 +
  SUM(cart_add_count) * 3 +
  SUM(purchase_count) * 5 AS rating
FROM `project.analytics.user_behavior`
WHERE interaction_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 90 DAY)
GROUP BY user_id, product_id
HAVING rating > 0;

-- Step 2: Train matrix factorization model
CREATE OR REPLACE MODEL `project.ml_models.product_recommendations`
OPTIONS(
  model_type='MATRIX_FACTORIZATION',
  user_col='user_id',
  item_col='product_id',
  rating_col='rating',
  feedback_type='implicit',  -- No explicit ratings, only interactions
  num_factors=50,  -- Embedding dimensions
  l2_reg=0.1,
  max_iterations=30
) AS
SELECT user_id, product_id, rating
FROM `project.ml_data.user_product_interactions`;

-- Step 3: Generate recommendations for all users
CREATE OR REPLACE TABLE `project.recommendations.user_product_recommendations` AS
SELECT
  user_id,
  ARRAY_AGG(
    STRUCT(product_id, predicted_rating)
    ORDER BY predicted_rating DESC
    LIMIT 20
  ) AS recommendations
FROM ML.RECOMMEND(MODEL `project.ml_models.product_recommendations`)
GROUP BY user_id;

-- Step 4: Get recommendations for specific user (via Cloud Function)
-- This query would be called from Cloud Function API
SELECT
  product_id,
  predicted_rating AS score
FROM ML.RECOMMEND(MODEL `project.ml_models.product_recommendations`)
WHERE user_id = @user_id
ORDER BY predicted_rating DESC
LIMIT 10;
```

**Cloud Function for serving**:
```python
import functions_framework
from google.cloud import bigquery
import json

# Initialize BigQuery client
bq_client = bigquery.Client()

@functions_framework.http
def get_recommendations(request):
    """HTTP Cloud Function for product recommendations"""
    request_json = request.get_json()
    user_id = request_json.get('user_id')

    if not user_id:
        return {'error': 'user_id required'}, 400

    # Query pre-computed recommendations
    query = f'''
        SELECT recommendations
        FROM `project.recommendations.user_product_recommendations`
        WHERE user_id = @user_id
    '''

    job_config = bigquery.QueryJobConfig(
        query_parameters=[
            bigquery.ScalarQueryParameter('user_id', 'STRING', user_id)
        ]
    )

    result = bq_client.query(query, job_config=job_config).result()
    rows = list(result)

    if not rows:
        return {'error': 'User not found'}, 404

    recommendations = [
        {'product_id': r['product_id'], 'score': r['predicted_rating']}
        for r in rows[0]['recommendations'][:10]
    ]

    return {'user_id': user_id, 'recommendations': recommendations}
```

**Key Decisions**:
- Use BigQuery ML matrix factorization for collaborative filtering
- Pre-compute recommendations for all users (batch)
- Serve via Cloud Functions with caching in Memorystore for low latency
- Retrain weekly to incorporate new user behavior

### Scenario 4: Time Series Forecasting for Demand Planning

**Requirements**: Forecast product demand for next 30 days across 10,000 products.

**Solution Architecture**:
```
BigQuery (sales data) → BigQuery ML (ARIMA_PLUS per product) →
Scheduled queries (daily refresh) → Data Studio (visualization)
```

**Complete Implementation**:
```sql
-- Step 1: Prepare time series data
CREATE OR REPLACE TABLE `project.ml_data.product_demand_history` AS
SELECT
  DATE(order_timestamp) AS date,
  product_id,
  SUM(quantity) AS daily_demand,
  AVG(unit_price) AS avg_price,
  SUM(revenue) AS daily_revenue
FROM `project.sales.orders`
WHERE order_timestamp >= DATE_SUB(CURRENT_DATE(), INTERVAL 730 DAY)  -- 2 years history
GROUP BY date, product_id
ORDER BY product_id, date;

-- Step 2: Train ARIMA_PLUS model (one model for all products)
CREATE OR REPLACE MODEL `project.ml_models.demand_forecast`
OPTIONS(
  model_type='ARIMA_PLUS',
  time_series_timestamp_col='date',
  time_series_data_col='daily_demand',
  time_series_id_col='product_id',  -- Separate forecast per product
  auto_arima=TRUE,
  data_frequency='DAILY',
  holiday_region='US',
  adjust_step_changes=TRUE,  -- Detect promotions/events
  enable_auto_arima_seasonality=TRUE
) AS
SELECT date, product_id, daily_demand
FROM `project.ml_data.product_demand_history`;

-- Step 3: Generate 30-day forecasts
CREATE OR REPLACE TABLE `project.forecasts.demand_next_30_days` AS
SELECT
  forecast_timestamp AS date,
  time_series_id AS product_id,
  forecast_value AS predicted_demand,
  prediction_interval_lower_bound AS lower_bound,
  prediction_interval_upper_bound AS upper_bound,
  confidence_level,
  CURRENT_TIMESTAMP() AS forecast_generated_at
FROM ML.FORECAST(
  MODEL `project.ml_models.demand_forecast`,
  STRUCT(30 AS horizon, 0.95 AS confidence_level)
);

-- Step 4: Aggregate forecasts for purchasing decisions
CREATE OR REPLACE TABLE `project.forecasts.purchasing_plan` AS
SELECT
  product_id,
  SUM(predicted_demand) AS total_demand_30d,
  SUM(lower_bound) AS conservative_demand,
  SUM(upper_bound) AS optimistic_demand,
  AVG(predicted_demand) AS avg_daily_demand
FROM `project.forecasts.demand_next_30_days`
GROUP BY product_id;

-- Step 5: Detect anomalies in recent sales
CREATE OR REPLACE TABLE `project.monitoring.demand_anomalies` AS
SELECT
  history_timestamp AS date,
  time_series_id AS product_id,
  history_value AS actual_demand,
  is_anomaly,
  anomaly_probability,
  lower_bound AS expected_lower,
  upper_bound AS expected_upper
FROM ML.DETECT_ANOMALIES(
  MODEL `project.ml_models.demand_forecast`,
  STRUCT(0.9 AS anomaly_prob_threshold)
)
WHERE is_anomaly = TRUE
  AND history_timestamp >= DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
ORDER BY history_timestamp DESC, anomaly_probability DESC;
```

**Key Decisions**:
- Use BigQuery ML ARIMA_PLUS for time series forecasting
- Use `time_series_id_col` to train separate models for each product efficiently
- Include holiday effects and step change detection
- Generate anomaly detection for supply chain alerts

### Scenario 5: Image Classification Pipeline for E-commerce

**Requirements**: Classify product images into categories for search and recommendations.

**Solution Architecture**:
```
Cloud Storage (image upload) → Cloud Functions (trigger) →
Vertex AI AutoML Vision (classification) → Firestore (metadata) →
BigQuery (analytics)
```

**Complete Implementation**:
```python
# Cloud Function triggered by Cloud Storage upload
import functions_framework
from google.cloud import aiplatform, firestore, bigquery, storage
from datetime import datetime

@functions_framework.cloud_event
def classify_product_image(cloud_event):
    """Triggered by Cloud Storage upload"""

    data = cloud_event.data
    bucket_name = data['bucket']
    file_name = data['name']
    gcs_uri = f'gs://{bucket_name}/{file_name}'

    print(f'Processing image: {gcs_uri}')

    # Initialize clients
    aiplatform.init(project='my-project', location='us-central1')
    db = firestore.Client()
    bq_client = bigquery.Client()

    # Get prediction from AutoML Vision model
    endpoint = aiplatform.Endpoint('projects/123/locations/us-central1/endpoints/789')

    # For AutoML Vision, we pass the GCS URI
    prediction = endpoint.predict(instances=[{'content': gcs_uri}])

    # Extract top predictions
    top_predictions = sorted(
        prediction.predictions[0]['displayNames'],
        key=lambda x: x['confidence'],
        reverse=True
    )[:3]

    result = {
        'image_uri': gcs_uri,
        'product_id': file_name.split('/')[1].replace('.jpg', ''),
        'primary_category': top_predictions[0]['displayName'],
        'primary_confidence': top_predictions[0]['confidence'],
        'all_predictions': top_predictions,
        'classified_at': datetime.utcnow()
    }

    # Write to Firestore for application access
    doc_ref = db.collection('product_classifications').document(result['product_id'])
    doc_ref.set(result)

    # Write to BigQuery for analytics
    table_id = 'my-project.ml_analytics.image_classifications'
    errors = bq_client.insert_rows_json(table_id, [result])

    if errors:
        print(f'BigQuery insert errors: {errors}')
    else:
        print(f'Successfully classified: {result["product_id"]} as {result["primary_category"]}')

    return result
```

**Training AutoML Vision model**:
```python
from google.cloud import aiplatform

aiplatform.init(project='my-project', location='us-central1')

# Create dataset
dataset = aiplatform.ImageDataset.create(
    display_name='product_categories',
    gcs_source='gs://my-bucket/training_images/metadata.csv',
    import_schema_uri=aiplatform.schema.dataset.ioformat.image.single_label_classification
)

# Train AutoML Vision model
job = aiplatform.AutoMLImageTrainingJob(
    display_name='product_classifier',
    prediction_type='classification',
    model_type='CLOUD',
    multi_label=False
)

model = job.run(
    dataset=dataset,
    model_display_name='product_category_classifier',
    training_fraction_split=0.8,
    validation_fraction_split=0.1,
    test_fraction_split=0.1,
    budget_milli_node_hours=8000  # 8 hours
)

# Deploy model
endpoint = model.deploy(
    deployed_model_display_name='product_classifier_v1',
    machine_type='n1-standard-4',
    min_replica_count=1,
    max_replica_count=5
)
```

**Key Decisions**:
- Use Vertex AI AutoML Vision for image classification without custom code
- Use Cloud Functions for event-driven processing
- Store metadata in Firestore for low-latency application access
- Store analytics in BigQuery for reporting

### Scenario 6: Sentiment Analysis Pipeline for Customer Reviews

**Requirements**: Analyze sentiment of customer reviews in real-time for product insights.

**Solution Architecture**:
```
Application → Pub/Sub (streaming) → Dataflow (enrichment) →
Natural Language API (sentiment) → BigQuery (storage/analysis)
```

**Complete Implementation**:
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions
from google.cloud import language_v1
import json

class SentimentAnalysisFn(beam.DoFn):
    def setup(self):
        self.client = language_v1.LanguageServiceClient()

    def process(self, review):
        # Analyze sentiment
        document = language_v1.Document(
            content=review['review_text'],
            type_=language_v1.Document.Type.PLAIN_TEXT
        )

        sentiment = self.client.analyze_sentiment(request={'document': document}).document_sentiment
        entities = self.client.analyze_entities(request={'document': document}).entities

        # Enrich review with sentiment
        enriched_review = {
            'review_id': review['review_id'],
            'product_id': review['product_id'],
            'customer_id': review['customer_id'],
            'review_text': review['review_text'],
            'sentiment_score': sentiment.score,  # -1 to 1
            'sentiment_magnitude': sentiment.magnitude,  # 0+ (strength)
            'sentiment_label': self._get_sentiment_label(sentiment.score),
            'entities': [{'name': e.name, 'type': e.type_.name} for e in entities[:5]],
            'processed_at': beam.window.TimestampedValue.now()
        }

        yield enriched_review

    def _get_sentiment_label(self, score):
        if score >= 0.25:
            return 'POSITIVE'
        elif score <= -0.25:
            return 'NEGATIVE'
        else:
            return 'NEUTRAL'

def run_sentiment_pipeline():
    options = PipelineOptions(
        streaming=True,
        project='my-project',
        region='us-central1'
    )

    with beam.Pipeline(options=options) as p:
        enriched_reviews = (
            p
            | 'ReadPubSub' >> beam.io.ReadFromPubSub(
                subscription='projects/my-project/subscriptions/customer-reviews'
            )
            | 'ParseJSON' >> beam.Map(lambda x: json.loads(x.decode('utf-8')))
            | 'AnalyzeSentiment' >> beam.ParDo(SentimentAnalysisFn())
            | 'WriteToBigQuery' >> beam.io.WriteToBigQuery(
                table='my-project:reviews.sentiment_analysis',
                schema='review_id:STRING,product_id:STRING,customer_id:STRING,review_text:STRING,sentiment_score:FLOAT,sentiment_magnitude:FLOAT,sentiment_label:STRING,entities:STRING,processed_at:TIMESTAMP',
                write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND
            )
        )

run_sentiment_pipeline()
```

**Analysis queries**:
```sql
-- Product sentiment dashboard
CREATE OR REPLACE VIEW `project.analytics.product_sentiment_summary` AS
SELECT
  product_id,
  COUNT(*) AS total_reviews,
  COUNTIF(sentiment_label = 'POSITIVE') AS positive_count,
  COUNTIF(sentiment_label = 'NEGATIVE') AS negative_count,
  COUNTIF(sentiment_label = 'NEUTRAL') AS neutral_count,
  AVG(sentiment_score) AS avg_sentiment_score,
  AVG(sentiment_magnitude) AS avg_sentiment_magnitude,
  -- Overall sentiment
  CASE
    WHEN AVG(sentiment_score) >= 0.25 THEN 'POSITIVE'
    WHEN AVG(sentiment_score) <= -0.25 THEN 'NEGATIVE'
    ELSE 'NEUTRAL'
  END AS overall_sentiment
FROM `project.reviews.sentiment_analysis`
WHERE processed_at >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY product_id;
```

**Key Decisions**:
- Use Natural Language API for pre-trained sentiment analysis
- Use Dataflow for stream processing at scale
- Store enriched data in BigQuery for analytics and dashboards
- Real-time processing enables immediate product insights

## Exam Tips for Professional Data Engineer

### Key Decision Framework

**When to use BigQuery ML vs Vertex AI**:

| Factor | Use BigQuery ML | Use Vertex AI |
|--------|----------------|---------------|
| **Data location** | Data in BigQuery | Data in GCS or distributed |
| **Model complexity** | Linear/logistic regression, DNN, K-means, ARIMA | Custom TensorFlow/PyTorch, complex architectures |
| **Latency requirements** | Batch predictions (minutes) | Real-time predictions (<100ms) |
| **Team skills** | SQL proficiency | Python/ML engineering expertise |
| **Deployment** | In-database predictions | Microservices, endpoints |
| **Integration** | Existing BigQuery workflows | Kubernetes, microservices |
| **Cost model** | Pay per query (bytes scanned) | Pay for compute (nodes, accelerators) |

**Choosing model types**:
- **Classification** (binary/multiclass): LOGISTIC_REG, DNN_CLASSIFIER, BOOSTED_TREE_CLASSIFIER
- **Regression** (numeric prediction): LINEAR_REG, DNN_REGRESSOR, BOOSTED_TREE_REGRESSOR
- **Clustering** (grouping): KMEANS
- **Recommendation**: MATRIX_FACTORIZATION
- **Time series**: ARIMA_PLUS
- **Auto selection**: AUTOML_CLASSIFIER, AUTOML_REGRESSOR

**Serving pattern decisions**:
- **Batch** (millions/day): BigQuery ML predictions, Vertex AI batch prediction
- **Real-time** (<100ms): Vertex AI endpoints with auto-scaling
- **Near real-time** (seconds): Dataflow + BigQuery ML or Vertex AI
- **Micro-batch** (minutes): Scheduled BigQuery queries

### Common Exam Question Patterns

**Pattern 1**: "Company needs to predict customer churn using data in BigQuery"
- **Answer**: BigQuery ML with LOGISTIC_REG or BOOSTED_TREE_CLASSIFIER
- **Why**: Data already in BigQuery, batch predictions sufficient, SQL-based

**Pattern 2**: "Real-time fraud detection with <200ms latency requirement"
- **Answer**: Vertex AI endpoint with Dataflow for feature engineering
- **Why**: Low latency requirement, real-time serving needed

**Pattern 3**: "Forecast sales for 50,000 products over next 30 days"
- **Answer**: BigQuery ML ARIMA_PLUS with time_series_id_col
- **Why**: Multiple time series, batch forecasting, efficient for many series

**Pattern 4**: "Analyze sentiment of customer reviews in real-time"
- **Answer**: Pub/Sub → Dataflow → Natural Language API → BigQuery
- **Why**: Pre-trained NLP model, streaming data, scale to millions

**Pattern 5**: "Classify product images uploaded by users"
- **Answer**: Cloud Storage → Cloud Functions → Vertex AI AutoML Vision
- **Why**: Image classification, event-driven, no ML expertise needed

**Pattern 6**: "Product recommendations for millions of users"
- **Answer**: BigQuery ML MATRIX_FACTORIZATION with batch pre-computation
- **Why**: Collaborative filtering, batch processing, serve from cache

**Pattern 7**: "Model performance degrading, need automated retraining"
- **Answer**: Cloud Monitoring alerts → Pub/Sub → Cloud Functions → Vertex AI Pipelines
- **Why**: Monitoring-driven automation, retraining workflow orchestration

**Pattern 8**: "A/B test two model versions"
- **Answer**: Vertex AI endpoint with traffic splitting
- **Why**: Built-in traffic management, gradual rollout capability

### Critical Exam Knowledge

**BigQuery ML Essentials**:
- `CREATE MODEL` with OPTIONS (model_type, input_label_cols, etc.)
- `ML.EVALUATE` for model assessment (precision, recall, F1, AUC)
- `ML.PREDICT` for generating predictions
- `ML.FEATURE_IMPORTANCE` for understanding models
- `ML.EXPLAIN_PREDICT` for individual prediction explanations
- `ML.FORECAST` for time series predictions
- `ML.DETECT_ANOMALIES` for anomaly detection
- `ML.RECOMMEND` for recommendations
- Export/import models with `EXPORT MODEL` and `CREATE MODEL` with model_path

**Vertex AI Essentials**:
- AutoML for automatic model selection (TabularDataset, ImageDataset)
- Custom training with CustomTrainingJob (TensorFlow, PyTorch, Scikit-learn)
- Model deployment to Endpoint (online serving)
- Batch prediction for large-scale inference
- Hyperparameter tuning with HyperparameterTuningJobSpec
- Feature Store for centralized feature management
- Model monitoring for drift detection
- Vertex AI Pipelines (Kubeflow) for ML workflows

**Feature Engineering Techniques**:
- Temporal features (EXTRACT, DATE functions)
- Window functions for aggregation (AVG OVER, LAG, LEAD)
- ML.FEATURE_CROSS for interaction features
- ML.BUCKETIZE for discretization
- ML.QUANTILE_BUCKETIZE for equal-sized bins
- ML.NGRAMS for text features
- One-hot encoding (automatic for STRING columns)
- Normalization (automatic or ML.STANDARD_SCALER)

**MLOps Best Practices**:
- Model versioning with labels and version_aliases
- Monitoring: data drift, prediction drift, training-serving skew
- Continuous training triggers: scheduled, performance-based, drift-based
- A/B testing with traffic splitting
- Canary deployments (gradual rollout)
- Rollback strategies (quick revert to previous version)

**Data Pipeline Integration**:
- BigQuery → BigQuery ML (simplest for batch)
- Pub/Sub → Dataflow → Vertex AI Endpoint (real-time)
- Dataflow → Feature Store → Vertex AI (feature management)
- Cloud Composer → BigQuery ML (orchestrated batch ML)
- Vertex AI Pipelines (end-to-end ML workflow automation)

**Cost Optimization**:
- BigQuery ML: Optimize query costs (partitioned tables, clustered columns)
- Vertex AI training: Right-size machine types, use preemptible instances
- Vertex AI serving: Auto-scaling, batch predictions for non-real-time use cases
- Pre-trained APIs: Batch API calls, cache results
- Feature Store: Optimize online/offline serving based on use case

### Hands-on Practice Checklist

- [ ] Create BigQuery ML models for all major types (LINEAR_REG, LOGISTIC_REG, DNN, KMEANS, ARIMA_PLUS)
- [ ] Train and deploy Vertex AI AutoML model (Tabular or Vision)
- [ ] Build custom training job with TensorFlow on Vertex AI
- [ ] Deploy model to Vertex AI endpoint and make predictions
- [ ] Create Vertex AI Pipeline with multiple components
- [ ] Set up model monitoring for drift detection
- [ ] Implement A/B testing with traffic splitting
- [ ] Build Dataflow pipeline with ML.PREDICT or Vertex AI endpoint calls
- [ ] Create Cloud Composer DAG for ML retraining
- [ ] Use Natural Language API or Vision API in data pipeline
- [ ] Implement feature engineering with window functions and ML functions
- [ ] Export BigQuery ML model and import into Vertex AI

### Common Pitfalls to Avoid

1. **Using Vertex AI when BigQuery ML is sufficient** - If data is in BigQuery and batch predictions work, use BigQuery ML
2. **Not considering latency requirements** - Real-time (<100ms) requires Vertex AI endpoints, not BigQuery ML
3. **Forgetting auto_class_weights for imbalanced data** - Always use for classification with class imbalance
4. **Not using time_series_id_col for multiple time series** - Efficient way to forecast many series at once
5. **Ignoring feature engineering** - Temporal features, lag features, and window aggregations are critical
6. **Not monitoring for drift** - Production models need data drift and prediction drift monitoring
7. **Using wrong model type** - BOOSTED_TREE often better than LINEAR for tabular data
8. **Not versioning models** - Always version models with labels and metadata
9. **Deploying without evaluation** - Always evaluate on held-out test set before deployment
10. **Overcomplicating architecture** - Choose simplest solution that meets requirements

### Quick Reference Commands

**BigQuery ML**:
```sql
-- Create model
CREATE MODEL dataset.model OPTIONS(model_type='LOGISTIC_REG', input_label_cols=['label']) AS SELECT * FROM dataset.training;

-- Evaluate
SELECT * FROM ML.EVALUATE(MODEL dataset.model);

-- Predict
SELECT * FROM ML.PREDICT(MODEL dataset.model, (SELECT * FROM dataset.input));

-- Feature importance
SELECT * FROM ML.FEATURE_IMPORTANCE(MODEL dataset.model);

-- Export
EXPORT MODEL dataset.model OPTIONS(uri='gs://bucket/model/*');
```

**Vertex AI Python**:
```python
# Train AutoML
from google.cloud import aiplatform
aiplatform.init(project='project', location='region')
dataset = aiplatform.TabularDataset.create(display_name='ds', bq_source='bq://project.dataset.table')
job = aiplatform.AutoMLTabularTrainingJob(display_name='job', optimization_prediction_type='classification')
model = job.run(dataset=dataset, target_column='label')

# Deploy
endpoint = model.deploy(machine_type='n1-standard-4', min_replica_count=1)

# Predict
predictions = endpoint.predict(instances=[{'feature1': value1, 'feature2': value2}])
```

**gcloud commands**:
```bash
# Create training job
gcloud ai custom-jobs create --region=us-central1 --display-name=job --worker-pool-spec=machine-type=n1-standard-8,replica-count=1,python-package-uris=gs://bucket/trainer.tar.gz,python-module=trainer.task

# Deploy model
gcloud ai endpoints create --region=us-central1 --display-name=endpoint
gcloud ai endpoints deploy-model ENDPOINT_ID --region=us-central1 --model=MODEL_ID --display-name=deployed --machine-type=n1-standard-4

# Batch prediction
gcloud ai batch-prediction-jobs create --region=us-central1 --model=MODEL_ID --job-display-name=batch-job --gcs-source-uri=gs://bucket/input.csv --gcs-destination-output-uri-prefix=gs://bucket/output/
```

## Additional Resources

### Official Documentation
- [BigQuery ML Documentation](https://cloud.google.com/bigquery-ml/docs) - Complete BigQuery ML reference
- [Vertex AI Documentation](https://cloud.google.com/vertex-ai/docs) - Unified ML platform guide
- [ML on GCP Best Practices](https://cloud.google.com/architecture/ml-on-gcp-best-practices) - Architecture patterns
- [MLOps Maturity Model](https://cloud.google.com/architecture/mlops-continuous-delivery-and-automation-pipelines-in-machine-learning) - Production ML workflows
- [BigQuery ML Tutorial](https://cloud.google.com/bigquery-ml/docs/tutorials) - Step-by-step guides
- [Vertex AI Samples](https://github.com/GoogleCloudPlatform/vertex-ai-samples) - Code examples

### Key Reference Pages
- [BigQuery ML Model Types](https://cloud.google.com/bigquery-ml/docs/reference/standard-sql/bigqueryml-syntax-create) - All model types and options
- [ML Functions Reference](https://cloud.google.com/bigquery-ml/docs/reference/standard-sql/bigqueryml-syntax-predict) - Complete SQL ML function list
- [Vertex AI Python Client](https://cloud.google.com/python/docs/reference/aiplatform/latest) - Python SDK reference
- [Pre-trained APIs](https://cloud.google.com/products/ai) - Vision, NLP, Translation, Speech APIs

### Practice Labs
- [BigQuery ML Qwiklabs](https://www.cloudskillsboost.google/catalog?keywords=bigquery%20ml) - Hands-on labs
- [Vertex AI Qwiklabs](https://www.cloudskillsboost.google/catalog?keywords=vertex%20ai) - Practical exercises
- [Data Engineering Quest](https://www.cloudskillsboost.google/quests/25) - Professional Data Engineer path

### Exam Preparation
- [Professional Data Engineer Exam Guide](https://cloud.google.com/certification/data-engineer) - Official exam topics
- [Sample Questions](https://cloud.google.com/certification/sample-questions/data-engineer) - Practice questions
- [Certification Forum](https://www.googlecloudcommunity.com/gc/Certification/bd-p/cloud-certification) - Community discussions

---

**Document Statistics**:
- Original length: 238 lines
- Enhanced length: ~3,400 lines
- Coverage: Comprehensive BigQuery ML (all model types), Vertex AI integration, complete ML scenarios, MLOps patterns, and exam-focused tips
- Code examples: 50+ complete implementations
- Scenarios: 6 end-to-end data engineering ML scenarios
- Focus: Professional Data Engineer exam requirements for ML operations
