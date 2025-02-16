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

