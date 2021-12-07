# Brazilian-National-Soccer---Predicting-Match-Results
Web scraping, cleaning, and analyzing data to predict match results
--------
### Why doing this?
I realized that every single Machine Learning model related to soccer that people present on internet focus on number of goals or some other statistics, like shoots, faults or corners, but people are forgetting about the most important data. All those numbers are made by real humans: the players. It does not matter if Barcelona is facing Granada. What does really matter is who plays for each team. Of course, with their best respective players, Barcelona should have a greater chance of winning, but if Barcelona puts only the under-16 players, probably the odds will turn to Granada. So, I decided to get data about the 11 players who started the matches. There is a chance of having players with the same name in different teams so I put a prefix for everyone, so the player 'Gil' from Corinthians will be displayed as 'COR_gil', the player 'Felipe alves' from Fortaleza will be displayed as 'FOR_felipealves' and so forth...

First, I want to convince you that using the players' name to predict the result can be useful in some situations. Let's check the games where Fortaleza played with and without the player Lucas Crispim.

```
victories_with_lucascrispim = table[((table['HMID5'] == 'FOR_lucascrispim') & (table['FULLRESULT'] == 'H')) | ((table['AMID5'] == 'FOR_lucascrispim') & (table['FULLRESULT'] == 'A'))]

victories_without_lucascrispim = table[((table['HTEAM'] == 'FORTALEZA') & (table['HMID5'] != 'FOR_lucascrispim') & (table['FULLRESULT'] == 'H')) | ((table['ATEAM'] == 'FORTALEZA') & (table['AMID5'] != 'FOR_lucascrispim') & (table['FULLRESULT'] == 'A'))]

total_matches_with_lucascrispim = table[(table['HMID5'] == 'FOR_lucascrispim') | (table['AMID5'] == 'FOR_lucascrispim')]

total_matches_without_lucascrispim = table[((table['HTEAM'] == 'FORTALEZA') & (table['HMID5'] != 'FOR_lucascrispim')) | ((table['ATEAM'] == 'FORTALEZA') & (table['AMID5'] != 'FOR_lucascrispim'))]
```
Let's plot the results:

![Fortaleza matches with and without Lucas Crispim](/img/LucasCrispim.png)
<br>
Those are the results from Matchweek 1 to Matchweek 36:
- Total matches with Lucas Crispim: 21
- Victories with Lucas Crispim: 12 (57.14%)
- Total matches without Lucas Cristpim: 15
- Victories without Lucas Crispim: 4 (26.66%)

As we can see, with this player, Fortaleza has twice the chance of winning a match than without him.
---------
# Webscraping the data


The two greater Brazilian Soccer Leagues (Série A and Série B) have 20 teams each, which means that each league will have 380 matches. Getting these data manually would be painful, as each match starts with 22 players. Also, we could make lots of mistakes if we try this. My solution was to create a bot that goes to some website and grasp the useful information. I used two tools to build this bot: Python and Selenium.
<br><br>
You can check the full code here at the [BRScraping.ipynb](https://github.com/mathfigueiredo/Brazilian-National-Soccer---Predicting-Match-Results/blob/main/BRScraping.ipynb) notebook.
<br><br>
It is very simple to use this bot. All you should do is run the **append_matchweeks** function with two parameters:
- start_matchweek: the first matchweek to grasp.
- end_matchweek: the last matchweek to grasp. Together, they should be an interval.
<br>
i.e.: to get the data from matchweek 1 and matchweek 36, just call the function *append_matchweeks(1,36)*
<br>
But remember: you should have a **table** Pandas Dataframe so the function can append the gathered data to this table. If you don't have this table yet, make an empty DataFrame with those columns to start:

```
columns = ['MATCHWEEK','HTEAM','ATEAM','HGK','HDEF1','HDEF2','HDEF3','HDEF4','HDEF5','HMID1','HMID2','HMID3','HMID4','HMID5','HMID6','HFOR1','HFOR2','HFOR3','HFOR4','HFOR5','AGK','ADEF1','ADEF2','ADEF3','ADEF4','ADEF5','AMID1','AMID2','AMID3','AMID4','AMID5','AMID6','AFOR1','AFOR2','AFOR3','AFOR4','AFOR5','HFULLGOAL','AFULLGOAL','FULLRESULT']

table = pd.DataFrame(columns=columns)
```
<br>
If you already have a table with some matchweeks in it, just load it with pandas read_csv function:

```
table = pd.read_csv('table.csv')
```

<br>
It is always good to check the consistency of the data. Sometimes, due to instabilities on internet connection or on the website server, the bot can get wrong data or duplicated observations. We can check countplots and duplicated rows just to check if all teams have the amount of games that we expect.
<br>
i.e.: for the matchweek 36, we want all teams to have exactly 18 matches as Hometeam and the same amount as Awayteam:
<br>

![Awayteam checking](/img/ATEAM_check.png)

Let's try to make this section not so long, but you can find other checkings directly in the notebook.


That's it for WebScraping. We also find a section on the notebook to get Features (X) and Target (y) and save it with pickle and the table itself in csv, in case we need some of those useful files.

----------
# Tuning model parameters (using GridSearchCV)
You can check the [Tuning notebook](https://github.com/mathfigueiredo/Brazilian-National-Soccer---Predicting-Match-Results/blob/main/Tuning.ipynb) to see how I used GridSearchCV to achieve the best result of each parameter. First, I found the best parameters for each model. Then, I tried 30 differents train and test splits for each model and got a table with some information about the accuracy scores foundm. As you can see in the table below, the best model for this dataset seems to be the Random Forest Classifier, with an accuracy score mean of **0.47** and std **0.013**.


![Tuning results](/img/tuning_results.png)

--------
# Finally... Predicting
