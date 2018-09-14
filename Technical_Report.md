## Predicting Totals Bets for NBA Games

Outline:
- [Introduction](#intro)
- [Dataset](#data)
- [Feature Engineering/EDA](#eda)
- [Modeling](#model)
- [Conclusion](#conclusion)

<a id = 'intro'></a>
### Introduction
This project attempts to predict whether an NBA game's total score will be over or under a predetermined total line, 
set by Las Vegas casinos. The goal is to create a model which returns a prediction on whether a game's will likely be over or under the total line. With these predictions and their associated probabilities, I will set a threshold to create a set of bets I will term "confident bets." With these confident bets, I will create a strategy to (hopefully) profit off of this information.

In sports betting, a bookkeeper will create a bet based on the total final score of an NBA game. Bettors can bet either an "Over," betting that the total score of the game will be higher than the total set by the bookeeper, or an "Under," betting that the total score of the game will be lower than the total set by the bookkeeper. If the game's final total score lands on the book's point total, the bet is termed a "Push," and the book returns the bettor's money. 

The bookeeper sets the line at a number at which they believes will create action for both "Over" bets and "Under" bets; this strategy helps set either outcome as favorable to the bookkeeper. For a bookkeeper, having a lot of money bet on one team, or in this case on one side of the Over | Under bet, puts them in a riskier position, where they stand to lose a large amount of money (relatively). If possible, bookkeepers would prefer to minimize this risk by having money spread out between both sides of a bet. I say all that to say, the point total set by a sports book doesn't reflect the book's belief in the final total score of a game, but rather the total at which they believe will draw equal action on both sides of a bet. This distinction will be important in interpreting the results of my model.
 

<a id = 'data'></a>
### Data
I decided to analyze the past 5 season of NBA games for the purposes of this project. There are 82 games in an NBA season, and 30 teams in the league; I collected data for 6150 games in total. For both of the websites, I used the Python package BeautifulSoup to parse the HTML of the webpages and return the information I needed.

- For the basketball data I scraped box scores for every game off of basketball-reference.com. These box score stats are comprised of the counting stats (points, assists, rebounds, etc.) for a basketball game, for both the target team and their opponent. I stored this data as a collection of JSON files, ordered by team and season. 
- With a dataset comprised of the past 5 seasons of NBA game data, I was able to create a list of dates which NBA games were played on in the past 5 seasons. With the list of dates, I was able to scrape gambling lines off of sportsbookreview.com, querying each date and pulling the historical betting lines for each game on that date. 
- In order to run the scrapes continuously, I hosted the code on an AWS server. In total, the scrape of the box scores for the NBA games took ~4 hours (~45 minutes per season), and the scrape of the point totals for each game took around 20 minutes.


Data Cleaning: The dataset of box scores from basketball-reference.com, and the dataset of betting lines from sportsbookreview.com, were relatively clean to begin with. 

- There were missing betting lines for a few games; to rectify this I went to additional gambling websites with historical data to manually look up the lines for these games. 
- There was also one game missing entirely from the gambling data, which I then imputed (Miami-Washington, 03/06/2015)
- In 2015 season, the second in my data, the Charlotte Bobcats changes their name to the Charlotte Hornets. I addition, the team slug changed as well, from "CHA" to "CHO." This change caused me some problems, so when cleaning my data, I had to make sure that any reference to a team or team slug accounted for this change.
- I got the date for each game by splitting on the values in my 'gid' column. I then used the dates, as well as the team slugs, to merge the games dataframe and the betting lines dataframe.
- I conducted Regex string matching on the betting lines to change the '1/2' symbol, which because of it's encoding was being picked up by Jupyter Notebook as a special character, to a '.5,' for all the betting lines.

<a id = 'eda'></a>
### Feature Engineering/EDA

I caluculated a few additional features for my dataset, on top of the box scores stats I scraped off of basketball-reference. 

- I calculated whether each game ended up being an over, an under, or a push(final score same as total line), by subtracting the book's point total and subtracting that from the actual game total. I used these features as my target variables. 

- I imputed an offensive rating for each team for each game. Offensive rating is an "advanced" statistic (called advanced because it is not a counting stat) which is a summary metric for a team's offensive performance in a game. I imputed this statistic in the hopes it would capture much of the variability present in my other offensive statistics (points, rebounds, assists, etc.)

Accounting for Time Series Data:

- To predict the total score for a game, I decided to impute each game as the representation of the three games prior for each team playing in the day's game. I decided against using a rolling mean because I wanted to capture more of the game-to-game variability present in the data.


#### Exploratory Data Analysis

- The game totals over all 5 seasons were mostly normally distributed, with a mean around 195.

![][distribution]

[distribution]: technical_report_graphs/download.png

- The offensive rating statistic for a single game appears to be highly correlated with a game's total score. This observation make sense intuitively (The better a team is performing offensively, the higher the score of a game should be). Due to the correlation, I'm guessing the offensive rating of previous games will have a statistically significant impact on the game totals for a future game.

![][heat]

[heat]: technical_report_graphs/heat_map.png


- Pushes occur more infrequently than the other outcomes, occurring in about 1.4% of games, compared to around 49% for Over and Under outcomes. 

![][frequency]

[frequency]: frequency_of_outcomes.png

<a id = 'model'></a>
### Modeling 
When splitting my data into training and testing sets, I split on the seasons, putting the first 4 seasons in my training data, and putting the last season in my testing data to make predictions on. 

I chose a few different types of models, chief among them Logistic Regression and Random Forest Classifier models. Becuase I am cross-validating my models, I need to account for the time series component when training my models. To do this, I used the TimeSeriesSplit module to properly cross-validate my training data. 

Principal Component Analysis was not useful working with my dataset, it did not perform better than other model that I used.

Here are my results:

- Logistic Regression (All features)

|   (Scoring: Negative Log Loss)   | Training Data Score | Testing Data Score |
|:--------------------------------:|:-------------------:|:------------------:|
|  Logistic Regression (Over Bets) |        0.552        |        0.511       |
| Logistic Regression (Under Bets) |        0.563        |        0.514       |

- Random Forest Classifier (All features, GridSearch)

|     (Scoring: ROC-AUC)     | Training Data Score | Testing Data Score |
|:--------------------------:|:-------------------:|:------------------:|
|  Random Forest (Over Bets) |        0.715        |        0.517       |
| Random Forest (Under Bets) |        0.993        |        0.487       |

- Logistic Regression (w/ select 150 KBest features, GridSearch)

|          (Scoring: ROC-AUC, Features: 150)         | Training Data Score | Testing Data Score |
|:--------------------------------------------------:|:-------------------:|:------------------:|
|  Logistic Regression w/ KBest Features (Over Bets) |        0.571        |        0.520       |
| Logistic Regression w/ KBest Features (Under Bets) |        0.572        |        0.530       |

- Logistic Regression (w/ Principal Component Analysis, GridSearch)

|            (Scoring: ROC-AUC)           | Training Data Score | Testing Data Score |
|:---------------------------------------:|:-------------------:|:------------------:|
|  Logistic Regression w/ PCA (Over Bets) |        0.557        |        0.505       |
| Logistic Regression w/ PCA (Under Bets) |        0.556        |        0.508       |

- Logistic Regression (w/ SelectFromModel features, Gridsearch)

|    (Scoring: ROC-AUC, Features: SelectFromModel)    | Training Data Score | Testing Data Score |
|:---------------------------------------------------:|:-------------------:|:------------------:|
|  Logistic Regression w/ SelectFromModel (Over Bets) |        0.586        |        0.528       |
| Logistic Regression w/ SelectFromModel (Under Bets) |        0.584        |        0.530       |

- A note on the bets-won columns that I'm predicting on. I decieded not to account for pushes when predicting the probablity of an outcome occurring for a game. A push occurs when the result of a game or event lands right on the listed point spread, or the game ends in a draw. For my data, a push refers to the situation where a game's point total is equal to the bookie's listed point total. In the case of a push, the bet is considered as if it had never happened, and all of the money gambled is returned. In my observed dataset, games where I had a push occurred, as stated in my EDA, 1.4% of the time. Because pushes represented a small percentage of the outcomes, and it's outcome is more of a null factor than a win or a loss in terms of the money bet, I decided to not explicitly encode for pushes in my model, instead opting to encode them as "not-wins."


#### Model Interpretation/Plotting: 

The model which performed the best out of those I tested was my GridSearched Logistic Regression Model, with feature selection done by SelectFromModel. SelectFromModel is a method of feature selection which chooses the most important features from a model based on their importance weights. A feature with a level of importance smaller than a given threshold will be dropped. Using SelectFromModel, I was left with 228 features.

The features with the greatest importance in my model are shown below:

|    Features    | ft_1_3_opp | ft_1_3 | fga_1_1_opp | fga_1_1 | fg%_1_1 | fg%_1_1_opp | off_rating_2_1 |
|:--------------:|:----------:|:------:|-------------|---------|---------|-------------|----------------|
| Feature Weight | 0.1918     | 0.1918 | 0.1875      | 0.1875  | 0.1693  | 0.1693      | 0.1577         |


|    Features    | fta_1_3_opp | fta_1_3 | pts_2_1_opp | pts_2_1 | game_total_score_1 | game_total_score_1_opp | fta_2_3 |
|:--------------:|:-----------:|:-------:|-------------|---------|--------------------|------------------------|---------|
| Feature Weight | -0.2197     | -0.2197 | -0.1873     | -0.1873 | -0.1590            | -0.1590                | -0.1551 |

- Interesting that the game total score for the previous game takes on a negative weight in my model. I do not think the feature loadings in my model offer significant insight into features that makes sense in terms of explaining the variabiltiy captured by the models; stats from previous games seems to be both positively and negatively affecting my model's prediction for a game to be either over or under. My best guess for the variance my model is accounting for is that there is measurable shift in how the bookkeeper is adjusting the lines to account for the previous games results.


Below are the plots of my confident bet predictions, shown as how the predictions change as the threshold for what is considered "confident" is changed. The line at 54% for the mean of confident predictions represents a level below which betting on the confident predictions would not return money. The baseline for predicting the total of a game stands at around 49% for both overs and unders. Considering a typical bet, where one bets $105 to win $100, 54% represents a level at which the number of confident bets that win overcomes the loss of money from the terms of the bet.

##### Under


![][under]

[under]: technical_report_graphs/UNDER_model_graphs.png

##### Over

![][over]

[over]: technical_report_graphs/OVER_model_graphs.png



#### Simulation

As shown in my modeling notebook, I created a basic simulation, combining the predictions for each game in the 2018 NBA season. Starting with $10,000, and making a bet each time my models predicted a confident bet (here set at a prediction with > 60% confidence), I show how my model performs informing a betting strategy over the course of the season. 

![][pooled]

[pooled]: technical_report_graphs/pooled_bets_graph.png

<a id = 'conclusion'></a>
### Conclusion
Everybody who gambles is looking for an edge. Through an exploration of using basic machine learning models to predict the total score of a game, as it relates to betting lines, I have learned a good deal about how inherently difficult it is to model this problem, due to inconsistency from a game to game perspective, and also how efficient the methods have become for setting these lines. 

Although I was able to achieve a predictive capability with my model high enough to win money using it to bet game totals, I think that the rate of confident predictions is a bit low. There were 174 "confident predictions" (for the purpose of this model, here confident means prediction was > 62% confident in picking the winning class). Because I have each game duplicated for the purposes of capturing each team's perspective on a game, I will be able to make "confident" bets on 104 games. While this figure is significant, for a professional sports gambler who may want to be in the action constantly, I doubt this model can be used as a the comprehensive blueprint for creating an optimal betting strategy.



#### Next Steps: 
- Oftentimes (particularly since the advent of mobile technology), bookeapers will offer live-betting, offering continually updated odds on different aspects of the game. I would want to look at using Bayesian statistics to predict the final over/under for a game, with halftime statistics for a live game.
- I think there are a number of features, both from an in-game statistical perspective(PER, pace statistics) and from a context standpoint (the distance each team has traveled in the past week) which could be imputed to try and explain more of the variance present in the game totals.
- I think it would be worthwhile to calculate the rolling mean for the past *x* games and see how imputing an average of a team's stats over a period of time compares to the way I have the previous games' stats imputed now (as a collection of the 3 prior games).
- An ensemble model may be able to take the various predictions returned by the models I created, in addition to others, and return a more confident prediction on certain games. I'm unsure how much optimization can be done on this dataset to maximize prediction confidence, however, because of the lack of significant correlation between the features and the target.



