---
---
{% include nav.html %}


<link rel="stylesheet" href="/SportsAnalytics/assets/css/custom.css">

# The Home Run King

### History and Origins: Henry "Hank" Aaron

The game between the Atlanta Braves and the Los Angeles Dodgers was played at Atlanta-Fulton County Stadium on April 8, 1974. One hundred eighty-five miles to the north, a kid sat alone in a house in Paint Rock, Tennessee, on the floor in front of the family television.  

He adjusted the knobs carefully, fine-tuning the UHF channel hoping to get the best picture possible. He was proud of how clear it looked. Then he took a deep breath and waited.  

The Braves started right-hander Ron Reed. The Dodgers countered with left-hander Al Downing. Hank Aaron batted cleanup and played left field. Darrell Evans and Dusty Baker were also in the lineup that night.  

In the bottom of the second inning, with the game scoreless, Aaron led off and drew a walk on five pitches. He never took the bat off his shoulder. He appeared almost detached, surveying the field without emotion. The crowd booed the pitcher for the walk!  

Dusty Baker followed with a double to left. An error by Bill Buckner allowed Aaron to score from first, giving Atlanta a 1-0 lead. In the process, Aaron set a **new National League record for career runs scored.**  

The Dodgers answered immediately. Steve Garvey singled. Bill Russell doubled. Al Downing singled to center, driving in Garvey and moving Russell to third. A ground ball cut down Russell at the plate, but after a popout, Jim Wynn doubled to left, driving in two more runs. Los Angeles led 3-1.  

The kid was still sitting on the floor. He knew Aaron would bat again in the next inning.  

The fourth inning began with Darrell Evans reaching on an error by Bill Russell. **Henry Aaron came to the plate** as the tying run. The crowd of 53,775 was the largest in the stadium's history and all in attendance appeared to rise with a low, electric buzz. Aaron stepped into the box sitting on 714.  

Downing's first pitch was low, in the dirt. The count moved to 1-0.  

The kid rose to his knees and instinctively reached out to touch the television. He noticed the time.  

On the very next pitch, at approximately 9:07 p.m. local time, **Aaron swung for the first time that night and launched the ball over the left-center-field wall for home run number 715.**   

The kid stood and watched the scene unfold. He knew this would be talked about ... forever!  

Years later, that same kid was in college, looking through a box of old baseball cards. One card caught his attention. On the front, it proclaimed Hank Aaron the "Home Run King."  

It was the back of the card that mattered most. **It told a story with numbers!**  

He bought the baseball card and took it home.  

**1974 Topps Hank Aaron "Home Run King" Baseball Card**  

