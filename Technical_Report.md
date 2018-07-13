## Predicting Totals Bets for NBA Games

Problem: This project attempts to predict whether an NBA game's total score will be over or under a predetermined total line, 
set by Las Vegas casinos. In sports betting, a bookkeaper will create a bet called a total on most team games. The total is set at the predicted total score for the game; with knowledge of this total, you are able to bet an "Over," betting that the total score of the game will be higher than the total set by the bookeaper, or you can bet an "Under," a number at which the bookkeaper believes will create action for both "Over" bets and "Under" bets. The goal of this project is to create a model which can discern some advantage/confidence in predicting these total bets, and then create a strategy to profit off of this information. 

Data: I decided to analyze the past 5 season of NBA games for the purposes of this project. For the basketball data I scraped box scores for every game off of basketball-reference.com. These box score stats are comprised of the counting stats (points, assists, rebounds, etc.) for a basketball game, for both the target team and their opponent. 

With this dataset, I was able to create a list of dates which NBA games were played on in the past 5 seasons. With the list of dates, I was able to scrape gambling lines off of sportsbookreview.com, querying each date and pulling the historical betting lines for each game on that date. 

Data Cleaning: The dataset of box scores from basketball-reference.com, and the dataset of betting lines from sportsbookreview.com, were relatively clean to begin with. There were missing betting lines for a few games; to rectify this I went to additional gambling websites with historical data to manually look up the lines for these games. There was also one game missing entirely from the gambling data, which I then imputed (Miami-Washington, 03/06/2015)


Feature Engineering/Accounting for Time Series: I caluculated a few features for my dataset. I calculated whether each game ended up being an over, an under, or a push(final score same as total line), and used these features as my response variables. I wanted a metric that was representative of a team's total 

To predict the total score for a game, I decided to impute each game as the representation of the three games prior for each team playing. I decided against using a rolling mean because I wanted to capture more of the game-to-game variability present in the data.

Exploratory Data Analysis: 

EDA Plotting:

Modeling: I chose a few different types of models 

Model Interpretation/Plotting:

Conclusion:
Everybody who gambles is looking for an edge. Through an exploration of using basic machine learning models to predict the total score of a game, as it relates to betting lines, I have learned a good deal about how inherently difficult it is to model this problem, due to extreme inconsistency from a game to game perspective. 

Stretch Goals: 
- Oftentimes (particularly since the advent of mobile technology), bookeapers will offer live-betting, offering continually updated odds on different aspects of the game. I would want to look at using Bayesian statistics to predict the final over/under for a game, with halftime statistics for a live game.
- I think there are a number of features, both from an in-game statistical perspective(PER, pace statistics) and from a context standpoint (the distance each team has traveled in the past week) which could be imputed to try and explain more of the variance present in the game totals.


