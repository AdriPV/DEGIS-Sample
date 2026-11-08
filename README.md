# ====================================
# Title: Evaluating the 2021 Chicago Speed Camera Policy
# Author: Adriana Puerta
# Date: 2022.07.22
# Description: This script cleans Chicago speed camera violation data and runs a fixed effects regression to evaluate the impact of the March 2021 policy change.
# =====================================

library(tidyverse)
library(lubridate)
library(ggplot2)
library(scales)

# 1. Load Data
# Note: Data downloaded from [[https://data.cityofchicago.org/Transportation/speed-violation/4ueg-apyt]]
speed_tickets <- read_csv("data/Speed_Camera_Violations.csv", show_col_types = FALSE)

# 2. Data Cleaning & Aggregation
# Aggregate violations to monthly level for plotting
monthly_violations <- speed_tickets %>%
  select(`VIOLATION DATE`, VIOLATIONS) %>%
  mutate(date = mdy(`VIOLATION DATE`)) %>%
  filter(date >= as.Date("2018-01-01") & date <= as.Date("2021-12-31")) %>%
  mutate(year = year(date), 
         month = month(date, label = TRUE, abbr = TRUE)) %>%
  group_by(year, month) %>%
  summarize(total_violations = sum(VIOLATIONS), .groups = 'drop')

# 3. Data Visualization
p1 <- ggplot(monthly_violations, aes(x = month, y = total_violations, 
                                     group = factor(year), color = factor(year))) +
  geom_line(linewidth = 1) +
  geom_point(size = 2) +
  scale_y_continuous(labels = comma) +
  scale_color_brewer(palette = "Set1") +
  labs(
    title = "Total Monthly Speed Camera Violations in Chicago (2018-2021)",
    subtitle = "Policy change took effect in March 2021",
    x = "Month", 
    y = "Total Violations",
    color = "Year"
  ) +
  theme_minimal()

ggsave("output/monthly_violations_plot.png", plot = p1, width = 8, height = 5)

# 4. Fixed Effects Regression
# Filter for cameras active before 2018 to establish a baseline
baseline_cameras <- speed_tickets %>%
  mutate(date = mdy(`VIOLATION DATE`)) %>%
  filter(date < as.Date("2018-01-01")) %>%
  distinct(`CAMERA ID`)

# Join back to get monthly violations for the baseline cameras
model_data <- speed_tickets %>%
  inner_join(baseline_cameras, by = "CAMERA ID") %>%
  mutate(date = mdy(`VIOLATION DATE`)) %>%
  filter(date >= as.Date("2018-01-01") & date <= as.Date("2021-12-31")) %>%
  mutate(
    post = ifelse(date >= as.Date("2021-03-01"), 1, 0),
    month_num = month(date)
  )

# Run Two-Way Fixed Effects Model
fe_model <- lm(VIOLATIONS ~ post + factor(`CAMERA ID`) + factor(month_num), 
               data = model_data)

# Save model results to a text file
sink("output/fe_model_results.txt")
print(summary(fe_model))
sink()

message("Analysis complete. Check the output/ folder for results.")
