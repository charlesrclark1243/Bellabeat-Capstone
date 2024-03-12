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
  - [`joined_daily_activity_calories_intensity_steps.csv` Analysis Results](#joined_daily_activity_calories_intensity_stepscsv-analysis-results)
    - [**...Grouped by Weekday Analysis Results**](#grouped-by-weekday-analysis-results)
  - [`daily_sleep_merged_cleaned.csv` Analysis Results](#daily_sleep_merged_cleanedcsv-analysis-results)
  - [`weight_log_info_merged_cleaned.csv` Analysis Results](#weight_log_info_merged_cleanedcsv-analysis-results)
    - [**...Grouped by ID BMI Analysis Results**](#grouped-by-id-bmi-analysis-results)
  - [Insights](#insights)
    - [**...From `joined_daily_activity_calories_intensity_steps.csv` Analysis Results**](#from-joined_daily_activity_calories_intensity_stepscsv-analysis-results)
    - [**...From `daily_sleep_merged_cleaned.csv` Analysis Results**](#from-daily_sleep_merged_cleanedcsv-analysis-results)
    - [**...From `weight_log_info_merged_cleaned.csv` Analysis Results**](#from-weight_log_info_merged_cleanedcsv-analysis-results)
- [Share Phase](#share-phase)
- [Act Phase](#act-phase)
- [References](#references)

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

The only processing step performed on the data in this file was to delete the `Fat` column, as it only contained 2 non-null entries, and to create the `BMIWeightClass` feature based on United Kingdom National Health Service's (NHS) guidelines on weight classes from BMI ranges.

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

The only processing steps performed on the data in this file was to remove both features which are mostly zeros (specifically the `LoggedActivitiesDistance` and `SedentaryActiveDistance` features) and those which overlap with the other data files (specifically the `TotalSteps`, `VeryActiveDistance`, `ModeratelyActiveDistance`, `LightActiveDistance`, `VeryActiveMinutes`, `FairlyActiveMinutes`, `LightlyActiveMinutes`, `SedentaryMinutes`, and `Calories` features), as well as to adding the `Weekday` and `TotalActiveMinutes` feature. I expect other inconsistencies which were not handled here to be filtered out when I perform the `INNER JOIN` between this file and the `dailySteps_merged.csv`, `dailyIntensities_merged.csv`, `dailyCalories_merged.csv` files.

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

### `joined_daily_activity_calories_intensity_steps.csv` Analysis Results

I performed a standard summarizing analysis on the `Calories`, `StepTotal`, `SedentaryMinutes`, `LightlyActiveMinutes`, `ModeratelyActiveMinutes`, and `VeryActiveMinutes` features, obtaining the following results which are saved in `./data/analysis_data/joined_daily_activity_calories_intensities_steps_analysis.csv`:

|        | Calories | StepTotal | SedentaryMinutes | LightlyActiveMinutes | ModeratelyActiveMinutes | VeryActiveMinutes |
| ------ | -------- | --------- | ---------------- | -------------------- | ----------------------- | ----------------- |
| Mean   | 2356.77 |	8445.63	| 700.73	| 215.40	| 17.60	| 24.46 |
| Standard Deviation | 759.93	| 4085.55	| 135.72	| 86.20	| 22.38	| 35.90 |
| Min    | 741.00 |	254.00	| 125.00	| 17.00	| 0.00	| 0.00 |
| 25th Percentile | 1788.00	| 5183.00	| 631.00	| 156.00	| 0.00	| 0.00 |
| Median | 2180.00	| 8863.00	| 412.50	| 67.00	| 10.00	| 0.00 |
| 75th Percentile | 2896.00	| 11193.00	| 776.00	| 263.00	| 26.00	| 36.00 |
| Max    | 4900.00	| 22770.00	| 1062.00	| 518.00	| 143.00	| 210.00 |


#### **...Grouped by Weekday Analysis Results**

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

From this query, we obtained the following results:

| Weekday	| AvgTotalActiveMinutes	| AvgCalories	| AvgStepTotal |
| ------- | --------------------- | ----------- | ------------ |
| Sunday	| 236.77	| 2262.37	| 7423.62 |
| Monday	| 269.76	| 2373.43	| 9020.50 |
| Tuesday	| 266.17	| 2500.47	| 9183.23 |
| Wednesday	| 243.10	| 2361.79	| 7922.96 |
| Thursday	| 238.33	| 2226.07	| 7992.87 |
| Friday	| 262.53	| 2335.06	| 8047.13 |
| Saturday	| 294.92	| 2449.38	| 9715.98 |

### `daily_sleep_merged_cleaned.csv` Analysis Results

I first used Google Sheets to calculate the `TotalHoursAsleep` and `TotalHoursInBed` features from the `TotalMinutesAsleep` and `TotalMinutesInBed` (previously `TotalTimeInBed`), and then I calculated the difference between these two hourly-rated metrics using the formula

```
HoursAsleepHoursInBedDiff = TotalHoursInBed - TotalHoursAsleep
```

to determine how much time was spent in bed but not sleeping for each record.

From here, I performed a standard summarizing analysis on the `TotalHoursAsleep`, `TotalHoursInBed`, and `HoursAsleepHoursInBedDiff` features, obtaining the following results which are saved in `daily_sleep_analysis.csv`:

|	     | TotalHoursAsleep	| TotalHoursInBed	| HoursAsleepHoursInBedDiff |
| ---- | ---------------- | --------------- | ------------------------- |
| Mean |	6.9911	| 7.6440	| 0.6529 |
| Standard Deviation	| 1.9724	| 2.1184	| 0.7762 |
| Min	| 0.9667	| 1.0167	| 0.0000 |
| 25th Percentile | 6.0167	| 6.7167	| 0.2833 |
| Median	| 7.2167	| 7.7167	| 0.4167 |
| 75th Percentile	| 8.1667	| 8.7667	| 0.6667 |
| Max |	13.2667	| 16.0167	| 6.1833 |

### `weight_log_info_merged_cleaned.csv` Analysis Results

I also performed a standard summarizing analysis on the `WeightPounds` and `BMI` features (I excluded `WeightKg` from this analysis because I was already using pounds as the unit of weight, so using kg would be unnecessarily redundant) and obtained the following results which are saved in `weight_log_info_analysis.csv`:

|	     | WeightPounds	| BMI |
| ---- | ------------ | --- |
| Mean	| 158.8118 |	25.1852 |
| Standard Deviation	| 30.6954	| 3.0670 |
| Min	| 115.9631	| 21.4500 |
| 25th Percentile	| 135.3638	| 23.9600 |
| Median	| 137.7889	| 24.3900 |
| 75th Percentile	| 187.5032	| 25.5600 |
| Max	| 294.3171	| 47.5400 |

#### **...Grouped by ID BMI Analysis Results**

I also felt it important to categorize the different users by their average BMIs in order to better understand the user base of such fitness devices. I had intended to import the data into BigQuery, however I received an error when submitting the load job, so I decided to use a Google Sheets pivot table instead; the results (which are saved in `weight_log_info_bmi_analysis.csv`) are shown below:

| Id	| AVERAGE of BMI |
| --  | -------------- |
| 1503960366	| 22.65 |
| 1927972279	| 47.54 |
| 2873212765	| 21.57 |
| 4319703577	| 27.41 |
| 4558609924	| 27.21 |
| 5577150313	| 28.00 |
| 6962181067	| 24.03 |
| 8877689391	| 25.49 |
| Grand Total	| 25.19 |

### Insights

#### **...From `joined_daily_activity_calories_intensity_steps.csv` Analysis Results**

From any standard summarizing analysis of the data in, one can get a general picture of what the distributions for each of the included features look like:
  - when mean > median, the distribution is likely positively (right) skewed.
  - when mean < median, the distribution is likely negatively (left) skewed.
  - when mean = median, the distribution is likely symmetric (zero skew).

Therefore, the distributions of the features included in `joined_daily_activity_calories_intensities_steps_analysis.csv` are best described as follows:

| Feature | Mean | Median | Skew |
| ------- | ---- | ------ | ----------------------------- |
| Calories | 2356.77 | 2180.00 | Positive |
| StepTotal | 8445.63 | 8863.00 | Negative |
| Sedentary Minutes | 700.73 | 412.50 | Positive |
| LightlyActiveMinutes | 215.40 | 67.00 | Positive |
| ModeratelyActiveMinutes | 17.60	| 10.00 | Positive |
| VeryActiveMinutes | 24.46 | 0.00 | Positive |

According to the Centers for Disease Control and Prevention (CDC), people should aim to achieve 10,000 steps per day. Based on my summarizing analysis, the average participating FitBit user in this data achieves fewer than 10,000 steps per day; this implies that the average participating FitBit user might not be as active as is recommended. This is further supported by the activity intensity data: about 50% of participating FitBit users spend less than about 412 minutes engaging in sedentary activity, 67 minutes engaging in light activity, 10 minutes engaging in moderate activity, and no time at all engaging in intense activity. 

In my weekday analysis of `joined_daily_activity_calories_intensity_steps.csv`, I found that participating FitBit users spend the most time on average engaging in non-sedentary activity on Saturdays and the least time on average on Sundays. If we were able to make inferences about the general FitBit user population, perhaps this would mean that more people are staying home on Sundays but going out on Saturdays; further research would be required to answer this question. Note, I haven't been making any inferences about the general FitBit user population with the insights from this data set's analyses because `joined_daily_activity_calories_intensities_steps_analysis.csv` only contains records for 24 users, not the 30 required to satisfy the central limit theorem (CLT). Going forward, though, I will "pretend" that this data set does contain a sufficient sample size, for the sake of generalizing to the Bellabeat user population later on.

#### **...From `daily_sleep_merged_cleaned.csv` Analysis Results**

The distribution of the features in `daily_sleep_merged_cleaned.csv` are best described as follows:

| Feature | Mean | Median | Skew |
| ------- | ---- | ------ | ---- |
| TotalHoursAsleep | 6.9911	| 7.2167 | Negative |
| TotalHoursInBed | 7.6440 | 7.7167	| Negative |
| HoursAsleepHoursInBedDiff | 0.6529 | 0.4167 | Positive |

According to the CDC, adults over the age of 18 should be getting at least 7 hours of sleep; my analysis leads to the inference that more than 50% of FitBit user sleep records meet this recommended amount of sleep. Furthermore, it can be inferred that most FitBit user sleep records will show that the submitting users are able to fall asleep relatively quickly (within an hour), since the average amount of time the participants spent in bed but not sleeping is less than an hour. 

#### **...From `weight_log_info_merged_cleaned.csv` Analysis Results**

The distribution of the features in `weight_log_info_merged_cleaned.csv` are best described as follows:

| Feature | Mean | Median | Skew | 
| ------- | ---- | ------ | ---- |
| WeightPounds | 158.8118 | 137.7889 | Positive |
| BMI | 25.1852 | 24.3900 | Positive |

Because the sample size for the cleaned weight log data is less than 30 users, I wouldn't be justified in using this data to make inferences about the entire FitBit user population. However, if I did have enough of a sample size to satisfy the central limit theorem (CLT), based on the summarizing data, I could infer that more than 50% of weight log records show that their submitting FitBit user is at least overweight based on their BMI (according to the United Kingdom National Health Service, or NHS).

My average BMI analysis for each of the participants appearing in the cleaned weight log data shows the following counts, grouped by average BMI weight class (again, according to the NHS):

| BMI Classification | Number of Participants |
| ------------------ | ---------------------- |
| Underweight | 0 |
| Healthy Weight | 3 |
| Overweight | 5 |
| Obese | 0 |
| Severely/Morbidly Obese | 1 |

Based on these results, it is evident that most of the participants appearing in the cleaned weight log data were at least overweight. Again, the sample size is less than 30, so we cannot apply this observation towards an inference about the general FitBit user population. Going forward, though, I will "pretend" that this data set does contain a sufficient sample size, for the sake of generalizing to the Bellabeat user population later on.

## Share Phase

I decided to use Tableau to create visualizations using the cleaned data sets, particularly the data in the `joined_daily_activity_calories_intensities_steps_analysis.csv`, `weight_log_info_merged_cleaned.csv`, and `daily_sleep_merged_cleaned.csv` files.

The final current Tableau workbook can be found on [my Tableau Public profile](https://public.tableau.com/app/profile/charles.clark4861/vizzes): it contains a dashboard with viusalizations depicting activity levels by weekday, participating userbase separated by BMI weight class, the distribution of amount of time sleeping, and the distribution of the amount of time in bed but not sleeping.

**Note:** The "Avg. Normal distribution approximation" feature in the "Time sleeping" visualization was only used to correctly color the distribution in the histogram. As such, it doesn't really have much analytical value.

## Act Phase

From the insights I extracted from the data during my analysis, I have come up with a few different marketing strategy suggestions for Bellabeat, assuming the trends depicted in the FitBit user data carry over:
- Additional research should be performed to obtain and analyze weight data from a large enough sample so the Central Limit Theorem (CLT) can be applied in asserting the insights captured in this analysis. If this further analysis confirms that the trends depicted in the FitBit user data, i.e., that half or more of the device userbase is at least overweight, then Bellabeat should focus on advertising to populations who meet this condition and want to work towards a healthier weight. Furthermore, the development of new products specifically aimed to help these populations track their progress and continue to lose weight would be benficial considering the size of Bellabeat's userbase that would find use in these products.
- Additional research should be done into why the userbase seems to be more active on Saturdays and least active on Sundays, and also if these trends carry over to populations outside the userbase. If this research indicates that non-users are more active on Saturdays as well because they are going out to enjoy time off from weekday-based jobs, perhaps it would benefit Bellabeat to consider increased funding towards radio, music streaming app, and electronic billboard advertising on Saturdays. Furthermore, if this research indicates that non-users are also least active on Sundays because they are relaxing at home before the start of a new workweek, it might be beneficial to allocate more funding towards television and streaming app (like Amazon Prime, Hulu, etc.) advertising on this day. 
- It could also be beneficial to apply the insights derived from the FitBit user sleep data torwards Bellabeat's advertising. While I can't logically say that owning a fitness device alone will lead to better sleeping habits, it would certainly be fair to suggest that such devices allow users a better understanding of their sleeping habits. This could be especially useful for people who are trying to fix their sleep schedules. In addition, it is also possible that the activities that fitness device users engage in to lose weight or stay fit themselves lead to better sleeping habits: one study found that consitent exercise can lead to improved sleep quality, even in those dealing with sleeping disorders such as insomnia. Therefore, Bellabeat can leverage the notion that engaging in fitness activities using its products can lead to better sleeping habits and quality in such a way that makes the non-user population (especially the non-user population that struggles with healthy sleeping habits) want to purchase its products.

## References
1. [The United Kingdom National Health Service (NHS), "Body mass index (BMI)"](https://www.nhsinform.scot/healthy-living/food-and-nutrition/healthy-eating-and-weight-loss/body-mass-index-bmi)
2. [The Centers for Disease Control and Prevention (CDC), "Lifestyle Coach Facilitation Guide: Post-Core"](https://www.cdc.gov/diabetes/prevention/pdf/postcurriculum_session8.pdf)
3. [The Centers for Disease Control and Prevention (CDC), "How Much Sleep Do I Need?"](https://www.cdc.gov/sleep/about_sleep/how_much_sleep.html)
4. [M.A. Alnawwar et al., "The Effect of Physical Activity on Sleep Quality and Sleep Disorder: A Systematic Review," Aug. 2023](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10503965/#:~:text=Studies%20have%20shown%20that%20regular,did%20not%20exercise%20%5B19%5D.)




