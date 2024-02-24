<!-- omit in toc -->
# Google Data Analytics Professional Certificate: Bellabeat Case Study

*Author:* Charles R. Clark <br />
*Started:* February 17, 2024 <br />
*Finished:* ???, 2024

<!-- omit in toc -->
## Table of Contents
- [Ask Phase](#ask-phase)
  - [Business Task](#business-task)
  - [Key Stakeholders](#key-stakeholders)
- [Prepare Phase](#prepare-phase)
  - [Data Limitations](#data-limitations)
- [Process Phase](#process-phase)
  - [`weightLogInfo_merged.csv` Processing](#weightloginfo_mergedcsv-processing)
  - [`dailySteps_merged.csv` Processing](#dailysteps_mergedcsv-processing)
  - [`dailyIntensities_merged.csv` Processing](#dailyintensities_mergedcsv-processing)
  - [`dailyCalories_merged.csv` Processing](#dailycalories_mergedcsv-processing)
  - [`dailyActivity_merged.csv` Processing](#dailyactivity_mergedcsv-processing)
  - [`INNER JOIN` Between the Four `daily*_merged.csv` Files](#inner-join-between-the-four-daily_mergedcsv-files)
- [Analyze Phase](#analyze-phase)
  - [`joined_daily_activity_calories_intensity_steps.csv` Analysis](#joined_daily_activity_calories_intensity_stepscsv-analysis)
    - [Grouped by Weekday](#grouped-by-weekday)

## Ask Phase

### Business Task

Analyze existing non-Bellabeat product consumer data to identify Bellabeat's areas of opportunity for growth. Specifically, to determine how consumers use health-geared smart devices in their everyday lives and apply these general trends to Bellabeat's customers in order to influence the company's marketing strategies.

### Key Stakeholders
 - *Urška Sršen:* Bellabeat co-founder, Chief Creative Officer
 - *Sando Mur:* Bellabeat co-founder, member of Bellabeat Executive Team
 - *Bellabeat Marketing Analytics Team:* Data analyst team responsible for guiding Bellabeat's marketing strategies.

## Prepare Phase

The data used throughout this project is the "FitBit Fitness Tracker Data" by Möbius on Kaggle (found [here](https://www.kaggle.com/datasets/arashnic/fitbit/data)). All the data are stored in CSV files located in the `./data/original_data` directory, and each file contains data collected between April 12, 2016 and May 12, 2016 from 30 FitBit users in either long or wide format. 

### Data Limitations

 - The data was collected during a month-long window of time; this is a relatively short amount of time and may not account for seasonal trends in the data.
 - The data being used are almost 8 years old at the time of this case study. Because the data are not up-to-date, they may not reflect more recent trends in the data.
 - The data also suffers from a limited sample size. The central limit theorem (CLT) states that a sample size of at least 30 subjects is generally enough to perform a valid analysis, but making statistically significant observations may be more challenging with the smaller sample size as confidence intervals and other statistical metrics allow for smaller windows of uncertainty when sample sizes are larger.

## Process Phase

The data in each CSV file seem to be relatively clean already. However, I did perform a minimal amount of processing with Google Sheets on some of the data files; this processing is described here. Lastly, I obtained the final data sets to be used in my analysis by importing the Google Sheets-cleaned data into BigQuery and obtaining the data from overlapping dates (only the dates which were still present in all of the data files after initial processing). All processed data can be found in the `./data/cleaned_data` directory.

###  `weightLogInfo_merged.csv` Processing

The only processing step performed on the data in this file was to delete the `Fat` column, as it only contained 2 non-null entries. 

### `dailySteps_merged.csv` Processing

The data in this file required some more processing that the previous file. First, I filtered the data by the `StepTotal` feature and found that a considerable number of records had 0 entries; these records were removed, as my own experience with FitBit devices is that they record 0 steps when the user isn't wearing the device for the day. From here, I removed records with step counts less than 50, as I cannot imagine a person taking fewer than 50 steps while consistently wearing their FitBit device. 

### `dailyIntensities_merged.csv` Processing

The data in this file required some more extensive processing as well. First, I removed the records where the sum of all activity times (see formula in code block below) was less than 2 hours; this was done to limit the effect of inconsistent FitBit device wearing on any analyses we may perform later.

```
SumHours = floor((SedentaryMinutes + LightActivityMinutes + FairlyActiveMinutes + VeryActiveMinutes)
```

Next, I removed the records where the sum of all activity times (see formula in code block above) was greater than 20 hours, as this may be indicative of the FitBit device never detecting the user's sleep. From here, I removed the one remaining record where the amount of time spent in sedintary activity was greater than 18 hours (1,080 minutes) for the same reason. 

### `dailyCalories_merged.csv` Processing

The only processing step performed on the data in this file was to remove records where the `Calories` feature contained a value less than 100, as this is indicative of incosistent FitBit device wearing.

### `dailyActivity_merged.csv` Processing

The only processing step performed on the data in this file was to remove both features which are mostly zeros (specifically the `LoggedActivitiesDistance` and `SedentaryActiveDistance` features) and those which overlap with the other data files (specifically the `TotalSteps`, `VeryActiveDistance`, `ModeratelyActiveDistance`, `LightActiveDistance`, `VeryActiveMinutes`, `FairlyActiveMinutes`, `LightlyActiveMinutes`, `SedentaryMinutes`, and `Calories` features). I expect any other inconsistencies to be filtered out when I perform the `INNER JOIN` between this file and the `dailySteps_merged.csv`, `dailyIntensities_merged.csv`, `dailyCalories_merged.csv` files.

### `INNER JOIN` Between the Four `daily*_merged.csv` Files 

The following SQL query was performed using Google BigQuery in order to join together the four cleaned `daily*_merged.csv` files: all null containing records were filtered out, and names were standardized (specifically among the `dailyIntensities_merged.csv` file's records).

```
SELECT 
  activity.Id,
  activity.ActivityDate,
  activity.TotalDistance,
  activity.TrackerDistance,
  calories.Calories,
  intensities.SedentaryMinutes,
  intensities.LightlyActiveMinutes,
  intensities.FairlyActiveMinutes AS ModeratelyActiveMinutes,
  intensities.VeryActiveMinutes,
  intensities.SedentaryActiveDistance,
  intensities.LightActiveDistance,
  intensities.ModeratelyActiveDistance,
  intensities.VeryActiveDistance,
  steps.StepTotal
FROM
  `big_query_project.bellabeat_case_study.activity` AS activity
  INNER JOIN
    `big_query_project.bellabeat_case_study.calories` AS calories
  ON
    activity.Id = calories.Id
  AND
    activity.ActivityDate = calories.ActivityDay
  INNER JOIN
    `big_query_project.bellabeat_case_study.intensities` AS intensities
  ON
    calories.Id = intensities.Id
  AND
    calories.ActivityDay = intensities.ActivityDay
  INNER JOIN
    `big_query_project.bellabeat_case_study.steps` AS steps
  ON
    intensities.Id = steps.Id
  AND
    intensities.ActivityDay = steps.ActivityDay;
```

The data set resulting from this join operation was exported out of BigQuery into the `joined_daily_activity_calories_intensity_steps.csv`. I then imported the joined data set into Google Sheets, made some further naming convention updates, and prepared for analysis.

## Analyze Phase

### `joined_daily_activity_calories_intensity_steps.csv` Analysis


#### Grouped by Weekday

I imported the `joined_daily_activity_calories_intensity_steps.csv` file into BigQuery and executed the SQL queery below; the results were saved in `./data/analysis_data/weekday_activity_analysis.csv`.

```
SELECT
  FORMAT_DATE("%A", ActivityDate) AS Weekday,
  AVG(LightlyActiveMinutes + ModeratelyActiveMinutes + VeryActiveMinutes) AS AvgTotalActiveMinutes,
  AVG(Calories) AS AvgCalories,
  AVG(StepTotal) AS AvgStepTotal
FROM
  `big_query_project.bellabeat_case_study.joined_dailies`
GROUP BY
  Weekday
ORDER BY
  CASE
    WHEN Weekday = "Sunday" THEN 1
    WHEN Weekday = "Monday" THEN 2
    WHEN Weekday = "Tuesday" THEN 3
    WHEN Weekday = "Wednesday" THEN 4
    WHEN Weekday = "Thursday" THEN 5
    WHEN Weekday = "Friday" THEN 6
    WHEN Weekday = "Saturday" THEN 7
  END ASC;
```

We found that of all the days in the week, people seem to be the most active on Saturday and the least active on Sunday.
