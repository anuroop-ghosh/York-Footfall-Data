---
author: Anuroop Ghosh
output:
  html_document:
    toc: true
    toc_depth: 3
  pdf_document:
    toc: true
    toc_depth: 3
---

`{r setup, message=FALSE} #All required packages for the codes to run library(tidyverse) library(dplyr) library(scales) library(knitr)`

# Section 1

## Data Dictionary

### Overview

-   **Dataset Name:** York Footfall Data
-   **Source:**
    [Link](https://my.wbs.ac.uk/$/$/$/event/cmsfile/t/item/i/1766014/v/4/f/1/n/York_Footfall_data.csv)

### Variable Definitions

  --------------------------------------------------------------------------------
  Variable Name      Description    Data Type        Format         Valid Values
  ------------------ -------------- ---------------- -------------- --------------
  **Date**           Date of        Date             Date           2015-01-01 to
                     recorded data                   (YYYY-MM-DD)   2019-12-31

  **SiteName**       Name of the    Character                       York
                     site                                           

  **LocationName**   Name of the    Character                       Church Street,
                     Location of                                    Coney Street,
                     collected data                                 Micklegate,
                                                                    Parliament
                                                                    Street,
                                                                    Parliament
                                                                    Street at M&S,
                                                                    Stonegate

  **WeekDay**        Day of the     Character                       Monday,
                     week                                           Tuesday,
                                                                    Wednesday,
                                                                    Thursday,
                                                                    Friday,
                                                                    Saturday,
                                                                    Sunday

  **TotalCount**     Total number   Number/Numeric                  402 - 328310
                     of footfall                                    

  **Recording_ID**   ID of recorded Number/Numeric                  
                     data                                           
  --------------------------------------------------------------------------------

## Task 1

``` {r}
#First we import  the data into a variable
York_Footfall_data <- read_csv("York_Footfall_data.csv")

#After importing the data we check for any duplicate values in the dataset
duplicates <- York_Footfall_data[duplicated(York_Footfall_data), ]
duplicates
```

We observe that there are no duplicates in the data.

``` {r}
#Now we check the number of missing values in each column of the dataset
print(summary(is.na(York_Footfall_data)))
```

The dataset has 110 missing values, with 10 in the TotalCount column and
100 in the Recording_ID column. For our analysis, we interpret
Recording_ID as an identifier linked to the footfall count for a
specific day, and we assume that its missing values are likely due to a
technical issue. There is a chance these missing Recording_ID values
could indicate errors in footfall data entry; however, due to limited
information, we decide that Recording_ID is not critical for our
analysis. Therefore, we only address the missing values in TotalCount to
move forward.

``` {r}
#Storing in a new variable for ease of operation
data <- York_Footfall_data %>%
  filter(!is.na(TotalCount))

#We check the data now
str(data)
unique(data$SiteName)
unique(data$LocationName)
unique(data$WeekDay)
```

We see all the data is for the Site of York and the days of the week are
also entered correctly without any spelling mistakes, but we notice two
location names 'Parliament Street' and 'Parliament Street at M&S', with
similar names. Upon cross-checking with our [Reference 2 and 3](#5) we
find that, 'Parliament Street' and 'Parliament Street at M&S' refers to
the same place, which could indicate towards a potential data entry
error. In order to fix that, we will update the data by renaming both of
them as 'Parliament Street'.

``` {r}
#Updating the data by renaming Parliament Street at M&S to Parliament Street
data <- data %>%
  mutate(LocationName = ifelse(LocationName == "Parliament Street at M&S", "Parliament Street", LocationName))
```

Now we shall visualize the data.

``` {r}
#Using Histogram
ggplot(data, aes(x = TotalCount) ) +
  geom_histogram(alpha = 0.9, binwidth = 1000) + 
  scale_x_continuous(labels = comma) +
  labs(title = "Histogram of Footfall", x = "Footfall", y = "Frequency")
```

Visually, the data does not appear to be Normal, which we try confirming
using the Kolmogoroz-Smirnov Test and Q-Q Plot.

``` {r}
ks.test(data$TotalCount, "pnorm", mean=mean(data$TotalCount, sd=sd(data$TotalCount)))
qqnorm(data$TotalCount)
qqline(data$TotalCount, col = "red", lwd = 2)
```

We observe that the p-value is extremely low (\<0.05) and the D
statistic value is extremely high (\>0.05). Furthermore, the Q-Q Plot
also shows that multiple data points deviate significantly from the
theoretical quantile line. Both these observations help us conclude that
the data is not normal or does not follow a normal distribution.

``` {r}
#Now we shall visualize the data for outliers
#We use Boxplot
ggplot(data, aes(y = TotalCount)) +
  geom_boxplot() +
  facet_wrap(~LocationName, ncol = 3,scales = "free_y") +
  scale_y_continuous(labels = comma) +
  labs(title = "Box Plot of Footfall by Location", y = "Footfall", x = "Location")
```

We observe quite a few outliers in the data. In order to be certain, we
will examine it further using z-score/standard score method.

``` {r}
z_scores <- scale(data$TotalCount)
sigma <- 3
outlier <- data$TotalCount[abs(z_scores) > sigma]
outlier
```

We observe a total of 15 values that are outliers. This prompts us to
proceed with the data with two approaches, data with outliers and data
without the outliers respectively.

## Task 1.1(Data with Outliers)

Now we shall create two summary tables (for the two datas) that will
show the following for each location where footfall was measured: - The
date of the first and last day when footfall was measured at this
location - The mean daily footfall - The standard deviation of the daily
footfall - The highest daily footfall - The lowest daily footfall

``` {r}
#Summary Table 1
summary_table_1 <- data %>%
  group_by(LocationName) %>%
  summarise(First_Footfall_day = min(Date), 
Last_Footfall_Day = max(Date),
Mean_Daily_Footfall = mean(TotalCount),
Standard_Deviation_of_Daily_Footfall = sd(TotalCount),
Highest_Daily_Footfall = max(TotalCount),
Lowest_Daily_Footfall = min(TotalCount))
```

Now we display this in an organized form to make it more presentable
using knitr commands.

``` {r}
knitr::kable(summary_table_1,
             caption = "Summary Table with Outliers",
             booktabs = TRUE,
             col.names = c("Name of Location","First Date of Recorded Footfall", "Last Date of Recorded Footfall", "Mean Daily Footfall", "Standard Deviation of Daily Footfall", "Highest Daily Footfall", "Lowest Daily Footfall"))
```

We will proceed similarly for data without outliers.

## Task 1.2(Data without outliers)

``` {r}
#Storing the no-outlier data in a new variable
data_no_outlier <- data %>%
  filter(!abs(z_scores) > sigma)

#Summary Table 2
summary_table_2 <- data_no_outlier %>%
  group_by(LocationName) %>%
  summarise(First_Footfall_day = min(Date), 
Last_Footfall_Day = max(Date),
Mean_Daily_Footfall = mean(TotalCount),
Standard_Deviation_of_Daily_Footfall = sd(TotalCount),
Highest_Daily_Footfall = max(TotalCount),
Lowest_Daily_Footfall = min(TotalCount))

#Displaying the table
knitr::kable(summary_table_2,
             caption = "Summary Table without Outliers",
             booktabs = TRUE,
             col.names = c("Name of Location","First Date of Recorded Footfall", "Last Date of Recorded Footfall", "Mean Daily Footfall", "Standard Deviation of Daily Footfall", "Highest Daily Footfall", "Lowest Daily Footfall"))
```

## Task 2

For the next analyses, we will be working with data from the year 2019
(the last full year before the COVID pandemic).

``` {r}
#Importing the 2019 data in a new variable
data2019 <- data %>%
  filter(grepl("2019",Date))
```

Now we shall visualize the data.

``` {r}
#Using Histogram
ggplot(data2019, aes(x = TotalCount) ) +
  geom_histogram(alpha = 0.9, binwidth = 700) + 
  scale_x_continuous(labels = comma) +
  labs(title = "Histogram of Footfall for the year 2019", x = "Footfall", y = "Frequency")
```

We observe that the 2019 data is skewed and it does not appear to follow
a Normal Distribution either. We shall further confirm this using
Kolmogorov-Smirnov Test and Q-Q Plot.

``` {r}
ks.test(data2019$TotalCount, "pnorm", mean=mean(data2019$TotalCount, sd=sd(data2019$TotalCount)))
qqnorm(data2019$TotalCount)
qqline(data2019$TotalCount, col = "red", lwd = 2)
```

We observe that the p-value is extremely low (\<0.05) and the D
statistic value is extremely high (\>0.05). Furthermore, the Q-Q Plot
also shows that multiple data points deviate significantly from the
theoretical quantile line. Both these observations help us conclude that
the data is not normal or does not follow a normal distribution.

Now we shall visualize the data for outliers.

``` {r}
#We use Boxplot
ggplot(data2019, aes(y = TotalCount)) +
  geom_boxplot() +
  facet_wrap(~LocationName, ncol = 4,scales = "free_y") +
  scale_y_continuous(labels = comma) +
  labs(title = "Box Plot of Footfall by Location", y = "Footfall", x = "Location")
```

We can observe multiple outliers in the 2019 data. We will be working
with z-scores once again to check for the present outliers in the data.

``` {r}
z_scores_2 <- scale(data2019$TotalCount)
sigma <- 3
outlier_2 <- data2019$TotalCount[abs(z_scores) > sigma] 
outlier_2
```

Contrary to the observations in the Box-plot, the z-scores indicate that
there are no outliers in the data. To verify this, we re-visualize the
data using Scatter-plot.

``` {r}
ggplot(data2019, aes(x = Date, y = TotalCount, color = LocationName) ) +
  geom_point(alpha = 0.9) +
  facet_wrap(~LocationName, nrow = 3,scales = "free_y") +
  labs(title = "Scatter Plot of Footfall by Location for the year 2019", y = "Footfall", x = "Location") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

The Scatter-plot points out that the data points, while potentially
deviating from the overall trend, are not significantly extreme, which
is why the z-score did not identify any outliers. Another instance why
Box-plots can be misleading occasionally.

Now to show the distribution of footfall across all days in the dataset,
separated by each Location Name, we will be using a Histogram.

``` {r}
#Histogram
ggplot(data2019, aes(x = TotalCount)) +
  geom_histogram(binwidth = 150, fill = "steelblue", color = "black") +  # Adjust binwidth as needed
  labs(title = "Distribution of Daily Footfall in 2019",
       x = "Daily Footfall",
       y = "Number of Days") +
  facet_wrap(~LocationName) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

For the next parts, we store the datas of Coney Street and Stonegate in
separate variables.

``` {r}
#We store the Coney Street Data in a variable
Coney_Street_data_2019 <- data2019 %>%
  filter(grepl("Coney Street",LocationName,ignore.case = TRUE))

#We do the same for Stonegate
Stonegate_data_2019 <- data2019 %>%
  filter(grepl("Stonegate",LocationName,ignore.case = TRUE))
```

Now, in order to perform the t-tests, we compare the variances of both
the locations and proceed with the t-test according to our findings.

``` {r}
#For data with outliers
#We compare the variances of both the datasets
var(Coney_Street_data_2019$TotalCount) == var(Stonegate_data_2019$TotalCount)

#We perform Welch t-test as their variances are unequal
t_test_1 <- t.test(Coney_Street_data_2019$TotalCount, Stonegate_data_2019$TotalCount, alternative = "two.sided", var.equal = FALSE)
print(t_test_1)
```

Next, we proceed with our analysis for the weekend data for Coney Street
and Stonegate.

``` {r}
#We store Weekend Data for Coney Street
Coney_Street_Weekend_2019 <- Coney_Street_data_2019 %>%
  filter(WeekDay %in% c("Saturday", "Sunday"), ignore.case = TRUE)

#Similarly for Stonegate
Stonegate_Weekend_2019 <- Stonegate_data_2019 %>%
  filter(WeekDay %in% c("Saturday", "Sunday"), ignore.case = TRUE)

#We compare the variances of these two datasets
var(Coney_Street_Weekend_2019$TotalCount) == var(Stonegate_Weekend_2019$TotalCount)

#We perform Welch t-test for the weekend data as their variances are also unequal
t_test_2 <- t.test(Coney_Street_Weekend_2019$TotalCount, Stonegate_Weekend_2019$TotalCount, alternative = "two.sided", var.equal = FALSE)
print(t_test_2)
```

------------------------------------------------------------------------

# Section 2

## Objective

In this project, we analyzed footfall data from York, focusing on
2019.Our objective was to advise a promoter on the best site to put
their stall, primarily between Coney Street and Stonegate, in order to
draw as many people as possible during the week, and on the weekends.

## Insights

### Overall Footfall Comparison:

1)  Coney Street had a consistently higher average footfall than
    Stonegate over the entire week.
2)  Statistical analysis suggests there is a 95% chance that the
    difference in average footfall between these locations ranges from
    an additional 671 to 2,556 people at Coney Street. This means that
    placing a stall at Coney Street could potentially lead to
    significantly more foot traffic.

