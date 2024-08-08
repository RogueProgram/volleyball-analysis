How does height influence success in the 2024 VNL?
In men’s volleyball, the height of the net is 243 cm (or 7 feet, 11⅝ inches). With such a high net, the ability to reach over it is of utmost importance. 


It seems intuitive to prefer height among volleyball players, especially those who are at the top tier of the sport - the annual Volleyball Nation’s League, which pits 16 of the best international teams against each other. 


Using this data I want to see if, at this level of play, there is any significant relationship between the height of the players and their overall ability in the game - or, at least, their respective position. 


First, I collected all of the data from the VNL website into Excel. It was a simple copy-paste from their extensive tables breaking down how each player performs in overall scoring, as well as in attacking, blocking, setting, digging (an attacked ball), receiving (a serve), and serving. 


After scraping the data, I standardized the column names. 


My key table is the Players table, which lists all players in the VNL by team, position, height and birth year. I also removed all of the diacriticals from the names, for ease of typing.


Once all of the players represented in the tournament were included in the Players worksheet, I did some basic statistical analysis on the data there. 


Team
Average of Height
Max of Height
Min of Height
ARG
195.11
205
175
BRA
197.59
208
185
BUL
197.90
208
175
CAN
198.93
210
178
CUB
195.64
208
175
FRA
197.33
210
183
GER
195.57
210
180
IRI
197.00
206
180
ITA
194.81
208
176
JPN
189.22
204
171
NED
198.64
208
185
POL
197.24
210
180
SLO
197.44
214
185
SRB
198.07
207
181
TUR
200.00
211
186
USA
196.78
210
175
Grand Total
196.68
214
171



The average height of all players in the tournament is 196.68. France, the winner of the tournament, is just slightly above average, while Japan, who won silver, is the shortest team on average by more than 5 cm. 


I also put together a box chart for quick visual reference.







Finally, I took a look at the same stats organized by position rather than height. 


Position
Average of Height
Max of Height
Min of Height
L
183.09
198
171
MB
203.42
214
195
O
200.82
211
187
OH
197.11
207
185
S
191.36
207
175





It’s no surprise that the shortest players are liberos, who never attack, and setters, who attack rarely. 
Some interesting insights from these statistics: 


Japan holds some interesting statistics here, where their players are often the shortest for their position, while also being ranked among the best. Yamamoto (L) and Sekita (S) are each the shortest libero and setter, were chosen for the Dream Team, and are in the top 10 for their positions. Sanchez Pages (S, ARG) is also 175, but his team didn’t win silver. 


The shortest attacker of any position is Koops (OH, NED), who tied 93rd for best scorer. 
However, the shortest opposite is everyone’s favorite Little Giant, Nishida (JPN), who ranked among the top 10 best attackers and scorers. 


Finally, I was surprised to see that the shortest middle blocker, a notoriously tall position, goes again to Japan’s Larry Evbade-Dan. Evbade-Dan isn’t quite as recognizable as his teammates (yet?). Overall, it’s notable that the silver-winning team is relatively short across the board in a sport where height is such an advantage. But don’t forget - Nishida is still 187, well above the global average of 174. 


From the other end, we see Slovakia’s Stalekar (MB) come in at a towering 214 cm. Stalekar shared a tied 25th rank for best blocker.


Next tallest is Adis Lagumdzija (O, TUR), also ranked 25 for best attacker. 


Moving down to outside hitter and setter, both of which have tallest players of 207 cm. The tallest setter in the VNL was S. Nikolov (BUL), 4th best setter above Sekita’s 7th. The tallest OH is France’s Patry, one of the top 10 players in the game. 


Finally, the tallest libero. In a position famous for being the shortest, the tallest, Iran’s Salehi, is an impressive 198. 


These highlights hint that either: 
There is no relationship between height and ability at this level of play (probable)
There is an inverse relationship between height and skill (unlikely)
There’s something in the water in Japan (insufficient data)


But further analysis will make one of these more clear. 


Next, I dropped these tables into BigQuery for SQL analysis, but there’s one bit of cleaning left to do. 


As mentioned, the official rosters do not include all players in the league. To make sure I caught all of the missing entries, I compared the rosters with the best scorers list (which includes all players, even those who scored 0). 


