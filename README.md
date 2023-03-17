# IBM Applied Data Science Capstone
#### Table of contents

## Executive Summary
We collected historical Falcon 9 launch data from the SpaceX launch API along with information available on Wikipedia. After data processing and exploratory data analysis with some basic visualizations and SQL queries, we perform features engineering on the data. We then utilized interactive analytics to investigate how the launch success rate is related to the launch site location and payload mass. Finally, several machine learning models were trained and tuned 5 machine learning models to predict whether the first stage of a Falcon 9 rocket will land successfully and be able to be reused. We found that flight number, launch site, and payload mass were related to the launch success rate. Each of the 5 models trained obtained an 83% prediction accuracy on the test data.

## Introduction
Much of the cost savings of the Falcon 9 rocket come from the fact that the first stage can be reused if landed successfully. By predicting whether or not the first stage can be successfully recovered, we can estimate the cost of a launch (which could be useful to another company hoping to bid against SpaceX). We attempt to predict the success of SpaceX Falcon 9 launches from prior launch data. We hope to answer the following questions.
* How is the launch success rate related to flight number, launch site, and payload mass?
* Can we predict whether or not SpaceX will be able to reuse the first stage of a Falcon 9 launch?

## Methodology
In this section, we summarize our methodology regarding data collection, data processing, exploratory analysis, visual analytics, and predictive analytics.

### Data collection

#### SpaceX REST API
We made a request from the SpaceX REST API at `https://api.spacexdata.com/v4/launches/past` to collect historical launch data.

**include api flowchart here**

The request was successful

**include response.status_code snippet**

The response was then converted to a Pandas dataframe using the `.json_normalize()` method.

#### Web scraping from Wikipedia
Additional data was scraped from Wikipedia tables at `https://en.wikipedia.org/w/index.php?title=List_of_Falcon_9_and_Falcon_Heavy_launches&oldid=1027686922` using `BeautifulSoup` and extracted to a list of HTML tables using `soup.find_all('tables')`.

### Data processing

#### Wrangling the REST API data
Much of the data collected from the SpaceX REST API referenced identification numbers. Several functions were used to get more easily interpretable information from the REST API. For example, the `getBoosterVersion()` makes a separate request from the REST API to extract the booster version from the identification number in the original response data.

**getBoosterVersion() code snippet**

There are analogous `getLaunchSite()`, `getPayloadData()`, and `getCoreData()` functions.

After the identification numbers were replaced as described above, several irrelevant columns were dropped, and the data was filtered to include only the Falcon 9 launch data. There were 5 missing payload values, which were replaced by the average payload.

**include code snippet**

Below is a preview of the resulting dataframe.

**dataframe head**

#### Wrangling the web scraped data
From the list of HTML tables scraped from `BeautifulSoup`, the relevant data was parsed into a Python dictionary and then converted into a Pandas dataframe, previewed below.

**dataframe head**

### Exploratory analysis
Exploratory analysis was performed on the processed data using SQL and some simple visualizations in `Seaborn`.

### Features Engineering
From the REST API data, there was a column labeled `Outcome` which tracked the landing outcome for the first stage of a Falcon 9 launch. This column had 8 possibilities, shown below.

**snippet**

Since we are only concerned with whether the landing was a success or not, we introduced a column `Class` that indicates a successful landing as `1` and a failure as `0`. The `Outcome` column was then dropped.

After this, one-hot encoding the remaining categorical variables and the entire dataframe was cast as type `float64`.

### Visual analytics
We explored the launch site locations using `Folium` and created an interactive dashboard using `Plotly Dash` to investigate the launch success rates as related to launch site and payload mass.

### Predictive analytics
We trained 5 classification machine learning models in an attempt to predict the launch outcome (i.e. the `Class` column). After normalizing and splitting the data into training and testing sets, we trained the following models.
- Logistic Regression
- Support Vector Machine
- Decision Tree
- *k*-Nearest Neighbors
- XGBClassifier

Hyperparameters were tuned using 10-fold cross validation with `GridSearchCV`. Model performance was evaluated using their accuracy score on the test data. 

## Results and insights

### Exploratory data analysis

### Visual analytics

### Predictive analytics

## Conclusions
* It appears possible to predict whether or not a Falcon 9 rocket's first stage can be reused.
* However: the models should be trained on a larger dataset if we would actually like to determine optimal performance.

## Appendix
