# When the Lights Go Out: Analyzing Power Outage Patterns Across the U.S.

**Name**: Asha Diekmann

## Introduction

This dataset contains information on major power outages in the continental 
United States from January 2000 to July 2016. It has 1,534 rows, where 
each row represents one major power outage event.

**Central Question: Where and when do major power outages tend to occur, 
and what characteristics are associated with higher severity?**

Understanding patterns in power outages matters for energy companies, 
policymakers, and everyday people who depend on reliable electricity. 
By identifying which regions and time periods are most vulnerable, 
we can better prepare for future outages.

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