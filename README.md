# Big Data Architecture for E-Commerce Analytics & Customer Satisfaction Prediction

> Group project for **Big Data Processing** (Group 3, Class LB01), BINUS University.
> Built on the Brazilian Olist e-commerce dataset.

## Overview

E-commerce platforms generate massive transaction data, but customer dissatisfaction often goes undetected because it isn't always voiced through formal complaints. This project builds a big data pipeline on the Brazilian Olist dataset (~96K orders) using HDFS and Apache Spark, then trains classification models to predict whether a customer will be satisfied with an order.

**Result:** the CatBoost classifier reached **81.98% accuracy**, outperforming Random Forest (72.62%). Delivery time and late delivery were the strongest drivers of satisfaction, far ahead of price or payment value.

## My Role

This was a group project. My contributions:

- **Business Problem Definition:** framed the e-commerce satisfaction problem and the insights worth analyzing
- **Big Data Justification:** explained why the dataset's volume, variety, and velocity call for a distributed approach over traditional systems
- **Machine Learning / Advanced Analytics:** developed and compared CatBoost and Random Forest classifiers, handled class imbalance, and evaluated with F1-score, AUC-ROC, and cross-validation

## Tech Stack

| Layer | Tools |
|---|---|
| Storage | Hadoop HDFS (single-node, Cloudera VM) |
| Processing | Apache Spark (PySpark), Pandas |
| Machine Learning | CatBoost, Random Forest (Spark MLlib), ADASYN, Gaussian noise augmentation |
| Experiments | Google Colab |
| Visualization | Power BI |

## Architecture

Batch-oriented layered architecture: **CSV data sources → ETL ingestion → HDFS (data lake) → Spark processing & ML → Power BI dashboard.** Batch processing was chosen because the Olist data is historical and does not require real-time analysis.

## Dataset

The raw CSV files are **not included** in this repository because of their size. Download them from Kaggle:

**[Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)**

The project uses all 9 tables:

`olist_orders_dataset`, `olist_customers_dataset`, `olist_order_items_dataset`, `olist_order_reviews_dataset`, `olist_order_payments_dataset`, `olist_products_dataset`, `olist_geolocation_dataset`, `olist_sellers_dataset`, `product_category_name_translation`


## Methodology

1. **Data load:** read the 9 raw tables into Spark DataFrames
2. **Cleaning:** filter to delivered orders, handle missing values, cap outliers with the IQR method
3. **Transformation:** cast timestamps, standardize text, cast numeric types
4. **Aggregation:** aggregate items, payments, and reviews to one row per `order_id` to avoid row explosion on joins
5. **Feature engineering:** `is_satisfied` (target: review score ≥ 4), `delivery_time`, `is_late`, and more
6. **Modeling:** CatBoost vs Random Forest, class weighting for imbalance (79% satisfied / 21% dissatisfied), stratified train/test split and 10-fold cross-validation
7. **Visualization:** 3-page interactive Power BI dashboard

Final modeling dataset: **95,830 orders** and **11 features**.

## Results

### Model comparison (test set)

| Metric | CatBoost | Random Forest |
|---|---|---|
| Accuracy | **0.8198** | 0.7262 |
| F1-Score | **0.7761** | 0.7389 |
| Precision | **0.8103** | 0.7577 |
| Recall | **0.8198** | 0.7262 |
| AUC-ROC | **0.6979** | 0.6843 |

### Key insights

- **Delivery is what matters most.** Top CatBoost features: `delivery_time` (31.11), `is_late` (27.79), `total_items` (16.07), `customer_state` (6.75), `total_freight` (5.38).
- **Late delivery hurts satisfaction badly:** ~82.85% satisfaction for on-time orders vs ~34.68% for late orders.
- **Regional gaps:** states such as RR, MA, AL, PA, and SE show lower satisfaction and longer delivery times (around 20–30 days).
- **Order complexity:** single-item orders reach ~81% satisfaction, while orders with more than five items drop to ~55%.


## Limitations

- Accuracy plateaued around 82% due to class imbalance and overlapping classes (satisfied and dissatisfied customers look similar in the operational data), so AUC-ROC stays modest at ~0.70.
- The pipeline runs on a single-node Hadoop VM and is batch-only, so it cannot support real-time monitoring.

## Future Work

- Real-time streaming with Apache Kafka and Spark Structured Streaming
- Other models for imbalanced, overlapping data (SVM with RBF kernel, TabPFN, neural networks) and richer customer-behavior features
- Multi-node Hadoop cluster for scalability

## Team

Group 3, Class LB01, BINUS University. This was a collaborative project with 11 members.
