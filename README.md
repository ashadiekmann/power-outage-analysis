# When the Lights Go Out: Analyzing Power Outage Patterns Across the U.S.

**Name**: Asha Diekmann

## Introduction

This project analyzes 1,534 major power outages across the continental
United States from 2000 to 2016. Each row represents one outage event.

**Central Question: What actually predicts how severe a power outage
will be, and is that prediction fair across all regions of the US?**

Understanding what drives outage severity matters for utility companies,
policymakers, and the millions of people who depend on reliable electricity.
Between 2000 and 2016, major outages affected an average of over 100,000
customers per event, and the number of outages nearly tripled between 2009
and 2011 alone. In the East North Central region, outages lasted nearly
4 days on average - far longer than any other region. By understanding
what predicts severity, utility companies can better allocate emergency
resources before an outage even ends.

Relevant columns and their descriptions:

| Column | Description |
|--------|-------------|
| YEAR | The year the outage occurred |
| MONTH | The month the outage occurred |
| U.S._STATE | The state where the outage occurred |
| CLIMATE.REGION | The U.S. climate region |
| CAUSE.CATEGORY | The general cause of the outage |
| OUTAGE.DURATION | How long the outage lasted in minutes |
| CUSTOMERS.AFFECTED | The number of customers who lost power |
| DEMAND.LOSS.MW | The amount of peak demand lost in megawatts |
| OUTAGE.START | The date and time the outage began |
| OUTAGE.RESTORATION | The date and time power was restored |

## Data Cleaning and Exploratory Data Analysis

For data cleaning, I first dropped the 'variables' and 'OBS' columns which 
were metadata from the original Excel file. I also dropped the first row 
which contained units like "mins" and "Megawatt" rather than actual data. 
I then combined the separate date and time columns into single timestamp 
columns called OUTAGE.START and OUTAGE.RESTORATION, and dropped the 
originals. I converted YEAR and MONTH from floats to integers, and converted 
OUTAGE.DURATION, CUSTOMERS.AFFECTED, and DEMAND.LOSS.MW to numeric values 
using pd.to_numeric, which turned any non-numeric entries into NaN. After 
cleaning, the three columns with the most missing values were DEMAND.LOSS.MW 
(705 missing), CUSTOMERS.AFFECTED (443 missing), and OUTAGE.DURATION (58 missing).

Here are the first 5 rows of the cleaned dataset:

|   YEAR |   MONTH | U.S._STATE   | CLIMATE.REGION     | CAUSE.CATEGORY     |   OUTAGE.DURATION |   CUSTOMERS.AFFECTED |
|-------:|--------:|:-------------|:-------------------|:-------------------|------------------:|---------------------:|
|   2011 |       7 | Minnesota    | East North Central | severe weather     |              3060 |                70000 |
|   2014 |       5 | Minnesota    | East North Central | intentional attack |                 1 |                  nan |
|   2010 |      10 | Minnesota    | East North Central | severe weather     |              3000 |                70000 |
|   2012 |       6 | Minnesota    | East North Central | severe weather     |              2550 |                68200 |
|   2015 |       7 | Minnesota    | East North Central | severe weather     |              1740 |               250000 |

The Northeast region has the most power outages by far with around 350, 
followed by the South and West. This suggests that more densely populated 
regions tend to experience more outages.

<iframe
  src="assets/outages_by_region.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

Most power outages are resolved relatively quickly, with the majority 
lasting under 2,000 minutes. However there are some extreme outliers 
that last much longer, showing the distribution is heavily right-skewed.

<iframe
  src="assets/outage_duration.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

Fuel supply emergencies tend to last the longest on average, while 
intentional attacks are resolved more quickly. This makes sense as 
fuel-related issues may take longer to diagnose and fix.

<iframe
  src="assets/duration_by_cause.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

The number of outages increased dramatically around 2011, likely due to 
a series of major storms that hit the US that year. There is a general 
upward trend from 2000 to 2011 followed by a decline.

<iframe
  src="assets/outages_per_year.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

## Assessment of Missingness

