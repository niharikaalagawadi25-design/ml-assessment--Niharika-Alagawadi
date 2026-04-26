
# B1. Problem Formulation

## (a) Machine Learning Formulation

This problem can be framed as a **supervised machine learning regression problem**.

### Target Variable

* `items_sold`

### Candidate Input Features

* Store ID
* Store size
* Location type (urban / semi-urban / rural)
* Monthly footfall
* Competition density
* Customer demographics
* Promotion type
* Month / season
* Weekend / festival indicators
* Historical sales trends
* Previous promotion results

### Problem Type Justification

The business wants to predict the number of items likely to be sold under different promotion choices. Since the output is a continuous numeric variable, regression is the correct ML approach.

---

## (b) Why Items Sold is Better than Revenue

Revenue can be influenced by many external factors:

* Price changes
* Discounts reducing unit price
* Product mix differences
* Inflation
* Premium product sales

A promotion may successfully increase customer purchases, but revenue may still look lower because discounted prices were used.

`items_sold` directly measures customer response and demand generation, making it a cleaner target for promotion effectiveness.

### Broader Principle

The target variable must align with the real business objective. Choosing the wrong target leads to models that optimise misleading outcomes.

---

## (c) Better Alternative to One Global Model

Use a **segmented or hierarchical modelling strategy**.

### Example Segments

* Urban stores
* Semi-urban stores
* Rural stores

or clusters based on:

* Similar footfall
* Competition density
* Customer demographics

### Justification

Different store types react differently to promotions. A segmented model captures local behaviour better than one global model and improves recommendation accuracy.

---

# B2. Data and EDA Strategy

## (a) Joining Raw Tables

### Source Tables

* Transactions
* Store attributes
* Promotion details
* Calendar table

### Join Keys

* `store_id` → join transactions with store attributes
* `promotion_id` or promotion type → join promotion details
* `transaction_date` → join calendar table

### Final Dataset Grain

**One row = one store for one month**

Example:

| store_id | month | promotion | items_sold |
| -------- | ----- | --------- | ---------- |

### Aggregations

From transaction-level data:

* Total items sold per month
* Total footfall
* Average basket size
* Number of transactions
* Promotion used that month
* Festival/weekend counts
* Lagged sales features (previous month sales)

---

## (b) EDA Before Modelling

## 1. Promotion Performance Comparison

Bar chart of average items sold by promotion type.

### Look For:

Which promotions generally perform best.

### Impact:

Useful baseline and promotion ranking.

---

## 2. Sales by Location Type

Boxplot of items sold across urban / semi-urban / rural stores.

### Look For:

Location-based differences.

### Impact:

Need interaction features or segmented models.

---

## 3. Monthly Trend / Seasonality

Line chart of monthly sales over time.

### Look For:

Seasonal peaks such as festivals or year-end.

### Impact:

Create month, quarter, festival features.

---

## 4. Correlation Heatmap

Numerical feature correlations.

### Look For:

Drivers like footfall, competition, store size.

### Impact:

Feature selection and multicollinearity checks.

---

## 5. Outlier Detection

Boxplots for items sold and footfall.

### Look For:

Abnormal stores or data quality issues.

### Impact:

Winsorization, log transform, anomaly handling.

---

## (c) 80% No Promotion Imbalance

The model may learn that "no promotion" is safest and underpredict gains from actual promotions.

### Steps to Address

* Stratified sampling by promotion category
* Weight promotional rows more heavily
* Oversample rare promotion cases
* Evaluate performance separately for promoted months
* Use uplift modelling if possible

---

# B3. Model Evaluation and Deployment

## (a) Train-Test Split and Metrics

### Correct Split

Use a **time-based split**.

Example:

* Train = first 2.5 years
* Test = last 6 months

Or rolling time-series validation.

### Why Random Split is Wrong

Random split leaks future information into training data and gives unrealistic performance estimates.

### Metrics

## RMSE

Penalises large errors strongly.

Use when big misses are costly.

## MAE

Average absolute error.

Easy for business teams to understand.

## MAPE (optional)

Percentage error.

Useful for comparing stores of different sizes.

## R²

Explains variance captured by model.

Higher means stronger fit.

---

## (b) Explaining Different Recommendations

Use feature importance and explainability tools such as:

* Global feature importance
* SHAP values
* Partial dependence plots

### Example for Store 12

December recommendation = Loyalty Points Bonus because:

* Festival season
* Higher expected footfall
* Gift shopping behaviour
* Higher customer retention value

March recommendation = Flat Discount because:

* Lower demand season
* Price-sensitive buying behaviour
* Need traffic generation

### Communication Style

Provide simple charts showing which factors drove each recommendation.

---

## (c) Deployment Process

## 1. Save Model

Use:

* `joblib`
* `pickle`

Store preprocessing pipeline + trained model together.

## 2. Monthly Data Preparation

At start of each month:

* Collect latest store data
* Update footfall, competition, season, calendar flags
* Apply same preprocessing pipeline

## 3. Generate Recommendations

For each store:

* Score all five promotions
* Select promotion with highest predicted items sold

## 4. Deliver Output

Dashboard or CSV with:

* Store ID
* Recommended promotion
* Predicted uplift
* Confidence score

## 5. Monitoring

Track monthly:

* Prediction error vs actual sales
* Drift in footfall/customer mix
* Promotion response decline
* Store behaviour changes

## 6. Retraining Trigger

Retrain when:

* Errors rise consistently
* New promotions introduced
* Major market behaviour changes
* Seasonal patterns shift
