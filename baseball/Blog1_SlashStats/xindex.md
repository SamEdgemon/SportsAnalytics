## Calculating Baseball’s Slash Stats in SAS
### A Tribute to the Game’s First Data Analyst

Baseball fans today live by the “slash line” — that set of numbers showing a player’s **batting average, on-base percentage, and slugging percentage**. 

You’ll see it in every broadcast and box score: .300 / .400 / .500

It looks simple, but behind it lies a story that stretches back more than 160 years to one Englishman with a notebook, a sharp mind for numbers, and a deep love for a new American game. <br><br>




## The First Data Analyst in Baseball History
Long before Moneyball or Statcast, **Henry Chadwick** was collecting baseball data by hand. Born in England in 1824, Chadwick moved to the United States as a teenager and began his career covering cricket. In the 1850s, he discovered baseball and was instantly hooked.

Chadwick believed the game needed ways to describe what happened such that performance could be compared. He concluded that the game needed **structure and statistics**. In 1859, he published the first **box score**, recording runs, hits, putouts, assists, and errors. He even invented the “K” for strikeout.

Example; An Early Box Score

![Box Score](https://samedgemon.github.io/Baseball/Blog1_SlashStats/Images/HenryChadwickBoxScore.jpg)


Chadwick also introduced the **Batting Average** — giving baseball its first true performance metric. His innovations made baseball measurable. <br><br>


## From Box Scores to Slash Lines
The game evolved, and so did its numbers.

Batting average alone couldn’t tell the whole story, so analysts and scorekeepers built on Chadwick’s foundation:

![Slash Line Table](https://samedgemon.github.io/Baseball/Blog1_SlashStats/Images/SlashLineTable.png)




Together, BA / OBP / SLG form the **slash line** — and OPS ties them into one simple number.

If Chadwick’s box score was the seed, the slash line is the modern tree that grew from it.

When Chadwick was creating box scores and calculating batting averages it was with a very manual process. Today we have data and programing languages like SAS to help us accomplish the slash line. Let’s do that! <br><br>


## Calculating the Slash Line in SAS

I’m using SAS Workbench to process my data and calculate the metrics. The workflow is simple:

1. Export player batting data as a .CSV from Lahman’s Baseball Database
2. I import the file into SAS
3. Create the slash line using a short data step


Learners can gain access (with your academic email address) to SAS Workbench <a href="https://www.sas.com/en_us/software/viya-workbench-for-learners.html" target="_blank">here</a>.



The data used can be downloaded [here](https://samedgemon.github.io/Baseball/Blog1_SlashStats/Data/BattingExtract.csv)



Here is the code I use to import the data;

```markdown
filename mycsv '"<your directory path>/BattingExtract.csv";
proc import datafile=mycsv
   out=temp
   dbms=csv
   replace;
   getnames=yes;
run;
```


Here is the code to I used to create the slash line:

```markdown
data SlashStats;
   set work.temp;
   '1B'n = H -'2B'n - '3B'n - HR;
   BA  = H / AB;
   OBP = (H + BB + HBP)/(AB + BB + HBP + SF);
   SLG = ('1B'n + 2*'2B'n + 3*'3B'n + 4*HR) / AB;
   OPS = OBP + SLG;
run;
```

This short program does exactly what Chadwick did (and a little more) — but in milliseconds instead of hours with pen and paper. The output gives you the same view fans have loved for generations: who’s getting on base, who’s hitting for power, and who’s doing both.

Here is the code I ran to evaluate my BA calculations:

```markdown
data BA;
    set work.SlashStats;
    where yearid >= 1970 and AB >= 200;
run;
proc sort data=work.BA; by descending BA; run;

title 'Top 10: BA since 1970';
proc print data=work.ba (obs=10); 
    var playerID yearID H AB BA; 
    format BA 9.3;
run;
```

Results. The Top 10 Batting Averages Since 1970

![Results Batting Average](https://samedgemon.github.io/Baseball/Blog1_SlashStats/Images/outputBattingAverage.png)



For those that want to recreate my entire process, I’ve included my workflow with more extensive details. <br><br>


## Why Henry Chadwick Matters

When Henry Chadwick started keeping score, he wasn’t just documenting baseball — he was translating a game into data. He made performance measurable, trends visible, and debate possible.

Every slash line, OPS leaderboard, and advanced stat we use today still traces back to Chadwick’s simple insight: Numbers can tell the story of the game.

And in that way, every analyst who writes code is a little bit like Henry Chadwick — building on his dream of making baseball understandable through data. <br><br>


**Next up**: We’ll take what we’ve learned here and recreate a classic Hank Aaron baseball card — complete with his full slash line and OPS.












