# Calculating Baseball’s Slash Stats in SAS
### A Tribute to the Game’s First Data Analyst x2

### History and Origins
Baseball fans today live by the “slash line” — batting average, on-base percentage, and slugging percentage.  
You’ll see it everywhere:
.300 / .400 / .500

Behind those numbers lies a story stretching back more than 160 years to one Englishman with a notebook and a deep love for a new American game.  

Henry Chadwick (1824–1908) was the first data analyst in baseball history, and he concluded that the sport needed structure and statistics. Long before Baseball Reference or the Baseball Abstract, Chadwick was collecting data by hand. Born in England and beginning his journalism career in the U.S. in the 1850s, he discovered baseball in 1856 and was instantly hooked.  

Chadwick believed the game needed ways to measure performance. He created the batting average and earned run average, and he helped shape the box score — including the familiar “K” for strikeout.
<!--box score image-->  

His work gave baseball its first true performance metrics and made the game measurable.  

---

### From Box Scores to Slash Lines

As baseball evolved, so did its statistics. Batting average alone couldn’t tell the whole story, so scorekeepers added:  

- 	On-Base Percentage (OBP)
- 	Slugging Percentage (SLG)  

Later, OBP and SLG were combined into OPS, now one of the most widely used indicators of offensive value.  

Now let’s use modern tools to build the slash line ourselves.

---  

### Building the Modern Slash Line from Historical Data  

To stay focused, we’ll lay out the objective, the formulas behind each metric, the requirements and workflow, and the tools we’ll use.  

**The Objective**  

Calculate the slash line — BA, OBP, SLG — and then combine OBP and SLG to compute OPS.  

**What We Know**  

Formulas:
- BA = H / AB  
- OBP = (H + BB + HBP) / (AB + BB + HBP + SF)  
- SLG = (1B + 2×2B + 3×3B + 4×HR) / AB  
- OPS = OBP + SLG  

**Requirements / Strategies**  

Variables needed:
H, AB, BB, HBP, SF, 2B, 3B, HR  

Additional variables for later analysis:  
G, R, RBI, SB, CS  

Workflow:
1.	Extract data using SQL
2.	Export to CSV
3.	Import into SAS
4.	Calculate slash line metrics  


**Data and Tools**  
•	Data from **Lahman’s Baseball Database** (SQLite)
•	**SQL** for extracting a data subset
•	**SAS Workbench** for importing, processing, and computing metrics  

---

### Hitting the Database  
With the metrics and workflow in place, it’s time to write some code. I’ve opened my SQLite interface, and from experience I know two things:
1.	The **Batting** table has everything we need.  
2.	I want to subset the data: seasons from 1954 forward (first consistent sacrifice fly data), and minimum 50 AB to reduce noise.  

**SQL Code for Query**  
``` SQL
SELECT  playerID, yearID, teamID, G, AB, H,
        [2B], [3B], HR, R, RBI, SB, CS, BB,
        SF, HBP
FROM    Batting
WHERE   yearID >= 1954 AND
        AB > 50;  
```
 
I exported the results as BattingExtract.csv.  
You can download BatttingExtract.csv here.  

### SAS Workbench  
You can get SAS Workbench here.  

Upload the CSV and import it:
``` SAS
filename mycsv "<your directory path>/BattingExtract.csv";
proc import datafile=mycsv
   out=temp
   dbms=csv
   replace;
   getnames=yes;
run;
```  

A temporary dataset called temp is created.  

**Calculating Slash Line Metrics**
``` SAS
data SlashStats;
   set work.temp;
   '1B'n = H - '2B'n - '3B'n - HR;
   BA  = H / AB;
   OBP = (H + BB + HBP)/(AB + BB + HBP + SF);
   SLG = ('1B'n + 2*'2B'n + 3*'3B'n + 4*HR) / AB;
   OPS = OBP + SLG;
run;  
```  

This short program produces the same measures once calculated by hand, but in milliseconds.

**Exploring the Results**  
``` SAS
data BA;
   set work.SlashStats;
   where yearid >= 1970 and AB >= 200;
run;

proc sort data=work.BA; 
   by descending BA; 
run;

title 'Top 10 Batting Averages (BA)';
title2 'since 1970';
proc print data=work.ba (obs=10) noobs label;
    var playerID yearID H AB BA;
    label playerID = 'Player ID'
          yearID = 'Year';
    format BA 9.3;
run;
```

**Results:** Top 10 Batting Averages Since 1970  
<!--insert table--> 
![Top 10 BA](https://samedgemon.github.io/baseball/Blog1_SlashStats/Images/TOP10BA_1970.png)

---

### Why Henry Chadwick Matters
When Henry Chadwick started keeping score, he wasn’t just documenting baseball — he was translating a game into data.
He made performance measurable, trends visible, and debate possible.  

Every slash line, OPS leaderboard, and modern metric traces back to his insight that **numbers can tell the story of the game.**

**A Note on Data Science Skills**  
*One purpose of this series is to teach data science through sports. In this single exercise, we’ve defined the objective,  
identified the variables we need, laid out the workflow, selected tools, extracted data using SQL, imported it into an  
analytical environment, engineered new features, wrote reproducible code, and produced a simple report. These are the  
same steps analysts use every day—just applied here to baseball instead of finance, healthcare, or operations.*

**Next up:** Recreating a classic **Hank Aaron** baseball card — complete with his full slash line and OPS.

