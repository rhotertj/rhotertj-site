+++
date="2023-06-11"
title="Ranking all darts players in Lower-Saxony."
+++

I am a passionate dart player in a club that also participates in the [DBH](https://dbhev.de) ligues.
In my first season I decided to take on the role of team captain. A team captain is responsible for the formation of the team and the organization of the matches.
Since team formation was often a guessing game - I did not know my opponents yet - I decided to create a ranking system to know which opponents to fear and which ones to underestimate ;)

### Rules 
Each dart team has a minimum of four players, although the team size increases in higher leagues.
When two dart teams play against each other, 12 matches are played. The number of singles and doubles varies and depends on the league, but for now we assume 8 singles and 4 doubles are played.
In each half, four singles and two doubles are played, so each player plays two matches. After the first half, it is possible to substitute any number of players. Opponents who played each other in the first half may not play each other in the second half.
If a club has more than one team, players from the lower leagues can help out in the higher leagues up to two times. 

### Motivation
When I think about the lineup of my team, I think about who I want to play against whom. I would look at my opponent's past games and the current standings in the league-wide player rankings. While people who have been playing darts in my town for 20 years would know everyone on those teams, I had no idea who they were. The problem was that since I did not know anyone, I could not possibly know how to evaluate my opponents game history. How would I know whether a loss against Alice and a win against Bob was to be expected because Alice is really good and Bob does not know how to throw - or that this was a very unlikely outcome. 

I wanted to have a ranking system that would give my an overview of the skill of all players in the league. A nice side effect would be that I would also get insights about my own team, which I hardly knew. A ranking system would also be useful next season: Which players were too good for the league? Send them up! Which players were not so good? Maybe a team playing in the lower leagues would be a better fit.

### Approach
After checking out a few ranking systems, I finally settled on [_TrueSkill_](https://www.microsoft.com/en-us/research/project/trueskill-ranking-system/). TrueSkill was developed by _Microsoft Research_ to improve matchmaking on XBOX Live. They wanted to estimate the skill of each player and create teams in online games to maximize the likelihood of a draw. This can be considered a very fair matchmaking process. Of course, I want to have an unfair matchmaking process that maximizes my team's probability of winning!

A _TrueSkill_ rating is between 0 and 50. To estimate a player's skill, _TrueSkill_ uses a normal distribution with default values for mean μ=25 and standard deviation σ=8.333, which covers all observable skills within μ ± (3 * σ). When a player defeats another player, their skills are updated. With each game played, the standard distribution of a player's rank decreases. The mean is updated based on the players' rankings. We expect the *better player* to win, which, assuming the rankings are correct, would mean that the *higher ranked player* wins. In this case, we slightly increase the mean of the winning player, slightly decrease the mean of the losing player, and decrease both standard deviations. In the case that the lower ranked player wins, we decrease the loser's ranking by a large amount and increase the winner's mean by a large amount. The detailed equations can be found in the [_TrueSkill_ paper](https://www.microsoft.com/en-us/research/wp-content/uploads/2007/01/NIPS2006_0688.pdf).

An advantage of _TrueSkill_ over other ranking systems such as ELO (used for chess) is that it converges quickly - the rankings are reasonably reliable after only 12 games. Because it uses Gaussian distributions rather than a single value, it is possible to calculate a winning probability for any given game.

We only estimate the skill of the players in singles, because doubles have a different dynamic. The rhythm is very different and the game is slower, which results in some players performing much worse.
Each player receives one skill estimate per league since it is impossible to compare rankings across leagues. A player with a skill level of 25 in a high league will probably be much better than a player who has played himself to a skill level of 26 in a lower league. Therefore, players who help out in a higher league will start with the default skill rating.

### Implementation
Unfortunately, there is no API or simple webpage to crawl the game results from. It is a [webapp](https://dbhev.de/liga/) written in php with overlays and animations that require a lot of clicking.
We use the selenium webdriver to crawl the competing players, matches, clubs and teams. These are written to a json file which we can process into a sqlite database. We use sqlalchemy as the database abstraction layer. 

The [trueskill python package](https://trueskill.org/#trueskill.rate_1vs1) is used for calculating the skill distributions.
The code can be found on my [github](https://github.com/rhotertj/ndv-elo).

To save some time we only crawl the leagues regarding the darts association in Hannover and Lower-Saxony - these are the leagues my club plays in. That leaves us with 21 leagues and 12,000 singles played by 1873 players!

### Match Report
For each match we can now plot the probability of winning for each pair. 

{{< figure src="match_qualities.png" title="Winning probabilites for players on the left side." >}}

The winner is displayed on the left side.

We can also visualize all distributions of all teams in a ridge plot to quickly see which team is better and identify the strongest players.

{{< figure src="ridge_dist.png" title="A ridge plot, showing every player and their skill probability density function." >}}



### Season Report
In order to plan teams for the next season, it is helpful to know which players are "out of distribution" given the strength of the leagues. We are looking for players that were underperforming - which we can move to a lower league next season - or players that were too good - which we can move to a higher team. We visualize this with a scatterplot that ranks the teams by their final standings and the players by their _conservative ranking_, which is r = μ - (3 * σ). This ranking takes into account the standard deviation and marks the worst ranking that can be estimated from the underlying distribution.

{{< figure src="season_report.png" title="A ridge plot, showing every player and their skill probability density function." >}}
