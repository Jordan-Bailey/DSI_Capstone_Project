## Predicting Totals Bets for NBA Games

Problem: This project attempts to predict whether an NBA game's total score will be over or under a predetermined total line, 
set by Las Vegas casinos. In sports betting, a bookkeaper will create a bet called a total on most team games. The total is set at the predicted total score for the game; with knowledge of this total, you are able to bet an "Over," betting that the total score of the game will be higher than the total set by the bookeaper, or you can bet an "Under," a number at which the bookkeaper believes will create action for both "Over" bets and "Under" bets. The goal of this project is to create a model which can in predicting these total bets, return a number of what I will term "confident bets," and then create a strategy to profit off of this information. 


Data: I decided to analyze the past 5 season of NBA games for the purposes of this project. For the basketball data I scraped box scores for every game off of basketball-reference.com. These box score stats are comprised of the counting stats (points, assists, rebounds, etc.) for a basketball game, for both the target team and their opponent. I stored this data as a collection of JSON files, ordered by team and season.

With a dataset comprised of the past 5 seasons of NBA game data, I was able to create a list of dates which NBA games were played on in the past 5 seasons. With the list of dates, I was able to scrape gambling lines off of sportsbookreview.com, querying each date and pulling the historical betting lines for each game on that date. 


Data Cleaning: The dataset of box scores from basketball-reference.com, and the dataset of betting lines from sportsbookreview.com, were relatively clean to begin with. 
- There were missing betting lines for a few games; to rectify this I went to additional gambling websites with historical data to manually look up the lines for these games. 
- There was also one game missing entirely from the gambling data, which I then imputed (Miami-Washington, 03/06/2015)
- In 2015 season, the second in my data, the Charlotte Bobcats changes their name to the Charlotte Hornets. I addition, the team slug changed as well, from "CHA" to "CHO." This change caused me some problems, so when cleaning my data, I had to make sure that any reference to a team or team slug accounted for this change.
- I got the date for each game by splitting on the values in my 'gid' column. I then used the dates, as well as the team slugs, to merge the games dataframe and the betting lines dataframe.
- I conducted Regex string matching on the betting lines to change the '1/2' symbol, which because of it's encoding was being picked up by Jupyter Notebook as a special character, to a '.5,' for all the betting lines.


Feature Engineering/Accounting for Time Series: I caluculated a few additional features for my dataset, on top of the box scores stats I scraped off of basketball-reference. 

- I calculated whether each game ended up being an over, an under, or a push(final score same as total line), by subtracting the book's point total and subtracting that from the actual game total. I used these features as my target variables. 

- To predict the total score for a game, I decided to impute each game as the representation of the three games prior for each team playing. I decided against using a rolling mean because I wanted to capture more of the game-to-game variability present in the data.

- I imputed an offensive rating for each team for each game. Offensive rating is an "advanced" statistic (called advanced because it is not a counting stat) which is a summary metric for a team's offensive performance in a game. I imputed this statistic in the hopes it would capture much of the variability present in my other offensive statistics (points, rebounds, assists, etc.)


Exploratory Data Analysis: 
- The game totals were mostly normally distributed, with a mean around 195.
- My baseline scores for each class 

EDA Plotting:

Modeling: When splitting my data into training and testing sets, I split on the seasons, putting the first 4 seasons in my training data, and putting the last season in my testing data to make predictions on. 

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



Model Interpretation/Plotting: 

The model which performed the best out of those I tested was my GridSearched Logistic Regression Model, with feature selection done by SelectFromModel. SelectFromModel is a method of feature selection which chooses the most important features from a model based on their importance weights. A feature with a level of importance smaller than a given threshold will be dropped. Using SelectFromModel, I was left with 228 features.

Conclusion:
Everybody who gambles is looking for an edge. Through an exploration of using basic machine learning models to predict the total score of a game, as it relates to betting lines, I have learned a good deal about how inherently difficult it is to model this problem, due to inconsistency from a game to game perspective, and also how efficient the methods have become for setting these lines. 

Although I was able to achieve a predictive capability with my model high enough to win money using it to bet game totals, I think that the rate of confident predictions is a bit low. There were 184 "confident predictions" (for the purpose of this model, here confident means prediction was > 60% confident in picking the winning class). Because I have each game duplicated for the purposes of capturing each team's perspective on a game, I will be able to make "confident" bets on 92 games. 

Stretch Goals: 
- Oftentimes (particularly since the advent of mobile technology), bookeapers will offer live-betting, offering continually updated odds on different aspects of the game. I would want to look at using Bayesian statistics to predict the final over/under for a game, with halftime statistics for a live game.
- I think there are a number of features, both from an in-game statistical perspective(PER, pace statistics) and from a context standpoint (the distance each team has traveled in the past week) which could be imputed to try and explain more of the variance present in the game totals.
- An ensemble model may be able to take the various predictions returned by the models I created, in addition to others, and return a more confident prediction on certain games. I'm unsure how much optimization can be done on this dataset to maximize prediction confidence, however, because of the lack of significant correlation between the features and the target.


