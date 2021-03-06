---
title: "NOAA at a glance"
author: "Chau Pham"
date: "4/7/2022"
output: 
    html_document:
        keep_md : true
---



## Synopsis

This project is carried out as part of the course Reproducible Research run by John Hopkins University on Coursera. Some quick cleaning and wrangling are done on the dataset prior to any explorary analyses are taken. From the data, it appears that hurricanes and tornados bring significant damages to the well-being of people and the economic conditions of the US.

-------------------------------------------------------

## Data processing
The NOAA dataset can be downloaded [here][1]. More information is available on the [documentation][2] of the database and the [FAQ][3] page.

[1]: https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2
[2]: https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf
[3]: https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2FNCDC%20Storm%20Events-FAQ%20Page.pdf

We first download the bz2 file containing the dataset to our local computer and load into R. **Warning: the dataset is around <span style="color: red;">475 MB</span> after unzipping.**

```r
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2", "noaa.bz2") ##optional if the dataset has already been downloaded
library(readr)
noaa <- read_csv("noaa.bz2")
```

Having loaded the dataset, let's have a first look at what we are working with.

```r
dim(noaa)
```

```
## [1] 902297     37
```
We have a dataset with 37 variables and more than 900,000 observations. There are a lot of recorded events in the dataset and it is useful to know how many there are, and what types of events occur most frequently.

```r
library(dplyr)
noaa$EVTYPE %>% unique() %>% length()
```

```
## [1] 977
```

```r
noaa$EVTYPE %>% unique() %>% .[c(1:10, 700:710, 800:810)] 
```

```
##  [1] "TORNADO"                   "TSTM WIND"                 "HAIL"                     
##  [4] "FREEZING RAIN"             "SNOW"                      "ICE STORM/FLASH FLOOD"    
##  [7] "SNOW/ICE"                  "WINTER STORM"              "HURRICANE OPAL/HIGH WINDS"
## [10] "THUNDERSTORM WINDS"        "Summary of May 13"         "Summary of May 14"        
## [13] "Summary of May 22 am"      "Summary of May 22 pm"      "Heatburst"                
## [16] "Summary of May 26 am"      "Summary of May 26 pm"      "Metro Storm, May 26"      
## [19] "Summary of May 31 am"      "Summary of May 31 pm"      "Summary of June 3"        
## [22] "Wintry mix"                "Frost"                     "Frost/Freeze"             
## [25] "RAIN (HEAVY)"              "Record Warmth"             "Prolong Cold"             
## [28] "Cold and Frost"            "URBAN/SML STREAM FLDG"     "STRONG WIND GUST"         
## [31] "LATE FREEZE"               "BLOW-OUT TIDE"
```

```r
noaa_freq <- noaa %>% group_by(EVTYPE) %>% tally(sort = TRUE)
noaa_freq
```

```
## # A tibble: 977 x 2
##    EVTYPE                  n
##    <chr>               <int>
##  1 HAIL               288661
##  2 TSTM WIND          219944
##  3 THUNDERSTORM WIND   82563
##  4 TORNADO             60652
##  5 FLASH FLOOD         54278
##  6 FLOOD               25326
##  7 THUNDERSTORM WINDS  20843
##  8 HIGH WIND           20212
##  9 LIGHTNING           15755
## 10 HEAVY SNOW          15708
## # ... with 967 more rows
```
There are close to 1,000 types of events recorded. However, at a glance, it appears this gargantuan number signifies at least two problems namely, poor naming convention and typos. To address these two problems, we will need to aggregate different entries of similar types into one category. Another issue we need to address is the existence of "Summary" entries.

