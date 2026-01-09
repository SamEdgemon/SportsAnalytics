---
---
{% include nav.html %}


<link rel="stylesheet" href="/SportsAnalytics/assets/css/custom.css">

# The Eras of Baseball

### History and Origins

The game of baseball changes when rules change.
It changes when the ball itself is manufactured differently, or when materials evolve.
It changes with league expansion, labor dynamics, performance-enhancing drugs, and, more recently, analytics.  

When meaningful change occurs, it leaves a trace.
If we know where to look, those changes can be detected visually in the data.  

Baseball fans often describe these shifts in terms of Eras. From the rough-and-tumble 19th Century to the modern age of analytics, the game has continually evolved. Rules changed. Strategies shifted. Players adapted. Each era tells a story—not just about baseball, but about the culture and conditions that shaped it.  

In this blog, we’ll walk through the major Eras of baseball with a specific purpose: to see how change reveals itself in the data. By examining runs scored per game across seasons, we can observe when the game fundamentally shifted—and begin to ask why. Whether you're a lifelong fan, a student of the game, or just baseball-curious, this approach offers a clear example of how data helps us understand change over time.  
 

### Building a Visualization for the Eras of Baseball

**Objective**

The objective is to create a visualization that clearly illustrates the Eras of Baseball such that structural change is visible over time.  

We will visualize "change" by looking at the average Runs Per Game (RPG) scored each season.


**Metrics and Formulas**

Runs Per Game (RPG)  = R / G

Runs per game is intentionally simple. It does not attempt to explain why offense changes, but it is well-suited to indicate when it does. When meaningful change occurs in the game, we expect to see it reflected in this metric.  


**Required Variables**

From the Teams table in Lahman’s Baseball Database, we require:   
- yearID
- R (runs scored)
- G (games played)    

No player-level data is needed. This is a league-level question.

**Workflow**  

The workflow for this analysis is as follows:

- Extract historical team data from Lahman’s Baseball Database (SQL)

- Export the result as a CSV file

- Import the CSV file into SAS Workbench (PROC IMPORT)

- Calculate Runs Per Game at the team level (SAS Data Step)

- Aggregate by season (PROC MEANS)

- Assign each season to a historically meaningful era (SAS Data Step)

- Visualize the result using PROC SGPLOT

Each step serves a purpose: extracting raw data, transforming it into a meaningful metric, aggregating it correctly, and finally presenting it in a way that allows change to be seen.


**Data and Tools**

- Lahman’s Baseball Database (Teams table)

- SQLite for querying and extracting data

- SAS Workbench for data preparation, aggregation, and visualization

See the Resources section for access to SAS Workbench.

### Code / Implementation

**SQL Query**  
``` SQL
select yearID, R, G
from Teams;
```

*This query returns one row per team per season. At this stage, no aggregation is performed.*

**SAS Code**  

***Import the CSV File***
``` SAS
filename mycsv "<your directory path>/rpg00.csv";
proc import datafile=mycsv
   out=temp
   dbms=csv
   replace;
   getnames=yes;
run;
```

*A temporary dataset called temp is created.*

***Calculate Runs Per Game (RPG)***

``` SAS
data rpg;
   set temp;
   RPG = R / G;
run;
```
*Runs Per Game (RPG) is calculated at the team-season level.*

***Aggregate RPG for each season***

``` SAS
proc means data=work.rpg nway;
   class yearID;
   var rpg;
   output out=avgRPG00;
run;
```

*This aggregation step is critical. We are no longer analyzing teams; we are analyzing eras.*

***Defining the Eras in Code***

For visualization, we explicitly encode baseball history (Eras) into the dataset.

``` SAS
data avgRPG (keep=yearID era avgRPG);
   set work.avgRPG00;
   where _STAT_ = 'MEAN';
   rename rpg = avgRPG;

   length Era $15;

   if yearID <= 1901 then Era = "19th Century";
   else if 1902 <= yearID < 1921 then Era = "Dead Ball";
   else if 1921 <= yearID < 1942 then Era = "Live Ball";
   else if 1942 <= yearID < 1947 then Era = "WWII";
   else if 1947 <= yearID < 1963 then Era = "Integration";
   else if 1963 <= yearID < 1973 then Era = "Expansion";
   else if 1973 <= yearID < 1994 then Era = "FA/DH";
   else if 1994 <= yearID < 2005 then Era = "Steroid";
   else if 2005 <= yearID < 2015 then Era = "Post Steroid";
   else if yearID >= 2015 then Era = "Analytics";
run;
```

*We are making our assumptions explicit. The visualization will now reflect both the data and our understanding of baseball history.*

**Creating the Visualization**

``` SAS
proc sgplot data=avgRPG;
   title "Average Runs Per Game by Season";

   scatter x=yearID y=avgRPG /
       group=era
       markerattrs=(symbol=circlefilled size=8);

   refline 1871 1902 1921 1942 1947 1963 1973 1994 2005 2015 /
       axis=x
       lineattrs=(pattern=shortdash color=red)
       label=("19th Cent" "Dead Ball" "Live Ball" "WWII" "Integration"
              "Expansion" "FA/DH" "Steroid" "Post Steroid" "Analytics");

   xaxis label="Season" values=(1860 to 2030 by 10);
   yaxis label="Avg Runs Per Game";
run;
```

![AvgRPG](https://samedgemon.github.io/SportsAnalytics/baseball/Blog3_Eras/Images/avgRPG.png)

*Each point represents a season.*
*Color distinguishes eras.*
*Vertical reference lines help anchor historical transitions.*

*The result is not merely a chart; it is a historical narrative expressed in data. It tells a story!*
*And like all good stories, it becomes clearer when structure is made visiible.*  

The chart above raises obvious questions.
Why do runs spike here?
Why do they collapse there?  

Baseball historians have long grouped these patterns into what we call eras.
For readers who want a deeper historical walkthrough of each era—rules, equipment, players, and cultural forces—I’ve created a companion reference document:

The Eras of Baseball (PDF)
*(A historical guide aligned with the visualization above)*



### Why Visualizations Matter

Rules change.
Baseballs change.
The league expands.
Performance-enhancing drugs appear—and later disappear.

When a change is meaningful, it often becomes visible once the right metric is plotted over time.

Baseball provides a uniquely rich environment to demonstrate this principle. Its long history, consistent record-keeping, and well-documented rule changes make it an ideal case study for understanding structural change through data.

Once the data is visualized, the questions arise naturally:

- Why does scoring jump here?
- Why does it decline there?
- What changed—and why?  

The visualization does not answer these questions.
It tells us where to look.

### A Note on Data Science Skills

In analytics, evidence of impact is often revealed not through complexity, but through clarity.  

One of the most important skills a data scientist can develop is the ability to detect change.  

Changing materials on a manufacturing line? Plot the data.
Adjusting interest rates? Plot the data.
Increasing advertising frequency? Plot the data.  

Visualization is rarely the end of analysis.
More often, it is the beginning of understanding.  

### Next Up

Bill James—and how asking better questions reshaped both the game of baseball and the way data is used to understand it.