SELECT BScore.Team, BScore.Player_Name
FROM `level-chassis-411403.VNL_2024.Best Scorers` AS BScore
LEFT JOIN `VNL_2024.Players`AS Players
ON BScore.Player_Name = Players.Player_Name
WHERE Players.Player_Name IS NULL
;


To make absolutely sure, I quickly compared all of the other tables against the Best Scorers table for any null values using this query and replaced the table name as needed.


SELECT BScore.Player_Name, BScore.Team 
FROM `level-chassis-411403.VNL_2024.Best Scorers` AS BScore
LEFT JOIN `VNL_2024.Best Blockers`AS Blockers
ON Blockers.Player_Name = BScore.Player_Name
WHERE BScore.Player_Name IS NULL
;


But it all came up empty, so it looks like the roster is truly complete. I quickly updated the data in Excel and reuploaded, so my data matched across all tables.


Then, I created a new table that combined the Players roster (which includes height and position) with the Best Scorers table. This table also drops Liberos, who are not scoring players.


CREATE TABLE VNL_2024.Player_Score
AS
SELECT * FROM `VNL_2024.Players`AS Players
Left join `level-chassis-411403.VNL_2024.Best Scorers` AS BScore
Using(Player_Name, Team)
WHERE Players.Position != "L"
ORDER BY Total_Pts DESC
;

Then, I broke the average score down by height and, for ease of digestion, divided the population into 5cm “height bins”. 

select
case
  when Height <=174 then '170-174'
  when Height >=175 and Height <= 179 then '175-179'
  when Height >=180 and Height <= 184 then '180-184'
  when Height >=185 and Height <= 189 then '185-189'
  when Height >=190 and Height <= 194 then '190-194'
  when Height >=195 and Height <= 199 then '195-199'
  when Height >=200 and Height <= 204 then '200-204'
  when Height >=205 and Height <= 209 then '205-209'
  when Height >=210 and Height <= 214 then '210-214'
  else 'NA'
END AS Height_bins,
ROUND(AVG(Total_Pts),2) AS Avg_Pts,
COUNT(Height) AS No_Players
from`VNL_2024.Player_Score`
GROUP BY
  case
  when Height <=174 then '170-174'
  when Height >=175 and Height <= 179 then '175-179'
  when Height >=180 and Height <= 184 then '180-184'
  when Height >=185 and Height <= 189 then '185-189'
  when Height >=190 and Height <= 194 then '190-194'
  when Height >=195 and Height <= 199 then '195-199'
  when Height >=200 and Height <= 204 then '200-204'
  when Height >=205 and Height <= 209 then '205-209'
  when Height >=210 and Height <= 214 then '210-214'
  else 'NA'
  end
Order by Height_bins DESC
;




Height_bins
Avg_Pts
No_Players
210-214
56.87
8
205-209
50.27
45
200-204
62.07
90
195-199
48.14
62
190-194
39.16
39
185-189
27.89
19
180-184
4.0
3
175-179
3.0
3





From here, it’s clear that taller players score more points on average. However, this is also not surprising given that there are more tall players. 205 players - more than two thirds of the players in the league - are over 195 cm tall. They are statistically more likely to “be scoring” as there are just more of them. 


Let’s add position into the equation.


select
Position,
case
  when Height <=174 then '170-174'
  when Height >=175 and Height <= 179 then '175-179'
  when Height >=180 and Height <= 184 then '180-184'
  when Height >=185 and Height <= 189 then '185-189'
  when Height >=190 and Height <= 194 then '190-194'
  when Height >=195 and Height <= 199 then '195-199'
  when Height >=200 and Height <= 204 then '200-204'
  when Height >=205 and Height <= 209 then '205-209'
  when Height >=210 and Height <= 214 then '210-214'
  else 'NA'
END AS Height_bins,
ROUND(AVG(Total_Pts),2) AS Avg_Pts,
COUNT(Height) AS No_Players,
from`VNL_2024.Player_Score`
GROUP BY
Position,
  case
  when Height <=174 then '170-174'
  when Height >=175 and Height <= 179 then '175-179'
  when Height >=180 and Height <= 184 then '180-184'
  when Height >=185 and Height <= 189 then '185-189'
  when Height >=190 and Height <= 194 then '190-194'
  when Height >=195 and Height <= 199 then '195-199'
  when Height >=200 and Height <= 204 then '200-204'
  when Height >=205 and Height <= 209 then '205-209'
  when Height >=210 and Height <= 214 then '210-214'
  else 'NA'
  end
