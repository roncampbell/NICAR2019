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

> Wildfires <- read_excel("California Wildfires_100.xlsx")