![Front](https://samedgemon.github.io/SportsAnalytics/baseball/Blog2_715/Images/Aaron1974Toppsv3.png)
![Back](https://samedgemon.github.io/SportsAnalytics/baseball/Blog2_715/Images/Aaron1974ToppsBackv3.png)


### Hank's Card Needs the Modern Slash Line  

When Hank Aaron played, the statistics most consistently shown were batting average and home runs. If he played today, it would be difficult to discuss his career without referencing on-base percentage (OBP) and OPS.  

This project updates the **1974 Topps Hank Aaron Home Run King baseball card** with modern sabermetric measures. The goal is to create a report that replicates the original card, but also adds context and insight using metrics that better describe offensive value. Let's add the the *slash stats!*  

<br>

## Building the Modern Slash Line from Historical Data  

We will state the objective, what we know (formulas), requirements, workflow, and finally the data and tools available to us:  


**Objective**  

Recreate the 1974 Topps Hank Aaron baseball card while incorporating modern slash line metrics: BA, OBP, SLG, and OPS.  


**Metrics and Formulas**  

- BA  = H / AB
- OBP = (H + BB + HBP) / (AB + BB + HBP + SF)
- SLG = (1B + 2*2B + 3*3B + 4*HR) / AB
- OPS = OBP + SLG  


**Required Variables**  

From the original card, we can see that we need:
- Year, Team, AB, H, 2B, 3B, HR, RBI, Batting Average

To compute additional metrics, we also require:
- BB, HBP, and SF 


**Workflow**
1. Import an existing CSV file (BattingExtract.csv)*
2. Calculate modern slash line metrics
3. Recreate the baseball card as a tabular report

***BattingExtract.csv** was created in the last blog, and it can be downloaded [here](https://samedgemon.github.io/SportsAnalytics/baseball/Blog1_SlashStats/Data/BattingExtract.csv).  


**Data and Tools**
- Historical batting data from a prior project: BattingExtract.csv
- SAS Workbench for data processing, metric calculation, and reporting  

<br>

## Code / Implementation

We will use **SAS Workbench** to complete this task. You can get SAS Workbench <a href="https://www.sas.com/en_us/software/viya-workbench-for-learners.html" target="_blank">here</a>.

**Import** the CSV file:
``` SAS
filename mycsv "<your directory path>/BattingExtract.csv";
proc import datafile=mycsv
   out=temp
   dbms=csv
   replace;
   getnames=yes;
run;
```  

*A temporary dataset (temp) is created.*  

<br>

**Calculate** slash line metrics:  
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
*What once required manual calculation now completes in milliseconds.*
*Note that singles must be calculated for the SLG calculation.*  

<br>  

**Subset** the data using Hank Aaron's playerID (aaronha01) and the yearID to match the baseball card.

``` SAS
data Aaron;
   set SlashStats;
   where playerid='aaronha01' and
         yearid <= 1973;
run;
proc freq; 
   table playerid; 
   title 'Valid: PlayerID';
run;
proc means nonobs n min max ndec=0; 
   var yearid; 
   title 'Valid: YearID';
run;
```
*It is a **best practice to validate** at key steps.*
*We use PROC FREQ to confirm identity, and PROC MEANS to confirm coverage across seasons.*

**Validation Results**

![Freq](https://samedgemon.github.io/SportsAnalytics/baseball/Blog2_715/Images/FreqValid.png) 
![Means](https://samedgemon.github.io/SportsAnalytics/baseball/Blog2_715/Images/MeansValid.png)  

*The tables tell us that all 20 records match Henry Aaron's playerID, and that we have the correct years to match the baseball card.*

<br>


**Creating the Report is not straightforward.** It is important to realize that the “career row” on a baseball card is not another row in the data; rather **it is a derived aggregation.** To put it another way, this is an opportunity to make the classic mistake of *averaging averages.* To avoid making this mistake, we must understand our tools, and we must position those tools for success! **We must improvise!**  

<br>

**Improvising.** Use PROC SQL to calculate the career numbers and create macro variables that we can use later.

``` SAS
proc sql noprint;
    select sum(G), sum(AB), sum(H), sum('1B'n), sum('2B'n), sum('3B'n),
           sum(HR), sum(RBI), sum(SB),
           sum(H)/sum(AB) as BA,
           (sum(H)+sum(BB)+sum(HBP)) / (sum(AB)+sum(BB)+sum(HBP)+sum(SF)) as OBP,
           (sum('1B'n) + 2*sum('2B'n) + 3*sum('3B'n) + 4*sum(HR)) / sum(AB) as SLG
    into :careerG, :careerAB, :careerH, :career1B, :career2B, :career3B,
         :careerHR, :careerRBI, :careerSB, :careerBA, :careerOBP, :careerSLG
    from Aaron;
quit;
%let careerOPS = %sysevalf(&careerOBP + &careerSLG);
```  
*This problem might have been solved with PROC SUMMARY, or a DATA step with retained totals, or maybe with BY group and LAST logic ... but, the choice was to solve this problem with **PROC SQL** because it better replicates our thought processes, i.e. "we need Aaron's career numbers ... what are they?"*

<br>

Grab those macro variables and create a dataset:
``` SAS
data AaronCareer;
    length yearID $10 teamID $10;
    yearID = "CAREER";
    teamID = "";
    G = &careerG;
    AB = &careerAB;
    H = &careerH;
    '1B'n = &career1B;
    '2B'n = &career2B;
    '3B'n = &career3B;
    HR = &careerHR;
    RBI = &careerRBI;
    SB = &careerSB;
    BA = &careerBA;
    OBP = &careerOBP;
    SLG = &careerSLG;
    OPS = &careerOPS;
run;
```
*Let's create a dataset using those macro variables with these 2 things in mind: (1) if it can be avoided, don't compute metrics in reports, and with that in mind (2) calculate once, but use many times!*

*Note that yearID has been assigned the value of "Career" which will be an inconsistent format considering the rest of the data.*  

<br>


Address the format issue and create the final dataset:
``` SAS
data AaronChar;
    set Aaron;
    yearID_char = put(yearID, 10.);
    drop yearID;
    rename yearID_char = yearID;
run;

data AaronFinal;
   retain playerID yearID teamID;
   set AaronChar AaronCareer;
run;
```
*YearID is addressed regarding the format, and a dataset, AaronChar, is created with a converted yearID. AaronFinal SETs together AaronChar and AaronCareer, and RETAIN is simply used to order the variables in the dataset.* 

*Note that Setting these tables is how we are appending the derived career row.*  

<br>

It has all come together so let's create the report:
``` SAS
proc print noobs label;
   var yearID teamID G AB H '2B'n '3B'n HR RBI BA OBP SLG OPS;
   label teamID='Team'
         yearID='Year';
   format G AB H RBI comma9. BA OBP SLG OPS 9.3;
   title 'Hank Aaron Baseball Card';
   title2 '1974 Topps Home Run King';
run;
```  

*Some might say that PROC PRINT failed, but it did exactly what it was supposed to do. Assuming that a reporting PROC could replace an analytical one (or an analytical process) would, however, constitute a failure. The "win" in the case was to recognize that the derived metrics must be recomputed after aggregation.*  

<br>

**The Report: Updated 1974 Topps Henry Aaron Home Run King Baseball Card.**

![Freq](https://samedgemon.github.io/SportsAnalytics/baseball/Blog2_715/Images/Card.png) 

<br>

### Why Baseball Cards Matter
For generations, baseball cards were an introduction to statistics and often to mathematics itself. Kids learned to compare, rank, sum, and average long before they encountered formal coursework. Recreating a baseball card from historical data demonstrates the power of reporting and communication. The value of analytics is not just in calculation, but in presenting information clearly and meaningfully.

<br>

### A Note on Data Science Skills
Watching Hank Aaron hit #715 and later rediscovering his baseball card illustrates how statistics tell a story. In this project, we used modern tools to retell that story with greater clarity and depth. We worked through a complete analytics workflow. We imported an existing CSV file, and wrote SAS code to engineer modern performance metrics. We used macro variables to capture and reuse summary statistics. We recomputed derived measures at the correct level of aggregation, avoiding the common mistake of averaging averages (or ratios). Ultimately, we created a clean and reproducible report that updates a classic baseball card with modern slash stats.

These steps mirror how analysts work in any domain: the data was explored and considered, aggregated (correctly), metrics recomputed, and analytical logic was created separate from the presentation.

<br>

### Next Up
The Eras of the Game. How have changes to rules affected the game? What about changes in the manufacturer of baseballs, or even social change? We will look at runs scored throughout the history of the game and visualize how change has affected the scoring of runs from season to season.

