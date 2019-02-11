# Gathering and cleaning data in R

This is the third of a three-part introduction to <code>R</code>. We'll import California demographic data from the web that requires some cleaning. We'll also analyze airport flight delays that will make sense only when it is merged with other data. With a little practice, these skills will become second nature. But first let's load the R packages we'll need. The ace IRE/NICAR staff has already installed them for the conference.

(Hint: If you're not at the conference, remember that you must install packages before you can use them. To do so, make sure you have a working web connection and then type at the console prompt this command: <code>install.packages("xxx")</code> where xxx is the name of the package.)

> library(tidyverse)

> library(xml2)

The xml2 package is a "non-core" part of the <code>tidyverse</code>. It's one of the dozen or so elements of the tidyverse that must be individually activated. Core elements like <code>ggplot</code>, <code>dplyr</code>, <code>tidyr</code> and <code>stringr</code> are activated with the single command <code>library(tidyverse)</code>.

The California Department of Finance tracks demographic trends using census, birth and death records to project population trends. We'll import a file that covers all 58 counties from 1970 through 2050.

> <code>download_html("http://tinyurl.com/y82z52uw", "ca_popest.csv")</code>

This XML2 command does three things. It tells R to download some HTML; it directs the browser to a particular website; and it downloads the content to a local file, to which we give a name. 

> <code>CA_popest <- read_csv("ca_popest.csv")
Parsed with column specification:
cols(
  fips = col_double(),
  county = col_character(),
  year = col_double(),
  age = col_double(),
  pop_female = col_double(),
  pop_male = col_double(),
  pop_total = col_double()
)</code>
  
> View(CA_popest) 

The dataframe contains 474,125 rows and has 7 columns. If you're familiar with FIPS codes, you'll immediately notice a problem with the (lower-case) fips column in CA_popest. The FIPS code for California is "06", but it appears here without the leading zero. This is a potential problem if we want to merge the data or map it. 

Fortunately it's easy to solve with the core tidyverse package stringr, which manipulates strings. The current fips field is 4 characters wide and lacks a leading zero. The field should be 5 characters wide (2 characters to identify the state, 3 to identify the county). Since the state is California in all cases, the first two characters should be "06"; we know from sorting the field in ascending and descending order that every row begins with a "6". All we need to do is widen the field from four characters to five and place an "0" in the first position, before the "6". 

> CA_popest$FIPS <- str_pad(CA_popest$fips, 5, "left", pad = "0")

This command creates a new field, FIPS (all caps), based on the existing field fips (lower case); it's 5 characters wide, padded on the left with the character "0". If you use the command View(CA_popest), you'll see the new field at the far right of the dataframe. Now all we have to do is move it to the left, where it will be more convenient. We can do that with column indexing.

> <code>CA_popest <- CA_popest[,c(8,1,2:7)]</code>

A reminder: Whenever you see numbers in R inside square brackets, separated by a comma, they refer to rows and columns; rows are to the left of the comma, columns to the right. In the code above, we move column 8 in front of column 1 while doing nothing to the rows. We use the "c()" to indicate we're combining or concatenating several objects.

We have a wealth of information here. If you're say, a school district administrator in Calaveras County, you can use this data to see how many 5-year-old children to expect in your kindergarten classes in 2025. But for most people this is too much of a good thing. We need to summmarize. We also want to be able to document our work. We can do both with a script.

To create a script in R Studio, click on the green "+" button at the upper left corner of the screen and then click on the words "R Script". I usually begin a script by listing the packages needed for that script; this way I can run the script independently of any other work I am doing. To stay organized, I usually keep a project in a folder or a set of folders on my computer; all the R scripts go in a subdirectory called, naturally, Scripts. 

Here's the code for summarizing population change by county by decade. 

library(tidyverse)              # make sure tidyverse is on
CA_popsum <- CA_popest %>%      # create new dataframe
  group_by(county, year) %>%    # grouping variables
  filter(year == 1970 | year == 1980 | year == 1990 | year == 2000 | 
           year == 2010 | year == 2020 | year == 2030 | year == 2040 | 
           year == 2050) %>%    # once-a-decade data
  summarise(
    Population = sum(pop_total, na.rm=T)) # get population for each year
    
I commented each line of this script using a hash mark (#). You can comment scripts as much (or as little) as you want. Just remember -- you may be going back to your script months later and wondering "What was I thinking when I wrote this?" Your comments, preceded by a hash (#) mark will be valuable clues. I use old scripts for two reasons. The first is to recycle old data for new stories. The second and by far the more important is to tweak the code for some new and unexpected use. Think of the data that the code produces as bricks that you can tear down and reuse a limited number of times and code as tools that you can employ over and over and over again in an endless combination of ways. The more you use the code, the more ways you will think of using it.

This script produces a 522-line dataframe. The data would make more sense if each year were in its own column. We can do this with the tidyverse package tidyr. This package has two main functions, gather() and spread(). The former takes wide tables and makes them long; the latter takes long tables and makes them wide. They both work by using key:value pairs. 

Here's an example: On line 1 of CA_popsum the key is 1970 and the value is 1072985; these two values are associated with each other (and with ALAMEDA). We'll make a new dataframe built of key:value frames like this.

library(tidyverse)
ca_popsum1 <- ca_popsum %>% 
  spread(key=year, value=Population)
  
