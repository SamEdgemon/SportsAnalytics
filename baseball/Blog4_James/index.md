---
---
{% include nav.html %}


<link rel="stylesheet" href="/SportsAnalytics/assets/css/custom.css"> 

v8

# Bill James and the Two Numbers <br> that Explain Winning  

### History and Origins  

Often referred to as the father of modern baseball analytics, Bill James was born in 1949 in Kansas and began his journey into baseball analysis while working as a security guard. Because he was not embedded in the game’s traditional institutions, **James approached baseball by questioning its underlying assumptions.** Rather than refining accepted measures, he asked whether those measures were answering the right questions at all. Driven by curiosity and a passion for the game, he examined baseball statistics in a rigorous, systematic way, laying the groundwork for what would become modern *sabermetrics.*  

He coined and popularized that term "sabermetrics" to describe the objective, statistical analysis of baseball but the label mattered less than the shift in reasoning behind it. Prior baseball statistics focused on what should be counted. **James focused on what should be explained.** With a clear framework for thinking statistically, James was ready to communicate his ideas to a broader audience.  

Through a series of self-published books he called *Baseball Abstracts*, which he began publishing in 1977, **James introduced a new way of thinking about performance.** His writing combined statistical rigor with wit and skepticism, challenging assumptions that had gone largely unquestioned for decades. Metrics such as Runs Created, Range Factor, Defensive Efficiency Rating, Win Shares, and the Pythagorean Expectation were not presented as clever inventions, but as tools designed to test his central question: *what truly causes teams to win?*  

What ultimately separated James from his predecessors was not technical sophistication, but the question he insisted on asking. Rather than refining descriptive measures of performance, he asked a more fundamental question: What actually causes teams to win baseball games? That question — and the discipline to follow it wherever the data led — reframed how offense, defense, and value would be understood going forward. This focus on identifying the core drivers of winning naturally led him to ask whether team performance could be distilled to just two numbers: *runs scored and runs allowed.*  


### The Pythagorean Theorem for Baseball  

Reducing the complexity of a baseball season — 162 games, thousands of at-bats, dozens of players — into just two numbers was Bill James’ revolutionary insight. Winning is not about style, tradition, or reputation; it is arithmetic. Building on his focus on what truly drives wins, James formalized this insight mathematically with what he called the Pythagorean Theorem for Baseball (sometimes referred to as the Pythagorean Expectation).  

James discovered that a team’s winning percentage could be estimated using only Runs Scored (R) and Runs Allowed (RA):  

<br>

$$
WP \approx \frac{R^2}{R^2 + RA^2}
$$

<br>


This deceptively simple formula proved remarkably accurate, and its implication profound: nearly everything in *baseball boils down to how many runs a team scores and how many it allows.*  

In this framework, outs are the scarce resource, and runs are the currency. Traditional offensive metrics — batting average or RBIs — may feel important, but they are subordinate to the ultimate goal of producing runs efficiently. **Teams that understand the relationship between runs and wins can prioritize the metrics that truly matter.**  

James’ Pythagorean Expectation reframed the game with the realization that **improving winning percentage is directly tied to increasing runs scored or decreasing runs allowed**. This perspective naturally sets up the next question: which offensive metrics most effectively produce runs?  

But before we discuss metrics, let’s evaluate the Pythagorean Theorem for Baseball.

And, let's start our evaluation by defining the objective, what we know (formulas), requirements regarding data, a workflow, and finally declare where we are getting data and what tools we will use.  

<br>

**Objective**  

Validate Bill James' Pythagorean Theorem for Baseball  

<br>

**What we know (Metrics and Formulas)**  

- Winning Percentage: $WP = \frac{W}{W+L}$

- Estimated Winning Percentage: $estWP = \frac{R^{2}}{R^{2} + RA^{2}}$  

<br>


**Required Variables**   

From the Teams table in Lahman’s Baseball Database, we need:   
- yearID (year ID)
- teamID (team ID)
- R (runs scored)
- RA (runs scored against)
- W (wins)
- L (losses)  

<br>

**Workflow**  
1. **Extract** data from database using SQL
2. **Export** CSV file from results
3. **Import** CSV file into SAS
4. **Calculate** the Pythagorean Expectation in SAS
5. **Create** visuals in SAS.  
  
<br>

**Data and Tools**    

- Data from **Lahman's Baseball Database** (SQLite)
- **SQL** for extracting data subset
- **SAS Workbench** for importing, processing, and computing metrics  

You can get SAS Workbench <a href="https://www.sas.com/en_us/software/viya-workbench-for-learners.html" target="_blank">here</a>.  

<br>

### Hitting the Database

We will refer to the formulas and workflow as we begin writing code.  

With the SQLite interface open the data will be extracted from the **Teams** table.  
Criteria for this extraction will include data from 1954 forward.  

<br>

**SQL Code**

``` SQL
select yearID, teamID, R, RA, W, L
from Teams
where yearID>1954;
```

*This query retrieves team performance statistics (runs scored, runs allowed, wins, losses)*
*for all seasons after 1954 from the Teams table.*  

*The results were exported to a CSV file called PythagExtract1.csv*  

