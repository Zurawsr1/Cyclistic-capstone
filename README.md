---
title: "Google Data Analytics Case Study 1: Divvy Bike-Share"
output:
  html_document:
    output_dir: "C:/Users/rober/OneDrive/Documents/Cyclistic_capstone_git"
author: Robert Zurawski
date: "2025-03-01"
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Introduction

This is optional path 1 of the Google Professional Data Analytics Certification Capstone project. In this case study, I will be using r to perform a real-world task for a fictional company in order to demonstrate the analytic ability of a junior data analyst. Data was collected from the real bike-share company, Divvy.

```{r scenario, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

### Scenario

I am a junior data analyst working on the marketing analyst team at Cyclistic, a bike-share company in Chicago. The director of marketing believes the company’s future success depends on maximizing the number of annual memberships. Therefore, my team wants to understand how casual riders and annual members use Cyclistic bikes differently. From these insights, my team will design a new marketing strategy to convert casual riders into annual members. But first, Cyclistic executives must approve my recommendations, so they must be backed up with compelling data insights and professional data visualizations.

```{r company, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

### About the Company.

Cyclistic is a bike-share program that features more than 5,800 bicycles and 600 docking stations. Cyclistic sets itself apart by also offering reclining bikes, hand tricycles, and cargo bikes, making bike-share more inclusive to people with disabilities and riders who can't use a standard two-wheeled bike. The majority of riders opt for traditional bikes; about 8% of riders use the assistive options. Cyclistic users are more likely to ride for leisure, but about 30% use the bikes to commute to work each day.

```{r shareholders, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

### Key Shareholders

Lily Moreno: The director of marketing and my manager. Moreno is responsible for the development of campaigns and initiatives to promote the bike-share program. These may include email, social media, and other channels.

Cyclistic executive team: The notoriously detail-oriented executive team will decide whether to approve the recommended marketing program.

```{r task, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

### Business Task:

Three questions will guide the future marketing program:

1.  How do annual members and casual riders use Cyclistic bikes differently?

2.  Why would casual riders buy Cyclistic annual memberships?

3.  How can Cyclistc use digital media to influence casual riders to become members?

```{r About_Data, echo=FALSE, message=FALSE, warning=FALSE}

library(tidyverse)
library(scales)
Data_1 <- read_csv("202401-divvy-tripdata.csv")
Data_2 <- read_csv("202402-divvy-tripdata.csv")
Data_3 <- read_csv("202403-divvy-tripdata.csv")
Data_4 <- read_csv("202404-divvy-tripdata.csv")
Data_5 <- read_csv("202405-divvy-tripdata.csv")
Data_6 <- read_csv("202406-divvy-tripdata.csv")
Data_7 <- read_csv("202407-divvy-tripdata.csv")
Data_8 <- read_csv("202408-divvy-tripdata.csv")
Data_9 <- read_csv("202409-divvy-tripdata.csv")
Data_10 <- read_csv("202410-divvy-tripdata.csv")
Data_11 <- read_csv("202411-divvy-tripdata.csv")
Data_12 <- read_csv("202412-divvy-tripdata.csv")

Full_Data <- rbind(Data_1, Data_2, Data_3, Data_4, Data_5, Data_6, Data_7, Data_8, Data_9, Data_10, Data_11, Data_12)

## Cleaning Data

## Step 1: remove unneeded columns (longitute latitute)
	cleaned_data <- Full_Data %>% select(-start_station_name, -start_station_id, -end_station_name, -end_station_id, -start_lat, -start_lng, -end_lat, -end_lng)
## Step 2: Clean the data to make sure everything is properly formatted (ymd_hms)
# Check for parsing issues in started_at
invalid_started_at <- cleaned_data %>%
  filter(is.na(ymd_hms(started_at)))

# Check for parsing issues in ended_at
invalid_ended_at <- cleaned_data %>%
  filter(is.na(ymd_hms(ended_at)))

# View the counts of problematic rows
#cat("Invalid started_at rows: ", nrow(invalid_started_at), "\n")
#cat("Invalid ended_at rows: ", nrow(invalid_ended_at), "\n")

# Convert started_at and ended_at to datetime format
cleaned_data <- cleaned_data %>%
  mutate(
    started_at = ymd_hms(started_at),
    ended_at = ymd_hms(ended_at)
  )

#Remove all invalid rows
cleaned_data <- cleaned_data %>%
  filter(!is.na(started_at) & !is.na(ended_at))

# Confirm no NA values in started_at and ended_at
#any(is.na(cleaned_data$started_at)) # Should return FALSE
#any(is.na(cleaned_data$ended_at))   # Should return FALSE

## Step 3: create new columns for day of week and Month and ride time
cleaned_data <- cleaned_data %>%
  mutate(
    #extract day of week
    day_of_week = wday(started_at, label = TRUE, abbr = FALSE),  # Full name of the day
    
    #extract the month name
    month_name = month(started_at, label = TRUE, abbr = FALSE),  # Full month name
    
    #calculate ride time in minutes
    ride_time = as.numeric(difftime(ended_at, started_at, units = "mins"))
  )

## Step 4: Transform any unusuable data into usable data 
cleaned_data <- cleaned_data %>%
  mutate(month_name_abbr = case_when(
    month_name == "January" ~ "Jan",
    month_name == "February" ~ "Feb",
    month_name == "March" ~ "Mar",
    month_name == "April" ~ "Apr",
    month_name == "May" ~ "May",
    month_name == "June" ~ "Jun",
    month_name == "July" ~ "Jul",
    month_name == "August" ~ "Aug",
    month_name == "September" ~ "Sep",
    month_name == "October" ~ "Oct",
    month_name == "November" ~ "Nov",
    month_name == "December" ~ "Dec",
    TRUE ~ month_name  # Fallback in case something is unexpected
  ))



## Step 5: Remove any uneeded data again after new columns and transformed data

cleaned_data <- cleaned_data %>%
  select(-started_at, -ended_at)

## Step 6: create bins for ride time with a maximum. 
# Create ride time bins with larger intervals and "60+ minutes"
cleaned_data <- cleaned_data %>%
  mutate(
    ride_time_bin = cut(
      ride_time,
      breaks = c(seq(0, 60, by = 10), Inf), # 10-minute intervals up to 60, then 60+
      labels = c(
        paste(seq(10, 60, by = 10) - 10, "to", seq(10, 60, by = 10), "minutes"),
        "60+ minutes"
      ),
      include.lowest = TRUE,
      right = FALSE # Left-inclusive intervals
    )
  )

weekend_graph <- ggplot(cleaned_data, aes(x = day_of_week, fill = member_casual)) +
  geom_bar(position = "dodge", color = "black") +
  labs(
    title = "Rides by Day of Week",
    x = "Day of the Week",
    y = "Number of Rides",
    fill = "User Type"
  ) +
  theme_minimal() +
  scale_x_discrete(limits = c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday")) + 
scale_y_continuous(labels = comma) + theme(axis.text.x = element_text(angle = 45, hjust = 1))

# Load necessary libraries
library(ggplot2)
library(scales)  # For comma formatting in y-axis

# Create the plot
graph_months <- ggplot(cleaned_data, aes(x = month_name_abbr, fill = member_casual)) +
  geom_bar(position = "dodge", color = "black") +
  labs(
    title = "Rides by Month and User Type",
    x = "Month",
    y = "Number of Rides",
    fill = "User Type"
  ) +
  theme_minimal() +
  scale_x_discrete(
    limits = c("Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec")
  ) +
  scale_y_continuous(labels = comma)  # Format y-axis numbers with commas





graph_bike_style <-ggplot(cleaned_data, aes(x = rideable_type, fill = member_casual)) +
    geom_bar(position = "dodge", color = "black") +
    labs(
        title = "Rideable Type",
        x = "Type of Bike",
        y = "Number of Rides",
        fill = "User Type"
    ) +
    theme_minimal() + scale_y_continuous(labels = comma)

graph_ride_time <-ggplot(cleaned_data %>% drop_na(ride_time_bin), aes(x = ride_time_bin, fill = member_casual)) +
  geom_bar(position = "dodge", color = "black") +
  labs(
    title = "User Ride Time",
    x = "Ride Time",
    y = "Number of Rides",
    fill = "User Type"
  ) +
  theme_minimal() +
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1) # Rotate x-axis labels for clarity
  ) + 
  scale_y_continuous(labels = comma)


```

### About the Data.

In this Project, we collected data from the bike-share company Divvy. We collected the most recent 12 months of data, from January 2024 to December 2024. The data had all the information we needed, most importantly, start and end time, which included the date, and time of day they started and finished their ride. First, we cleaned this data by making sure all of the dates and times were in the proper format. Then we transformed this data to show us the day of week, ride time, and the month name. We used these variables to see what the differences in behavior are for annual members and casual riders.

### Graphing our findings

#### Day of Week

  First, we had a suspicion that annual members use the service more on weekdays for transportation to and from work, while casual members use it more on weekends for leisure. We confirmed that suspicion by plotting the counts for each type of user compared with the day of the week. In order to convert these casual riders into annual memberships, we can launch a new weekend only membership. This will be offered at a slight discount and have an "Upgrade to Annual Membership" option. 

```{r Week_Day, echo=FALSE}
plot(weekend_graph) 
```

#### Rider Seasonality

  Next, since the data measures users in Chicago, we assumed all users would take a dip during the Winter months. If this hypothesis is correct, it will not affect our analysis, but we still must check to make sure there are no other patterns between user types. The team confirmed this by graphing rides per month for casual riders and annual members.

```{r Month_Name, echo=FALSE}
plot(graph_months)
```

#### Kind of Bike

  The data shows bike usage differences between annual members and casual riders. Annual members have much higher overall use than casual riders. Both users prefer the electric bikes to a classic bike, but there isn't a huge discrepancy between these. Casual riders ride electric scooters more than annual members, but overall scooter usage is comparatively low to bikes.

```{r bike_type, echo=FALSE}
plot(graph_bike_style)
```

#### Ride Time

  To get an idea of how long users were riding, we broke down ride time between casual riders and annual members. Overall, most users were riding for less than 20 minutes a session, however, longer rides are more popular with casual riders than annual members. Because of this, we should consider targeting casual users with longer rides with email signups that will get them a slightly discounted ride. We can then start an email advertising campaign to sway these casual users. 

```{r ride_time, echo=FALSE}
plot(graph_ride_time)
```

```{r analysis, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
# Step 1: Filter for casual users and weekends
weekend_data <- cleaned_data %>%
  filter(
    member_casual == "casual" & 
      day_of_week %in% c("Friday", "Saturday", "Sunday")
  )

# Step 2: Identify repeat users (repeated ride_id)
repeat_counts <- weekend_data %>%
  group_by(ride_id) %>%
  summarize(count = n()) %>%
  ungroup()

# Find the total number of casual rides and repeat rides
total_casual_weekend_rides <- nrow(weekend_data) # Total casual weekend rides
repeat_rides <- repeat_counts %>%
  filter(count > 1) %>%
  summarize(repeat_total = n()) %>%
  pull(repeat_total) # Count of repeating ride_id's

# Step 3: Calculate the percentage of repeats
repeat_percentage <- (repeat_rides / total_casual_weekend_rides) * 100

# Output the results
cat("Total casual weekend rides:", total_casual_weekend_rides, "\n")
cat("Number of repeating casual ride_id's:", repeat_rides, "\n")
cat("Percentage of repeating casual ride_id's:", round(repeat_percentage, 2), "%", "\n")

```

### Conclusion

Our analysis shows that casual users tend to ride more on weekends. Only a small percentage of these weekend riders are repeat users (.01%). Also, while the majority of both types of riders aren't taking long rides, casual users take more long rides.

```{r Conclusion, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Recommendation

From our findings, we recommend:

1.  Creating repeat casual customers by using targeting advertising on social media. We can target our weekend casual riders and launch ads from Thursdays to Sundays on their social media platforms.

2.  Convert the repeat casual users into membership users by creating a weekend only membership. This will be offered at a slight discount from the annual membership and will also include an "upgrade to annual membership" option.

3.  Consider targeting casual users on longer rides with email sign ups for a slightly discounted ride in order to start an email advertising campaign to sway casual users.
