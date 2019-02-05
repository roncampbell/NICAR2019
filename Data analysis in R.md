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
Classes ‘tbl_df’, ‘tbl’ and 'data.frame':	11795 obs. of  14 variables:
 $ YEAR      : num  2016 2016 2016 2016 2016 ...
 $ STATE     : chr  "California" "California" "California" "California" ...
 $ AGENCY    : chr  "National Park Service" "California Department of Forestry

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

