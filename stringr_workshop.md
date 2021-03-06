---
title: "stringr_workshop"
author: "Kai Foerster and Max Eckert"
date: "07/10/2021"
output:
  html_document:
    keep_md: true
---



### String Manipulation using stringR

![What working with strings can feel like](pics/why-cant-you-just-be-normal.gif){width=300px}

<br>
Strings! Numeric value's unruly, difficult, unpopular cousin? We think not! String values play not only an important role in data cleaning, but texts itself are increasingly treated as rich sources of data for research in political science and public policy (see here, for example https://journals.sagepub.com/doi/full/10.1177/20531680211022206, or here: https://onlinelibrary.wiley.com/doi/10.1111/padm.12656). So today, we want to dive deeper into how to manipulate strings using the stringR package. 
<br>

![StringR to the rescue](pics/Meme_stringR_girl.jpg){width=300px}
<br>

## A Brief Overview
Most of us already became somewhat familiar with the basic functionality of the stringR package when we used it for the previous assignments. Part of the tidyverse, the stringR package was published in 2019 and is currently running version 1.4.0. <b>All functions in stringr start with "str_" and the first argument within the bracket is always the vector of strings that you want to modify, e.g. str_replace(<i>argument1</i>, ...) which comes in handy when using the the pipe to write your code.</b>

## Table of Contents
In the following document, we want to walk you through the following key functions of the stringR package  and some applications:

* Subsetting Strings
* Mutating Strings
* String ordering and sorting
* Joining and Splitting
* Order Strings


First, let's load the necessary packages we'll need for this exercise:

```r
library(stringr)
library(xml2)
```

For the purpose of this workshops we will be working with a tried and trusted data source: newspaper headlines! Thanks to the folks at the Internet Archive (https://archive.org), we will look at newspaper headlines across different points in time. Below, we can see the link to the Guardian headlines from the first day of classes, September 1, 2021.


```r
guardian_url <- "https://web.archive.org/web/20210901040912/https://www.theguardian.com/international"
```

Let's extract the newspaper headlines from that day.


```r
guardian_headlines <- guardian_url %>% read_html() %>% 
rvest::html_nodes(xpath = '//a[contains(concat( " ", @class, " " ), concat( " ", "js-headline-text", " " ))]') %>% rvest::html_text()
guardian_headlines[c(1:5)]
```

```
## [1] "Biden calls for new era in US foreign policy in defensive speech"           
## [2] "Biden sets himself apart by placing Afghanistan blame at predecessors??? feet"
## [3] "Wheelchair basketball, road cycling, badminton begins and more ?????live!"     
## [4] "Father seeking $2m before stepping down as conservator, court filing claims"
## [5] "Up to half of world???s wild tree species could be at risk of extinction"
```
Nice! What if we wanted to access the headlines of the following day?
Let's take a moment and think about the logic behind the URL:
<br>
https://web.archive.org/web/20210901040912/https://www.theguardian.com/international
<br>
We have the Internet Archive ".../web" followed by a time stamp, and the URL of our news source. Let's split up the string to manipulate time stamp into year, month, date, and time.


```r
guardian_split <- guardian_url %>% str_split(pattern = "web/") 
url_archive <- guardian_split[[1]][1]
url_date <- guardian_split[[1]][2]
url_archive
```

```
## [1] "https://web.archive.org/"
```

```r
url_date
```

```
## [1] "20210901040912/https://www.theguardian.com/international"
```
We can see that the first four digits are the year, followed by two digits for the month, two for the day, and the remaining 6 for the time of day at which the information was collected (we can disregard the time of day for now). Using the str_sub command, specifying the start and end position in the string, we subset year, month, and day. (Negative indexing is also possible.)


```r
url_year <- url_date %>% str_sub(start = 1, end = 4) 
url_month <- url_date %>% str_sub(start = 5, end = 6) 
url_day <- url_date %>% str_sub(start = 7, end = 8) 

url_year
```

```
## [1] "2021"
```

```r
url_month
```

```
## [1] "09"
```

```r
url_day
```

```
## [1] "01"
```

If we wanted to replace the information about year, month, or day, we could also use the str_replace function and assign a new value to the positions in the URL.


```r
str_sub(url_date, start = 7, end = 8) <- '02'
url_date
```

```
## [1] "20210902040912/https://www.theguardian.com/international"
```

Now that we have changed the day of the URL to September 2, 2021 we are merging the two substrings again. Keep in mind that earlier, we separated the URL at 'web/', so we need to add this again to receive a working URL.


```r
new_url <- str_c(url_archive, url_date, sep = "web/")
new_url
```

```
## [1] "https://web.archive.org/web/20210902040912/https://www.theguardian.com/international"
```

With our new URL, we can now scrape the headlines for the next day's edition of the Guardian newspaper!


```r
guardian_headlines_new <- new_url %>% read_html() %>% 
rvest::html_nodes(xpath = '//a[contains(concat( " ", @class, " " ), concat( " ", "js-headline-text", " " ))]') %>% rvest::html_text()
guardian_headlines_new[c(1:5)]
```

```
## [1] "Texas law 'blatantly violates' constitutional rights, says Biden"             
## [2] "Sackler family set to pay $4.5bn to settle claims "                           
## [3] "US military chief could work with Taliban on IS counter-terror strikes"       
## [4] "Joe Rogan has the virus ??? and his treatment will make health experts feel ill"
## [5] "Site bans Covid misinformation forum after ???go dark??? protest"
```

