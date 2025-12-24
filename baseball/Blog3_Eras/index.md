---
---
{% include nav.html %}


<link rel="stylesheet" href="/SportsAnalytics/assets/css/custom.css">

# The Eras of Baseball

### History and Origins

Baseball changes when rules change.  
It changes when the ball itself changes.  
It changes with expansion, integration, labor dynamics, performance-enhancing drugs, and analytics.  

If those changes matter, they should leave a footprint in the data.  


### Building a Visualization for the Eras of Baseball

**Objective**

The objective of this analysis is to create a visualization that clearly illustrates the Eras of Baseball and makes structural change visible over time.


**Metrics and Formulas**

Runs Per Game (RPG)  = R / G

Runs per game is intentionally simple. It does not attempt to explain why offense changes, but it is expected to indicate when it does.


**Required Variables**

From the Teams table in Lahman’s Baseball Database, we require: yearID, R, G  

No player-level data is needed. This is a league-level question.

**Workflow**

- Extract historical team data from Lahman’s Baseball Database

- Export the result as a CSV file

- Import the CSV file into SAS Workbench

- Calculate Runs Per Game at the team level (SAS Data Step)

- Aggregate by season (Proc Means)

- Assign each season to a historically meaningful era (SAS Data Step)

- Visualize the result using PROC SGPLOT

Each step serves a purpose. None are accidental.  


**Data and Tools**

- Lahman’s Baseball Database (Teams table)

- SQLite for querying and extracting data

- SAS Workbench for data preparation, aggregation, and visualization

### Code / Implementation

**SQL Query**  
``` SQL
select yearID, R, G
from Teams;
```

*This query returns one row per team per season. At this stage, no aggregation is performed.*

**Import the CSV File**
``` SAS
filename mycsv "<your directory path>/rpg00.csv";
proc import datafile=mycsv
   out=temp
   dbms=csv
   replace;
   getnames=yes;
run;
```

*A temporary dataset (temp) is created. This keeps the workflow reproducible and transparent.*

Calculate Runs Per Game

``` SAS
data rpg;
   set temp;
   RPG = r / g;
run;
```
*Runs Per Game (RPG) is calculated at the team-season level.*

Move to a league-wide view for each season

``` SAS
proc means data=work.rpg nway;
   class yearID;
   var rpg;
   output out=avgRPG00;
run;
```

*This aggregation step is critical. We are no longer analyzing teams; we are analyzing eras.*

**Defining the Eras in Code**

Before visualizing the data, we explicitly encode baseball history (Eras) into the dataset.

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

*Each point represents a season.*
*Color distinguishes eras.*
*Vertical reference lines mark transitions.*

*The result is not merely a chart; it is a historical narrative expressed in data. It tells a story!*

### Why Visualizations Matter

Rules change.
Baseballs change.
The league expands.
Performance-enhancing drugs appear—and later disappear.

Regardless of which lever is pulled, if the change is meaningful, it often becomes visible when the right metric is plotted over time.

Baseball offers a uniquely rich setting to demonstrate this idea. Once the data is visualized, the questions arise naturally:

Why does scoring jump here?
Why does it decline there?
What changed and why?

### A Note on Data Science Skills

In analytics, evidence of impact is often revealed not through complexity, but through clarity.

Changing materials on a manufacturing line? Plot the data.
Adjusting interest rates? Plot the data.
Increasing advertising frequency? Plot the data.

Visualization is rarely the end of analysis.
More often, it is the beginning of understanding.

### Next Up

Bill James, and how asking better questions reshaped the way the game—and data—are understood.