I believe DEMAND.LOSS.MW is likely NMAR. The missingness is probably related 
to the value itself — smaller outages likely didn't have demand loss measured 
or reported at all. If we had data on which utility company reported each 
outage, that could explain the missingness and make it MAR.

The permutation test for CAUSE.CATEGORY produced a p-value < 0.002, meaning 
the missingness of CUSTOMERS.AFFECTED is dependent on CAUSE.CATEGORY. 
Certain types of outages are more likely to have missing customer data than others.

<iframe
  src="assets/missingness.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

The permutation test for MONTH produced a p-value of 1.0, meaning the 
missingness of CUSTOMERS.AFFECTED does not depend on the month the outage 
occurred. This makes sense since missing customer data is unlikely to be 
related to the time of year.

## Hypothesis Testing

**Null Hypothesis:** Outages in summer and outages in winter affect the same 
number of customers on average, and any difference is due to random chance.

**Alternative Hypothesis:** Outages in summer affect more customers on average 
than outages in winter.

**Test Statistic:** Difference in means (mean customers affected in summer minus 
mean customers affected in winter). Significance level: 0.05.

The observed difference in means was about -8190 customers, meaning winter 
outages actually affected more customers on average than summer outages. 
The p-value was 0.672, which is well above our significance level of 0.05. 
We fail to reject the null hypothesis. The data does not provide enough 
evidence to conclude that summer outages affect more customers than winter 
outages. This was a surprising finding since we might expect summer heat 
waves to cause larger outages, but winter storms may actually be more 
damaging to the power grid.

## Framing a Prediction Problem

**Prediction Problem:** Predict the number of customers affected by a power 
outage (CUSTOMERS.AFFECTED).

**Type:** Regression, since I am predicting a continuous numerical value.

**Response Variable:** CUSTOMERS.AFFECTED. I chose this because it is a direct 
measure of how severe an outage is, and understanding what drives it could 
help energy companies better prepare for large outages.

**Metric:** RMSE (Root Mean Squared Error). I chose RMSE over R squared because 
it is in the same units as my response variable (number of customers), making 
it easier to interpret.

**Features I would know at the time of prediction:** At the time an outage 
starts, I would know the cause category, climate region, state, month, year, 
and total customers in the area. I would not include outage duration since 
that information is only available after the outage is already over.

## Baseline Model

My baseline model uses Linear Regression to predict CUSTOMERS.AFFECTED with 
two features. CAUSE.CATEGORY is nominal so I one hot encoded it using 
OneHotEncoder. TOTAL.CUSTOMERS is quantitative so I left it as is. In total 
I have 1 nominal feature and 1 quantitative feature. The training RMSE was 
about 267,126 customers and the test RMSE was about 339,307 customers. I 
don't think this is a very good model since the RMSE is really high relative 
to the scale of the data. The gap between training and test RMSE also suggests 
some overfitting.

## Final Model

For my final model I added two new features: CLIMATE.REGION (nominal, one hot 
encoded) and MONTH (quantitative). I switched from LinearRegression to 
RandomForestRegressor since it can capture non-linear relationships between 
features and the target variable. The best hyperparameters from GridSearchCV 
were max_depth of 3 and n_estimators of 200. The final test RMSE dropped from 
about 339,307 to about 330,062 customers, which is an improvement over the 
baseline.

## Fairness Analysis

**Group X:** Northeast region outages

**Group Y:** All other region outages

**Evaluation Metric:** RMSE

**Null Hypothesis:** My model is fair. The RMSE for Northeast outages and 
non-Northeast outages are roughly the same, and any difference is due to 
random chance.

**Alternative Hypothesis:** My model is unfair. The RMSE for Northeast outages 
is lower than for non-Northeast outages.

**Test Statistic:** Difference in RMSE. Significance level: 0.05.

The observed difference in RMSE was about -287,571, meaning the model 
actually performs better on Northeast outages than on outages in other 
regions. The p-value was 0.002, which is below our significance level of 
0.05, so we reject the null hypothesis. The data suggests that my model 
may not be perfectly fair. This is likely because the Northeast has the 
most outages in the dataset, giving the model more examples to learn from 
for that region. In the future, collecting more data from underrepresented 
regions could help improve fairness.