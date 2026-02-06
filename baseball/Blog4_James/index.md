---
---
<!--{% include nav.html %}


<link rel="stylesheet" href="/SportsAnalytics/assets/css/custom.css">  -->

v3

# Bill James and the Two Numbers
# that Explain Winning  

### History and Origins  

Often referred to as the father of modern baseball analytics, Bill James was born in 1949 in Kansas and began his journey into baseball analysis while working as a security guard. Because he was not embedded in the game’s traditional institutions, **James approached baseball by questioning its underlying assumptions.** Rather than refining accepted measures, he asked whether those measures were answering the right questions at all. Driven by curiosity and a passion for the game, he examined baseball statistics in a rigorous, systematic way, laying the groundwork for what would become modern *sabermetrics.*  

He coined and popularized that term "sabermetrics" to describe the objective, statistical analysis of baseball but the label mattered less than the shift in reasoning behind it. Prior baseball statistics focused on what should be counted. **James focused on what should be explained.** With a clear framework for thinking statistically, James was ready to communicate his ideas to a broader audience.  

Through a series of self-published books he called *Baseball Abstracts*, which he began publishing in 1977, **James introduced a new way of thinking about performance.** His writing combined statistical rigor with wit and skepticism, challenging assumptions that had gone largely unquestioned for decades. Metrics such as Runs Created, Range Factor, Defensive Efficiency Rating, Win Shares, and the Pythagorean Expectation were not presented as clever inventions, but as tools designed to test his central question: *what truly causes teams to win?*  

What ultimately separated James from his predecessors was not technical sophistication, but the question he insisted on asking. Rather than refining descriptive measures of performance, he asked a more fundamental question: What actually causes teams to win baseball games? That question — and the discipline to follow it wherever the data led — reframed how offense, defense, and value would be understood going forward. This focus on identifying the core drivers of winning naturally led him to ask whether team performance could be distilled to just two numbers: *runs scored and runs allowed.*  

### The Pythagorean Theorem for Baseball  

Reducing the complexity of a baseball season — 162 games, thousands of at-bats, dozens of players — into just two numbers was Bill James’ revolutionary insight. Winning is not about style, tradition, or reputation; it is arithmetic. Building on his focus on what truly drives wins, James formalized this insight mathematically with what he called the Pythagorean Theorem for Baseball (sometimes referred to as the Pythagorean Expectation).  

James discovered that a team’s winning percentage could be estimated using only Runs Scored (R) and Runs Allowed (RA):  

<!--WP≈R^2/(R^2+RA^2 )-->

$WP \approx \frac{R^2}{R^2 + RA^2}$


This deceptively simple formula proved remarkably accurate. Its implication was profound: nearly everything in *baseball boils down to how many runs a team scores and how many it allows.*  

In this framework, outs are the scarce resource, and runs are the currency. Traditional offensive metrics — batting average or RBIs — may feel important, but they are subordinate to the ultimate goal of producing runs efficiently. Teams that understand the relationship between runs and wins can prioritize the metrics that truly matter.  

James’ Pythagorean insight reframed the game: improving winning percentage is directly tied to increasing runs scored or decreasing runs allowed. This perspective naturally sets up the next question: which offensive metrics most effectively produce runs?  

But before we discuss metrics, let’s evaluate the Pythagorean Theorem for Baseball.

Objective  

Validate Bill Jame's Pythagorean Theorem for Baseball  

What we know  

- WP = W / (W + L)
- estWP = R^2 / (R^2 + RA^2)  

Requirements / strategies  

W, G, R, RA

Data and tools  

- Lahman's Baseball Database
- SQL
- SAS 


SQL

select yearID, teamID, R, RA, W, L
from Teams
where yearID>1954;

SAS Code  

import the pythag extract

calculate WP and the Pythagorean estimate  

data pythag;
   set pythag0;
   WP=w/(W+L);
   estWP=R**2/(R**2+RA**2);
run;

Caculate r

proc corr; 
   var wp estWP; 
run;

Print and Graph

Radomly pick a year (show code)
proc Print
proc sgplot

Why is this important?

Data Science

Next up  
A shorter blog, but one of importance. Let's build a model!
