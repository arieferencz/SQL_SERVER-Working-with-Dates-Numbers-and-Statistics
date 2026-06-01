# SQL Server — Working with Dates, Numbers and Statistics

This repository contains T-SQL exercises using Microsoft's **AdventureWorks2022** database, organised into 2 topic folders.

Each exercise includes a question, one or more solutions, the T-SQL code, the query output, and a step-by-step explanation.

---

## 📋 Table of contents

- [Folder 1 — Dates](#folder-1--dates)
- [Folder 2 — Numbers and statistics](#folder-2--numbers-and-statistics)

---

## 📁 Folder 1 — Dates

| Exercise | Description |
|---|---|
| [Adding and subtracting days, months and years to a date](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Adding%20and%20Subtracting%20days%2C%20months%20and%20years%20to%20a%20date.md) | Uses DATEADD to perform arithmetic on date values |
| [Calculate first day, last day and second to last day of current month](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Calculate%20first%20day%2C%20last%20day%20and%20second%20to%20last%20day%20of%20current%20month.md) | Derives key boundary dates of the current month dynamically |
| [Counting the number of business days between 2 dates](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Counting%20the%20number%20of%20business%20days%20between%202%20dates.md) | Calculates working days between two dates excluding weekends |
| [Counting the number of days between 2 dates](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Counting%20the%20number%20of%20days%20between%202%20dates.md) | Uses DATEDIFF to count calendar days between two dates |
| [Counting the number of days between hiring dates on same column](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Counting%20the%20number%20of%20days%20between%20hiring%20dates%20on%20same%20column.md) | Uses LAG to calculate gaps between consecutive hire dates |
| [Counting the number of days in a year](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Counting%20the%20number%20of%20days%20in%20a%20year.md) | Determines total days in a year accounting for leap years |
| [Counting the number of employees hired every month of every year](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Counting%20the%20number%20of%20employees%20hired%20every%20month%20of%20every%20year.md) | Groups hire dates by month and year to show hiring trends |
| [Counting the number of occurrences for every weekday in a year](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Counting%20the%20number%20of%20occurrences%20for%20every%20weekday%20in%20a%20year.md) | Counts how many times each weekday appears in a given year |
| [Creating a calendar for current month](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Creating%20a%20calendar%20for%20current%20month) | Generates a full calendar grid for the current month using T-SQL |
| [Retrieving the dates of a specific weekday in a year](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Retrieving%20the%20dates%20of%20a%20specific%20weekday%20in%20a%20year) | Lists every occurrence of a chosen weekday throughout a year |
| [Retrieving the dates of first and last occurrences for a specific weekday in a month](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Retrieving%20the%20dates%20of%20first%20and%20last%20occurrences%20for%20a%20specific%20weekday%20in%20a%20month) | Finds the first and last occurrence of a weekday within a month |
| [Retrieving the start and end dates for a specific quarter](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Retrieving%20the%20start%20and%20end%20dates%20for%20a%20specific%20quarter) | Calculates the first and last day of any given quarter |
| [Retrieving the start and end dates for all 4 quarters in a year](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Retrieving%20start%20and%20end%20dates%20for%20all%204%20quarters%20in%20a%20year) | Generates start and end dates for all four quarters dynamically |
| [Retrieving the lists of employees hired on the same date](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/SQL_SERVER-Dates/Retrieving%20the%20lists%20of%20employees%20that%20were%20hired%20on%20the%20same%20date%20(min.%205%20employees)) | Finds groups of 5 or more employees sharing the same hire date |

---

## 📁 Folder 2 — Numbers and statistics

| Exercise | Description |
|---|---|
| [Average calculations](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Average%20calculations) | Calculates averages using AVG, SUM/COUNT, and GROUP BY ROLLUP |
| [Creating horizontal histograms](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Creating%20horizontal%20histograms) | Builds text-based horizontal histograms using REPLICATE |
| [Creating vertical histograms](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Creating%20vertical%20histograms) | Builds text-based vertical histograms using T-SQL |
| [Highest and lowest dollar value sales orders calculations](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Highest%20and%20lowest%20dollar%20value%20sales%20orders%20calculations) | Retrieves top and bottom sales orders by total value |
| [Median calculations using PERCENTILE_DISC and PERCENTILE_CONT](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Median%20calculations%20using%20PERCENTILE_DISC()%20and%20PERCENTILE_CONT()) | Compares discrete and continuous median calculations |
| [Mode calculations using DENSE_RANK](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Mode%20calculations%20using%20DENSE_RANK()) | Identifies the most frequently occurring value using DENSE_RANK |
| [Percentile calculations using PERCENT_RANK and CUME_DIST](https://github.com/arieferencz/SQL_SERVER-
