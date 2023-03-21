# IBM Applied Data Science Capstone
This is the final project from the IBM Applied Data Science Capstone course on Coursera. This document contains a summary of the methodology and results. The `final_presentation.pdf` is a slide deck version of the contents here.

#### Table of contents
1. [Executive summary](#executive-summary)
2. [Introduction](#introduction)
3. [Methodology](#methodology)
   1. [Data collection](#data-collection)
   2. [Data processing](#data-processing)
   3. [Exploratory analysis](#exploratory-analysis)
   4. [Features engineering](#features-engineering)
   5. [Visual analytics](#visual-analytics)
   6. [Predictive analytics](#predictive-analytics)
4. [Results and insights](#results-and-insights)
   1. [Exploratory data analysis](#exploratory-data-analysis)
   2. [Visual analytics](#svisual-analytics)
   3. [Predictive analytics](#spredictive-analytics)
5. [Conclusions](#conclusions)

## Executive summary <a name='executive-summary'></a>
We collected historical Falcon 9 launch data from the SpaceX launch API along with information available on Wikipedia. After data processing and exploratory data analysis with some basic visualizations and SQL queries, we performed features engineering on the data. We then utilized interactive analytics to investigate how the launch success rate is related to the data features, including flight number, launch site location, orbit type, and payload mass. Finally, we trained and tuned 5 machine learning models to predict whether the first stage of a Falcon 9 rocket will land successfully and be able to be reused. Each of the 5 models obtained similar performance, with an 83% prediction accuracy on the test data.

## Introduction <a name='introduction'></a>
Much of the cost savings of the SpaceX Falcon 9 rocket come from the fact that the first stage can be reused if landed successfully. By predicting whether or not the first stage can be successfully recovered, we can estimate the cost of a launch (which could be useful to another company hoping to bid against SpaceX). We hope to answer the following question.

> Can we predict whether or not the first stage of a Falcon 9 rocket will land successfully?

## Methodology <a name='methodology'></a>
In this section, we summarize our methodology regarding data collection, data processing, exploratory analysis, visual analytics, and predictive analytics.

### Data collection <a name='data-collection'></a>

#### SpaceX REST API
We made a request from the SpaceX REST API at `spacex_url = https://api.spacexdata.com/v4/launches/past` to collect historical launch data.

![api_diagram](https://user-images.githubusercontent.com/50031286/226355714-18f81be7-dbfa-4854-bec6-3a1d8e0b41ec.png)

The request was successful

![response_success](https://user-images.githubusercontent.com/50031286/226356184-fee46f47-0597-4263-957b-1262916e24f6.png)

The response was then converted to a Pandas dataframe using the `.json_normalize()` method.

![api_data_1](https://user-images.githubusercontent.com/50031286/226356671-e5a7c312-43bc-4928-85b0-ca518e5e8c42.png)

#### Web scraping from Wikipedia
Additional data was scraped from Wikipedia tables at `https://en.wikipedia.org/w/index.php?title=List_of_Falcon_9_and_Falcon_Heavy_launches&oldid=1027686922` using `BeautifulSoup` and extracted to a list of HTML tables using `soup.find_all('tables')`.

### Data processing <a name='data-processing'></a>

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

### Exploratory analysis <a name='exploratory-analysis'></a>
Exploratory analysis was performed on the processed data using `SQL` and some simple visualizations in `Seaborn`.

### Features Engineering <a name='features-engineering'></a>
From the REST API data, there was a column labeled `Outcome` which tracked the landing outcome for the first stage of a Falcon 9 launch. This column had 8 possibilities, shown below.

![landing_outcomes](https://user-images.githubusercontent.com/50031286/226359907-2c3c6a0f-7367-47ae-9556-50bfc1b46110.png)

Since we are only concerned with whether the landing was a success or not, we introduced a column `Class` that indicates a successful landing as `1` and a failure as `0`. The `Outcome` column was then dropped.

![data_with_class](https://user-images.githubusercontent.com/50031286/226360275-065fb381-330e-451c-af3b-95ffa87d7051.png)

After this, we one-hot encoded the remaining categorical variables using `get_dummies`. The resulting dataframe was cast as type `float64`.

### Visual analytics <a name='visual-analytics'></a>
We explored the launch site locations using `Folium` and created an interactive dashboard using `Plotly Dash` to investigate the launch success rates as related to launch site and payload mass.

### Predictive analytics <a name='predictive-analytics'></a>
We trained 5 classification machine learning models in an attempt to predict the launch outcome (i.e. the `Class` column). After normalizing with `StandardScaler` and splitting the data into training and testing sets with `train_test_split`, we trained the following models from `scikit-learn`.
- Logistic Regression
- Support Vector Machine
- Decision Tree
- *k*-Nearest Neighbors
- XGBClassifier

Note that the testing data has only 18 entries--this ultimately results in models that are difficult to distinguish from one another.

![train_test_split](https://user-images.githubusercontent.com/50031286/226391846-0a753a37-be5e-4f8e-8600-3492cd7ed86f.png)
![y_test](https://user-images.githubusercontent.com/50031286/226391889-f66a173c-c0f5-4dc0-ad4d-a9aa736ac928.png)

Hyperparameters were tuned using 10-fold cross validation with `GridSearchCV`. Model performance was evaluated using their accuracy score on the test data. 

![model_training_tuning](https://user-images.githubusercontent.com/50031286/226390787-934a2ec1-2694-4639-81d0-12cb7e14387a.png)

## Results and insights

### Exploratory data analysis <a name='exploratory-data-analysis'></a>
This section summarizes the results and insights drawn from our exploratory data analysis. For more detail, see the following notebooks from the `Exploratory analytics` folder.
- `EDA with SQL.ipynb`
- `Visualization and features engineering.ipynb`

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

- Flight Number vs. Launch Site

![flightno_vs_launchsite](https://user-images.githubusercontent.com/50031286/226370235-4af0c7b3-5c70-4c61-b24b-7615131c29ab.png)

Notice, the success rate appears to increase with the flight number and different launch sites have different success rates. Also observe how the choice of launch site changes with flight number.

- Payload mass vs. Launch Site

![payload_vs_launchsite](https://user-images.githubusercontent.com/50031286/226371218-5a3716e2-0c41-4033-99bc-3e754f315c4f.png)

Launch site VAFB SLC-4E has no launches with payload greater than 10,000 kg.

- Orbit success rates

![orbit_success](https://user-images.githubusercontent.com/50031286/226371837-6018f3ea-8815-464b-9f09-6b43379a5e07.png)

The orbits with the highest success rate are ES-L1, GEO, HEO, and SSO.

- Flight number vs. Orbit

![flightno_vs_orbit](https://user-images.githubusercontent.com/50031286/226372072-deac00a3-683f-4717-92ab-d0baef1712d8.png)

The success rate for LEO increases with flight number. However, there does not appear to be a relationship between flight number and success rate for GTO launches.

- Payload mass vs. Orbit

![payload_vs_orbit](https://user-images.githubusercontent.com/50031286/226372490-8fd9fc52-dc9f-428f-a411-f6e6d3404ad8.png)

Heavy payloads (greater than 10,000 kg) have more success with LEO and ISS launches.

- Launch success rate yearly trend

![year_vs_success](https://user-images.githubusercontent.com/50031286/226373346-730c2643-0866-4b48-9795-516a9762579b.png)

The launch success rate has increased from 2010 to 2020.

### Visual analytics <a name='svisual-analytics'></a>
This section contains results and insights from our visual analytics. For more detail, see the following notebooks from the `Visual analytics` folder.
- `Exploring launch site locations with Folium.ipynb` 
- `Launch records dashboard.ipynb`

#### Launch site geography

- Launch site locations

![launch_site_locations](https://user-images.githubusercontent.com/50031286/226375730-e1036ca9-61d6-4038-a9a6-d94a04282108.png)

There are 4 distinct launch sites, 1 of which is in California with the rest in close proximity to one another near Cape Canaveral, Florida.

- Successful launches by site

The launch site in California (`VAFB SLC-4E`) has 10 launches, while the Florida launch sites have a combined 46 launches.

![launch_success_1](https://user-images.githubusercontent.com/50031286/226377202-74787518-a901-4732-84e8-518768f9fafc.png)

Clicking on the marker, we can see the successful launches colored in green.

![launch_success_2](https://user-images.githubusercontent.com/50031286/226378085-130ae1c5-e7fa-4860-8527-74e0f1709949.png)

From the Folium map, we can infer the following approximate launch success rates for each site.

| Launch site  | Success rate on Folium Map|
| -----------  | :-------------------------: |
| CCAFS LC-40  | 27%                       |
| CCAFS SLC-40 | 43%                       |
| KSC LC-39A   | 77%                       |
| VAFB SLC-4E  | 40%                       |

- Launch site proximities

![launch_site_proximities](https://user-images.githubusercontent.com/50031286/226382619-e42e14a4-e4ce-4b82-add3-233024ff9ef2.png)

We observed that launch sites tend to be close to railways, highways, and coastlines. Close proximity to railways and highways makes it easier to transport resources (including people) to and from the launch site. 

We also noticed that launch sites tend to be close to coastlines but at least 15 km away from cities. These locational constraints can help minimize the risk of collateral damage in the event of a malfunction.
  
#### Launch site and payload dashboard

- Total successful launches by site

![launch_success_site_pie](https://user-images.githubusercontent.com/50031286/226384798-ba7ce219-2fc2-4369-8339-37c8f8380cbe.png)

KSC LC-39A has the highest proportion of successful launches.

- Successful launches from KSC LC-39A

![successful_launches_ksc](https://user-images.githubusercontent.com/50031286/226385413-368d7225-1fdb-4014-94a0-ddd5fd77143f.png)

KSC LC-39A also has the highest launch success rate of approximately 77%.

- Payload vs. Success

![payload_success_all](https://user-images.githubusercontent.com/50031286/226387168-2e94cbb9-a36d-4216-be38-190a51d1f05f.png)

Generally, as payload increases, the success rate for all sites decreases. Below are some plots showing the success rate for small and heavy payload ranges.

![payload_success_smallest](https://user-images.githubusercontent.com/50031286/226387306-ba6ea4dc-bf37-4554-8ffc-3db488d8d8ac.png)

![payload_success_largest](https://user-images.githubusercontent.com/50031286/226387336-59c0aedd-4750-4610-9c56-4c9dc255ba5b.png)

A payload range between 2,500 kg and 3,500 kg has a success rate of about 67%.

![payload_success_small](https://user-images.githubusercontent.com/50031286/226387617-2962aa72-4f3c-4524-8598-50d0b36dff29.png)

Meanwhile, the payload range from 6,000 kg to 7,500 kg has a success rate of 0%.

![payload_success_large](https://user-images.githubusercontent.com/50031286/226388032-150ef48a-6330-4990-94d4-9d9fe446a8fd.png)

### Predictive analytics <a name='spredictive-analytics'></a>
This section summarizes the results and insights drawn from our predictive analytics. For more detail, see the following notebook from the `Predictive analytics` folder.
- `Predictive analytics (classification).ipynb`

#### Model performance

![model_performance](https://user-images.githubusercontent.com/50031286/226389871-ee215cf5-68f2-482a-8d67-aaee66841bf0.png)

Each of the 5 trained models performed similarly, achieving an accuracy of about 83% on the testing data.

#### Confusion matrix

![confusion_matrix](https://user-images.githubusercontent.com/50031286/226390010-d4bf8091-448c-4e3a-894d-6ed4ecde6741.png)

Moreover, each model produced the same confusion matrix. The confusion matrix indicates that any of the models can be used to predict the landing outcome, with a primary caveat of false positives.

## Conclusions <a name='conclusions'></a>

- It appears possible to predict whether or not a Falcon 9 rocket's first stage can be reused.
- However, the models should be trained and tested on a larger dataset if we would actually like to determine optimal performance.
