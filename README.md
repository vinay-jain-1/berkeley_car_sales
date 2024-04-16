# berkeley_car_sales
UC Berkeley - Professional Data Science and Machine Learning - Assignment 10


# Goal of this exercise:
The goal is to understand what factors make a used car more or less expensive.  Expected outcome is a clear set of recommendations to the client -- a used car dealership -- as to what consumers value in a used car.

# Link to Jupyter notebook:
 https://github.com/vinay-jain-1/berkeley_car_sales/blob/main/prompt_II.ipynb

# Approach:
### CRISP approach
CRISP DM framework was used to approach how this exercise should be conducted. 

### Business understanding
Business understanding was gained by reviewing the raw data and problem statement. Documented all the various sections as defined in the CRISP manual (https://github.com/vinay-jain-1/berkeley_car_sales/blob/main/docs/crisp-dm-manual.pdf). 

### Data understanding
The data was then analyzed to get a good understanding of what is available. The following steps were followed to analyze the data:
1. Read CSV data into a Dataframe.
2. Understand all the columns involved, how many are non-null, which ones are categorical or numerical.
3. For categorical columns, identify if any of them contain a large number of unique values. Those columns will special consideration when defining a model.
4. For numerical values, identify what kind of outliers exist and what values should be considered for modeling.

These steps resulted in identifying the upper and lower fence values to use for numerical values. 
State and Region values seemed to exhibit similar outcome. So if modeling constraints dictated dropping one of them, we could choose to do so.

### Data Preparation
Once the data analysis was completed, data preparation steps were taken to identify how each of the categorical values should be encoded and if any new derived attributes should be created. The following attributes were created anew as part of this:
1. Age (derived from year)
2. age x odo (derived as a product of age and odometer reading).

Data preparation and analysis resulted in the following feature engineering summary:

| Feature | Transformation | Rationale |
|----------|----------|----------|
| age (derived from year)   | Standardization + Polynomial Features (degree 2 or 3) + Regularization    | Handles non-linear relationship between age and price, prevents overfitting.   |
| odometer    | Standardization + Polynomial Features (degree 2 or 3)    | Addresses scaling and potential non-linearity of mileage.    |
| age_x_odo (interaction_only term derived feature from age and odometer)    | Standardization + Polynomial Features (degree 5)   | Captures the interaction effect of age and mileage on price. |
| model    | Target Encoding + Regularization    | Captures model's impact on price while considering potential overfitting and data imbalance.    |
| All other categorical features    | One-hot encoding    | Represents distinct transmission types effectively and aligns with other categorical encodings.|

### Modeling
**Implementation Steps:**
- Define column transformer pipelines
- Prepare data: Split the data into development and testing sets.
- Model Training and Evaluation: Train and compare different models with the engineered features using cross-validation.

### Summary of findings

#### The model definition and execution was successfully completed. Following are the outcomes:
1. I believe I met the key business ask. I was able to successfully identify the importance of various features in identifying the price of a used car. Details about the key findings are in the next section.
2. The quality of the model was defined using a few metrics: R², RMSE and MSE.
    - **R² score: 0.83**: An R² value of 0.83 for predicting the price of used cars is generally considered a very good score. It indicates that my model explains 83% of the variance in car prices, which is a strong performance for this type of problem.
        - The high R² suggests that your model has successfully captured the relationships between:
            - the features: 'year', 'manufacturer', 'model', 'condition', 'cylinders', 'fuel', 'odometer', 'title_status', 'transmission', 'type', 'paint_color', 'state', 'region', and the age-odometer interaction AND
            - the target variable 'price'.
        - It implies that the feature engineering efforts and model selection were effective in representing the factors that influence used car prices.
    - **RMSE=5207.59**: Considering that the used car prices were in the range of $0 (minimum) to $57K (max) -- after taking out the outliers, an RMSE of 5207 would indicate the predictions are within $5.2K of the actual prices. So while its not particularly bad, the model has room for improvement.
    - **MSE=2.711907e+07**: Similar considerations as RMSE.
    - **Linear Regression model** did slightly better than Ridge based model (with winning alpha of 0.01). Linear had RMSE of 5207.597766, whereas Ridge had RMSE of 5215.112490.
    - Here is how the two models did (Model Performance Comparison):

#### Model Performance Comparison

| Model             | Test MSE          | Test RMSE        | Test R²         |
|-------------------|:-----------------:|:-----------------:|:----------------:|
| Linear Regression | 2.711907e+07     | 5207.597766     | <font color="green">0.826709</font> |
| Ridge Regression  | 2.719740e+07     | 5215.112490     | 0.826208        |


#### Importance of features along with their top 3 values (for categorical columns) are as follows:

| Feature           | Highest Importance | Rank 1 value | Rank 2 value | Rank 3 value |
|-------------------|:-------------------:|:-------------:|:-------------:|:-------------:|
| age_x_odometer     | 115,014           | --            | --            | --            |
| odometer          | 103,266           | --            | --            | --            |
| year (age)        |  23,775           | --            | --            | --            |
| manufacturer      |  22,770           | aston-martin  | ferrari      | tesla        |
| cylinders         |   8,392           | 12            | 3            | 8            | 
| fuel              |   6,108           | diesel        | electric     | hybrid       |
| region            |   5,993           | southwest MS  | southwest VA | Gainesville  |
| state             |   3,906           | LA            | AR            | OH           |
| title_status      |   3,142           | lien          | clean        | missing      |
| type              |   2,939           | offroad       | bus          | truck        |
| condition         |   1,789           | fair          | salvage      | new          |
| transmission      |   1,422           | other         | manual       | automatic    |
| paint_color       |    594           | purple        | brown        | green        |
| model             |    0.520          | --            | --            | --            |
