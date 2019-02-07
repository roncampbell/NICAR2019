# Data analysis and plotting in R

Thousands of packages make R the Swiss Army knife of software with a tool for almost every job. The NICAR staff has installed four packages for participants at the conference: tidverse, mosaic, janitor and descr. For those reading this tip sheet outside the conference, here's a reminder of how to install a package, using the tidyverse as an example:

> install.packages("tidyverse")

Notice the plural in "packages" and the quote marks around the name of the package to be installed. They're important. If you forget them, R will ding you with an error. But installing packages is not enough. Packages use memory and can slow performance; so R keeps them turned off unless you specifically activate them with the command library(xxx) where xxx -- no quote marks! -- is the name of a package. Let's get started by activating some previously installed packages.

> library(tidyverse)
 
> library(mosaic)

> library(janitor)

> library(descr)

And now for a curveball: The tidyverse is a package of packages -- the "core tidyverse" of widely used packages, including ggplot (charts), dplyr and tidyr (data manipulation), readr (simple data import), stringr (string manipulation) -- plus a dozen or so other packages that are NOT loaded automatically. Journalists love some of those non-core tidyverse packages, particular readxl (importing Excel) and lubridate (manipulating dates). We're going to need readxl later in the class, so let's get it now.

> library(readxl)

We're going to spend some time exploring Orange County with Census data. We'll use a key part of the tidyverse, readr, to open a comma-separated variable (csv) file on your computers. Since the Census Bureau habitually provides two rows of column headers, we'll tell R to skip the first line.

> OC_Residents <- read_csv("ACS_17_5YR_B05001 - tracts.csv", skip = 1)

Now type View(OC_Residents) and you'll see we have a problem - actually a couple of problems. First, the column headers are very strange - huge actually. Second and more important, most of the values appear to be strings rather than integers. We'll fix these problems right now. Sometimes we can fix column names with the janitor package and its self-explanatory clean_names function.

> OC_Residents <- janitor::clean_names(OC_Residents)

Unfortunately janitor leaves us with awkwardly long names. So let's clean them ourselves. First step is to learn how many columns there are.

> dim(OC_Residents)

[1] 584  15

This means there are 584 rows and 15 columns. We need to rename some of those columns and change a few of their types as well. To do both we'll use something called column indexing, using the index number of a column in brackets from [1] to [15]. To alter a name we'll use the function colnames(). To eliminate a column entirely we'll use the powerful word NULL; be very, very careful before you NULL a column because there's no going back.

Here are the first few name changes:

> <code>colnames(OC_Residents)[2] <- "ID"
colnames(OC_Residents)[3] <- "Geography"
colnames(OC_Residents)[4] <- "TotalPop"</code>                                     

We'll also rename the columns for "Native", "US_PR" (Puerto Rico), "US_BornAbroad", "Naturalized" and "Noncitizen". These are columns 6, 8, 10, 12 and 14; make sure you have the right column indexes when you rename these columns. We can also eliminate some columns if we wish. It's always a good idea to check the column name first, for example  by typing "colnames(OC_Residents)[15]" in the console and hiting enter; this will tell you for sure if you're about to zap the column you really intend to terminate with extreme prejudice. Remember, once you do the NULL trick, a column is gone.

> OC_Residents[15] <- NULL

Remember, some columns in the dataframe that should be numbers are currently characters. I'll show you how to change TotalPop from character to numeric, and then you do it for the Native, Naturalized and Noncitizen columns.

> OC_Residents$TotalPop <- as.numeric(OC_Residents$TotalPop)

Orange County has a surfer-dude, white-girl reputation. In fact, it's majority minority and has a very large immigrant population. To identify OC's immigrant neighborhoods, we'll create new columns in the dataframe using a function called mutate().

> <code>OC_Residents <- OC_Residents %>% 
group_by(ID) %>% 
mutate(Immigrants = sum(Naturalized + Noncitizen),
 ImmigrantPer = (Immigrants / TotalPop) * 100)</code>
 
There are several ways to crunch this data in R. A couple of my favorites are the Base R summary() function and the mosaic package's aptly named favstats tool. Let's try them both.
 
 > summary(OC_Residents$ImmigrantPer)
 
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
   
   0.00    16.76    26.59    28.38    38.83    63.01       1 