You can download **PythagExtract1.csv** [here](https://samedgemon.github.io/SportsAnalytics/baseball/Blog4_James/Data/PythagExtract1.csv).  

<br>


**SAS Code**  

**Import the CSV file**

``` SAS
proc import datafile='/workspaces/workspace/.sasuser.workbench/CSV/PythagExtract1.csv'
            out=pythag00
            replace;
run;
proc contents; run;
```
*Import the CSV file (PythagExtract1.csv).  
*Save as a SAS temp file (pythag00).*  
*Run PROC CONTENTS to see information about the temp file.*  

<br>

**Calculate Winning Percentage (WP) and the Pythagorean Expectation (estWP)**  

``` SAS
data pythag;
   set pythag00;
   WP=w/(W+L);
   estWP=R**2/(R**2+RA**2);
run;
```

*Read each observation from PYTHAG00 and write it to PYTHAG.*  
*While doing so, compute Winning Percentage (WP) from wins and losses,*  
*and the Pythagorean Expectation (estWP) from runs scored and allowed.*  

*Recall our objective, "validate Bill James' Pythagorean Theorem for Baseball."*
*In other words, does the Bill James' claim actually hold up with real data?*
*How can we do that?*

*One way to do that is with the Pearson correlation coefficient.*

<br>


**Calculate the Pearson Correlation Coefficient (r)**

``` SAS
proc corr; 
   var estWP; 
   with WP;
run;
```

We are using PROC CORR to examine the correlation between winning percentage (WP) and the Pythagorean Estimate (estWP). It computes the Pearson Correlation Coefficient (r), which measures the strength and direction of a linear relationship between two variables.

Here are the results;


| Variables Compared | Pearson Correlation (r) |
|--------------------|--------------------------|
| WP vs estWP        | 0.94                     |



*A correlation value close to +1 indicates a strong positive linear relationship.*  

In this case, the correlation between actual winning percentage (WP) and the Pythagorean estimate (estWP) is very close to +1. This tells us that teams with higher estimated winning percentages—based only on runs scored and runs allowed—also have higher actual winning percentages.  

*In practical terms, once runs are known, much of the variation in team success is already explained.*


**Plot Actual Winning Percentage (WP) vs. the Pythagorean Expecation (estWP)**

``` SAS
ods graphics / reset width=1280px height=960px imagename="pythag1" outputfmt=png;
ods listing gpath="/workspaces/workspace/.sasuser.workbench/Images";
proc sgplot data=pythag;
   scatter x=estWP y=WP / markerattrs=(symbol=circlefilled color=blue size=12);
   xaxis label="Pythagorean Expectation (estWP)"
         labelattrs=(size=18)
         valueattrs=(size=12);
   yaxis label="Winning Percentage (WP)"
         labelattrs=(size=18)
         valueattrs=(size=12);
   title height=2.5 "Winning Percentage vs Pythagorean Expectation";
run;
ods listing close;
```

<br>


Print and Graph

![Pythag1](https://samedgemon.github.io/SportsAnalytics/baseball/Blog4_James/Images/pythag1.png)  


The scatterplot shows each team’s actual winning percentage (WP) plotted against its Pythagorean estimate (estWP). Each point represents a single team-season.

If the Pythagorean formula perfectly predicted team performance, all points would lie exactly along a 45-degree line. Instead, we observe a tight upward-sloping cloud of points. Teams with higher estimated winning percentages almost always have higher actual winning percentages.

This visual pattern corresponds to the **Pearson correlation coefficient of r = 0.94**. The points cluster closely around a linear relationship, indicating that *most of the variation in team success is explained by runs scored and runs allowed*.

The small deviations from the implied line reflect randomness, sequencing effects, bullpen performance, and other factors that influence outcomes but are not captured directly by run totals.

<br>

This results motivate the next question: **if runs drive winning, which offensive metrics best explain runs?**



**Why Does the Pythagorean Theorem for Baseball Matter?**

It separates performance from results.

Wins are the outcome. Runs scored and runs allowed are the underlying process.

Our analysis shows a correlation of r = 0.94 between the Pythagorean estimate and actual winning percentage. Visually, the scatterplot confirms a tight linear relationship. Numerically, this implies that most of the variation in team success is associated with run differential.

In practical terms, once runs are known, team success is largely determined. This reframes how we think about performance evaluation:

- Are teams winning because they are fundamentally strong?

- Or, are they temporarily benefiting from sequencing, luck, or one-run game variance?

The Pythagorean formula gives us a principled way to answer that question.

More broadly, we have demonstrated that *analytics tests ideas against data and quantifies relationships that would otherwise remain intuitive claims.*  

<br>

**A Note on Data Science Skills**

In this post, we applied several foundational data science skills:

- Extracted structured data from a database using SQL

- Exported query results to a CSV file

- Imported external data into SAS

- Created derived variables (e.g., winning percentage, Pythagorean estimate)

- Conducted a correlation analysis using PROC CORR

- Visualized the relationship using PROC SGPLOT

Most importantly, *we learned how to interpret a correlation coefficient.*

Correlation is one of the most widely used tools in analytics. It quantifies the strength and direction of a linear relationship between two variables. In this case, it allowed us to evaluate whether Bill James’ formula meaningfully tracks actual team performance.

*Understanding correlation is foundational.* Before building predictive models, before running regressions, before applying machine learning algorithms, analysts must first understand whether variables are meaningfully related.

That skill — testing and quantifying relationships — is central to both sports analytics and business analytics.


**Next up**
A shorter blog, but one of importance. Let's build a model!
