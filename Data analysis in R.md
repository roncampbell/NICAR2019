# Data analysis and plotting in R

Thousands of packages make R the Swiss Army knife of software with a tool for almost every job. The NICAR staff has installed four packages for participants at the conference: tidverse, mosaic, janitor and descr. For those reading this tip sheet outside the conference, here's a reminder of how to install a package, using the tidyverse as an example:

install.packages("tidyverse")

Notice the plural in "packages" and the quote marks around the name of the package to be installed. They're important. If you forget them, R will ding you with an error. But installing packages is not enough. Packages use memory and can slow performance (bad); so R keeps them turned off unless you specifically activate them with the command library(xxx) where xxx -- no quote marks! -- is the name of a package. Let's get started by activating some previously installed packages.

library(tidyverse)

library(mosaic)

library(janitor)

library(descr)

And now for a curveball: The tidyverse is a package of packages -- the "core tidyverse" of widely used packages, including ggplot (charts), dplyr and tidyr (data manipulation), readr (simple data import), stringr (string manipulation) -- plus a dozen or so other packages. Journalists love some of those non-core tidyverse packages, particular readxl (importing Excel) and lubridate (manipulating dates). We're going to need readxl, so let's get it now.

library(readxl)

As you may have heard, California has two seasons, fire and flood. Let us now ponder fire. CalFire, the state's wildlands fire department, keeps track of wildfires going back decades on an Excel spreadsheet. We'll import, analyze and visualize California's fiery past.

> Wildfires <- read_excel("CA_wildfires.xlsx")

Now we'll get a quick idea of the Wildfires dataframe's structure with the str() function.

> str(Wildfires)
Classes ‘tbl_df’, ‘tbl’ and 'data.frame':	11977 obs. of  14 variables:
 $ YEAR      : num  1878 1895 1896 1898 1898 ...
 $ STATE     : chr  "California" "California" "California" "California" ...
 $ AGENCY    : chr  "Contract County" "Contract County" "Contract County" "Contract County" ...

Whoa! There are nearly 12,000 fires here ("11977 obs"). Let's make a chart and see how many of those fires are big. We'll do that using ggplot, the ingenious charting tool built into the tidyverse. We'll dive right in, and then I'll explain how it works:

> ggplot(Wildfires, aes(x=YEAR, y=GIS_ACRES)) + geom_point()

Every ggplot command begins the same way, by invoking the command ggplot, followed by parentheses and, inside the parentheses the data and the exact parts of the data we're going to map; these are known as aesthetics and abbreviated "aes". For this chart, the x or horizontal axis is the column YEAR (all caps because that's the way it is in the dataframe) and the y or vertical axis is GIS_ACRES. Next we specify how we want to display the data: a geom_point, aka a scatterplot, which places dots on an X-Y matrix.

Now you might notice something odd about the Y axis; the numbers are in scientific (exponential) notation. Fortunately there's an easy way to deal with that in R.

> options(scipen=999)

After you've run the options command, re-run the ggplot command, and you'll get a chart with real numbers in the Y-axis. But we can still do lots more. The great thing about ggplot is that it works in layers. We can add items to it.

> <code>ggplot(Wildfires, aes(x=YEAR, y=GIS_ACRES)) + geom_point() +
labs(title="California wildfire history",
 caption="Source: CalFire") +
 xlab("Year") + ylab("Acreage")</code>

Notice that when we add a new line (unless it's a continuation of something inside parentheses), we end the preceding line with a "+" sign; this is important because ggplot is fussy about grammar. 

We've added a title and caption while changing the labels for the x and y axes. But we could really dress up the inside of the chart too. Those black dots look rather dreary, and so does the standard gray background. If you like color, R has got you covered.

> colors()

Yes, R features 657 colors. So let's make two other changes to our wildfire plot:

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

Now let's come to Orange County and dig into one of my favorite things, Census data. We'll use a great part of the tidyverse, readr, to open a comma-separated variable (csv) file on your computers.

> OC_Residents <- read_csv("ACS_17_5YR_B05001 - tracts.csv")

Now type View(OC_Residents) and you'll see we have a problem - actually a couple of problems. First, the column headers appear in the first row; the actual headers are meaningless. Second and more important, most of the values appear to be strings rather than integers. We'll fix all of those right now. First step is to find out the dimensions of the file:

> dim(OC_Residents)

[1] 584  15

This means there are 584 rows and 15 columns. As you'll see, we don't need some of the columns. But we need to change the types of those we do need, and for convenience we'll want to rename them too. To do both we'll use something called column indexing, using the index number of a column in brackets from [1] to [15]. To alter a name we'll use the function colnames(). To eliminate a column entirely we'll use the powerful word NULL; be very, very careful with that one because there's no going back.

Here are the first few name changes:

> <code>colnames(OC_Residents)[2] <- "ID"
colnames(OC_Residents)[3] <- "Geography"
colnames(OC_Residents)[4] <- "TotalPop"</code>                                     

I'll make similar changes to the columns for "Native", "US_PR" (Puerto Rico), "US_BornAbroad", "Naturalized" and "Noncitizen". These are columns 6, 8, 10, 12 and 14; make sure you have the right column indexes when you rename these columns. Then it's time to eliminate columns. I usually start from the right side and move to left. It's always a good idea to check the column name first, for example  by typing "colnames(OC_Residents)[15]" in the console and hiting enter; this will tell you for sure if you're about to zap the column you really intend to terminate with extreme prejudice. Remember, once you do the NULL trick, a column is gone.

> OC_Residents[15] <- NULL

Bye-bye.

There are still a couple of things left to do. First that top row is useless. Let's get rid of it.

> OC_Residents <- OC_Residents[-1,]

This command tells R to remove the first row but to do nothing about any of the columns. Whenever you see brackets in R with a comma, everything to the left of the comma applies to rows, everything to the right applies to columns.

Now let's fix the columns, which are formatted as strings. We need to make them numbers. I'll show you how to do it for TotalPop, and then you do it for the Native, Naturalized and Noncitizen columns.

> OC_Residents$TotalPop <- as.numeric(OC_Residents$TotalPop)

Orange County has a surfer-dude, white-girl reputation. In fact, it's majority minority and has a very large immigrant population. To identify OC's immigrant neighborhoods, we'll create new columns in the dataframe using a function called mutate().

> <code>OC_Residents <- OC_Residents %>% 
group_by(ID) %>% 
mutate(Immigrants = sum(Naturalized + Noncitizen),
 ImmigrantPer = (Immigrants / TotalPop) * 100)</code>

> favstats(OC_Residents$ImmigrantPer)

 min       Q1   median      Q3     max     mean       sd   n missing
 
   0  16.76027  26.59105  38.8278  63.0132  28.37704  13.31743  582       1
 
 Oooh, let's graph this. And this time, let's plot a curve.
 
 > <code>ggplot(OC_Residents, aes(x=TotalPop, y=Immigrants)) + geom_point() +
geom_smooth()
`geom_smooth()` using method = 'loess' and formula 'y ~ x'</code>

![]()

The blue line with the gray error shading shows the apparent relationship between Orange County's total and immigrant population by tract. In most tracts close to a quarter of all residents were born abroad, but the error grows wider as population grows.