> favstats(OC_Residents$ImmigrantPer)

 min       Q1   median      Q3     max     mean       sd   n missing
 
   0  16.76027  26.59105  38.8278  63.0132  28.37704  13.31743  582       1
   
Summary() is good most of the time, but when you need precision and especially when you want to keep tabs on the standard deviation and missing values, favstats() lives up to its name. But nothing beats a good chart for capturing the meaning of a statistic. I've been dealing with stats for a couple of decades, and I still get more from a histogram than I do from a summary.
 
 > ggplot(OC_Residents, aes(ImmigrantPer)) + geom_histogram()
 
This is a basic histogram of the percentage of immigrants in all 583 Orange County census tracts. The structure of the command is essentially the same every time: the word ggplot, followed by parentheses, the name of the dataset, the word aes (short for aesthetic), and then in another set of parentheses, the variables to be charted; next the type of graphic. Everything after that is optional. You can add a lot of options.

First, let's change the color of the histogram and get rid of the dull gray background. Then let's add labels to the x- and y-axes. Add a title and a source too. Notice as we do that each line ends with a "+" sign, unless we're continuing material inside parentheses. Another thing: While lines and boundaries in ggplot have color, abbreviated "col", objects have "fill". 

And one last detail: You've got a lot of choices when it comes to color. How many choices? I'm glad you asked.

> colors()

Yeah, R has 657 colors. 

> <code>ggplot(OC_Residents, aes(ImmigrantPer)) +
geom_histogram(col="black", fill="lightskyblue2") +
labs(title="Immigrant population of Orange County census tracts",
caption="Source: U.S. Census Bureau") +
xlab("Immigrant percentage") + ylab("Census tracts") +
theme_minimal()</code>

