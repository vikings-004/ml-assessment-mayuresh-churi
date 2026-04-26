## B1. Problem Formulation

### (a) Machine Learning Formulation

The goal here is to figure out which promotion works best for each store in a given month, with the objective of maximising the number of items sold.

- **Target Variable:**  
  items_sold (number of items sold per store per month)

- **Input Features:**  
  - Store attributes: store_size, location_type, competition_density  
  - Promotion type: Flat Discount, BOGO, Free Gift, Category Offer, Loyalty Points  
  - Temporal features: month, seasonality, is_weekend, is_festival  
  - Store performance indicators: historical sales, footfall  
  - External factors: customer demographics, local competition  

- **Problem Type:**  
  Supervised regression  

**Justification:**  
We are trying to predict a numerical value (items_sold), so this naturally becomes a regression problem. Once we can estimate expected sales for each promotion, we can compare them and pick the one that gives the highest output. So effectively, the model is helping us make a better business decision.

---

### (b) Why Items Sold is a Better Target than Revenue

Using items_sold is more reliable than total revenue because revenue can be heavily influenced by pricing and discounting. For example, a deep discount might increase the number of items sold but reduce revenue per item, which can distort the true effectiveness of a promotion.

Items sold (volume) gives a cleaner signal of actual customer demand and how well a promotion drives purchases, without getting affected by price changes.

**Broader Principle:**  
The target variable should directly reflect the business objective and should not be influenced by the decision variable itself. If the target is affected by the action we are trying to optimise (like promotions affecting price), it can lead to biased or misleading results.

---

### (c) Alternative Modelling Strategy

Instead of using a single global model across all stores, a better approach would be to use a segmented or hierarchical modelling strategy.

For example:
- Build separate models for urban, semi-urban, and rural stores, or  
- Use a single model but include interaction effects between promotion type and store characteristics  

**Justification:**  
Different stores behave very differently. Customer preferences, purchasing power, and response to promotions vary by location. A single global model may average out these differences and miss important patterns. Segmenting the modelling approach helps capture these variations and leads to more accurate and actionable recommendations.

---

## B2. Data and EDA Strategy

### (a) Data Joining and Dataset Design

We would start by joining the four tables using appropriate keys:

- **transactions** (base table): store_id, transaction_date, items_sold  
- **store attributes**: joined on store_id  
- **promotion details**: joined using promotion_id or promotion_type along with date  
- **calendar**: joined on transaction_date  

After joining, the dataset should be aggregated to a consistent level.

- **Grain of the dataset:**  
  One row per *store per month*

- **Aggregations performed:**  
  - Total items_sold → monthly sales volume  
  - Number of transactions → proxy for footfall  
  - Average basket size (if available)  
  - Promotion applied in that month  
  - Calendar features (e.g., number of weekends or festival days in that month)  

This ensures that both features and target are aligned with the business decision, which is made at a store-month level.

---

### (b) Exploratory Data Analysis (EDA)

Before building any model, I would focus on understanding patterns in the data:

1. **Promotion vs Items Sold (Bar Chart / Box Plot):**  
   Compare average sales across promotion types.  
   → Helps identify which promotions are generally effective.

2. **Time Series Analysis (Line Chart):**  
   Plot sales over time for selected stores.  
   → Helps detect trends, seasonality, and unusual spikes.

3. **Location-wise Comparison:**  
   Compare performance across urban, semi-urban, and rural stores.  
   → Helps identify differences in customer behaviour and supports segmentation.

4. **Correlation Analysis (Heatmap):**  
   Check relationships between variables like competition_density, footfall, and sales.  
   → Helps identify strong predictors and potential redundancy.

5. **Distribution of Items Sold (Histogram):**  
   Check for skewness and outliers.  
   → May suggest transformations if the data is highly skewed.

These steps help in forming hypotheses, validating assumptions, and guiding feature engineering decisions.

---

### (c) Promotion Imbalance (80% No Promotion)

If 80% of transactions have no promotion, the model may end up learning mostly baseline behaviour and not properly capture the impact of promotions.

This can lead to:
- Underestimating promotion effectiveness  
- Poor differentiation between promotion types  

**Steps to address this:**
- Ensure better representation of promotion data (e.g., weighting or sampling)  
- Add a binary feature indicating promotion vs no promotion  
- Consider modelling baseline demand separately from incremental promotion impact  
- Evaluate model performance separately for promotion and non-promotion cases  

This helps ensure the model actually learns the effect of promotions.

---

## B3. Model Evaluation and Deployment

### (a) Train-Test Split and Evaluation Metrics

Since the data is time-based, we should use a **temporal split**:

- First ~2.5 years → training  
- Last ~6 months → testing  

**Why not a random split?**  
A random split would mix past and future data, causing data leakage. The model could learn from future patterns and appear more accurate than it actually is.

**Evaluation Metrics:**
- **RMSE:** Penalises large errors more — useful when big mistakes are costly  
- **MAE:** Easy to interpret — average error in number of items  
- **R² (optional):** Shows variance explained, but less directly useful for business decisions  

**Interpretation:**  
Lower RMSE and MAE mean better predictions, which directly translates to better promotion decisions and inventory planning.

---

### (b) Explaining Different Recommendations (Feature Importance)

The model gives different recommendations for December and March because the underlying conditions are different.

Using feature importance or interpretability tools:

- **December:** High seasonal demand (festivals, holidays), so loyalty-based promotions work better  
- **March:** Lower demand period, so discounts are more effective in driving sales  

**How to explain this to business teams:**
- Highlight key drivers like seasonality and store characteristics  
- Show predicted outcomes for different promotions  
- Keep explanation simple:  
  “In December, demand is already high, so loyalty incentives maximise value. In March, demand is lower, so discounts help boost sales.”

This builds trust and makes the model easier to accept.

---

### (c) Deployment and Monitoring Strategy

**1. Model Saving:**  
Save the full pipeline (preprocessing + model) using joblib or pickle.

**2. Monthly Data Preparation:**  
At the start of each month:
- Gather updated store data, calendar data, and promotion options  
- Apply the same preprocessing pipeline  
- Generate predictions for each promotion  

**3. Recommendation Step:**  
For each store, select the promotion with the highest predicted sales.

**4. Monitoring:**  
- Track RMSE and MAE over time  
- Compare predicted vs actual sales  
- Monitor data drift (changes in input distributions)  
- Monitor concept drift (changes in relationships, e.g., promotions losing effectiveness)

**5. Retraining:**  
- Retrain periodically (e.g., quarterly) or when performance drops  
- Use latest data to keep the model relevant  

This ensures the model continues to perform well in a dynamic business environment.