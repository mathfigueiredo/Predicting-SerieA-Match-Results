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

![Fortaleza matches with and without Lucas Crispim](/LucasCrispim.png)
<br>
Those are the results from Matchweek 1 to Matchweek 36:
- Total matches with Lucas Crispim: 21
- Victories with Lucas Crispim: 12 (57.14%)
- Total matches without Lucas Cristpim: 15
- Victories without Lucas Crispim: 4 (26.66%)

As we can see, with this player, Fortaleza has twice the chance of winning a match than without him.

# Webscraping the data


The two greater Brazilian Soccer Leagues (Série A and Série B) has 20 teams each, which means that each league will have 380 matches. 