![](https://github.com/roncampbell/NICAR2019/blob/images/Immig_Histogram.png?raw=true)

Another way to look at the data is to examine the relationship between the total population and the number of immigrants. We can do that with a scatterplot. This is just what it sounds like: a bunch of dots plotted against two variables on an X-Y axis. You can even add a line to show the trend of the data.
 
 > <code>ggplot(OC_Residents, aes(x=TotalPop, y=Immigrants)) + geom_point() +
geom_smooth()
`geom_smooth()` using method = 'loess' and formula 'y ~ x'</code>

![](https://github.com/roncampbell/NICAR2019/blob/images/OC_Pop_immigrants.png?raw=true)

The blue line with the gray error shading shows the apparent relationship between Orange County's total and immigrant population by tract. In most tracts close to a quarter of all residents were born abroad, but the error grows wider as population grows.

It's often useful to categorize data. We'll classify Orange County tracts by the percentage of immigrants each tract has. But before we do that, let's get a better handle on their immigrant share using R's quantile tool. The syntax is a bit tricky. We use the "c" operator, which stands for "concatenate" because we're throwing several numbers together. And we include the term "na.rm=T", short for "remove all NA values=TRUE", meaning "ignore every missing value in the data." 

> quantile(OC_Residents$ImmigrantPer, c(0.1, 0.25, 0.35, 0.5, 0.6, 0.75, 0.9, 0.95),na.rm=T)

     10%      25%      35%      50%      60%      75%      90%      95% 
     
12.08220  16.76027  21.08368  26.59105  31.25812  38.82780  47.13526  51.14030

We just split OC tracts into quantiles. In the lowest quarter of tracts, 16.76% of residents are immigrants. In the highest quarter of tracts, 38.83% of residents are immigrants. With this information we can now classify the data, using dplyr's mutate function. 

> <code>OC_Residents <- OC_Residents %>% 
mutate(Share = case_when(
ImmigrantPer <= 16.76027 ~ "Low",
ImmigrantPer > 16.76027 & ImmigrantPer <= 26.59105 ~ "MedLow",
ImmigrantPer > 26.59105 & ImmigrantPer <=38.82780 ~ "MedHigh",
ImmigrantPer > 38.82780 ~ "High"))</code>

We just created a new column in the dataframe called Share; each tract has one of four values, based on the percentage of immigrants.

Now let's look at income in OC census tracts. We'll import income data much the same way we brought in the citizenship data.

> OC_Income <- read_csv("ACS_17_5YR_B19013 - tracts.csv", skip= 1)

There are fewer columns this time, but the headers look even more obnoxious. Let's not bother with janitor::clean_names(). We'll change the column names ourselves. The name of Column 2 becomes "ID", Column 3 becomes "Geography", Column 4 becomes "MedianHHInc", Column 5 becomes "MOE". When you've renamed those columns, eliminate Column 1 because its name is too similar to Column 2.

If you've forgotten, here's how to rename columns:

> colnames(OC_Income)[2] <- "ID"

And I'm sure you all want to know just how much money is floating around Orange County. But first we have to change the MedianHHIncome column, currently formatted as a character, to numeric.

> OC_Income$MedianHHIncome <- as.numeric(OC_Income$MedianHHIncome)

> favstats(OC_Income$MedianHHIncome)

   min    Q1 median       Q3    max     mean       sd   n missing
   
 24211  61443   83214  106449.5  209095  86921.18  32116.53  579       4

Now let's do it up right in a graphic.

> <code>ggplot(OC_Income, aes(MedianHHIncome)) + 
geom_histogram(col='black', fill='green') +
labs(title="Median household income in Orange County census tracts") +
xlab("Median household income") + ylab("Count") +
theme_minimal()</code>

![](https://github.com/roncampbell/NICAR2019/blob/images/OC_income.png?raw=true)

So far we've analyzed Orange County immigration and household income by census tract. Is there a relation between the two? To find out, let's join the two datasets. If you've merged data in SQL, you'll find the process in R to be pretty familiar. We're joining two dataframes, OC_Residents and OC_Income, each with 583 rows, each with what looks like a common field called "ID". 

> OC_ResInc <- inner_join(OC_Residents, OC_Income, by="ID")

The resulting dataframe contains two duplicate columns -- Geography.x and Geography.y. We can solve that problem easily by eliminating the duplicate and renaming the other, first making sure we've got the right columns.

> <code>colnames(OC_ResInc)[19]
[1] "Geography.y"
> OC_ResInc[19] <- NULL
> colnames(OC_ResInc)[3]
[1] "Geography.x"
 > colnames(OC_ResInc)[3] <- "Geography"</code>

Let's first see if there's a correlation between median household income and the percentage of immigrants in OC tracts. When doing correlation - in fact with many statistical procedures - you must beware of NA values. So we'll throw in a bit of code to ward off the NA beast.

> <code>cor(OC_ResInc$ImmigrantPer, OC_ResInc$MedianHHIncome, use="pairwise.complete.obs")
[1] -0.5889496</code>

This suggests that the percentage of immigrants is associated with reduced median household income. Interesting, but we can do better. We categorized tracts by the percentage of immigrants so we could do a deeper analysis. So let's look specifically at median household income in each tract by its immigrant share category -- Low, Medium-Low, Medium-High and High. We'll use a graphic called a box plot to do that. And we'll use a "facet" to make it easy to compare the categories.

> <code>ggplot(OC_ResInc, aes(x="Income", y=MedianHHIncome)) +
geom_boxplot() +
facet_grid(Share ~ .)</code>

First a quick explanation of boxplots: The thick horizontal line in each box is the median; the lower and upper lines represent the 25th and 75th percentiles respectively and are known as the Interquartile Range, IQR for short. The vertical lines extending outward are known as whiskers and typically extend for 1-1/2 times the length of the IQR. Any points beyond the whisker are by definition outliers.

The boxplots clearly show that tracts with a high percentage of immigrants tend to have lower median household incomes. We can make the graphic better by removing the blank NA boxplot on the right. To do that, we'll filter the graphic to remove the three tracts with no immigrants. While we're at it, we'll also label the x- and y-axes.

> <code>ggplot(filter(OC_ResInc, Immigrants > 0), aes(x="Income", y=MedianHHIncome)) +
geom_boxplot() +
facet_grid(. ~ Share) +
xlab("Immigrant Percentage") + ylab("Median Household Income")</code>

![](https://github.com/roncampbell/NICAR2019/blob/images/ImmigInc_boxplot.png?raw=true)

As you may have heard, California has two seasons, fire and flood. Let us now ponder fire. CalFire, the state's wildlands fire department, keeps track of wildfires going back decades on an Excel spreadsheet. We'll import, analyze and visualize California's fiery past.

> Wildfires <- read_excel("CA_wildfires.xlsx")

Now we'll get a quick idea of the Wildfires dataframe's structure with the str() function.

> str(Wildfires)
Classes ‘tbl_df’, ‘tbl’ and 'data.frame':	11977 obs. of  14 variables:
 $ YEAR      : num  1878 1895 1896 1898 1898 ...
 $ STATE     : chr  "California" "California" "California" "California" ...
 $ AGENCY    : chr  "Contract County" "Contract County" "Contract County" "Contract County" ...

Whoa! There are nearly 12,000 fires here ("11977 obs"). Let's make a chart and see how many of those fires are big. We'll do that using ggplot, the ingenious charting tool built into the tidyverse. We'll dive right in, and then I'll explain how it works:

> <code>ggplot(Wildfires, aes(x=YEAR, y=GIS_ACRES)) + geom_point() +
labs(title="California wildfire history",
 caption="Source: CalFire") +
 xlab("Year") + ylab("Acreage")</code>

Now you might notice something odd about the Y axis; the numbers are in scientific (exponential) notation. Fortunately there's an easy way to deal with that in R.

> options(scipen=999)

Let's run it again - this time with color points and a blank background.

> <code>ggplot(Wildfires, aes(x=YEAR, y=GIS_ACRES)) + 
geom_point(col="firebrick1") +
labs(title="California wildfire history",
caption="Source: CalFire") +
xlab("Year") + ylab("Acreage") +
theme_minimal()</code>

![](https://github.com/roncampbell/NICAR2019/blob/images/Fire_scatter.png?raw=true)

The graph makes clear that there are thousands of relatively small fires and a handful of gigantic ones. Let's drill down and take a closer look using some tools to summarise the data. There are several ways to do that, and we're going to try two of them -- Base R's summary() and the mosaic package's aptly named favstats().

> summary(Wildfires$GIS_ACRES)

    Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
   100.0    236.5    512.9   2817.6   1465.7 501082.0 
   
> favstats(Wildfires$GIS_ACRES)

      min       Q1   median       Q3    max     mean       sd     n
 100.0275 236.4592 512.9036 1465.738 501082 2817.561 11611.32 11977
 missing
       0
       
The summary() function is good most of the time; if you want more precise values and really need the standard deviation and missing values, favstats() is useful. Both show that the typical wildfire is small, with a median size of around 512 acres and a mean (mathematical average) of 2,817 acres. But notice that very high standard deviation -- 11,611 acres. That's the tell, the sign that the mean is, uh, meaningless.

Let's see if there's a historical pattern. We have nearly data for nearly 12,000 fires for more than a century. Maybe we can see something if we break them down by decade instead of by year. To do that, we again will use dplyr's mutate tool. We'll also use something most of us non-math majors never encountered, the modulus, or remainder from division. Specifically we'll transform years into their decade-equivalents.

> <code>Wildfires <- Wildfires %>% 
mutate(
 Decade = YEAR - (YEAR %% 10))</code>
 
 Now we've got a new column called "Decade" at the 15th position on the far right end of the Wildfires dataframe. To keep things convenient, let's move it next to the YEAR column at the left. Here's how we'll do that. Again, if you're not sure exactly what you're doing, use colnames(dataframe)[] to make sure of the name of the column before you move it.
 
> Wildfires <- Wildfires[,c(1,15,2:14)]

Whenever you see something in square brackets in R with a comma, anything to the left of the comma refers to rows, and anything to the right refers to columns. In the command above, we concatenate -- there's that "c" again -- columns 1 and 15, followed by columns 2 through 14. This places the YEAR and Decade column next to each other followed by every other column in their former order.

Now let's see if there's a historic pattern to wildfires in California. We'll make a bar chart, using Decade as the x (categorical) axis and GIS_ACRES as the y-axis. Bar charts can contain one, two or more categorical variables. For that reason you can't just say geom_bar(). You have to be specific -- stat="identity" for one categorical variable or if you're stacking 2+ variables; position="dodged" if you want to place two categorical variables side-by-side.

> <code>ggplot(Wildfires, aes(x=Decade, y=GIS_ACRES)) + 
geom_bar(stat="identity", fill="orange2") +
theme_minimal() +
xlab("Decade") + ylab("Acres burned") +
labs(title="California wildfires",
caption="Source: CalFire")</code>

![]()




Let's narrow our focus to the most recent fires in the data, those that occurred from 2000 through 2017. First we'll filter the Wildfires dataframe and create a new dataframe with just the recent data. Then we'll analyze the new dataframe.

> <code>RecentFires <- Wildfires %>% 
 filter(YEAR >= 2000)</code>

We'll use another tool to summarize the data by acreage, quantiles. Pay close attention to this command -- it's tricky.

> quantile(RecentFires$GIS_ACRES, c(0.05, 0.1, 0.25, 0.5, 0.75, 0.9, 0.95, 0.99), na.rm=T)

        5%        10%        25%        50%        75%        90%        95%        99% 
        
  114.4208   133.8902   206.0898   501.5787  1949.1175  9343.3310 22146.7040 80108.3956
  
The worst 1% of wildfires burned at least 80,000 acres. We know that some fires in the past 20 years burned five times that much.

Let's graph the fires to get a better idea of their relative size. We'll use a graphic called a histogram, which breaks things up into "bins" based on their frequency. See if you follow the logic of the command.

> ggplot(RecentFires, aes(GIS_ACRES)) + geom_histogram(bins=50)
  
![](https://github.com/roncampbell/NICAR2019/blob/images/Wildfire_histogram.png?raw=true)

I divided the data into 50 groups or "bins". If I hadn't done that, ggplot would have automatically split the data into 30 bins. Even with the extra dividing, it's obvous that there are very, very few giant wildfires. The histogram reaffirms what the numbers have already told us: Most wildfires are (relatively) small.

But what causes the most fires? We can find out by crosstabbing the CAUSE and GIS_ACRES columns. One easy way to do that is with the aggregate() function in Base R. Since we're going to chart it, let's save the results to a variable.

> FireCauses <- aggregate(GIS_ACRES ~ CAUSE, RecentFires, sum)
> FireCauses
                           CAUSE    GIS_ACRES
1                         <Null>   46033.9913
2                  1 - Lightning 4529748.6658
3                   10 - Vehicle  355979.9253
4                 11 - Powerline  149804.5863
 
There are 19 listed causes, including the all-purpose "Null", meaning "nobody knows". Let's do a bar chart to put everything in perspective. Since bar charts can few different forms -- single, stacked or side-by-side -- we have to specify what we want. For a bar chart with one categorical variable and one continuous variable, use stat="identity". With two categorical variables, you have a choice -- either stack the bars or place them side by side; for the former, use different fills to indicate the variables; for the latter, use position="dodge".

> <code>ggplot(FireCauses, aes(x=CAUSE, y=GIS_ACRES)) + 
geom_bar(stat="identity")</code>

This gives a simple bar chart, and an ugly one with a run-on x-axis. We can do a lot better. The obvious cure is to tilt the x-axis on its side. There's some code online to help us do that. While we're at it, let's add some color to the bars, write a headline and change the axis labels. Again, we're going to write this chart in layers, adding pieces a bit at a time. One other note: In ggplot, "col" or color applies to lines like the outline of a bar; "fill" applies to the interior of an object.

><code>ggplot(FireCauses, aes(x=CAUSE, y=GIS_ACRES)) + 
geom_bar(stat="identity", col="black", fill="orange") +
theme(axis.text.x = element_text(angle=90,hjust=1,vjust=0.5)) +
labs(title="Causes of California wildfires, 2000-2017",
caption="Source: CalFire") +
xlab("Cause") + ylab("Acreage")</code>

![](https://github.com/roncampbell/NICAR2019/blob/images/Wildfire_causes.png?raw=true)

Who fights the fires? The answer is in the column AGENCY. Again, we can get a breakdown of firefighting by cause and agency with a simple crosstab. Or we can find out with a picture. Since we already know the x-axis with causes is hard to read, let's borrow the code we used last time. This time, however, we'll do something different. We'll "fill" the bars with the AGENCY variable to create a stacked bar graph. That will show which agencies did the most work fighting which types of fires.

> <code>ggplot(RecentFires, aes(x=CAUSE, y=GIS_ACRES, fill=AGENCY)) + 
geom_bar(stat="identity") +
theme(axis.text.x = element_text(angle=90,hjust=1,vjust=0.5))</code>

![](https://github.com/roncampbell/NICAR2019/blob/images/Wildfire_fighters.png?raw=true)


