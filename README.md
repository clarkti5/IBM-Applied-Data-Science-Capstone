# IBM Applied Data Science Capstone
#### Table of contents

## Executive Summary
We collected historical Falcon 9 launch data from the SpaceX launch API along with information available on Wikipedia. After data processing and exploratory data analysis with some basic visualizations and SQL queries, we performed features engineering on the data. We then utilized interactive analytics to investigate how the launch success rate is related to the data feautures, including flight number, launch site location, orbit type, and payload mass. Finally, we trained and tuned 5 machine learning models to predict whether the first stage of a Falcon 9 rocket will land successfully and be able to be reused. Each of the 5 models obtained similar performance, with an 83% prediction accuracy on the test data.

## Introduction
Much of the cost savings of the SpaceX Falcon 9 rocket come from the fact that the first stage can be reused if landed successfully. By predicting whether or not the first stage can be successfully recovered, we can estimate the cost of a launch (which could be useful to another company hoping to bid against SpaceX). We hope to answer the following question.

> Can we predict whether or not the first stage of a Falcon 9 rocket will land successfully?

## Methodology
In this section, we summarize our methodology regarding data collection, data processing, exploratory analysis, visual analytics, and predictive analytics.

### Data collection

#### SpaceX REST API
We made a request from the SpaceX REST API at `spacex_url = https://api.spacexdata.com/v4/launches/past` to collect historical launch data.