```r
library(forcats)
noaa$EVTYPE <- toupper(noaa$EVTYPE) 
noaa <- filter(noaa, !grepl("SUMMARY", EVTYPE))  ##Dropping summarising entries

## Categorization
noaa <- noaa %>% mutate(EVENT_CAT = case_when(
    grepl("THUNDE*RE*STORM *W+I*N*D*|^[^NO]*TSTM", EVTYPE) ~ "THUNDERSTORM WIND",
    grepl("HIGH WIND", EVTYPE) ~ "HIGH WIND",
    grepl("^(?=.*WIND)(?!.*THUNDERSTORM)(?!.*TSTM)(?!.*HIGH)", EVTYPE, perl = TRUE) ~ "OTHER WIND",
    grepl("HAIL", EVTYPE) ~ "HAIL",
    grepl("TORNADO", EVTYPE) ~ "TORNADO",
    grepl("FLOOD", EVTYPE) ~ "FLOOD", 
    grepl("SNOW", EVTYPE) ~ "SNOW",
    grepl("RAIN", EVTYPE) ~ "RAIN",
    grepl("LIGHTNING", EVTYPE) ~ "LIGHTNING",
    grepl("HEAT", EVTYPE) ~ "HEAT",
    grepl("COLD", EVTYPE) ~ "COLD", 
    grepl("FUNNEL", EVTYPE) ~ "FUNNEL",
    grepl("FOG", EVTYPE) ~ "FOG",
    grepl("HURRICANE|TYPHOON", EVTYPE) ~ "HURICCANE",
    grepl("BLIZZARD", EVTYPE) ~ "BLIZZARD", 
    grepl("RIP CURRENT", EVTYPE) ~ "RIP CURRENT",
    grepl("WATERSPOUT", EVTYPE) ~ "WATERSPOUT", 
    grepl("TSUNAMI", EVTYPE) ~ "TSUNAMI", 
    grepl("SURF", EVTYPE) ~ "SURF",
    grepl("AVALANCHE", EVTYPE) ~ "AVALANCHE", 
    grepl("DROUGHT", EVTYPE) ~ "DROUGHT",
    grepl("WILDFIRE", EVTYPE) ~ "WILDFIRE",
    grepl("SLEET", EVTYPE) ~ "SLEET",
    TRUE ~ "OTHER"
))

noaa %>% group_by(EVENT_CAT) %>% tally(sort = TRUE)
```

```
## # A tibble: 24 x 2
##    EVENT_CAT              n
##    <chr>              <int>
##  1 THUNDERSTORM WIND 330523
##  2 HAIL              289278
##  3 FLOOD              82711
##  4 TORNADO            60699
##  5 OTHER              38380
##  6 HIGH WIND          21953
##  7 SNOW               17677
##  8 LIGHTNING          15760
##  9 RAIN               12153
## 10 FUNNEL              6981
## # ... with 14 more rows
```
To deal with further categorization and date/time, further cleaning and processing are essential. It is, however, beyond the scope of this RMarkdown project and thus will be left as a future project.

## Results
### 1. Across the United States, which types of events (as indicated in the \color{red}{\verb|EVTYPE|}EVTYPE variable) are most harmful with respect to population health?
In order to address the question above, it is crucial we define and identify the variables associated with **population health**. From the dataset, we are mostly interested in variable **FATALITIES** & **INJURIES**.

We can measure the degree of harm by the total number of fatalities and/or injuries done by an event or the average fatalities and/or injuries by that event. These two different measures will likely give two distinct pictures as more frequent events might be less harmful and vice versa. I will make a plot reporting these two measures for events that have at least 100 instances.

```r
library(ggplot2)
library(tidyr)
reform <- noaa %>%
    group_by(EVENT_CAT) %>%
    filter(n() >= 100) %>%
    summarise(TOTAL_FATAL = sum(FATALITIES), TOTAL_INJUR = sum(INJURIES), AVG_FATAL = mean(FATALITIES), AVG_INJUR = mean(INJURIES)) %>%
    pivot_longer(cols = c("TOTAL_FATAL", "TOTAL_INJUR", "AVG_FATAL", "AVG_INJUR"), names_to = "MEASURE") %>%
    separate(col = MEASURE, into = c("MEASURE", "STATS"), "_") %>%
    mutate(EVENT_CAT = fct_reorder(EVENT_CAT, value))
reform %>%
    ggplot(aes(x = EVENT_CAT, fill = STATS)) + geom_bar(aes(y = value), stat = "identity", position = position_dodge()) +
    coord_flip() + facet_wrap(~MEASURE, nrow = 1, scales = "free_x") + labs(x = "", y = "", title = "Fatalities and Injuries by event") +
    scale_fill_discrete(name = "Measure", labels = c("Fatalities", "Injuries")) + theme(plot.title = element_text(hjust = 0.5))
```