### Weekend Footfall Comparison:

1)  For weekends specifically, we observed that both locations have
    similar average footfall numbers.
2)  There's a 95% probability that the difference in average weekend
    footfall could vary, potentially from a decrease of 2,362 people to
    an increase of 1,755 people at Coney Street. This wider range
    indicates that weekend foot traffic is more variable.

## Conclusion

Based on the 2019 data, Coney Street is the recommended location for
setting up a stall to achieve higher footfall during the week. For
weekends, both locations are likely to offer similar foot traffic, but
Coney Street still holds a slight advantage overall.

# References {#5}

1)  Fundamentals of Mathematical Statistics - S.C. Gupta, V.K. Kapoor
2)  [Arcgis Hub - City of York
    Council](https://hub.arcgis.com/datasets/CYC::footfall-cameras/explore?location=53.958999%2C-1.084650%2C14.98)
3)  [Google Maps - Parliament Street,
    York](https://www.google.co.uk/maps/place/Parliament+St,+York/@53.9589064,-1.0836557,17z/data=!3m1!4b1!4m6!3m5!1s0x487931a8b434c03f:0xc503d849a5b66094!8m2!3d53.9589033!4d-1.0810754!16s%2Fg%2F1vfp7qjg?entry=ttu&g_ep=EgoyMDI0MTAyNy4wIKXMDSoASAFQAw%3D%3D)
4)  [RMarkdown Cheat
    Sheet](https://github.com/rstudio/cheatsheets/raw/main/rmarkdown-2.0.pdf)
5)  [Gemini AI](https://gemini.google.com/app)
6)  [ChatGPT AI](https://chatgpt.com)
7)  [WBS Individual Assignment Question and
    Guidance](https://my.wbs.ac.uk/$/$/$/event/cmsfile/t/item/i/1766014/v/4/f/2/n/Individual-Assignment-1-IB94X---4--5-.pdf)
8)  [WBS Business Statistics
    Forum](https://my.wbs.ac.uk/-/academic/337712/forums/forum/354921/topic/1767072/#post_1058890)

------------------------------------------------------------------------