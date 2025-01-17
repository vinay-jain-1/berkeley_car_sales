UC Berkeley - Professional Data Science and Machine Learning - Assignment 11.1


# Goal of this exercise:
The goal is to understand what factors make a used car more or less expensive.  Expected outcome is a clear set of recommendations to the client -- a used car dealership -- as to what consumers value in a used car.

# Link to Jupyter notebook:
 https://github.com/vinay-jain-1/berkeley_car_sales/blob/main/prompt_II.ipynb


# CRISP-DM approach
CRISP-DM framework was used to approach how this exercise should be conducted. 

# Business understanding
Business understanding was gained by reviewing the raw data and problem statement. Documented all the various sections as defined in the CRISP manual (https://github.com/vinay-jain-1/berkeley_car_sales/blob/main/docs/crisp-dm-manual.pdf). 

# Data understanding
The data was then analyzed to get a good understanding of what is available. The following steps were followed to analyze the data:
1. Read CSV data into a Dataframe.
2. Understand all the columns involved, how many are non-null, which ones are categorical or numerical.
3. For categorical columns, identify if any of them contain a large number of unique values. Those columns will special consideration when defining a model.
4. For numerical values, identify what kind of outliers exist and what values should be considered for modeling.

Once all the data attributes were analyzed, the following observations were recorded.
### Data description observations summary:

1. Drop the ```id``` and ```VIN``` columns since they are meant to ID each vehicle and not useful for our analysis.
2. Retain most of the categorical attibutes (except ```size``` since it has a low count of non-null values).
3. Use the lower and upper fence values for numerical columns as calculated above.
4. Some of the non-integer attributes have quite a few unique values ('state' and 'manufacturer' for example). This could lead to a large explosion of width of data when techniques like one-hot encoder are applied.
5. Considering that 'model' attribute has thousands of unique values, it may be best to model it as a TargetEncoder instead of OneHotEncoder.
6. Update: Iterative development analysis showed that dropping ```drive``` helped improve the R^2 metric.

# Data Preparation
Once the data analysis was completed, data preparation steps were taken to identify how each of the categorical values should be encoded and if any new derived attributes should be created. The following attributes were created anew as part of this:
1. Age (derived from year)
2. age x odo (derived as a product of age and odometer reading).

Data preparation and analysis resulted in the following feature engineering summary:

| Feature | Transformation | Rationale |
|----------|----------|----------|
| age (derived from year)   | Polynomial Features (degree 6) + Standardization | Handles non-linear relationship between age and price, prevents overfitting.   |
| odometer    | Polynomial Features (degree 4) + Standardization  | Addresses scaling and potential non-linearity of mileage.    |
| age_x_odo (interaction_only term derived feature from age and odometer)    | Standardization + Polynomial Features (degree 5)   | Captures the interaction effect of age and mileage on price. |
| model    | Target Encoding   | Captures model's impact on price while considering potential overfitting and data imbalance.    |
| All other categorical features    | One-hot encoding    | Represents distinct transmission types effectively and aligns with other categorical encodings.|

# Modeling
**Implementation Steps:**
- Define column transformer pipelines
- Prepare data: Split the data into development and testing sets.
- Model Training and Evaluation: Train and compare different models with the engineered features using cross-validation.

## Column transformer pipelines setup
Columns were transformed as follows:
- **OneHot Encoded columns**: 'manufacturer', 'condition', 'cylinders', 'fuel', 'title_status', 'transmission', 'type', 'paint_color', 'state', 'region'
- **Numerical columns with Polynomial degree of 6 and standardized:** 'age', 'odometer'
- **Numerical columns with Polynomial degree of 4 and standardized**: age_x_odo
- **Target encoding**:'model'

![ColumnTransformer](https://github.com/vinay-jain-1/berkeley_car_sales/raw/main/images/ColumnTransformer.png)

## Model execution
- The model was executed using Linear regression and with Ridge regression with different values of alpha (0.05, 0.1, and 0.2). A cross validation was conducted using KFold with numnber of folds set to 5. GridSearchCV was used to execute different models. Results were compared using R², RMSE and MSE metrics. Finally importance of each of the features were derived to compile the recommendations for the used car dealership.

## The model definition and execution was successfully completed. Following are the outcomes:
1. I believe I met the key business ask. I was able to successfully identify the importance of various features in identifying the price of a used car. Details about the key findings are in the next section.
2. The quality of the model was defined using a few metrics: R², RMSE and MSE.
    - **R² score: 0.83**: An R² value of 0.83 for predicting the price of used cars is generally considered a very good score. It indicates that my model explains 83% of the variance in car prices, which is a strong performance for this type of problem.
        - The high R² suggests that the model has successfully captured the relationships between:
            - the features: 'year', 'manufacturer', 'model', 'condition', 'cylinders', 'fuel', 'odometer', 'title_status', 'transmission', 'type', 'paint_color', 'state', 'region', and the age-odometer interaction AND
            - the target variable 'price'.
        - It implies that the feature engineering efforts and model selection were effective in representing the factors that influence used car prices.
    - **RMSE=5207.59**: Considering that the used car prices were in the range of $0 (minimum) to $57K (max) -- after taking out the outliers, an RMSE of 5207 would indicate the predictions are within $5.2K of the actual prices (since RMSE is in the same units as the target variable- price). So while its not particularly bad, the model has room for improvement.
    - **MSE=2.711907e+07**: Similar considerations as RMSE but with a more heavier penalty for larger errors as it is a squared value metric. 
    - **Linear Regression model** did slightly better than Ridge based model (with winning alpha of 0.01). Linear had RMSE of 5207.597766, whereas Ridge had RMSE of 5215.112490.
    - Here is how the two models did (Model Performance Comparison):

## Model Performance Comparison

| Model             | Test MSE          | Test RMSE        | Test R²         |
|-------------------|:-----------------:|:-----------------:|:----------------:|
| Linear Regression | 2.711907e+07     | 5207.597766     | <font color="green">0.826709</font> |
| Ridge Regression (best alpha = 0.05) | 2.719740e+07     | 5215.112490     | 0.826208        |

![Model Comparison](https://github.com/vinay-jain-1/berkeley_car_sales/raw/main/images/model_comparison.png)


## Linear Regression model provided the final best outcome. 
However, Ridge was not far behind and it was so close that with some variation of data, Ridge could very well just be a tad bit ahead.

# Importance of features along with their top 3 values (for categorical columns) are as follows:

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


# Recommendations for the used car dealership

## Here are the key features that drive the price of the car listed in the order of ranking (rank 1 is listed first):

1. **Age along with Odometer reading**: This combined feature holds the highest importance. It suggests a complex relationship between age and mileage impacting car prices. For instance, high mileage might be less detrimental on a 15-year-old car compared to a 2-year-old car. Consider developing models that capture this interaction effect for better price prediction.
2. **Odometer reading**: As expected, odometer reading is a significant factor influencing car prices. Customers generally prefer cars with lower mileage, indicating less wear and tear. Focus on capturing accurate odometer readings and potentially consider its interaction with age.
3. **Year (age)**: While not as crucial as odometer reading, the car's age remains a significant factor. Newer cars generally command higher prices due to perceived better condition and updated features. Account for the year of manufacture in your pricing models.
4. **Manufacturer**: Brand reputation plays a substantial role, particularly for prestigious names like Aston Martin, Ferrari, and Tesla (ranked 1st, 2nd, and 3rd respectively). These brands often signify luxury, performance, and cutting-edge technology, commanding premium prices. Categorize and analyze car prices based on manufacturer reputation and brand value.
5. **Cylinders**:  The number of cylinders is a differentiating factor, especially for non-standard configurations like 12, 3, and 8 cylinders (ranked 1st, 2nd, and 3rd respectively). These often represent high-performance or fuel-efficient engines, which can significantly impact price. Consider engine specifications and cylinder count in your pricing models. 
6. **Fuel Type**:  The type of fuel the car uses (diesel, electric, hybrid - ranked 1st, 2nd, and 3rd) plays a role in determining price.  Consider incorporating fuel type and its associated costs (e.g., fuel efficiency, charging infrastructure) into your pricing models.
7. **Region**: The region where the car is located has an impact on its price. The top 3 regions impacting price are southwest MS, southwest VA, and Gainesville (ranked 1st, 2nd, and 3rd). Consider regional factors such as demand, supply, and cost of living when assessing car prices.
8. **State**: Similar to region, the state where the car is located can influence its price due to varying regulations, taxes, and market conditions.  The top 3 states are LA, AR, and OH (ranked 1st, 2nd, and 3rd). Account for state-specific factors in your pricing analysis. 
9. **Title Status**: The car's title status (lien, clean, missing - ranked 1st, 2nd, and 3rd) is important for determining its value and legal standing. Ensure accurate title information is available and factored into pricing decisions.
10. **Type**: The type of vehicle (offroad, bus, truck - ranked 1st, 2nd, and 3rd) significantly affects its price. Different vehicle types cater to different needs and preferences, leading to varying price points.  Clearly categorize and analyze prices based on vehicle type.
11. **Condition**: The car's condition (fair, salvage, new - ranked 1st, 2nd, and 3rd) is a crucial factor influencing its price.  Customers are willing to pay more for cars in better condition. Accurately assess and represent the car's condition in pricing models.
12. **Transmission**: The type of transmission (other, manual, automatic - ranked 1st, 2nd, and 3rd) can impact the car's desirability and price.  Consider the preferences of your target market and the availability of different transmission types.
13. **Paint Color**: While having a lower importance, paint color can still influence a car's price. The top 3 colors are purple, brown, and green (ranked 1st, 2nd, and 3rd). Certain colors may be more popular or desirable than others, leading to price variations.
14. **Model**:  The specific model of the car within a manufacturer's lineup can significantly influence its price due to varying features, performance, and demand. Account for model-specific attributes in your pricing analysis. 


Hope this helps the used car dealership in determining what factors to consider for maintaing inventory of used cars.   

# Next steps (improvements) recommendation for the project
The R² value of 0.83 is pretty good. However, it will good to identify what else can we do to further improve that number (and reduce RMSE/MSE). One such area that I feel we can look at exploring is the interaction relationship between features- just like there was an interaction identified between age and odometer. This could further help in explaining the factors that dictate the pricing. Interaction between Manufacturer and State features could be one such area to explore.