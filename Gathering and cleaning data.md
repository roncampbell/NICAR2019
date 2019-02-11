# Gathering and cleaning data

This is the third of a three-part introduction to <code>R</code>. We'll import California demographic data from the web that requires some cleaning. We'll also analyze airport flight delays that will make sense only when it is merged with other data. With a little practice, these skills will become second nature. But first let's load the R packages we'll need. The ace IRE/NICAR staff has already installed them for the conference.

(Hint: If you're not at the conference, remember that you must install packages before you can use them. To do so, make sure you have a working web connection and then type at the console prompt this command: <code>install.packages("xxx")</code> where xxx is the name of the package.)

> library(tidyverse)

> library(xml2)

The xml2 package is a "non-core" part of the <code>tidyverse</code>. It's one of the dozen or so elements of the tidyverse that must be individually activated. The core elements like <code>ggplot</code>, <code>dplyr</code>, <code>tidyr</code> and <code>stringr</code> are activated with the single command <code>library(tidyverse)</code>.

