Note: This project is still in progress

# Capstone Project
Capstone project ATP betting

# Betting

# Inital data

## Data cleaning

# elo-score
The elo-score / elo-rating is a value that describes the performance in the past of the player. It takes into account the wins and losses in the past but also the performance of the opponent. So a success against a strong comtetitor has a larger gain as against a weak opponent. Further information on [Wiki](https://en.wikipedia.org/wiki/Elo_rating_system)

The calculation-method of the elo-score is not unique and the algorithm is empirical with variats. Here is a rather simple implementation of the calculation. The *K-factor* e.g. is set constant to 32, and could be seen as *hyer-parameter*. (Working on that is for the future) 

The function is stored in [Python/elo_features.py](Python/elo_features.py)

It figured out that this feature belongs to the most important ones in this project.

(TODO: Create also Surface-dependant elo-scores for each underground)

# Tax

Due to the fact that I live in Germany I have to take into account that a tax of 5% at each bet has to be paid. This decreases the profitability and rises the benchmark to be reached. I don't know if and how there are similar taxes in other countries, I'll drag it along the code.

For comparability issues we'll bet 1 unit of money whichever currency and I don't think that it matters if it is € or Dollar.

Some bookmakers drop the tax from the initial wager. So the effective wager is only 95 cent we bet with. Some other bookmakers pull off 5% of the cashback in case of win. That dereases the gain more and the difference is another profit of the bookmaker. But there are also bookmakers where the tax is implemented in the odds. I don't know how far in this cases the odds and the profit is lower. I assume that there are not different odds just for one county and it is a mixed calculation over all betters over the world.

<img src="figures/eff_odds.png">


<img src="figures/req_perc.png">

As you can see, the tax matters not only a bit. It distorts straight average strategies. It also disturbs the balance between winning rate, odds, risk and confidence if an outcome appears. There is a threshold of odds / rates for the winning zone because treasury aggrevates the success in betting.

So I would expect that it makes no sense to bet on Nadal due to rather low odds (1.01) even it is nearly obvious that he wins.

Note: I observed that it looks like some bookmakers offer betting with a wager of 50 cent. The deducted tax is "only" 2 cent due to rounding issues. This reduces the tax to 4%. I don't estimate that this fact matter to the way reaching a significant gain.
 

# EDA
So let's have a look on the data we have and try if there is a way to make profit with simple strategies.

How to figure out the better player where we are confident that he wins?



## Distribution of outcomes with better elo and rank
Have a look on players that have a better elo-score **AND** a better ATP-ranking.
The following shows if the elo-score or the ATP-rank is suitable for the prediction of the right outcome. It does not consider the ROI.

<img src="figures/match_amount_1.png">

As you see in the figure above the differene of the elo-score promises a bit more chance that the player wins and that the much higher rank in the ATP list. The higher the difference of the elo-score the higher the probability to win. But according to this fact the odds will also fall towards the winner and the amount of such matches falls. 

On the one hand we could say, "So what..." let's bet just on the few nearly sure matches, just pick the matches above $\Delta$-elo 400, but on the other hand we have to keep in mind in our case the tax and the needed ratio of wins at all.

Let's see next if we want a minimum of odds. At least 1.06 for the tax and further 0.02 for our deposit, so 1.08.

<img src="figures/match_amount_2.png">

As you see this strategy wouldn't work properly, because very few matches let to bet over years. The ROI including the losses wouldn't look glorious also, I think. 

Let's have look how it is if we only consider matches of the *$1^{st}$ Round*, the differenc between the competitors should be larger. And if a weak player comes to the finals, he had a good run or a lucky week.

<img src="figures/match_amount_3.png">

So this look ok, due to the fact that the ratio of won matches is a bit higher than before, **but** we left the minimum odds at 1.0! **And** the total amount of matches decreased, see the scale of the upper graphs.

**Conclusion:** The elo-score is the better indicator than the ATP-ranking. But in the moment it is not sure if it helps us to make profit. We can estimate that a higher elo difference will cause lower odds and also the chance to make profit.(Maybe the ML Model could give us more confidence...)

## Simple strategy?
With the information above we can try to develop a simple strategy to make profit. Of course other Kaggle competitors already have shown, just betting on the favorite / the player with odds of a favorite, does not push the ROI very well. 

But let's have a try by using the differences of the rank and the elo-score... Maybe we find also the underestimated underdogs...

With a numpy meshgrid several constellations of +/- delta elo-score and +/- delta ATP-rank are observed. With pandas the matches are filteres out and the profit calculated. The observations reach over 10 years, from the beginning of 2009 until the end of 2019. 

I look on the strategies if there is a profit at all. In a second step I check if setting a minimum odd makes it better. Finally I check if with tax a possible profit is still there.

The animated development of the ROI according to the strategies with the different parameters are [here](Animated_gifs)

### Strategy: Betting the better
Straight forward, just betting on all matches with at least the shown delta values.

As you see there is no constellation 


### Strategy: Betting crossover 1

### Strategy: Betting crossover 2


# Feature engineering
Based on the given data several other features can be generated. 

## Past features

## One-hot features
The initial features "Series", "Court", "Surface", "Round", "Best of", "Tournament" were converted to encoded bool features. 
The player names were also one-hot encoded. For each outcome the name of the player and the name of the opponent.

## Missing values
Due to the fact that some past features are not applicapable everytime. So if a player had no matches durching the last few days the ratio of won matches e.g. is a NaN. Set it to 0 is not suitable because it would be a false information. Especially in the *duo_features*, that means the direct comparison of the players, contain many missing values. The chance that exactly these players matched each other in the rather short time period inspected (150 days) is low. Of course it can be discussed if such features are useful at all. But I would say that they are at least not useless...


# Prediction with XGBoost
The attempt is to predict the outcomes right and increase the ROI.
The decision to use XGBoost is based the fact that it can handle missing values. The missing values were generated in the features. So the features are not ready to use in an ANN or AdaBoost with trees or forests.

(To make it short: I didn't find a predictive model for long term gain yet...)

## Models
It figured out that the XGB models is very very sensitive to the train data. A slight variation of the train and eval data gives complete different scores and also the predictions of the test set is different. Due to this I used several equal models but with slightly different train data. So in this cas it is an ensemble of 9 XGB models with the hope that their common opinion will rise the ROI.

I played with some parameters, especially *learning rate, max_depth* and *early stopping*, on the one hand to increse the precision and the ROI on the other hand to avoid overfitting. I observed that better train loss often has a worse eval loss. That means that there remained a lot of work... I assume that it depends on the XGB-parameters but maybe also on the features.

## Split the data
The data is split to a train set, an eval set and a test set. The reference point is the start date of testing. A few hundred samples before a used as eval set and several thousend samples befor the eval set is used as train set. A few hundred after this point are used for testing.

The different models are trained with slightly different train and eval sets. For that there is a variation of the lenght of the eval set (the differences are in the range +/- 70)

## Loss function / scorer
On the one hand we want to predict the right outcome. Especially we want to decrease the amount of *false positive* predictions so the *error* (1-accuracy) could be an appropriate scorer to minimize. (Until now in test runs the eval error was above 0.3 that means that right predictions were below 70%. Afterwarts calculated ROI was negative...)

On the other hand we want to maximize the ROI / gain. So I implemented a scorer that calculates the absolute gain of the predictions according to the odds. Due to the fact that we use a gradient **decent** method the has to be flipped to negative, because XGB wants to minimize the score. The problem is that there is no chace to pass the odds to the function. It is not possible to extract the odds as feature from the passed DMatrix. For each run the scorer function has to be created new according to the train/eval/test set with the odds as constant values in it. In the scorer a kind of thershold also can be set as hyperparameter. 

Due to the fact that each match has two samples both outcomes are treated as unique matches. I observed in test runs that sometimes the prediction for both outcomes is high (above 0,5). For that they are compared and lower one is set to 0 and the higher one passed as it is.

## Voting systems
Due to the fact that the XGB models are so instable and different in their results a voting system is required and also a strategy how to compute the confidence.
Several systems are implemented.

### Confidence
How to make the confidence for the decision? There are a some different attempts to built a confidence.
- Straight probability. Just take the probabilities of the predictors `conf=proba`.
- Confidence by the odds. Division of the probabilities by the odds  `conf=proba/odd`. It's a kind of confirmation of the opinion. If the odd is low (towards 1) and the proba high the confidence will rise. If the odd rises (lower confidence of the bookmaker / other betters) the confidence decreases.
- Log_odd confidence. The idea is to punish the probas by the odds so that the predictors have to be very sure to make a decision. `conf=log10(odd)*(10**(proba))` Assuming that if there are higher odds the probas won't be as high. It also avoids that the majority of decisions won't bet on matches with low odds where also the gain is low and the chance/risk balance isn't very good.

### Voting
For the common opinion/decision according to the above mentioned confidences of the models here are:
- The mean of the confidences of the models. Then check if the mean if above the threshold.
- Majority voting. Count how many modles confidence is above the threshold. The amount of confident models is also a hyperparameter. Due to this the amount of models is not even.

Note: The different confidence starategies may need different thresholds.

### Prediction and results

# Future work
There are several points that can be improved for better results:

- calculate surface dependent elo-scores
- maybe imrove the fucntion for the elo-score
- parallelize the calculation of the features for accelerated computation and playing with the hyperparameters in the features.
- Try XGBmodels without odds in the features. The feature importance of the odds seems to be quite high so they have a large influence on the decisions. But the odds gives back the opinion / confidence of the bookmaker and other betters. I think it would be better to make an own opinion without the influence of the odds. Or maybe at least a combination of it in an ensemble that some models calculte with odds and others without and then check the voting.
- Find a way to handle the missing values un the generated features for usage with other predictors that do not accept NaNs. Then try ANN, Random Forests e.g. maybe with other boosters.
- Improve the precision. Check if the gain as loss function is the expediant way.
- Work / optimization of confidence calculation.
- Improve the voting system
- Filtering *samples* of best players and best tournament right before the model training and not only the *one-hot features*.
- Maybe implement a kind of gridsearch for the models to figure out better parameters. Due to the split and the scorer it won't be as easy.
- Redesign the whole system to make only one sample per match and let the model directly decide who wins.
- Voting by another model (ANN?)

# Appendix

## Features
