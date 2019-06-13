
# Module 1 Final Project Submissions


## File structure

Please, refer to [final_notebook](https://github.com/AnnaLara/dsc-1-final-project/blob/master/final_notebook.ipynb) as our main working notebook.

The directory also includes other notebooks that we used as drafts and for visualization purposes, as well as png images used in blog post.

## Presentation

Our presentation can be accessed with [this link](https://docs.google.com/presentation/d/1giwBhXJsj6OJRegxyEWuLGmiGKHhsnskNlKTO4_94Zc/edit?usp=sharing)

## Blogpost

The blogpost on log-transformation can be accessed with [this link](https://dev.to/annalara/king-county-house-sales-dataset-log-transformations-and-interpreting-results-19b2)

## Overview

**The goal** of the project was to create a linear regressiom model that predicts housing prices King County using [the dataset from Kaggle](https://www.kaggle.com/harlfoxem/housesalesprediction). The datase has around 22,500 datapoints and contain features like:

* **id** - unique identified for a house
* **dateDate** - house was sold
* **pricePrice** -  is prediction target
* **bedroomsNumber** -  of Bedrooms/House
* **bathroomsNumber** -  of bathrooms/bedrooms
* **sqft_livingsquare** -  footage of the home
* **sqft_lotsquare** -  footage of the lot
* **floorsTotal** -  floors (levels) in house
* **waterfront** - House which has a view to a waterfront
* **view** - Has been viewed
* **condition** - How good the condition is ( Overall )
* **grade** - overall grade given to the housing unit, based on King County grading system
* **sqft_above** - square footage of house apart from basement
* **sqft_basement** - square footage of the basement
* **yr_built** - Built Year
* **yr_renovated** - Year when house was renovated
* **zipcode** - zip
* **lat** - Latitude coordinate
* **long** - Longitude coordinate
* **sqft_living15** - The square footage of interior housing living space for the nearest 15 neighbors
* **sqft_lot15** - The square footage of the land lots of the nearest 15 neighbors

First of all, we created some beseline metrics to be able to see how do our models perform. We took an average price and calculated evaluaition metrics based on it:

- MSE: 134953101375.57
- RMSE: 367,359.63
- MAE: 234,028.18

### Data cleaning

We performed some data cleaning to be able to train out model on it.

1. Removed location features (we lated added it back)
2. Dropped ID, View, Renovation year (had little data) features
3. Data type transformations
4. Replaced '?' values in `sqft_basement` with defference in sdft_living and sqft_above

### Correlations

We checked for correlations between variables to check for multicollinearity. We found some obvious correlations betweet, for example, number of bathrooms and sqft_living, but thought that features still were significant per se, so we did not drop any of them.

### Feature engineering and choice of features to include to the model

- We created dummy variables for Waterfront feature and dropped all rows that had no data (around 150 rows).

- Created a new columnd `sold_y_ago` instead of `year_built`. It turned out that the only values it contained was 4 and 5, so we careated dummy variables for this feature too.

- We checked again for correlations:
    - Based on correlation between variables (multicollinearity) we decided to drop those variables that have correlation levels greater than o.8 which is sqft_above/sqft_living. We removed sqft_above, as sqft_living had greater correlation coefficient with target variable price.
    - After that we proceeded to choose variables for our model based on correlation level with price. We chose to include those where correlation index is greater than 0.1: bedrooms, bathrooms, sqft_living, floors, grade, sqft_basement, sqft_living15, WF_1.0.
    
- We tried different variations of features, building linear regression models with Statsmodel library. We ended up dopping some features because of large p-values. We also tried some data transformations on several variables, but it was not improving the model, so we did not implement them in the final model. We used R-squared, RMSE, MAE and cross-validated MAE to check the models' performance.

- The most significant feature engineering step was creating features based on location: distance from Seattle Downtown, from Bellvuew and from South Lake Union. We also log-transformed those variable. More on log-transformations in our blogpost: [King County House Sales dataset: log-transformations and interpreting results](https://dev.to/annalara/king-county-house-sales-dataset-log-transformations-and-interpreting-results-19b2)

We used `Geopy` library to extract distance from latitude and longitude.

- On the final steps we binned the number of bathrooms variable in to 5 categories to see if that improves our model performance. 

### Final model

When choosing our final model, we also decided to focus on a middle-class demographic who are not likely to be interested in multi-million dollar houses. Therefore, we can eliminate the outliers in the extreme upper range.

We calculated acceptable upper and lower bounds by using the interquartile range, and establish an acceptable limit above and below the third and first quartile, respectively. By printing these we can see that this gives us a lower limit of well below 0, which is fine, and an upper limit of 1.1 million, which is a reasonable maximum budget for our target audience.


Our final model included the following variables and coefficients:

|**Variable**|  **Cofficient** |
|---|---|
| sqft_basement  | -52.70529397568341  |
| sqft_living15  | 72.37624199160497  |
| sqft_living  | 161.1132233239232  |
| FL_NUM_2 | 5613.5486484063495 |
| bedrooms | -18195.009642388108 |
| FL_NUM_1 | 22405.63514503364 |
| FL_NUM_3 | -28019.183793439624 |
| bath_0-20% | 98775.4401659016 |
| mi_from_bellevue_log | -109855.76361744544 |
| bath_21-40% | 111239.01947057268 |
| bath_41-60% | 121021.85143872387 |
| mi_from_downtown_log | 210431.8366851622 |
| **bath_81-100%** | -235988.23720924056 |
| **WF_1.0** | 279274.5407692619 |
| **mi_from_south_lake_log** | -294919.76931348676 |
| y_intercept | 415562.4254950011 |

**Model performance**:

- R-squared: 0.727
- MAE : 83589.61
- Cross-validated MAE: -82730.77
- Improvement from baseline - RMSE: 70%
- Improvement from baseline - MAE (both csorr-validated and not-cross-validated): 64%

### Strongest predictors (for the homes up to 1.2 million):

- **Proximity to South Lake Union**
For every 500 ft increase in distance from South Lake Union, the price will decrease approximately  by $28,000

- **Waterfront location**
Increases the price by approximately  $280,000

- **3 - 5 bathrooms**
Increases the price by approximately  $121,000

### Some insights

- Distance from downtown has a positive relationship with the price, meaning that the further away from downtown, the higher the housing price. Possible explanations of this could be:
    - Buyers prefer other locations
    - The homes further from downtown can be bigger, thus more expensive
    
- Square ft: **price up by $161.11** for every additional square foot

- Square ft of the **nearest 15 houses**: **price up by $ 72.38**  for every additional square foot


