# Brazilian-National-Soccer---Predicting-Match-Results
Web scraping, cleaning, and analyzing data to predict match results
--------
### Why doing this?
I realized that every single Machine Learning model related to soccer that people present on internet focus on number of goals or some other statistics, like shoots, faults or corners, but people are forgetting about the most important data. All those numbers are made by real humans: the players. It dows not matter if Barcelona is facing Granada. What does really matter is who plays for each team. Of course, with their best respective players, Barcelona should have a greater chance of winning, but if Barcelona puts only the under-16 players, probably the odds will turn to Granada. So, I decided to get data about the 11 players who started the matches. There is a chance of having players with the same name in different teams so I put a prefix for everyone, so the player 'Gil' from Corinthians will be displayed as 'COR_gil', the player 'Felipe alves' from Fortaleza will be displayed as 'FOR_felipealves' and so forth...

First, I want to convince you that using the players name to predict the result can be useful in some situations. Let's check the games where Fortaleza played with and without the players Lucas Crispim.

The two greater Brazilian Soccer Leagues (Série A and Série B) has 20 teams each, which means that each league will have 380 matches. 