<img src="storm-analysis_files/figure-html/harmful-event-1.png" style="display: block; margin: auto;" />

If we are interested in the total numbers of injuries and fatalities caused, Tornado is the most harmful, whereas, on average, hurricane causes the most injuries while heat-related events are the most fatal.



### 2. Across the United States, which types of events have the greatest economic consequences?
We calculate the damages in dollars by aggregating estimated damages caused to property and damages caused to crop yields to determine the economic consequences of these events.

```r
## Leave only entries that have a valid record of damages caused
noaa$PROPDMGEXP <- toupper(noaa$PROPDMGEXP)
noaa$CROPDMGEXP <- toupper(noaa$CROPDMGEXP)
noaa <- noaa %>%
    filter(PROPDMGEXP %in% c("K", "M", "B", "H", NA), CROPDMGEXP %in% c("K", "M", "B", NA))
noaa <- noaa %>%
    mutate(MULTIPLIER1 = case_when(grepl("K", PROPDMGEXP) ~ 10^3, grepl("M", PROPDMGEXP) ~ 10^6, grepl("H", PROPDMGEXP) ~
        100, grepl("K", PROPDMGEXP) ~ 10^9, is.na(PROPDMGEXP) ~ 0), MULTIPLIER2 = case_when(grepl("K", PROPDMGEXP) ~
        10^3, grepl("M", PROPDMGEXP) ~ 10^6, grepl("K", PROPDMGEXP) ~ 10^9, is.na(PROPDMGEXP) ~ 0)) %>%
    mutate(PROP_DMG = PROPDMG * MULTIPLIER1, CROP_DMG = CROPDMG * MULTIPLIER2)
reform1 <- noaa %>%
    group_by(EVENT_CAT) %>%
    filter(n() >= 100) %>%
    summarise(TOTAL_PROP = sum(PROP_DMG, na.rm = TRUE), TOTAL_CROP = sum(CROP_DMG, na.rm = TRUE), AVG_PROP = mean(PROP_DMG,
        na.rm = TRUE), AVG_CROP = mean(CROP_DMG, na.rm = TRUE)) %>%
    pivot_longer(cols = 2:5, names_to = "MEASURE") %>%
    separate(col = MEASURE, into = c("MEASURE", "TYPE"), "_") %>%
    mutate(EVENT_CAT = fct_reorder(EVENT_CAT, value))
reform1 %>%
    ggplot(aes(x = EVENT_CAT, fill = TYPE)) + geom_bar(aes(y = value), stat = "identity") + coord_flip() + facet_wrap(~MEASURE,
    nrow = 1, scales = "free_x") + labs(title = "Economic impact of events", subtitle = "Damages measured in US Dollar",
    x = "", y = "") + scale_fill_discrete(name = "Types of damage", labels = c("Crop", "Property")) + theme(plot.title = element_text(hjust = 0.5),
    plot.subtitle = element_text(hjust = 0.5))
```

<img src="storm-analysis_files/figure-html/economic-consequences-1.png" style="display: block; margin: auto;" />
In aggregate, flood and tornado have done exorbitant amount of damages to the US economy, totalling close to \$150 billions and slightly more than \$75 billions. The former disproportionately affects crop yields and harvests while the latter affects property more. On average, however, hurricane is by far the most destructive with each occurrence causing close to $60 millions in damages, mostly in property. 
