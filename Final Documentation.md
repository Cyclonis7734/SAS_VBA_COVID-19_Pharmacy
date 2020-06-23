- [COVID-19 and Pharmacy Claims Counts](#covid-19-and-pharmacy-claims-counts)
- [Beginnings](#beginnings)
- [Cleaning Up the COVID-19 Data](#cleaning-up-the-covid-19-data)
  * [The COVID19 Guide File](#the-covid19-guide-file)
    + [The Guide File&#39;s Back End](#the-guide-file--39-s-back-end)
    + [Further Back End Necessities](#further-back-end-necessities)
  * [Pre and Post Clean Up Comparison](#pre-and-post-clean-up-comparison)
- [Getting SASsy](#getting-sassy)
  * [Improving Quality of Life in SAS](#improving-quality-of-life-in-sas)
  * [Back to COVID-19 Data](#back-to-covid-19-data)
  * [Cigna Pharmacy Data](#cigna-pharmacy-data)
  * [Is the Breakout Necessary?](#is-the-breakout-necessary-)
- [Combo Data Sets and Correlations](#combo-data-sets-and-correlations)
  * [Find the Patterns](#find-the-patterns)
    + [Why is this Happening?](#why-is-this-happening-)
- [Shoot Your Shot](#shoot-your-shot)
  * [Planetary Models](#planetary-models)
  * [No Model Objects](#no-model-objects)
- [Discussion Topics](#discussion-topics)
- [Conclusion](#conclusion)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

# COVID-19 and Pharmacy Claims Counts

Author: Thomas Seggerman

# Beginnings

It is no surprise that COVID-19 data is a topic of concern, during the midst of this pandemic. The world stands to learn a lot from data pertaining to the virus, and has already begun to reveal its use, on many platforms. I work for Cigna Healthcare, on a Pharmacy Reporting team, where I am responsible for creating many types of reports. Once the pandemic reared its ugly head, I was asked if I had any interest in mixing COVID-19 data with Cigna Pharmacy data, to do some Exploratory Data Analysis (EDA). I decided that would be a great topic for me to tackle, for my first Practicum!

Initially, I was curious how I would be able to obtain data from Cigna, so that I could use it in Python or R, per the Practicum&#39;s requirements. However, as I delved into the options for programs available within Cigna, that could do prediction modeling and visualization creation, I realized that SAS was a viable option. I have only used SAS for one or two various data queries in my current role, and never used it in previous positions. After discovering that SAS could be used for prediction modeling, and with the knowledge that my role would eventually require that I become more familiar with its use, I decided to try and run with it as being my language of choice for the assignment. I recommended that I use SAS for the main host of the data, for visualizations, and to produce prediction models. I also mentioned that I would, possibly, need to use VBA in Excel for some minor Extract, Translate, and Load (ETL) operations as well.

After it was assured that aggregated representations of Cigna&#39;s data was the only expression of their data that would be shown publically in any way, Cigna and ESI Management were more confident that the project would be approved, and the waiting game began. After waiting for approval, and beginning to get a better gauge on the breadth of the challenge at hand, now was a good time to discern what data was available for COVID-19. Researching on data sources available for COVID-19, showed that there were, basically, two major players. The first being the Worldometer, which is used widely for many various reporting agencies and bodies. The second main source,  John Hopkins University&#39;s (JHU) Center for Systems Science and Engineering, made a very informative dashboard. Fortunately, the JHU datasets are available to the public, and provide aggregated data from multiple sources. In reviewing the JHU data sources, you can see that they even use the Worldometer&#39;s dataset in creating their main COVID-19 dashboard.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%201%20-%20JHU%20Data%20Sources.png)

Armed with knowledge of the data that is available, I began trying to piece together a more refined version of the project, to Management. My aim was going to be to try and find any correlations with our Pharmacy data against COVID-19 data, which might be useful for Cigna Pharmacy. The potential for usefulness was not necessarily known at first, and it was decided to wait and see what could be found during EDA.

# Cleaning Up the COVID-19 Data

JHU&#39;s COVID-19 data is downloadable from their Github repository, in an assortment of text delimited (txt) files. They are saved files with the extension of &quot;.txt&quot; and are delimited with commas, making them, technically, comma delimited &quot;.csv&quot; files. After downloading all files desired, I simply renamed the files to have a .csv extension, rather than .txt. This is something that could have been done via VBA code. However, during the waiting process for the files to download, I opted to spend the small amount of time required to rename the files after downloading them, instead. There are individual files for each date that they have aggregated data into, for all days provided. The data they collected, begins on April 12th of 2020, which only gives us a total of around 50 days&#39; worth of data. IT policies, within Cigna, dictate that data storage on the SAS servers is heavily regulated and monitored. It is known that having too many large files within the allotted server storage space provided for SAS, can make IT flag your account. So it was necessary to find a way to merge all of the JHU delimited files together, into a single file for transferring to the SAS storage server space. VBA in Excel, wound up being the simplest solution to avoid being flagged.

## The COVID19 Guide File

My first step in this process, was to create an Excel file that could act as a guide for noting everything that I needed to create, or do, for the project, in various lists. I figured it would be a great place to keep track of everything, as well as make a host for VBA code that could be run at any point, to convert all downloaded JHU delimited files. Below, you can see the main tab which kept track of options and settings for the import of the JHU files. Various issues were discovered throughout the process, which are addressed in the next few sections.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%202%20-%20Guide%20File%20Main%20Tab.png)

Cell J1 (light blue highlighted cell in the above image), is a location that is used by the VBA code to hold a text value of a State from the USA, which has been read from the JHU files. Essentially, the VBA code will place a row&#39;s State value into this location, and the two cells below it will update via formulas. Those two cells (J2 and J3) below, will tell us whether to keep that row or not, and what the State&#39;s abbreviated text value is. The abbreviated text value will help us to match up information by state, later, if we discover that we need or want to do so, since that&#39;s how it&#39;s stored in Cigna&#39;s Pharmacy tables.

The formulas for J2 and J3 are:

J2: =IF(ISERROR(MATCH(J1,L:L,0)),&quot;N&quot;,&quot;Y&quot;)

J3: =IF(J2=&quot;Y&quot;,VLOOKUP(J1,L1:M50,2,0),&quot;&quot;)

The list in Column K is a copy of the column from the JHU files with the heading &quot;Province\_State,&quot; which is what was used to observe all possible values for the States being shown in the JHU data. The values in column L, hold the spelling of every state that we want to keep. This makes it so we can match the values from each row in the JHU files, by using a VLOOKUP function we obtain the two character abbreviations for each state.

### The Guide File&#39;s Back End

In Column C and D, in the above image, you&#39;ll see that row 7 is a header of sorts. Rows 8-10 in Column&#39;s C &amp; D, are yes/no settings that the VBA code will read, so that we can decide if we want to perform various steps during the merging of all the JHU files. This leads us to the next portion of the Guide file, where the VBA code ties into the Excel front-end, from the above image. The below images from VBA, shows the code that is used to perform various steps in the merging process. We&#39;ll stop and take a look at various spots, in order to try and clarify what the code is doing. In order to execute the VBA code, the button from the above Worksheet image (Create Combined csv File), just needs to be clicked.

Abiding by good programming practices, the first few lines of the VBA code will declare and set variables, as much as possible. Below, you can see the first line of code is making the &quot;Option Explicit,&quot; declaration. This enforces that all variables must be declared, before usage. To declare variables in VBA, we use the Dimension (Dim) command, to separate out a block of memory for the variable we want to create. After declaring all of our variables that we&#39;ll be using, we instantiate a few of them, as soon as we can, using the Set command for objects or just using an equals sign for primitive types. The range object being created, is to host a location where an Excel function can count the rows with values in them. As we move data from the JHU files, into our single Excel file that will be the final product, this number will iterate, automatically, giving us the proper row to push new data into. The Dir function on the last line, is a widely used function that provides a simple method for iterating through your directories in Windows environments. Its usage, in the manner shown below, dictates that it will return the first file it finds, that matches its simple RegEx expression. In this case, any csv files in the given directory.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%203%20-%20Guide%20File%20VBA%20Vars.png)

Now that we finished our declarations and sets for the variables that we are able to do so with, the process of looping through all of the JHU files can begin. Below, you can see that we use a Do loop to begin the process. Most of the code contains comments, describing what is happening at each line. The basic premise for the process, overall, is that we&#39;re looping through each of the JHU files, adjusting some of their layout where necessary, then pulling all rows into our final JHU file.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%204%20-%20Guide%20File%20VBA%20CCFC19%20Method.png)

### Further Back End Necessities

The VBA code up to this point, was for initial expected needs to be met. It was thought, mostly, that the state abbreviations would be an absolute necessity, so no parsing of that functionality was necessary. Setting the code up as an optional feature wound up becoming unnecessary shortly after beginning to write the code. However, it was still unknown as to whether more options for running various code, or functions, would be necessary. Thus the reason for creating a section to handle optional code running on the Guide file. After observing the finalized file more closely, a realization set in that the removal of rows where the state was not part of the 50 contiguous states of the USA, might be something worth doing. This was the main reason for the creation of Cells D8:D10, so that it would be easy to turn on and off, these options, if the need ever arose. In the event that it becomes desired to have less intrusion on the COVID-19 data, one could simply set those values to N, and be done. Below, is the remaining code for the Main Method (CreateCombinedFileCOVID19) from the previous pictures of code. As you can see, it is checking the 2 other options to be as Y or N, then calls the methods tied to those options, if desired.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%205%20-%20Guide%20File%20VBA%20Option%20Calls.png)

The method created for removing rows that did not have 1 of the 50 contiguous states in the US was called &quot;RemoveNOTStates.&quot; Below is the code for that method, and once again, you can see that the comments guide you through what is happening at each line. Basic premise here, is that the code is looping through all rows in the combined JHU file that was just made, and performing various tasks.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%206%20-%20Guide%20File%20VBA%20RNS%20Method.png)

After processing all of the files in SAS, it would become a necessity to obtain the daily counts of the values from the data. In order to do this, it was necessary to update the Excel VBA code file. The daily counts could be obtained by using a simple subtraction formula, after sorting the data. Basically, sort the data by State then date, and a historic listing for each state, alphabetically, is the result. The formula we use to get the counts, is then simply checking to see if the state is the same as the previous row. If that is true, then we subtract the current row&#39;s count from the previous row, returning the daily increase/decrease of every data point from the JHU files. Below, you can see the code that does just that. There&#39;s not as much commenting on this one, because the language used in most of the lines are fairly self-explanatory as to what each are doing. The nested &quot;With&quot; statement has several lines of code, towards the end of it, which are necessary to set when working with the Sort property of a Worksheet object. The variable &quot;longLast&quot; was set during the previous optional method, RemoveNOTStates. Since that method has always been used, it was not necessary to set it within this separate method, as the variable&#39;s scope was set to Public in this Module.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%207%20-%20Guide%20File%20VBA%20ADC%20Method.png)

## Pre and Post Clean Up Comparison

As with all Data projects of this nature, it is important to get a look at the data, before and after it has been cleaned. Below is an image from the original JHU file for 4/12/2020.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%208%20-%20csv%20file%20Pre%20VBA%20Runs.png)

Below is a transpose function ran on the header and the first 5 rows, along with a column I made, to show the column letter. This just looks a little bit cleaner than the above image, in some ways. You can clearly see that there are several columns which are simply not going to be used, for various reasons. The UID, ISO3, Country\_Region, Lat, and Long\_ columns are, mostly, for distinctions that we won&#39;t be needing. Many of these columns were probably used for placement on the JHU dashboard.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%209%20-%20csv%20file%20Pre%20VBA%20Runs%20transposed.png)

Next, we&#39;ll take a look at the clean version of the JHU files, after they&#39;ve been combined. Notice the sort command&#39;s effect on the data, and the accuracy of the math for columns U:Z making daily increase/decrease counts possible.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2010%20-%20Final%20file%20Post%20VBA%20Run%20transposed.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2011%20-%20Final%20file%20Post%20VBA%20Run.png)

# Getting SASsy

Now that we have a finalized version of the COVID-19 data, we can take it over to the SAS storage server, and use it to create tables on the SAS server. The COVID-19 data clean up and gathering, was the only portion of this project that was not done in SAS. The data from Cigna/ESI is mostly clean already, and requires SAS&#39;s SQL-like code in order to pull it in the format that we want. For now, we&#39;ll start out by looking at the SAS code which pulls the final version of the JHU file&#39;s data into a temp table, then we will perform some basic EDA.

To get the data from my local laptop for work, over to the SAS Storage server, Cigna provided a program called winscp.exe. WinSCP is a simple transfer program, presumably for transferring from Windows Operating Systems to Linux or Unix based Operating Systems. In my own experience, the term &quot;scp&quot; is a secure copy command on Linux based Operating Systems. I never found out what the SAS servers OS&#39;s were, but in the end, it doesn&#39;t matter. After pushing the file onto the storage server, the below code was used to access the file and import it into the SAS workspace as a temp table. The schema title &quot;WORK&quot; is SAS storage for temporary tables. Tables created in that schema, only exist for as long as the instance of SAS that made them is running. The first line of code uses the SYSECHO function, and is similar to a console output message, or a print text feature, in most programming languages. The SYSECHO function is being used repeatedly throughout this project, to dictate when a new section, or operation, has begun.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2012%20-%20SAS%20Code%20Import%20COVID%20File.png)

After importing the file, the raw file is interpreted by SAS automatically. Similarly to many other programming languages with automated methods for importing data sets, it is not accurately able to interpret the data. The next code steps take on the challenge of further interpretation of the file, and basically reimport the data from the raw import, into a new table with the corrected formats that will be used throughout the remaining SAS project. Below, you can see that the columns are all being pulled in, and only the fields we need are being kept. The Filename column is being heavily altered, with one line of code. This particular line is using a substring method within SAS, to parse out the filename, into a date field instead. Outside of the substring created date field and State field, only the data points that were relative to counts of people, falling under the various types of categories provided by the JHU files, were kept.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2013%20-%20SAS%20Code%20ETL%20from%20Raw%20COVID%20File.png)

At the end of the &quot;CREATE TABLE&#39;s&quot; semicolon, you can see the word &quot;QUIT;&quot; This dictates the end of the &quot;PROC SQL&quot; statement at the top. Basically, SAS uses declaration functions, which act as a sort of mode declaration. &quot;PROC SQL&quot; could be interpreted as &quot;SAS is now going to begin interpreting SQL-like instructions from you.&quot; During processing, SAS will attempt to convert this code into the SQL code required by your Database (DB) of choice. After &quot;PROC SQL&quot; commands have finished, we can see another &quot;proc&quot; statement, for a &quot;contents&quot; function call. This set of code is to review the table provided in the &quot;DATA&quot; parameter&#39;s attributes, and various details about the table&#39;s setup. This includes the row and column counts, Observations and Variables, respectively, along with a plethora of other details. Below, we&#39;ll take a look at the output from both the &quot;PROC SQL&quot; and the &#39;proc contents&quot; commands.

&#39;PROC SQL&#39; - CREATE TABLE command results:

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2014%20-%20SAS%20table%20COVID_CLEAN.png)

&#39;proc contents&#39; command results:

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2015%20-%20SAS%20proc%20contents%20COVID_CLEAN%201.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2016%20-%20SAS%20proc%20contents%20COVID_CLEAN%202.png)

The last table in the &#39;proc contents&#39; command, gives us the column types and their formats, along with the label and column name for reference.

## Improving Quality of Life in SAS

After manually altering the SAS code over the first few weeks, it became desirable to create some global variables. The point of adding them, was to remove the hassle of manually entering dates in, whenever new data became available. It is necessary, at this point, to simply note that this code exists, so that there is not confusion later. When the variables are used in various queries, this is where those variable&#39;s values are being set. To create globally scoped variables in SAS, you first have to list off all of the variables via a &quot;%GLOBAL&quot; declaration statement. This is the only distinction between global and local variables in SAS. Local variables can be declared and set, by using the &quot;%LET&quot; statement only. So, by declaring them as global, then setting them by using the &quot;%LET&quot; command, we accomplish making the scoping of the variable as global. The &quot;proc print&quot; command at the end of this code snippet, simply forces SAS to produce a pointless title in the Results output from running this code. The Title output can be seen below the code image. Notice that the &quot;tick&#39;s&quot; in the variable setting are not indicative of a string variable type. When using a &quot;%LET&quot; command, everything after the equal and following space characters becomes part of the variable, until SAS reads a semicolon character. The format of the dates shown below, are how SAS expects dates to be formatted, for use inside functions involving date comparisons.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2017%20-%20SAS%20Code%20Global%20Vars.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2018%20-%20SAS%20Global%20Vars%20Test%20Results.png)

## Back to COVID-19 Data

Now that we&#39;ve got some global variables setup, we can get back to the fun part of the project, where we explore our data! With COVID-19 data available in SAS tables, a good starting point was to recreate some of the more widely known stats from various news sources. Right away, I knew that New York would be a good state to focus on, since it has seen the deepest impact of COVID-19, bar none, in the USA. Below, you can see the code used to create a special temp table. This table was organized in such a way, so as to gather the final date&#39;s values for each of the JHU category types. This was accomplished by using the &quot;MAX&quot; command, rather than specifying a date, so that I did not have to try and figure out which date was going to be used anytime we updated the JHU file. The strategy of using &quot;MAX&quot; only worked for this particular EDA point. After creating the table COVID\_ST\_SUMM, the &quot;proc means&quot; function is run on it, so that we can review the various averages of the table&#39;s columns. Below, you can see both results from the code, in their respective order.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2019%20-%20SAS%20Code%20create%20COVID_ST_SUMM.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2020%20-%20SAS%20table%20COVID_ST_SUMM.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2021%20-%20SAS%20proc%20means%20COVID_ST_SUMM.png)

After reviewing the &quot;proc means&quot; function&#39;s results, the table&#39;s details were not very interesting to read. Considering that I know NY&#39;s counts are skewing the data drastically, a comparison of the 50 states, visually, would make for better impact. Below, you can see the &quot;proc SGPLOT&quot; function is being used to create a bar chart based on State, by Confirmed Cases count total. The first two lines of code, are forcing a graphical setting to make the &quot;SGPLOT&quot; function have a specific width and height for its print-out of the chart.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2022%20-%20SAS%20Code%20proc%20sgplot%20COVID_ST_SUMM.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2023%20-%20SAS%20proc%20sgplot%20COVID_ST_SUMM%20plot.png)

There we go. That&#39;s the scary data that New York experienced. New York more than doubles the counts from the second place state for confirmed cases of COVID-19. On a lighter note, this skewness was even worse, when I first started running the code to create this bar graph, around the first few weeks of this project. This means that New York has really made a dent in lowering their count of confirmed cases, up until this point.

Next, I wanted to get a basic overview of the 3 main points of data from the JHU files: confirmed, recovered, and deaths. Again, this was desired because it should match output seen on Worldometer&#39;s website, or at least be close in comparison. While the visualization of cumulative data would be slightly interesting, the daily counts of each type of data would be preferred. For ease of creating any plots from this perspective, it was decided to pull in the columns for all fields, except state, then create however many plots would be desired.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2024%20-%20SAS%20Code%20create%20COVID_DATE_SUMM.png)

With the above table ready for use, the next step was to figure out how to display the data on a plot of some sort. The below code works with the SAS function &quot;proc gplot&quot; to create two Y-axis breakouts. The multiple function calls before the &quot;proc gplot&quot; command are defining various pieces of the plot which will be passed during the main &quot;proc gplot&quot; function call. The reason for the separation of the Y-axis onto two sides, was to make it so that the confirmed cases total could be viewed separately on a different, much larger, scale. When all 3 data points were kept on the same axis, the &quot;CONFIRMED\_TOTAL&quot; column&#39;s point values dwarfed the visibility of the other two columns for &quot;RECOVERED\_TOTAL&quot; and &quot;DEATHS\_TOTAL&quot;. Splitting the Y-axis made it so that the progression differences, for the other two points of data, were actually visibly able to be seen fluctuating. When using a single axis, the two other column&#39;s values appeared to be a never changing line along the bottom of the plot.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2025%20-%20SAS%20Code%20proc%20gplot%20COVID_DATE_SUMM%20cumulative.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2026%20-%20SAS%20proc%20gplot%20COVID_DATE_SUMM%20cumulative%20plot.png)

The right side&#39;s Y-axis is for the &quot;CONFIRMED\_TOTAL&quot; count only. The left side is for the red and green (DEATHS\_TOTAL and RECOVERED\_TOTAL) stats only.

While this was at least mildly interesting to see, the more informative plot would revolve around the daily counts. I find this to be the case, because the whole point of the social distancing was to &quot;lower the curve&quot; down to a level that medical facilities can actually handle. That said, the daily counts seemed to be close to mirroring the visualizations found on Worldometer&#39;s website for the same view of the data, verifying that I was aggregating the data properly. Differences found could, most likely, be attributed to the removal of rows where the various States were not retained, in the VBA code used for merging the JHU files together.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2027%20-%20SAS%20Code%20proc%20gplot%20COVID_DATE_SUMM.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2028%20-%20SAS%20proc%20gplot%20COVID_DATE_SUMM%20plot.png)

The confirmed daily totals line is very similar to the data we&#39;ve seen on Worldometer&#39;s site, where we were hovering at 30k confirmed daily cases, on average, for a while. Over the past month, we&#39;ve slowly dipped down to around 25-20k confirmed cases daily instead. Of notable attention, the first entry for each stat has sudden high spikes that drop, after the first day. This shows us that the first day of every stat should probably not be used, as it is a summation for all of the entries that occurred up to that first date, which we do not have data for. If you were wondering, the confirmed daily total count&#39;s first date value is far higher than the Y-axis limit of 50k, sitting at 552,194.

## Cigna Pharmacy Data

Now that we&#39;ve taken a basic look at the COVID-19 data from JHU, I wanted to see what data I could obtain from Cigna that might be useful. Right away, I wanted to find details about specific drugs, used to treat COVID-19. My search started online, looking for various websites that might have this information. The final location I wound up using was a webpage written on the site goodrx.com, authored by Jennifer Tran, PharmD, RPh. She listed the top 10 drugs that are, or were at the time, being used to try and treat COVID-19 to some extent. The final list she created, can be seen in my comments in the code images below. The first set of code, was used to create a table of drugs which I could sift through to see if I could find each of the drugs listed on Tran&#39;s article. In the list of the 10 drugs mentioned, the drugs which have an arrow pointing right (---\&gt;), are stating that I found a drug which had a brand name that matched with the name of the drug listed on Tran&#39;s article. So, arrow equals found, no arrow equals not found in our drug lists. For Cigna data protection, I opted to not include the results of this list, just to be safe. Ultimately it was pretty straight forward to match on the brand name, and I wound up just doing this once, just to be certain which field (BRND\_NM or LABEL\_NM) could be used for flagging specific drugs.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2029%20-%20SAS%20Code%20create%20DRUGS_LIST.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2030%20-%20SAS%20Code%20Comments%20Results%20for%20Drug%20Search.png)

With a list on hand of what Brand Names could be found in our databases, I began to sift for other information that was available. Initially there might have been a use for supply amounts as a feasible data point, if for nothing else than to see if there was a massive amount of claim counts for specific supply quantities, at the beginning of the lockdown. Which, as you can see in the image below, was also a value that was pulled in as well, &quot;CLAIM\_COUNT.&quot; State data was also pulled, to try and discern how certain states might have been effected. Lastly, an attempt was made to try to find a way to identify if specific claims may have originated from pharmacies that mainly service hospitals, rather than all other pharmacies. While a good idea that could be very useful for working with Hospitals, it wound up not being easy to flag pharmacies that way. Cigna&#39;s database had a field that would seem to specify a pharmacy as servicing a hospital mainly, but in reviewing that field, it wound up leading to a dead end for various reasons. At any rate, the entire field was completely empty aside from 2 entries, making it useless anyway.

The only other method thought up, was to use the address as a means of creating a match near a hospital somehow. Maybe get a list of hospital addresses, and see what could be found? This could possibly be used as a way to narrow down which pharmacies had an address that was within, or nearby, a hospital. This also wound up being difficult to implement, for several reasons. The main one being that pharmacies which I was able to find, that definitely serviced a hospital mainly, wound up, usually having totally non-related names from the hospital, and often had an address with a totally different street name. This would mean there would have to be a system to declare a percentage match to a hospital, based on some sort of criteria. While possible, it would be a huge time suck, and in the end it wound up being thought best, to simply cut our losses instead. If nothing of use could be found from this data, Management would probably let me know.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2031%20-%20SAS%20Code%20create%20PHARM_CLAIMS.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2032%20-%20SAS%20table%20PHARM_CLAIMS.png)

Now that I have a list that could be used to aggregate Cigna/ESI&#39;s pharmacy data, I wanted to do some basic EDA on it. So, I decided to take a look at the brand name usage over time, and see if there has been a definite increase in usage of some of these drugs that are being used for COVID-19 treatment. Below, I created a table to be used in the graphical plotting, using the &quot;sgplot&quot; function. The column &quot;MOYR&quot; was used to create a view that would be based on Month and Year, so that the x-axis would not be overwhelmed, visibly, with dates. Then I simply summed up the claim counts per drug identified. I made sure to leave off the &quot;Other&quot; category, which would dwarf all of these specified drugs, if left in. The commented out line in the Where clause, was used to create a more narrow view, from 2019, rather than the range we wound up using in the above table&#39;s pull from 2017 onwards. The &quot;sgplot&quot; function&#39;s &quot;series&quot; method, wound up being the most interesting one to use for this particular visualization. After using the &quot;sgplot&quot; function in the earlier visualizations, my experimentation mindset dictated that I should try using the &quot;gplot&quot; function instead this time. This method wound up making it a little less cluttered with code, and provided a similarly striking visualization.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2033%20-%20SAS%20Code%20create%20and%20sgplot%20PHARM_DRUG_DATE.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2034%20-%20SAS%20sgplot%20PHARM_DRUG_DATE%20plot.png)

From the above data, you can clearly see that 2 drugs had a fairly obvious surge in claims, around the peak of the COVID-19 confirmed cases increases in March. Azithromycin (Red Line) and Hydroxy Chloroquine (Brown Line), specifically. The Hydroxy Chloroquine spike was nearly double the monthly usage up to that point, and dropped after March. A spike happened for Azithromycin, but not nearly in the same respective percentage as Hydroxy Chloroquine. The remaining drugs from Tran&#39;s list, did not appear to be as highly effected.

Lastly for Cigna/ESI&#39;s pharmacy data, I had another desired visualization for EDA, concerning the day supply/quantity ordered. The code for this table creation wound up being nearly identical to the drug specific table above. Basically we just swapped out &quot;BRAND\_NAME&quot; for &quot;SUPPLY\_AMOUNT&quot; in the Select statement. This time, I also requested a grid outline for both the x and y axis.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2035%20-%20SAS%20Code%20create%20and%20sgplot%20PHARM_SUPPLY.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2036%20-%20SAS%20sgplot%20PHARM_SUPPLY%20plot.png)

As you can see, 30 day subscriptions top the list for this request, by and large. Right away, we notice a spike in claims for 30 day subscriptions in March, then an immediate drop. The same seems to be true, at least mildly, for 30-90 day and temp usage subscriptions. The immediate drop would probably be due to the lockdown, when nobody wants to go out anywhere, or simply is not allowed to. So, this would appear to be something that might be useful for Cigna/ESI. Due to the, seemingly, inherent nature of the 30 day subscriptions to jump around by, roughly, 20-30k per month, it is difficult to say for certain that COVID-19 was the cause for a spike here, though.

## Is the Breakout Necessary?

After pulling the other information from the Cigna Pharmacy DB, further talks with Management wielded an uncertainty as to the usefulness of these points being predicted in any sense. The state data didn&#39;t seem to be a dead-ringer for interest as well, or at least would be uncertain as to its use, at best. The simple concept of claim count being able to be predicted, seemed to be a possible target that might be useful though. So, the next step was to combine COVID-19 data with the Pharmacy data, into a singular table that could have correlation testing and prediction models created for it. To do this, I would need to match up information from both data sets by date, and create a singular table.

# Combo Data Sets and Correlations

The steps used to create the combined table, involved creating two new temp tables first, then joining them together afterwards. This kept the process simple, and would allow me to have different aggregation types for each table.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2037%20-%20SAS%20Code%20create%20COVID_FIN%20and%20PHARM_FIN.png)

With both temp tables created, we could now just do a simple join statement to combine them.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2038%20-%20SAS%20Code%20create%20COVID_PHARM_FINAL.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2039%20-%20SAS%20table%20COVID_PHARM_FINAL.png)

As you can see, the above table worked out to be the main table to be used for the remainder of the project. The first date of April 12th, 2020 was skipped, by using the global variable for a begin date. The cumulative totals for each JHU column wound up being dropped, after attempting to use them in finding correlations proved to be completely useless. Just to do some due diligence, a review of the basic &quot;proc contents&quot; and &quot;proc means&quot; function&#39;s output tables were reviewed. Below, some of the &quot;proc contents&quot; function&#39;s results are not being shown, in favor of simply showing the useful portions of the results and not having to redact any information.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2040%20-%20SAS%20Code%20proc%20contents%20and%20means%20COVID_PHARM_FINAL.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2041%20-%20SAS%20proc%20contents%20COVID_PHARM_FINAL%201.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2042%20-%20SAS%20proc%20contents%20COVID_PHARM_FINAL%202.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2043%20-%20SAS%20proc%20means%20COVID_PHARM_FINAL.png)

## Find the Patterns

With a combined data table ready for usage, next it was time to figure out how SAS handles correlation determination. After researching and testing the available options for correlation determination in SAS, the basic &quot;PROC CORR&quot; function would suffice. The other various options/parameters for the function, wound up not really being terribly useful for this project&#39;s needs. Below, you can see that the &quot;PROC CORR&quot; function makes use of the graphics settings through the systemic &quot;ods&quot; function, as we set the preferred width and height. Within the function call, the option of &quot;PLOTS=MATRIX&quot; is what creates the output of charts, comparing all of the columns in the data used, to be compared against one another. This makes it easier to see if there are obvious correlations, visually, when we map the data points against each other. The parameter pass to the &quot;MATRIX&quot; option/function of &quot;HISTOGRAM&quot;, tell the &quot;PROC CORR&quot; function to create histograms of groupings of the data points, when they are compared to themselves. The &quot;NVAR=ALL&quot; parameter setting, tells the &quot;MATRIX&quot; function to use all of the columns available, in the data set being passed.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2044%20-%20SAS%20Code%20proc%20corr%20COVID_PHARM_FINAL.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2045%20-%20SAS%20proc%20corr%20COVID_PHARM_FINAL%201.png)

As you may have noticed, the title for the above data, says &quot;Correlations BEFORE Weekend Removal.&quot; This was due to the discovery of outliers for weekend data, which we&#39;ll discuss a little later. Below, is the matrix output, which can be used to find obvious patterns in the comparison of columns to one another. You&#39;ll notice that many columns have at least some sort of pattern that they follow. Whenever you see dots that lie outside of the various groupings or lines, those are usually outliers that reduce the correlation of the two columns to one another. Since claim count is our response variable, it became the main focus for observation, more than the other columns. Ideally, at least one column that has a strong correlation with &quot;CLAIM\_COUNT,&quot; would be present. In reviewing the correlation scatter plots below, you can clearly see that there seems to be two different groupings for each variable, when plotted against the claim count data. The above table (Pearson Correlation Coefficients) which shows the correlation coefficients (top) and p-values (bottom) for each cross comparison, shows no promising results for any columns to have a correlation with CLAIM\_COUNT. The highest correlation being with DEATHS, at a measly .25815 correlation coefficient and a .07 p-value.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2046%20-%20SAS%20proc%20corr%20COVID_PHARM_FINAL%202.png)

### Why is this Happening?

The problem with the data wound up being that the weekend claim counts were severely lower than the weekday claim counts. Thus, the name for the title being &quot;BEFORE&quot; weekend removals. This is also the reason that the column &quot;WDAY&quot; was implemented, so that a filter could be placed on the data, to specifically remove the weekend dates. The below code took care of this step and, at the same time, removed the Memorial Day holiday that took place during the given timeframe on May 25th. The 26th was also removed, as that day seems to have been a &quot;catch up&quot; day which had an unusually high claim count as well. The reason for these outliers to exist is still unknown for certain. However, my work colleagues and I have agreed that the most likely cause is that the &quot;filedate&quot; which is used to create a claim, is not necessarily the same date as when the prescriptions/subscriptions were actually given to someone to use. Thus, this makes it more of an admin staff paperwork issue, rather than an oddity, per se. After removing the weekend dates, and the holiday dates, I actually dropped the WDAY column as well, because I didn&#39;t need to use it for anything else, and it just took up excess space in the &quot;proc corr&quot; output&#39;s matrix. After the column alterations, we simply rerun the same &quot;proc corr&quot; function as previously done.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2047%20-%20SAS%20Code%20proc%20corr%20COVID_PHARM_FINAL%203.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2048%20-%20SAS%20proc%20corr%20COVID_PHARM_FINAL%204.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2049%20-%20SAS%20proc%20corr%20COVID_PHARM_FINAL%205.png)

As you can see from the above tables, the resulting correlations were much better than the previous run&#39;s results. In particular, a strong correlation was found with the CONFIRMED and DEATHS columns, of -.68630,\&lt;.0001 and -.48917,.0039 for the correlation coefficients and p-values, respectively. That said, it is safe to say that a Null Hypothesis could probably be rejected, given the extremely low p-values for both of those columns compared to claim counts. The proper method of expressing this would be:

**H**<sub>0</sub> = There **IS NOT** a correlation between CLAIM\_COUNT totals from Cigna/ESI pharmacy data, and<br/>at least one of the COVID-19 case counts from the JHU data.

**H**<sub>1</sub> = There **IS** a correlation between CLAIM\_COUNT totals from Cigna/ESI pharmacy data, and<br/>at least one of the COVID-19 case counts from the JHU data.

Given that two p-values for CONFIRMED and DEATHS totals from the JHU data are very close to 0, we can reject the Null Hypothesis that there is not a correlation between at least one of the COVID-19 case counts from the JHU data.

# Shoot Your Shot

Now that we are good to go with some evidence of a correlation between our COVID-19 data and the Pharmacy data, we should be able to create a prediction model that has some decent accuracy. First step for this, is to split our data into training and testing data sets. Below, you can see a simple select all statement is being used, from the same table. The global variable dateCHOP, is used to split the data into two separate tables at a specified date.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2050%20-%20SAS%20Code%20create%20FIN_TRAIN%20and%20FIN_TEST.png)

## Planetary Models

Now that we have the training and test data sets, it&#39;s time to actually create and use a prediction model. In this case, we know that we have the potential to have multiple columns with some possibility of correlation. This means that some sort of multivariate model might work well. SAS has taken great care to make it so that the prediction models they have available, are easily able to handle most situations that their users might come across. The first model type that caught my eye, was the adaptive regression function. The function &quot;ADAPTIVEREG&quot; creates non-parametric splines within your data to create a prediction model, which is based off of two people&#39;s published works. Patricia Smith of NASA implemented the concept of using knot theory, from physics, to create a weighted value to every &quot;cross over point&quot; of splines that could be drawn to fit through many fluctuations of plotted values. Jerome Friedman of Stanford University, took Smith&#39;s work, and pushed it to a new level, going on to make a trademarked function entitled Multivariate Adaptive Regression Splines, or MARS for short. SAS implemented their own version of the function, and, due to the trademark, named the function &quot;ADAPTIVEREG.&quot; In Python it is available via the &quot;py-earth&quot; package in the &quot;sci-kitlearn&quot; library, and in R it is known as the &quot;mars&quot; function from the &quot;mda&quot; library.

The MARS function, essentially, creates splines that can take the place of a linear, cubic, logarithmic, or whatever polynomial line&#39;s shape, that you can imagine. The processes taking place to create the line for a MARS function can, sometimes, be less computationally heavy, when compared to other methods. Knot theory mathematics is applied to all of the data points, and assists by searching for a location where an angle seems to appear in the data. The function to detect these sudden changes in direction of the flow of the data points, is referred to as a hinge function. A hinge function simply measures where a certain maximum value, in a given spline&#39;s direction, is occurring. For example, if a sudden direction change occurs, the max x or y value could be pushed into a simple line function (y=mx+b), so as to determine where no more points lie, beyond a tested max value. In order to make light of where the hinge functions &quot;settle,&quot; a specific plot can be produced, often referred to as a hockey stick chart. Once these hinges have been created, iterations take place which seek to determine where a spline can best be drawn between each of the hinges. Below are a few images of references that were used, to gain a greater understanding of the MARS function, specifically in SAS.

**NOTE** : These images have nothing to do with the COVID-19 project that this text is focused on. They are merely for demonstrating how the ADAPTIVEREG function in SAS works.

First, we can see a hockey stick report. Notice that the y value is 0, and that wherever the hinge function&#39;s values stop being 0, is dictated as a hinge location (-4, -2, and 0, for the data being used in the below image).

[http://www.wiilsu.org/EQzxioeazdFYT/SUSNov2013/Proceedings/Slides/Kuhfeld%20-%20Introducing%20Two%20New%20Advanced%20Regression%20Procedures%20-%20PROC%20ADAPTIVEREG%20and%20PROC%20QUANTSELECT.pdf](http://www.wiilsu.org/EQzxioeazdFYT/SUSNov2013/Proceedings/Slides/Kuhfeld%20-%20Introducing%20Two%20New%20Advanced%20Regression%20Procedures%20-%20PROC%20ADAPTIVEREG%20and%20PROC%20QUANTSELECT.pdf)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2051%20-%20SAS%20Documentation%20Hockey%20Stick%20Functions.png)

Next, you can see the hinge locations, plotted on a chart, where the resulting prediction model&#39;s splines have been plotted. Notice the locations where the splines shift directions, is at the location of each hinge; and that the resulting function for y, at the top, wields all of the hinge function&#39;s added up to make the final prediction model&#39;s function. In SAS, the hinge functions are referred to as Basis functions, thus the use of the β in the prediction model&#39;s function.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2052%20-%20SAS%20Documentation%20Splines%20Piecewise%20Linear%20Example.png)

The topic of MARS is a grandiose one, and well beyond the scope of this paper. It was a major learning point for me, personally, during this project. For now though, let&#39;s move on and take a look at the code required to make this model in SAS.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2053%20-%20SAS%20Code%20adaptivereg.png)

Above, you can see that a PROC call is made, then we feed in our parameters for the function. &quot;PLOTS=ALL,&quot; tells the function to create our plots with all of the available columns from the table we&#39;re passing. The &quot;SEED=10,&quot; is for reproducibility later, if desired, and serves exactly the same purpose of a seed function in R or Python. Below that, we can see the command &quot;MODEL&quot; is used, creating a &quot;formula&quot; of sorts, created with the columns we wish to attempt to use in our predictive model. You might have noticed that we&#39;re only passing 3 columns, of the 6 or 7 that we had available to us for using. The reasons for this were varied, but, essentially, the values that are not displayed kept having odd effects on the results of the prediction model. Specifically, the columns in question would sometimes have negative values, causing extreme swings of accuracy, to upwards of 50% losses. These columns could become negative, because of their natural tendency to decay. In example, the &quot;ACTIVE&quot; column would become a lower value, due to JHU wanting to keep an active cases value that had results of cases when possible. It would have been possible to not collect these values as a negative number, but would have been time consuming, and maybe not provided much value, so I opted to simply remove them. Instead, I kept the values which were always positive and seemed to be more of a cumulative total value, rather than an adjusting one.

## No Model Objects

In R and Python, when we create a model object, we can reference it, like it was an actual object. In SAS, the model is created, then dropped, when the ADAPTIVEREG function&#39;s ending RUN line is reached. This means that we must feed the created model the training and test data, then tell it how to give us results from the model. To do this, we can use the &quot;OUTPUT OUT&quot; functions to create a new table with the results from our model, on the training data set. Use the &quot;SCORE&quot; function, to create a table that predicts on a test data set. For the &quot;SCORE&quot; method, we are feeding the test data set into the &quot;DATA&quot; Parameter, and telling the function to output a new table with the results, using the &quot;OUT&quot; parameter. The parameters &quot;PRED&quot; and &quot;RESID&quot; are dictating that two columns should be present on the output table, for prediction and residual, respectively.

Once we have run the below model, a slew of tables is displayed in the Results tab of the Program run in SAS. Below, we&#39;ll take a look at some of the more interesting results produced.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2054%20-%20SAS%20adaptivereg%20Fit%20Statistics.png)![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2055%20-%20SAS%20adaptivereg%20Variable%20Importance.png)

The tables &quot;Fit Statistics&quot; and &quot;Variable Importance&quot; wound up being the most useful. The &quot;Fit Statistics&quot; table gives you the final selected model&#39;s stats, on the overall fit. As you can see we have a pretty good R-Square value of .9438. The &quot;Variable Importance&quot; table is useful for identifying which columns wound up carrying the most weight, in effectively predicting values. The three columns that I removed earlier, wound up wielding extremely low &quot;Importance&quot; values on this table, initially. This provided me with a desire to research why they were so low, leading to their removal. An Importance value that is closer to 100 is indicative of a stronger correlation within the prediction model.

Below, we&#39;ll take a look at the returned model selection results from our ADAPTIVEREG function&#39;s results.
![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2056%20-%20SAS%20adaptivereg%20Model%20Selection.png)

The above charts, show the progression in model selection changes that occurred. You can see that the 10-Basis14 model was the final selection, because it wielded the best overall output for a combination of Sum of Squared Error, and Cross Validation Error. Additionally SAS hands us a few more graphs which diagnose the fit of the model, in order for us to understand its accuracy more. For the top left graph, we can see that the spread of residuals is relatively minimal for all plotted points. A skew of up to 10k within a model predicting between a low of 200k and 300k is pretty good, all things considered. We also take a look at a bar chart that shows the residuals tending to be skewed on guessing higher at times, more so than lower. Though the bar chart fits into the desired bell curve for residuals, mostly. On the bottom middle chart, we get a good look at the predicted vs actual values, plotted against one another. Everything is mostly close to the line, but you can see that a larger portion of the points are under the line. This is also a demonstration of the skewness in tendency to favor higher guesses, rather than lower.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2057%20-%20SAS%20adaptivereg%20Model%20Fit%20Diagnostics.png)

So, the resulting output of data is easily able to be viewed in SAS, since it outputs a table for the model&#39;s predictions on the training and test data sets. The testing data sets will provide us with much more desired information at this point, so let&#39;s look at that first.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2058%20-%20SAS%20adaptivereg%20ARAG_TEST_RESULTS.png)

So, as you can see, the resulting predictions are not terribly far off their marks. Most are under 10k. However, this only wound up giving prediction numbers, when we should also have a percentage of accuracy at each row. To do this, we need to make some new columns on the table, which indicate a column to be used to create the Mean Average Error (MAE), and another column for 1-MAE to show accuracy. Below, you can see the SAS code required to create these two new columns, so that we can review the model&#39;s accuracy. Notice that we are simply creating a new table to hold the results in, and are adding the two new columns that calculate this information. The ALTER TABLE function is dropping out the values from the resulting table, for the columns that we did not use for our prediction model. After the PROC SQL is run, we are then using the PROC MEANS function call to obtain a table which gives us our mean averages for entire columns, to obtain the final accuracy, and error rate, of our model.

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2059%20-%20SAS%20Code%20multi%20ARAG_TEST_RESULTS.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2060%20-%20SAS%20table%20ARAG_TEST_RESULTS%20updated.png)

![alt text](https://github.com/Cyclonis7734/SAS_VBA_COVID-19_Pharmacy/blob/master/Final%20Paper%20pics/Pic%2061%20-%20SAS%20proc%20means%20ARAG_TEST_RESULTS%20updated.png)

As you can see, we&#39;re hitting an accuracy above 95%, using this model. This is a favorable ending to a long process of exploring the COVID-19 data for correlations with Cigna/ESI pharmacy data. The resulting model will require further testing, and adjustments, but is mostly finished at this point. Management is happy!

# Discussion Topics

Throughout the process of creating this project, I ran into many situations where a hefty amount of uncertainty was circulating, as far as goes what to do; and came later than was desirable. More specifically, towards the idea of me being able to make changes that would improve confidence in the overall project&#39;s output. The accuracy is definitely able to be seen as a positive highlight of the final product, but it comes with many stipulations at this point. For starters, we do not have any build up data, which can show us what the effects of the early days of COVID-19 were, on our claim counts. I realized this after the first week or two, but needed to get the necessary data right away and get started. It also did not help, that I had not fully understood where the data on JHU&#39;s Github repository was being stored, and why.

Essentially, the world wide data would have been better to use, as it actually contained USA data as well. However, the USA specific data set is what I went with. The reason that JHU separated these out, was because the number of columns had been more fully established and continually cleaned as of 4/12/2020. The world data sets wielded fewer data points, but at a far earlier date of capturing (As early as January 2020). The main columns I wound up using were present in these early files, and I will probably be making an effort to go back and capture them soon. The reason I did not pursue this, once discovered, was because I would have needed to rewrite most of the VBA code to handle an entirely different data file. While this would not be terribly difficult to do, it would still have been a major time-suck from the project. So, I opted to stay the course for now.

Another issue I will be addressing, after this project is over, are concerns about the inclusion of weekends and holidays as well as more data from Cigna/ESI. It is important that a prediction model is able to account for weekend/holiday dates as a simple flag in some manner. It is important that we should expect a prediction model to be trained to simply use the weekday number (The WDAY column in SAS), in tandem with month name maybe, in order to guesstimate the expected claim counts for any month. With that, we could probably also use an estimated sales, or new accounts increase/decrease value, that would allow for greater accuracy as well. The new accounts or new client counts would be necessary to include, because of Cigna/ESI&#39;s natural tendency to obtain more business over time. The slight uptick in claim counts over each year could probably be attributed to this factor, and would assist with accuracy for any point in time.

Additionally, once we had a model that could accurately predict claim counts for any given month, without COVID-19 data, we could then recreate a similar model with COVID-19 data. It would be very interesting to see both MARS prediction models plotted against each other. This would give a clear picture as to the effect of COVID-19&#39;s impact on claim counts, far more distinctly than anything else we&#39;ve seen at this point.

It is important to remember, that this model has many restrictions or factors that it relies on, which make it very specific in its use. Below are some of these factors:

1. Only using a subset of Pharmacy data (Cigna/ESI claims) from the USA, which may have unintended biases and not be a fair representation of the US population data.
2. Data for COVID-19 may contain inaccurate information due to the current chaos. We&#39;re using what is considered the &quot;gold standard&quot; at this point though, from JHU.
3. Claim counts are constantly shifting, due to pending statuses, approvals for paying, and denials happening at later dates.
4. Claim counts are for claims that have been paid only. It may be more helpful, or accurate, to include a claims count total that is based off of date, regardless of outcome of the claims.
5. Data is only representative of, what might be considered, impact during &quot;high&quot; case counts of COVID-19, at this point in time. The reasons for this being a factor at all, were addressed in the above paragraphs, relating to date of data capturing.

# Conclusion

With a 95% accuracy on our claim counts from the prediction model, in tandem with two highly correlated data points from COVID-19 data, it is fair to say that this project has been interesting, at the very least. We were able to reject the null hypothesis, but I still don&#39;t believe we can confidently say that it is safe to rely on the model. Given the restrictions and qualifiers for this model being slightly heavy on count and impact, only Cigna/ESI might find it to be useful right now, as a &quot;first glance&quot; on impact. That said, if I can address some of the topics from the above few paragraphs on restrictions/factors, we should be able to help ease this hesitation on reliability. I believe this is entirely able to be done, and I will probably be making it a point to tackle these issues, after this project has been completed for my Practicum.

In reflecting on the work done here, I believe that SAS was a decent choice of software to use in working on this project. If I did not have Cigna/ESI data to use, I would probably have tried to find some other sort of data on something that is being affected by COVID-19, which has been steadily correlated with the virus becoming more prevalent over time. While I got to learn about SAS syntax during this project, I also learned about the complexity of the MARS function. The MARS function will probably stand out as the most intriguing part of this project for me, personally. However, if I were to have used Python or R, I may have simply tried to use MARS with those languages as well. I believe that SAS, only having a few predictive models to choose from, wound up making it more of an interesting path of knowledge to walk down, though.

I would like to take a second and acknowledge something that I realized shortly into beginning this project as well. And that is, that the full impact of COVID-19 on our claims count will probably not be understood for quite some time. Nor will its true effects on any other data, or other ideas in general, be fully understood until much time has passed. Though we don&#39;t have as much data on previous pandemics, the data that we do have is still gaining realizations of impact, on all sorts of topics, even to this day, 100 years later (Spanish Flu for example ~1918). I&#39;m certain COVID-19 will go down in history in a similar manner, and with all of the data with today&#39;s technology, we should be able to help future proof our society even more!
