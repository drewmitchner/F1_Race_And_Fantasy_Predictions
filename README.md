# F1_Race_And_Fantasy_Predictions
ML and Monte-Carlo modeling for predicting Formula 1 race results and optimizing F1 Fantasy scores
<figure>   
  <img src="/plots/Abu-Dhabi_2024_MonteCarloResults.png" width="800" height="500">
    <figcaption><center>Monte-Carlo race place predictions for Abu-Dhabi, 2024.</center></figcaption>
</figure>

## Table of Contents
1. Overview
2. Dependencies
3. Methodology
4. Results
5. Further Work

## 1. Overview
This project uses historical F1 data from the F1db page (Citation needed) to build prediction models for Formula 1 races and fantasy game. Historic data is pre-processed, analyzed, and various prediction models are attempted with the best performing XGBoost model selected for further predictions. Individual driver predictions are pitted against each other using Monte-Carlo method to simulate each Grand Prix 500+ times. The race place probabilities of the Monte-Carlo are translated into predicted F1 Fantasy points, and Linear Programming is used to build an optimal F1 Fantasy team within a given budget.
The methodology presented implements new details generally not seen in order to improve performance:
* **Driver Form**: predictions for each race include the driver's previous race and fantasy results from the past N races. 2-3 past races were found to give the best results
* **Pre- and Post-Qualifying Predictions**: as anticipated, starting grid position is by far the single strongest predictor of race position. However, for F1 Fantasy, team selections must be locked be in before qualifying. Predictions are made using both just free practice results as well as post-qualifying to understand the delta in predictive accuracy. Monte-Carlo and Fantasy predictions all use the pre-qualifying models.
* **Monte-Carlo Simulation**: many similar projects make race predictions looking at a sole driver's data in isolation without factoring in other drivers present in the Grand Prix. By using Monte-Carlo methods, individual driver predictions are pitted against one another and the fine tuning of the gaps between drivers is shown to increase performance.

## 2. Depencies
* Python 3

## 3. Methodology
The project contains 5 Juptyter Notebooks:
1. [F1 Predictions 1 - Clean Data.ipynb](F1%Predictions%1%-%Clean%Data.ipynb)
2. [F1 Predictions 2 - EDA.ipynb](F1%Predictions%2%-%EDA.ipynb)
3. [F1 Predictions 3 - Model Building.ipynb](F1%Predictions%3%-%Model%Building.ipynb)
4. [F1 Predictions 4 - Monte Carlo.ipynb](F1%Predictions%4%-%Monte%Carlo.ipynb)
5. [F1 Predictions 5 - Team Optimization.ipynb](F1%Predictions%5%-%Team%Optimization.ipynb)

### F1 Predictions 1 - Clean Data.ipynb
This notebook reads in the raw data using yaml for race results, free practice, qualifying, fastest lap, and driver of the day voting. The data is merged into a single dataframe.
<br/><br/>
Note: Only the past 10 years (2014-2024) of data are used for simplicity. Data is available going back to 1950, but F1 rules have varied over that time period. Certainly an option moving forward to train on data going farther back.

### F1 Predictions 2 - EDA.ipynb
Further data cleaning and exploratory data analysis (EDA). Fantasy points are added for each race, producing the following distribution:
<figure>   
  <img src="/plots/F1_FantasyPoints_Histogram.png" width="500" height="300">
    <figcaption><center>F1 Fantasy points over the past 10 years.</center></figcaption>
</figure>
The data have a mean of 7.9 points and standard deviation of 16.4 points. The spike at -20 represents all the DNF's, which occurs for roughly 10% of the drivers over this period. The other peak to note is around 35 points, which is the typical score for a driver winning from pole position (10 points for qualifying first, 25 points for winning). Over the selected time period, the driver starting from pole has won 50.3% of the time (starting second has won 25%, and starting third has won ~10% of the time, which leaves about 15% wins for those starting off the top three).
<br/><br/>
Additionally in the EDA section, a histogram is created for each starting grid to visualize the likelihood for each finishing place. A beta distribution is then fit for each of the 20 starting grid positions.
<figure>   
  <img src="/plots/F1_Finish_vs_GridPosition_Fitted_1-8.png" width="900" height="400">
</figure>
<figure>   
  <img src="/plots/F1_Finish_vs_GridPosition_Fitted_9-16.png" width="900" height="400">
</figure>
<figure>   
  <img src="/plots/F1_Finish_vs_GridPosition_Fitted_17-20.png" width="900" height="200">
  <figcaption><center>Race place distribution based on each starting grid position.</center></figcaption>
</figure>
The fitted distibrutions are used in the Monte-Carlo simulations to inject some reasonable levels of variance into each drivers performance in line with historical data.
<br/><br/>
The beta distributions do a reasonable job of capturing the distributions with a couple caveats. In general the fitted line shows higher variance than the actual race data. Additionally, drivers starting near the back of the grid don't finish in the last few positions as often as the fit indicates. This is likely due to a couple drivers every race experiencing a DNF, bumping those starting in the back up a couple places. Future work may focus on refining these fitted distibutions.

