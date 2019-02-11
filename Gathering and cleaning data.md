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
