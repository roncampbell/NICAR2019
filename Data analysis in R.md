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

Whoa! There are nearly 12,000 fires here ("11795 obs"). Let's make a chart and see how many of them are big. We'll do that using ggplot, the ingenious charting tool built into the tidyverse. We'll dive right in, and then I'll explain how it works:

> ggplot(Wildfires, aes(x=YEAR, y=GIS_ACRES)) + geom_point()

Every ggplot command begins the same way, by invoking the command ggplot, followed by parentheses and, inside the parentheses the data and the exact parts of the data we're going to map; these are known as aesthetics and abbreviated "aes". For this chart, the x or horizontal axis is the column YEAR (all caps because that's the way it is in the dataframe) and the y or vertical axis is GIS_ACRES. Next we specify how we want to display the data: a geom_point, aka a scatterplot, which places dots on an X-Y matrix.

Now you might notice something odd about the Y axis; the numbers are in scientific (exponential) notation. Fortunately there's an easy way to deal with that in R.

> options(scipen=999)

After you've run the options command, re-run the ggplot command, and you'll get a chart with real numbers in the Y-axis. But we can still do lots more. The great thing about ggplot is that it works in layers. We can add items to it.

> ggplot(Wildfires, aes(x=YEAR, y=GIS_ACRES)) + geom_point() +
+     labs(title="California wildfire history",
+                caption="Source: CalFire") +
+     xlab("Year") + ylab("Acreage")

We've added a title and caption while changing the labels for the x and y axes. But we could really dress up the inside of the chart too. Those black dots look rather dreary, and so does the standard gray background. If you like color, R has got you covered.

> colors()

Yes, there are 657 colors. So let's make two other changes to our wildfire plot:

> ggplot(Wildfires, aes(x=YEAR, y=GIS_ACRES)) + 
+     geom_point(col="firebrick1") +
+     labs(title="California wildfire history",
+                caption="Source: CalFire") +
+     xlab("Year") + ylab("Acreage") +
+     theme_minimal()

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

> RecentFires <- Wildfires %>% 
+     filter(YEAR >= 2000)

We'll use another tool to summarize the data by acreage, quantiles. Pay close attention to this command -- it's tricky.

> quantile(RecentFires$GIS_ACRES, c(0.05, 0.1, 0.25, 0.5, 0.75, 0.9, 0.95, 0.99), na.rm=T)
        5%        10%        25%        50%        75%        90%        95%        99% 
  114.4208   133.8902   206.0898   501.5787  1949.1175  9343.3310 22146.7040 80108.3956
  
The worst 1% of wildfires burned at least 80,000 acres. We know that some fires in the past 20 years burned five times that much.
  
Now let's look at the causes of some of these fires using the handy tabyl function in the janitor package. (Janitor is also useful for cleaning up data - thus its name.)

> tabyl(RecentFires, CAUSE)
                         CAUSE   n      percent
                        <Null>   8 0.0035288928
                 1 - Lightning 705 0.3109836789
                  10 - Vehicle 127 0.0560211734
                11 - Powerline  91 0.0401411557
 