### F1 Predictions 3 - Model Building.ipynb
In this section multiple stats models are built using various regression and machine learning approaches with varying predictor data. Model performance is analyzed and compared to understand predictive capabilities and limitations. For all models the data is split into train and test sets (test split=0.2). The XGBoost Regressor model that performs the best on pre-qualifying data is selected to be used for final model predictions.
<br/><br/>
As a first baseline, multiple linear regression is used to predict both race place and fantasy points. The two plots show predicted fantasy points when both including and excluding qualifying data.
<figure>   
  <img src="/plots/F1_LinModel_FantasyPoints.png" width="400" height="250">
  <figcaption><center>F1 Fantasy points predictions when including qualifying data and starting grid information.</center></figcaption>
</figure>
<figure>   
  <img src="/plots/F1_LinModel_FantasyPoints_preQual.png" width="400" height="250">
  <figcaption><center>F1 Fantasy points predictions when only including pre-qualifying data.</center></figcaption>
</figure>
The two models show that the inclusion of qualifying data (Q1/2/3 times and positions, as well as starting grid position) improves the R<sub>2</sub> from 0.42 to 0.50. It's a notable dropoff to not have the qualifying data, but the pre-qualifying model still retains notable predictive capability despite lacking the single most effective predictor in starting grid position.
<br/><br/>
An important limitation of linear regression is the inability to handle missing data. This significantly cuts into an already limited dataset. To address this and to build models of increasing complexity, the next several models looked at are XGBoost forest methods. The next two plots look at predicting final race place, again both with and without qualifying results.
<figure>   
  <img src="/plots/F1_XGBoost_RacePosition.png" width="400" height="250">
  <figcaption><center>XGBoost race position predictions when including qualifying data and starting grid information.</center></figcaption>
</figure>
<figure>   
  <img src="/plotsF1_XGBoost_RacePosition_preQual.png" width="400" height="250">
  <figcaption><center>XGBoost race position predictions when only including pre-qualifying data.</center></figcaption>
</figure>
The XGBoost models show marked improvement over the simple regression models, increasing the R<sub>2</sub> performance on the test dataset from 0.5 to 0.67 and from 0.42 to 0.58 for both the post- and pre-qualifying predictor sets, respectively. It is interesting, and probably a comfort, that the decrease in performance when removing qualifying data is the same across the two methods. Additionally, note that the pre-qualifying XGBoost model outperforms even the post-qualifying linear regression model.
<br/><br/>
For these XGBoost models, inclusion of driver "form" (race place and fantasy points from the past N grand prix's) was experimented with. Inclusion of 2-3 prior race results was find to slightly boost performance on the order of a couple percentage points. Including additional prior races saw no benefit and even, in some cases, worse performance as the model began overfitting. The final model uses a driver's prior 2 grand prix results.
<br/><br/>
Finally, the best performing XGBoost regressor with pre-qualifying data only is selected to used in the Monte-Carlo simulation and fantasy team optimization in the remaining sections.

### F1 Predictions 4 - Monte Carlo.ipynb
In the model building section above, all the models only looked at one driver's data in isolation and did not consider the other competitors in the race. A Monte-Carlo method is an effective way of pitting driver predictions against one another. This allows for probabilistic predictions for each discrete race place and better fine tuning of the gaps between drivers. No longer is the model limited to "driver 1 is predicted ahead of driver 2", but understands how big the gap is between successive drivers and can factor that into its predictions.
<br/><br/>
The Monte-Carlo simulation process is split into 7 steps:
1. Get an expected race place for each driver based on the XGBoost regression model.
2. For each driver, identify which two beta fit models to use (whole number above and below the expected place).
3. Randomly select which of the two beta fit models to use, weighted accordingly.
4. Using the selected beta PDF, randomly generate a randomized, continuous race place for each driver.
5. Rank the drivers according to their random race place.
6. Get the predicted fantasy points for each driver based on this race order.
7. Repeat the above steps N times.*
\*The default number of runs is 500. This number was used to balance stable predictions and computation time. For specific Grand Prix one-offs, the number N can be increased to remove batch variance.
<br/><br/>
The table below breaks down the specific race place probabilities for each driver for the last race of 2024 in Abu-Dhabi.
<figure>   
  <img src="/plots/Abu-Dhabi_2024_MonteCarloResults.png" width="600" height="350">
    <figcaption><center>Monte-Carlo race place predictions for Abu-Dhabi, 2024.</center></figcaption>
</figure>
In the specific Grand Prix pictured, the model "correctly" identified the winner as Lando Norris. However, it underestimated the performance of the two Ferrari drivers, who had poor practice times but bounced back in the Grand Prix (though with a significant boost from Verstappen and Piastri touching and losing time at the first corner).
<br/><br/>
Finally for this section, predicted race place is converted into predicted fantasy points. To assist this, historical data was used to correlate finishing position with fantasy points. The two plots below show the correlations for finishers inside the top 10 and outside the top 10.
<figure>   
  <img src="/plots/F1_XGBoost_RacePosition.png" width="400" height="250">
  <figcaption><center>XGBoost race position predictions when including qualifying data and starting grid information.</center></figcaption>
</figure>
<figure>   
  <img src="/plotsF1_XGBoost_RacePosition_preQual.png" width="400" height="250">
  <figcaption><center>XGBoost race position predictions when only including pre-qualifying data.</center></figcaption>
</figure>