![api_diagram](https://user-images.githubusercontent.com/50031286/226355714-18f81be7-dbfa-4854-bec6-3a1d8e0b41ec.png)

The request was successful

![response_success](https://user-images.githubusercontent.com/50031286/226356184-fee46f47-0597-4263-957b-1262916e24f6.png)

The response was then converted to a Pandas dataframe using the `.json_normalize()` method.

![api_data_1](https://user-images.githubusercontent.com/50031286/226356671-e5a7c312-43bc-4928-85b0-ca518e5e8c42.png)

#### Web scraping from Wikipedia
Additional data was scraped from Wikipedia tables at `https://en.wikipedia.org/w/index.php?title=List_of_Falcon_9_and_Falcon_Heavy_launches&oldid=1027686922` using `BeautifulSoup` and extracted to a list of HTML tables using `soup.find_all('tables')`.

### Data processing

#### Wrangling the REST API data
Much of the data collected from the SpaceX REST API referenced identification numbers. Several functions were used to get more easily interpretable information from the REST API. For example, `getBoosterVersion()` makes a separate request from the REST API to extract the booster version from the identification number in the original response data.

![getBoosterVersion](https://user-images.githubusercontent.com/50031286/226357297-4af66521-93e8-4db0-8be6-ee92066b9548.png)

There are analogous `getLaunchSite()`, `getPayloadData()`, and `getCoreData()` functions.

After the identification numbers were replaced as described above, several irrelevant columns were removed (e.g. `links.youtube_id`) and the data was filtered to include only the Falcon 9 launch data. Below is a preview of the resulting data frame named `data_falcon9`.

![data_falcon9](https://user-images.githubusercontent.com/50031286/226358315-ba2750aa-3fcb-4de1-ab71-7f06b0db0c42.png)

There were 5 missing payload values, which were replaced by the average payload.

![missing_payload](https://user-images.githubusercontent.com/50031286/226358595-42c1602f-434c-414d-9aae-4ec9bf1526f9.png)

Missing values for `LandingPad` indicate when a landing pad was not used, so they were retained.

#### Wrangling the web scraped data
From the list of HTML tables scraped from `BeautifulSoup`, the relevant data was parsed into a Python dictionary named `launch_dict`, then converted into a Pandas dataframe, previewed below.

![data_scraped](https://user-images.githubusercontent.com/50031286/226359249-e607438c-009a-4422-a0bd-1a954457b8ec.png)

### Exploratory analysis
Exploratory analysis was performed on the processed data using `SQL` and some simple visualizations in `Seaborn`.

### Features Engineering
From the REST API data, there was a column labeled `Outcome` which tracked the landing outcome for the first stage of a Falcon 9 launch. This column had 8 possibilities, shown below.

![landing_outcomes](https://user-images.githubusercontent.com/50031286/226359907-2c3c6a0f-7367-47ae-9556-50bfc1b46110.png)

Since we are only concerned with whether the landing was a success or not, we introduced a column `Class` that indicates a successful landing as `1` and a failure as `0`. The `Outcome` column was then dropped.

![data_with_class](https://user-images.githubusercontent.com/50031286/226360275-065fb381-330e-451c-af3b-95ffa87d7051.png)

After this, we one-hot encoded the remaining categorical variables using `get_dummites`. The resulting dataframe was cast as type `float64`.

### Visual analytics
We explored the launch site locations using `Folium` and created an interactive dashboard using `Plotly Dash` to investigate the launch success rates as related to launch site and payload mass.

### Predictive analytics
We trained 5 classification machine learning models in an attempt to predict the launch outcome (i.e. the `Class` column). After normalizing with `StandardScaler` and splitting the data into training and testing sets with `train_test_split`, we trained the following models from `scikit-learn`.
- Logistic Regression
- Support Vector Machine
- Decision Tree
- *k*-Nearest Neighbors
- XGBClassifier

Hyperparameters were tuned using 10-fold cross validation with `GridSearchCV`. Model performance was evaluated using their accuracy score on the test data. 

## Results and insights
This sections summarizes the results and insights drawn from our investigations.

### Exploratory data analysis

#### EDA with SQL

- There are four distinct launch sites (`CCAFS LC-40`, `CCAFS SLC-40`, `KSC LC-39A`, and `VAFB SLC-4E`) used by SpaceX for Falcon 9 launches, as shown below.

![launch_sites_sql](https://user-images.githubusercontent.com/50031286/226362072-f318093b-5f29-4cf0-b15a-2d044e10d09e.png)

- Two of these sites (`CCAFS LC-40` and `CCAFS SLC-40`) are located in close proximity to each other near Cape Canaveral, Florida. Below are the first 5 launch records from these sites.

![cca_5](https://user-images.githubusercontent.com/50031286/226363143-57d68169-3a44-43d2-8fce-4d85c3cd9ad0.png)

- The total payload mass carried for NASA (CRS) was 45,956 kg. The total payload carried for NASA in general was 107,010 kg.

![nasa_payload_sql](https://user-images.githubusercontent.com/50031286/226365211-2fbffc42-61d9-4718-870b-c5e59b23cd9b.png)

- The average payload mass carried by the F9 v1.1 booster was 2,535 kg.

![avg_payload_sql](https://user-images.githubusercontent.com/50031286/226365646-87d93dbe-416c-45dd-87e9-8624a4069cda.png)

- The maximum payload was 15,600 kg carried by the boosters listed below.

![max_payload_sql](https://user-images.githubusercontent.com/50031286/226366136-4708498d-5e20-4a5e-b82e-3dcdf33ba5b9.png)

- The boosters that have successfully landed on a drone ship with payload between 4,000 and 6,000 kg are shown here.

![drone_ship_payload_range_sql](https://user-images.githubusercontent.com/50031286/226366615-7f8855df-b6b5-4ccf-9538-701cc8cfb446.png)

- The first successful landing on a ground pad was on 2015-12-22.

![ground_pad_first_sql](https://user-images.githubusercontent.com/50031286/226366935-773e382b-1882-409f-8677-11c1bdc76705.png)

- In total, there were 100 successful landings and 1 failure.

![outcomes_sql](https://user-images.githubusercontent.com/50031286/226367257-d2e525f7-d392-4b62-b358-4d06bf1bb445.png)

- The following describes the failed drone ship landings in 2015.

![fail_drone_sql](https://user-images.githubusercontent.com/50031286/226367686-a0835b3d-af74-48f9-a181-fd9805a10dd5.png)

- Below is the count of successful landing outcomes between 2010-06-04 and 2017-03-20.

![rank_sql](https://user-images.githubusercontent.com/50031286/226367958-cbe534f4-2574-4981-968f-190b82577c1d.png)

#### EDA with visualization



### Visual analytics

#### Launch site proximities

#### Launch site and payload dashboard

### Predictive analytics

#### Model performance

#### Confusion matrix

## Conclusions
* It appears possible to predict whether or not a Falcon 9 rocket's first stage can be reused.
* However: the models should be trained on a larger dataset if we would actually like to determine optimal performance.

## Appendix

* IBM Coursera Acknowledgement
* Jupyter notebooks (GitHub) link
