---
---
{% include nav.html %}


<link rel="stylesheet" href="/SportsAnalytics/assets/css/custom.css">

# The Home Run King
## Henry Aaron Broke Babe Ruth's Home Run Record on April 8, 1974

## History and Origins: 715 Home Runs  

The game between the Atlanta Braves and the Los Angeles Dodgers was played at Atlanta-Fulton County Stadium. One hundred eighty-five miles to the north, a kid sat alone in a house in Paint Rock, Tennessee, on the floor in front of the family television.  

He adjusted the knobs carefully, fine-tuning the UHF channel hoping to get the best picture possible. He was proud of how clear it looked. Then he took a deep breath and waited.  

The Braves started right-hander Ron Reed. The Dodgers countered with left-hander Al Downing. Henry Aaron batted cleanup and played left field. Darrell Evans and Dusty Baker were also in the lineup that night.  

In the bottom of the second inning, with the game scoreless, Aaron led off and drew a walk on five pitches. He never took the bat off his shoulder. He appeared almost detached, surveying the field without emotion. The crowd booed the walk.  

Dusty Baker followed with a double to left. An error by Bill Buckner allowed Aaron to score from first, giving Atlanta a 1-0 lead. In the process, Aaron set a **new National League record for career runs scored.**  

The Dodgers answered immediately. Steve Garvey singled. Bill Russell doubled. Al Downing singled to center, driving in Garvey and moving Russell to third. A ground ball cut down Russell at the plate, but after a popout, Jim Wynn doubled to left, driving in two more runs. Los Angeles led 3-1.  

The kid was still sitting on the floor. He knew Aaron would bat again in the next inning.  

The fourth inning began with Darrell Evans reaching on an error by Bill Russell. **Henry Aaron came to the plate** as the tying run. The crowd of 53,775 — the largest in the stadium's history — rose with a low, electric buzz. Aaron stepped into the box sitting on 714.  

Downing's first pitch was low, in the dirt. The count moved to 1-0.  

The kid rose to his knees and instinctively reached out to touch the television. He noticed the time.  

On the very next pitch, at approximately 9:07 p.m. local time, **Aaron swung for the first time that night and launched the ball over the left-center-field wall for home run number 715.**   

The kid stood and watched the scene unfold. He knew this would be talked about ... forever!  

Years later, that same kid was in college, looking through a box of old baseball cards. One card caught his attention. On the front, it proclaimed Henry Aaron the "Home Run King."  

It was the back of the card that mattered most. **It told a story with numbers!**  

The student bought the baseball card and took it home.  

**1974 Topps Henry Aaron "Home Run King" Baseball Card**

<!-->
<img src="/SportsAnalytics/baseball/Blog2_715/Images/Aaron1974Toppsv2.png"
     alt="Front"
     class="img-card">
-->

![Back](https://samedgemon.github.io/SportsAnalytics/baseball/Blog2_715/Images/Aaron1974ToppsV3.png)
![Back](https://samedgemon.github.io/SportsAnalytics/baseball/Blog2_715/Images/Aaron1974ToppsBackv2.png)


### Hank's Card Needs the Modern Slash Line  

When Henry Aaron played, the statistics most consistently shown were batting average and home runs. If he played today, it would be difficult to discuss his career without also referencing on-base percentage (OBP) and OPS.  

This project updates the **1974 Topps Henry Aaron Home Run King baseball card** using modern sabermetric measures. The goal is not to replace the original card, but to add context and insight using metrics that better describe offensive value.

## Building the Modern Slash Line from Historical Data  

### Objective  

Recreate the 1974 Topps Henry Aaron baseball card while incorporating modern slash line metrics: BA, OBP, SLG, and OPS.

### Metrics and Formulas  

- BA  = H / AB
- OBP = (H + BB + HBP) / (AB + BB + HBP + SF)
- SLG = (1B + 2*2B + 3*3B + 4*HR) / AB
- OPS = OBP + SLG

### Required Variables  

From the original card, we can see that we need:  
**Year, Team, AB, H, 2B, 3B, HR, RBI, Batting Average**

To compute addtional metrics, we also require:  
**BB, HBP, and SF**


### Workflow
1. Import an existing CSV file (BattingExtract.csv)*
2. Calculate modern slash line metrics
3. Recreate the baseball card as a tabular report

*BattingExtract.csv was created in the last blog, and it can be downloaded [here](https://samedgemon.github.io/SportsAnalytics/baseball/Blog1_SlashStats/Data/BattingExtract.csv).  

### Data and Tools
- Historical batting data from a prior project
- BattingExtract.csv
- SAS Workbench for data processing, metric calculation, and reporting

## Code / Implementation  

We will use **SAS Workbench** to complete this task. You can get SAS Workbench <a href="https://www.sas.com/en_us/software/viya-workbench-for-learners.html" target="_blank">here</a>.

Import the CSV file:
``` SAS
filename mycsv "<your directory path>/BattingExtract.csv";
proc import datafile=mycsv
   out=temp
   dbms=csv
   replace;
   getnames=yes;
run;
```  

A temporary dataset (temp) is created.

Calculate slash line metrics:  
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
What once required manual calculation now completes in milliseconds.

We need to "find" Henry Aaron in the data. His playerID is "aaronha01'

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
It is a **best practice to validate** at everystep. We check to make sure that all playerID's are what we expect, and that records exist for each year oh Henry Aaron's career up to and including 1973.

Validation.

![Freq](https://samedgemon.github.io/SportsAnalytics/baseball/Blog2_715/Images/FreqValid.png) 
![Means](https://samedgemon.github.io/SportsAnalytics/baseball/Blog2_715/Images/MeansValid.png)


Creating the Report:

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
Creating the Report:
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

Creating the Report:
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

Creating the Report:
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

The 1974 Topps Henry Aaron Home Run King Baseball Card:

![Freq](https://samedgemon.github.io/SportsAnalytics/baseball/Blog2_715/Images/Card.png) 



### Why Baseball Cards Matter
For generations, baseball cards were an introduction to statistics�and often to mathematics itself. Kids learned to compare, rank, sum, and average long before they encountered formal coursework.
Recreating a baseball card from historical data demonstrates the power of reporting and communication. The value of analytics is not just in calculation, but in presenting information clearly and meaningfully.

#### A Note on Data Science Skills
Watching Henry Aaron hit #715 and later rediscovering his baseball card illustrates how statistics tell a story. In this project, we used modern tools to retell that story with greater clarity and depth.
In doing so, we practiced core data science skills used by analysts every day:
* Importing external data from a CSV file
* Writing code to engineer new features and metrics
* Applying macro-style techniques to make calculations repeatable
* Updating a report to reflect modern analytical standards
* Communicating results through a clean, structured table
Along the way, we also defined objectives, identified required variables, selected appropriate tools, and followed a reproducible workflow from raw data to final output.
These same steps apply across industries�finance, healthcare, operations, manufacturing�not just baseball. The context may change, but the process remains the same.
In this case, that process allowed us to revisit one of baseball�s most historic moments and view it through a modern analytical lens.

### Next Up
The Eras of the Game.

