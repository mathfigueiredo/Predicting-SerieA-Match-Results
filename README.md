# Brazilian-National-Soccer---Predicting-Match-Results
Web scraping, cleaning, and analyzing data to predict match results

-------

### Summary
1. [Things you should know](#things-you-should-know)
2. [Why am I doing this?](#why-am-i-doing-this)
3. [Knowing our dataset](#knowing-our-dataset)
4. [Webscraping our data](#webscraping-our-data)
5. [Tuning model parameters (using GridSearchCV)](#tuning-model-parameters-using-gridsearchcv)
6. [Testing our model](#testing-our-model)
7. [Finally... Predicting!](#finally-predicting)
8. [Feel free to contact me!](#thank-you-for-reaching-here)
--------
### Things you should know
- We are going to get our data from the [Globo Esporte website](https://ge.globo.com/futebol/brasileirao-serie-a/)
- To make real time predictions, we should go to the website provided above, look for a specific match and get the link at the bottom of the match container (the green text saying (**Acompanhe em tempo real**)
- This text normally appears 60 to 30 minutes before the match starts, with all the players and tacticals of each team, which we will use to make our predictions
- In the datasets of this project, you will see the columns **result** and **pred** (prediction). They can assume 3 values:
    - **'H'** (Home team wins)
    - **'A'** (Away team wins) and
    - **'D'** (Draw)

--------

### Why am I doing this?
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

### Knowing our dataset
To understand our dataset:

Each team uses different sets of tactical arrangements, so I decided to separete them in 4 major positions:
- GK: Goalkeeper (1)
- DEF: Defenders (max. 5)
- MID: Midfielders (max. 6)
- FOR: Forward (max. 5)


The initial letter of each player columns are H (Home) or A (Away):
- HDEF1 stands for Home team Defender 1
- AMID3 stands for Away team Midfielder 3
- and so forth...
- When there is no player in such position, we use 'none'

The last 3 columns are HFULLGOAL (Home goals), AFULLGOAL(Away goals), FULLRESULT (Result of match).

----------

### Webscraping our data
The two greater Brazilian Soccer Leagues (Série A and Série B) have 20 teams each, which means that each league will have 380 matches. Getting these data manually would be painful, as each match starts with 22 players. Also, we could make lots of mistakes if we try this. My solution was to create a bot that goes to some website and grasp the useful information. I used two tools to build this bot: Python and Selenium.
<br>
You can check the full code here at the [BRScraping.ipynb](https://github.com/mathfigueiredo/Brazilian-National-Soccer---Predicting-Match-Results/blob/main/BRScraping.ipynb) notebook.


It is very simple to use this bot. All you should do is run the **append_matchweeks** function with two parameters:
- start_matchweek: the first matchweek to grasp.
- end_matchweek: the last matchweek to grasp. Together, they should be an interval.


i.e.: to get the data from matchweek 1 and matchweek 36, just call the function *append_matchweeks(1,36)* <br>
i.e.: to get data from matchweek 28, call the function *append_matchweeks(28,28)*


But remember: you should have a **table** Pandas Dataframe so the function can append the gathered data to this table. If you don't have this table yet, make an empty DataFrame with those columns to start:

```
columns = ['MATCHWEEK','HTEAM','ATEAM','HGK','HDEF1','HDEF2','HDEF3','HDEF4','HDEF5','HMID1','HMID2','HMID3','HMID4','HMID5','HMID6','HFOR1','HFOR2','HFOR3','HFOR4','HFOR5','AGK','ADEF1','ADEF2','ADEF3','ADEF4','ADEF5','AMID1','AMID2','AMID3','AMID4','AMID5','AMID6','AFOR1','AFOR2','AFOR3','AFOR4','AFOR5','HFULLGOAL','AFULLGOAL','FULLRESULT']

table = pd.DataFrame(columns=columns)
```

If you already have a table with some matchweeks in it, just load it with pandas read_csv function:

```
table = pd.read_csv('BR-21.csv')
```
In my case, I saved it with the name 'BR-21.csv'. In this line, I'm just assigning my saved table to the **table** variable.


It is always good to check the consistency of our data. Sometimes, due to instabilities on internet connection or on the website server, our bot can get wrong data or duplicated observations. We can check countplots and duplicated rows just to be sure that all teams have the amount of games that we expect.


i.e.: for the matchweek 36, we want all teams to have exactly 18 matches as Hometeam and the same amount as Awayteam:


![Awayteam checking](/img/ATEAM_check.png)

Let's try to make this section not so long, but you can find other checkings directly in the notebook.


That's it for WebScraping. We also find a section on the notebook to get Features (X) and Target (y) and save it with pickle and the table itself in csv, in case we need some of those useful files later.

----------

### Tuning model parameters (using GridSearchCV)
You can check the [Tuning notebook](https://github.com/mathfigueiredo/Brazilian-National-Soccer---Predicting-Match-Results/blob/main/Tuning.ipynb) to see how I used GridSearchCV to achieve the best result of each parameter. First, I found the best parameters for each model. Then, I tried 30 differents train and test splits for each model and got a table with some information about the accuracy scores found. As you can see in the table below, the best model for this dataset seems to be the Random Forest Classifier, with an accuracy score mean of **0.47** and std **0.013**.


![Tuning results](/img/tuning_results.png)

--------

### Testing our Model
You can check out the [Predicting notebook](https://github.com/mathfigueiredo/Brazilian-National-Soccer---Predicting-Match-Results/blob/main/Predict.ipynb) and make your own analysis.

First, we need to load the **table** (named exactly like this):

```
table = pd.read_csv('BR-21.csv')
```

I coded a function called **random_forest_matchweek** to check the predictions and the real results of past matches. It receives a single parameter (*matchweek*). This is how it works:
- If we call *random_forest_matchweek(15)*, it will split our data from matchweek 1 to matchweek 14 and predict the result of matchweek 15 and the probabilities of Hometeam winning, Awayteam winning and Draw. Then, it brings the real results of the matches to compare, printing the accuracy of predictions from 0 (0%) to 1 (100%).
<br>
This function is very useful because we can simulate real life scenarios. If we pass some matchweek to the function, it will train the algorithm with the data of past matches and predict results using only data that was available at the moment of our matchweek.

Check out the output of calling random_forest_matchweek(34):

![example of random forest matchweek function](/img/random_forest_matchweek_example.png)

The function created a model and trained it with data from matchweek 1 to matchweek 33. As you can see, it predicted 60% of results correctly.


So, to test how our model would perform through the year, let's consider predictions from matchweek 8 to the end. Why? Because 8 matchweeks are enough to get at least 3 matches of each team as home team and as away team, so the model can use these data to make the first predictions. It would not be smart to start predicting results from the second or third matchweek, because we just could not have consistent data to train our model.

What we will do here is create a DataFrame to keep our predictions:

```
all_predictions = pd.DataFrame(columns = ['Home','Away','result','pred','probH','probD','probA'])
```
Note:
- result: real result of the match
- pred: our prediction
- probH: probability of Home team winning
- probD: probability of Draw
- probA: probability of Away team winning


Then, we want to predict all matchweeks since 8th and append them to our new DataFrame **all_predictions.**

```
for i in np.arange(8,37):
    t = random_forest_matchweek(i)
    all_predictions = all_predictions.append(t)
```

This will take a while and will print the accuracy score of each matchweek.

Now, with some code that you can check directly in the notebook, we transform the probabilities columns into integers columns and create two functions to determine the precision of our model:
- calc_precision: this function simply receives two parameters (df, result) and calculate the amount of right predictions of the passed result.
- calc_precision_with_perc: this function gives us more detailed information about the precisions based on the probabilities found by our model.

Using **calc_precision**:

```
print("Precision when predicting H: {:.2f}".format(calc_precision(all_predictions,'H')))
print("Precision when predicting A: {:.2f}".format(calc_precision(all_predictions,'A')))
print("Precision when predicting D: {:.2f}".format(calc_precision(all_predictions,'D')))
```

Output:

![calc precision example](/img/calc_precision_function.png)

Of course we would be happier with a more accurate predict, but let's keep in mind that match results are not normal distributions and we can get more right predictions with our model than with some random classification. Let's check that out.

```
print("Percentage of real result H starting from matchweek 8: {:.2f}".format(len(all_predictions[all_predictions['result'] == 'H']) / len(all_predictions)))
print("Percentage of real result A starting from matchweek 8: {:.2f}".format(len(all_predictions[all_predictions['result'] == 'A']) / len(all_predictions)))
print("Percentage of real result D starting from matchweek 8: {:.2f}".format(len(all_predictions[all_predictions['result'] == 'D']) / len(all_predictions)))
```

Output:

![percentage of real result](/img/percentage_of_real_result.png)

As we can see, if we'd say all the matches would be H, A or D, we would get a worse score than with our model.
But we can extract more refined statements from this. Let's consider the precision score based on the probabilities given by our model.

Let's create a DataFrame for each possible predicted value (H, A and D), using **calc_precision_with_perc** function to get more refined information about precision and the amount of occurencies for each probability predicted by our model (the code is a bit long, so, please check it out in the notebook).

The output looks like this:

![calc percentage with perc function example](/img/calc_precision_with_perc_function.png)

Wow. Calm down. Let's relax and analyse this responsibly.<br>
We can see that:
- when our model predicts **'H'** with a probability of **at least 50%**, we have a precision of **63%.** It occured **55** times (35:heavy_check_mark: / 20:x:)
- when our model predicts **'H'** with a probability of **at least 60%**, we have a precision of **65%.** It occured **20** times (13:heavy_check_mark: / 7:x:)
- when our model predicts **'D'** with a probability of **at least 55%**, we have a precision of **43%.** It occured **14** times (6:heavy_check_mark: / 8:x:)
- when our model predicts **'D'** with a probability of **at least 65%**, we have a precision of **100%.** It occured **2** times (2:heavy_check_mark: / 0:x:)
- when our model predicts **'A'** with a probability of **at least 55%**, we have a precision of **33%.** It occured **6** times (2:heavy_check_mark: / 4:x:)
- when our model predicts **'A'** with a probability of **at least 60%**, we have a precision of **50%.** It occured **2** times (1:heavy_check_mark: / 1:x:)

-------

### Finally... Predicting!
To make real time predictions, we should go to the [Globo Esporte website](https://ge.globo.com/futebol/brasileirao-serie-a/) 60 to 30 minutes before our match start and click on the green button saying **Acompanhe em tempo real**. It will take us to the specific match webpage containing information about the players and tacticals of each team.

The only thing we need to do now is create a python list **links** with all the the match links we want to predict. Each link should be a string and if there is more than one, separate them by comma.

```
links = [
    'https://ge.globo.com/rj/futebol/brasileirao-serie-a/jogo/06-12-2021/flamengo-santos.ghtml',
    'https://ge.globo.com/rs/futebol/brasileirao-serie-a/jogo/06-12-2021/internacional-atletico-go.ghtml',
    'https://ge.globo.com/mt/futebol/brasileirao-serie-a/jogo/06-12-2021/cuiaba-fortaleza.ghtml'
]
```

Finally, just call **predict(links)**. The predict function receives just one parameter: a list containing the direct links to the match webpage. It will now make our bot work to get all the information needed and print out the predictions.

-----

### Thank you for reaching here!
If you have anything to talk to me, feel free to contact me at:<br>
Email: contact@mathfigueiredo.com <br>
Linkedin: https://www.linkedin.com/in/mathfigueiredo <br>
Professional Portfolio: https://mathfigueiredo.com <br>