Order by Position, Height_bins DESC
;


Position
Height_bins
Avg_Pts
No_Players
MB
210-214
48
6
MB
205-209
39.48
26
MB
200-204
43.89
38
MB
195-199
40.55
12
O
210-214
83.5
2
O
205-209
84.5
8
O
200-204
95
21
O
195-199
73
3
O
190-194
70.75
6
O
185-189
185
1
OH
205-209
49.4
10
OH
200-204
69.04
27
OH
195-199
64.74
33
OH
190-194
50.65
20
OH
185-189
41.14
7
S
205-209
55
1
S
200-204
10.25
4
S
195-199
9.23
14
S
190-194
11.77
13
S
185-189
2.9
11
S
180-184
4
3
S
175-179
3
3



Broken down by position, most points tend to go to the most populated height bins, with the notable exceptions of the tallest setter (S. Nikolov, BUL), and the shortest opposite (Nishida, JPN), whose averages are not reduced by others in their bins. 


This suggests even further that height is not a significant factor among players at this level. Apart from significant outliers, players in the middle of the range for their position tended to score more highly than those at the top, particularly if that height bin had a higher number of players. 


Instead of looking at points scored, I then looked at the success rate of each position and player. 


The VNL statistics includes a percentage of success for each skill: Attacking, Blocking, Serving, Setting, Digging and Receiving. All I want to do is put these core stats into one table so I can analyze them all at one time.


All of these are calculated as a percentage of success out of the total attempts except for blocking, which is a negative percentage representing the difference in percentage points between block kills (points) and block outs (errors). So if 30% of a player’s blocks were kills and 50% were block outs, then they would have a blocking percentage of -20%. 


In any case, I used this query to create the aggregate table of percentages by skill. 


CREATE TABLE VNL_2024.Total_Percentage AS
SELECT Players.Team,
      Players.Player_Name,
      Players.Position,
      Players.Height,
      Players.Birth_Year,
      Blockers.p_Block,
      Setters.p_Set,
      Attackers.p_Attack,
      Servers.p_Serve,
      Diggers.p_Dig,
      Receivers.p_Receive,
FROM `level-chassis-411403.VNL_2024.Players` AS Players
  LEFT JOIN
  `VNL_2024.Best Blockers` AS Blockers
  ON Players.Player_Name = Blockers.Player_Name
  LEFT JOIN
  `VNL_2024.Best Setters` AS Setters
  ON Players.Player_Name = Setters.Player_Name
  LEFT JOIN
  `VNL_2024.Best Attackers` AS Attackers
  ON Players.Player_Name = Attackers.Player_Name
  LEFT JOIN
  `VNL_2024.Best Servers` AS Servers
  ON Players.Player_Name = Servers.Player_Name
  LEFT JOIN
  `VNL_2024.Best Diggers` AS Diggers
  ON Players.Player_Name = Diggers.Player_Name
  LEFT JOIN
  `VNL_2024.Best Receivers` AS Receivers
  ON Players.Player_Name = Receivers.Player_Name
;


Then I took this aggregate table and put it into Tableau to visualize. I organized it by position, and added trend lines to instantly see the relationship between position, height and success.



What resulted was a lot of output to essentially say, no, there’s no direct relationship between height and skill in professional men’s volleyball. 


This isn’t surprising. To get to this level, these players have already been filtered for skill, not to mention height. The average height of all the men in this tournament was nearly 2 meters. So if we were to compare these players against players who aren’t literally world-class, we may see that height is the deciding factor. It may be that the volleyball players who do develop their skills to this level feel that they have a lower physical barrier to overcome than their shorter colleagues. 


But the success of shorter players such as Yuji Nishida suggests that being shorter than most is not a death sentence in volleyball, and the statistics support this. What appears to matter more is the skill to attain this level of play, which can supersede the attribute that carries so much cachet in volleyball. 


