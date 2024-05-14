# Project SQL: Deep Exploration of Global Causes of Death (1990–2019)


### Goal of this project:
The primary goal was to identify and analyze the leading causes of death globally between 1990-2019. Specifically, I aimed to explore patterns and trends related to cardiovascular diseases, drug and alcohol-related deaths, and suicide rates. Additionally, I sought to understand the disparities between countries and regions, considering factors such as culture, politics, and economic conditions.

1) How do global mortality trends vary across different regions and countries, particularly concerning causes such as cardiovascular diseases, drug and alcohol-related deaths, and suicides?
2) What factors contribute to the disparities in mortality rates observed worldwide, and how do cultural, economic, and political differences influence these trends?
3) Are there identifiable patterns or correlations between socioeconomic indicators, such as income levels or education, and mortality rates related to specific causes of death?
4) How effective are existing public health policies and interventions in addressing mortality trends associated with alcohol and drug use, and what areas require further attention or improvement?

### Data source:
[Dataset location](https://ourworldindata.org/grapher/annual-number-of-deaths-by-cause)
The downloaded csv should be named "annual-number-of-deaths-by-cause". The dataset included information on countries,years, and the number of deaths per cause. Additional information is included on income levels, and larger regions.

### Tools Used:
1. Microsoft Excel
3. Microsoft SQL Server
   - [Download](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16)

- First step will be to open the data set Microsoft Excel for easy viewing. It also allows me to see opportunities to transform and clean the data.

- Next step will be to upload the csv file into Microsoft SQL Server

### Data Transformation:
Change Table name to Cdeath from annual-number-of-deaths-by-cause: I did this to help keep my table name short and for ease of use.
So lets take a look at the data:
```sql
Select *
From Cdeath
```



