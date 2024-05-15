# Project SQL: Deep Exploration of Global Causes of Death (1990–2019)
---

## Table of Contents

- [Goal of this Project](#goal-of-this-project)
- [Data source](#data-source)
- [Tools used](#tools-used)
- [Data Transformation](#data-transformation)
- [Analysis](#analysis)
- [What did I learn](#what-did-i-learn)
- [Conclusion](#conclusion)

### Goal of this project:
---
The primary goal was to identify and analyze the leading causes of death globally between 1990-2019. Specifically, I aimed to explore patterns and trends related to cardiovascular diseases, drug and alcohol-related deaths, and suicide rates. Additionally, I sought to understand the disparities between countries and regions, considering factors such as culture, politics, and economic conditions.

1) How do global mortality trends vary across different regions and countries, particularly concerning causes such as cardiovascular diseases, drug and alcohol-related deaths, and suicides?
2) What factors contribute to the disparities in mortality rates observed worldwide, and how do cultural, economic, and political differences influence these trends?
3) Are there identifiable patterns or correlations between socioeconomic indicators, such as income levels or education, and mortality rates related to specific causes of death?
4) How effective are existing public health policies and interventions in addressing mortality trends associated with alcohol and drug use, and what areas require further attention or improvement?

### Data source:
---
[Dataset location](https://ourworldindata.org/grapher/annual-number-of-deaths-by-cause)
The downloaded csv should be named "annual-number-of-deaths-by-cause". The dataset included information on countries,years, and the number of deaths per cause. Additional information is included on income levels, and larger regions.

### Tools Used:
---
1. Microsoft Excel
3. Microsoft SQL Server
   - [Download](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver16)

- First step will be to open the data set Microsoft Excel for easy viewing. It also allows me to see opportunities to transform and clean the data.
![Excel sheet](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/ce34da0d-b4f1-40b3-b95d-01d7af84c7ed)
- Next step will be to upload the csv file into Microsoft SQL Server


### Data Transformation:
---
Change Table name to Cdeath from "annual-number-of-deaths-by-cause": I did this to help keep my table name short and for ease of use.
So lets take a look at the data:
```sql
Select *
From Cdeath
```
![Startup layout](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/b5be95c2-cde9-4b80-a659-0eaded946a65)

I wasn't a fan of this layout. The first step was changing the "entity" column to "Country". In Microsoft SQL server this can be accomplished using the stored procedure function.
```sql
EXEC sp_rename 'Cdeath.Entity', 'Country', 'column'
```

Another way I am organizing my data is by grouping each cause of death into one column. I accomplish this using Unpivot and a Subquery function. Doing this allows me to perform aggregate functions on my data. This Unpivot will also provide us with the sums for each cause of death by the year and country. We will use this shortly to apply analysis to the data.
```sql
SELECT Country, Year, Causes, Total_deaths
FROM
(
SELECT Country,Year,
Meningitis,AlzheimerDisease,ParkinsonDisease,Nutritional,Malaria,Drowning,Interpersonalviolence,MaternalDisorder,HIV_AIDS,DrugUse,Tuberculosis,
CardiovascularDiseases,RespiratoryInf,NeonatalDisorder,AlcoholUseDisorders,Selfharm,Expnature,DiarrhealDiseases,
HeatColdExposure,Neoplasms,Conflict_terrorism,Diabetes_mellitus,KidneyDisease,Poison,Malnutrition,Road_injuries,Chronic_Respiratory_Diseases,
Chronic_Liver_Diseases,Digestive_Diseases,Fire_heat_hot_subs,Acute_Hepatitis,Measles
     
FROM Cdeath
) AS maint

UNPIVOT

(total_deaths for causes in
(Meningitis,AlzheimerDisease,ParkinsonDisease,Nutritional,Malaria,Drowning,Interpersonalviolence,MaternalDisorder,HIV_AIDS,DrugUse,Tuberculosis,
CardiovascularDiseases,RespiratoryInf,NeonatalDisorder,AlcoholUseDisorders,Selfharm,Expnature,DiarrhealDiseases,
HeatColdExposure,Neoplasms,Conflict_terrorism,Diabetes_mellitus,KidneyDisease,Poison,Malnutrition,Road_injuries,Chronic_Respiratory_Diseases,
Chronic_Liver_Diseases,Digestive_Diseases,Fire_heat_hot_subs,Acute_Hepatitis,Measles)
) as unpivottable
```
![Unpivot view](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/25b4a54c-3e5c-4b9a-9dab-bb4571db2361)

Now I would like to start performing analysis on this view provided from the last query. This can be accomplished by creating a temporary table with the new data layout. Remember our last unpivot function didn't edit the layout of our original dataset, it just provides us with a temporary view of the dataset. However, for the analysis we want to accomplish it would be cumbersome to write out the long unpivot function each time.

This is where temporary tables are useful.
Here are some reasons why you should use temporary tables:

- Working with large amounts of Data
- I want to perform multiple analysis using this specific view of the data
- Quick referencing for table views

```sql
---Creating temp table
Create table #Cdeath1
(Country varchar(255), Year nvarchar(255), Causes varchar(255), Total_deaths bigint)

---Inserting previous data view query into temp table
Insert into #Cdeath1

SELECT Country, Year, Causes, Total_deaths
FROM
(
SELECT Country,Year,
Meningitis,AlzheimerDisease,ParkinsonDisease,Nutritional,Malaria,Drowning,Interpersonalviolence,MaternalDisorder,HIV_AIDS,DrugUse,Tuberculosis,
CardiovascularDiseases,RespiratoryInf,NeonatalDisorder,AlcoholUseDisorders,Selfharm,Expnature,DiarrhealDiseases,
HeatColdExposure,Neoplasms,Conflict_terrorism,Diabetes_mellitus,KidneyDisease,Poison,Malnutrition,Road_injuries,Chronic_Respiratory_Diseases,
Chronic_Liver_Diseases,Digestive_Diseases,Fire_heat_hot_subs,Acute_Hepatitis,Measles
     
FROM Cdeath
) AS maint

UNPIVOT

(total_deaths for causes in
(Meningitis,AlzheimerDisease,ParkinsonDisease,Nutritional,Malaria,Drowning,Interpersonalviolence,MaternalDisorder,HIV_AIDS,DrugUse,Tuberculosis,
CardiovascularDiseases,RespiratoryInf,NeonatalDisorder,AlcoholUseDisorders,Selfharm,Expnature,DiarrhealDiseases,
HeatColdExposure,Neoplasms,Conflict_terrorism,Diabetes_mellitus,KidneyDisease,Poison,Malnutrition,Road_injuries,Chronic_Respiratory_Diseases,
Chronic_Liver_Diseases,Digestive_Diseases,Fire_heat_hot_subs,Acute_Hepatitis,Measles)
) as unpivottable
```

As you can see I named my temporary table #Cdeath1. The first step will be to test our temp table.
```sql
Select * 
From #Cdeath1
```
If created successfully, you will have a saved temp table that we can apply multiple functions for our analysis.

### Analysis
---
Let's get an overview of the data by using the distinct function we can see all of our countries in one list.
```sql
Select 
distinct(Country)
From
#Cdeath1
where country like '%United%states%' ---- Filter by regions
where country like '%Income%' ---- Filter by certain income
```
Next, we can check the max, min, sums, and average cause of deaths. Using the Top 20 syntax to provide us with the top 20 results.
```sql
Select Top 20
Causes,
Sum(Total_deaths) as TotalNumberDeaths,
Max(Total_deaths) as MaxDeaths,
avg(Total_deaths) as AvgNumberofdeaths
From
#Cdeath1
Group by Causes
Order by TotalNumberDeaths desc
```

Here we can see that the highest cause of death experienced in the world between 1990 and 2019 was Cardiovascular disease. Followed by Chronic Respiratory Diseases, Respiratory Infections, Neonatal Disorder, Digestive Diseases, Diarrheal Diseases, Tuberculosis, Chronic Liver Diseases and Road injuries.

- Let's look at the total number of deaths by cause for each country separately.
```sql
Select 
Country, Causes, SUM(Total_deaths) AS TotalNumberDeaths
from #Cdeath1
group by Country, Causes
order by TotalNumberDeaths desc

--- Lets add filters by regions
Select 
Country, Causes, SUM(Total_deaths) AS TotalNumberDeaths
from #Cdeath1
Where Country in ('Nigeria', 'India', 'Kenya', 'Bangladesh', 'Pakistan') 
group by Country, Causes
order by Country, TotalNumberDeaths desc
```

Here I can see that the data in most cases is consistent that Cardiovascular diseases, have resulted in the highest number of deaths for most countries with only a few outliers. By adding the Where function, we can filter the total number of deaths for certain countries.

I wanted to get a better idea of these outliers. What causes of death are prevalent in some countries but not in others? Thus, my goal here was to gauge which regions of the world are least impacted by certain causes of death eg/Cardiovascular diseases. This information could be used later to identify if a country is at risk of certain causes or the rate has risen, where it previously wasnt a big issue.

 - For easier viewing, I thought it would be great to rank each cause of death for each country.
   
```sql
--- 1)Create a query to rank my causes of death by country. 
Select Country, Causes, Sum(Total_deaths) as TotalNumberDeaths,
Rank() Over(partition by country order by sum(total_deaths) desc) AS RankNumber
from #cdeath1
group by country, causes
```
- So what regions do not have Cardiovascular disease in their top 5 causes of death?

I can use a CTE (Common table expression). This allows me to apply a one-time filter on the Ranked results.

```sql
WITH Ranked_causes as 
(
Select Country, Causes, Sum(Total_deaths) as TotalNumberDeaths,
Rank() Over(partition by country order by sum(total_deaths) desc) as RankNumber
from #cdeath1
group by country, causes
)

SELECT 
    Country,
    Causes,
 Ranknumber
FROM Ranked_Causes
WHERE Country NOT IN (
    SELECT Country
    FROM Ranked_Causes
    WHERE Ranknumber < 5 AND Causes = 'CardiovascularDiseases' 
)
ORDER BY Country, TotalNumberDeaths DESC;
```
This data shows that African countries have suffered less than other countries as it relates to cardiovascular diseases. But it doesn't tell the full story. The data also suggests that several other causes of death hold priority such as Malaria, HIV/AIDS, and Diarrheal diseases.

When exploring the various causes of death per country, I became interested in 3 categories:
1) Alcohol use
2) Drug use
3) Self-harm/Suicide

My first goal here was to find out which specific countries have a higher death rate in relation to alcohol, drug use, and self-harm/suicidal deaths.

To accomplish this, I can use the ranked causes list I previously created with CTE. However, I will apply a filter to these results using my 3 categories. Below you can see how I can add these filters to find countries with higher/lower rankings. Remove --- to activate a specific filter.

```sql
WITH Ranked_causes as 
(
Select 
Country, Causes, Sum(Total_deaths) as TotalNumberDeaths,
Rank() Over(partition by country order by sum(total_deaths) desc) AS RankNumber
from #cdeath1
group by country, causes
)

Select Country, Causes, TotalNumberDeaths,RankNumber
from Ranked_causes
---Where RankNumber <= 10 and Causes LIKE '%alcohol%'
---Where RankNumber <= 15 and Causes like '%drug%'
---Where RankNumber <= 10 and Causes LIKE '%Selfharm%' 
---and Country in ('United Kingdom', 'European Region (WHO)','Scotland','England') 
order by TotalNumberDeaths desc
```


1) Alcohol use related deaths (World Ranking)
```sql
Where RankNumber <= 10 and Causes LIKE '%alcohol%
```
![Alcohol](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/41d7fa80-44f5-42f1-a795-07695176f73e)

This allows us to see which countries have alcohol related deaths between 1 -10.
The lower the rank the higher proportion of overall deaths per country. 

- This gave me the opportunity to dig deeper. Why were these countries dealing with high rates of alcohol use related deaths?

A pattern was emerging. I found here that there was a high prevalence of Alcohol use related deaths in European countries. For example, Ukraine ranks 8 for all overall deaths between 1990–2019. As a comparison, Alcohol use disorder deaths rank as 25 for all countries during this timeframe. [This is also in support of literature which states that European countries rank amongst the highest alcohol drinkers in the world in comparison to other countries in the world.](https://www.euronews.com/health/2023/06/30/so-long-dry-january-which-country-drinks-the-most-alcohol-in-europe) Further contributing to the results and the elevated numbers.

2) Drug use related deaths (World Ranking)
```sql
Where RankNumber <= 15 and Causes LIKE '%drug%'
```
![Drug](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/0039a4a0-ef50-48ef-8160-6ca61899e27a)

This allows us to see which countries have drug use related deaths ranked between 1–15.

With these initial findings, It did not surprise me to the USA high on the list. However, it did surprise me to see countries within the United Kingdom with a higher ranking. One in particular is Scotland. [When gathering more data information on this, it turns out that Scotland has one of the worst drug problems in Europe. But why is this the case? According to reports, one of the biggest attributes was poverty, deprivation, and trauma.](https://www.sdf.org.uk/blog-poverty-is-the-root-of-scotlands-fatal-drug-overdose-crisis/) To further delve into this information specific drugs used could be investigated, helping to implement policies that protect others from future harm. Additionally, strategies that help the population strive out of poverty could be helpful.

3) Self Harm/Suicidal related deaths (World Ranking)
   ```sql
   Where RankNumber <= 10 and Causes LIKE '%Selfharm%'
   ```
  ![self harm top 10](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/7fbaefae-3cb8-4c9e-b665-79d2dfd2efc9)

This allows us to see which countries have self harm/suicidal related deaths ranked between 1–10.

With these results I was surprised to see Sri Lanka so high on the list, thus, I needed to delve further. [Upon my research, large contributors to the suicide rates within the country were the result of pesticide self-poisoning.](https://centrepsp.org/media/news/sri-lankan-suicide-rate-stable-during-pandemic)

But why is this the case? 
Like many low-income countries, Suicide rates are often the result of societal issues such as poverty and unemployment, leading to an increase in depression/anxiety which can fuel suicidal thoughts. Rural farming communities also increase overall access to lethal pesticides. In the same article, reports suggest that there was not an increase in suicidal deaths during the pandemic. While this data set only contains results up to 2019, we are somewhat limited.

4) What countries have a lower ranking of suicidal deaths?
```sql
Where RankNumber >=11 and Causes LIKE '%Selfharm%'
```
![Lower rank self harm](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/bf8ccda5-5a9f-4940-a6c3-fd284f3941ce)

This allows us to see which countries have a lower prevalence of suicidal deaths in proportion to deaths within that region. In this case, all rankings were all higher than 11.

By altering the query to look at higher rankings we are able to see which regions have lower reported suicide deaths. [Amongst this group were the Middle east/North Africa. Research seems to support this may be due to cultural factors such as the religious practice of Islam which holds strong governance on individuals' lives.](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC9178353/) Further emphasizing how culture and religion may impact mortality rates in different regions.

- But what if we try to gain an idea of how these rates have progressed over time?
Let's take a look at Sri Lanka's suicidal rates from 1990 and onwards
```sql
Select 
*
From
#Cdeath1
Where country like '%Sri%Lanka%' and Causes LIKE '%selfharm%'
```
![sri lanka](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/23263505-d49e-4705-b27a-ae6527e35185)

Here we can see how these rates have fluctuated throughout the years. As you can see there was a steep increase in suicidal deaths in the 1990s but since then, rates have decreased.

- But what influenced this decrease? This made me think about the impact of policies within the country and whether they may have an effect.
  
In the 1990s, Sri Lanka implemented policies that restricted the availability of toxic pesticides within the country which could suggest the reason for the decrease in suicidal rates in the years 2000 and onwards.

But this doesn't tell the full story.

- To gain a bigger picture let's look at this progression in neighboring countries such as India
  
![India self harm](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/4dfc1b1b-8191-484f-b958-ce51dc2c402c)

In comparison to Sri Lanka, India's suicide death rate has steadily increased, boasting one of the highest rates in the world. Like Sri Lanka, economic instability, poverty, and unemployment can contribute to heightened stress levels and mental health issues, increasing the risk of suicide. To address these issues, mental health access is essential in getting people the help they need. [However, according to my research, access to mental health care is sparse, with over 75% of the population with mental health disorders not receiving treatment.](https://speakingofmedicine.plos.org/2023/05/25/mind-matters-indias-mental-health-budget-crisis/)

- But why is this the case?
The National Mental Health Programme (NMHP) was implemented in 1982 to integrate mental health care with primary healthcare services. However, despite increases in funding to address treatment gaps, overall utilization of mental health services has been low. This means that today, less than 1% of the country's total budget for health is allocated to mental health. Considerable, low despite the increase in suicidal rates. Cultural factors between the 2 countries may also prevent individuals from seeking help or discussing their struggles openly, increasing the overall risk.

- Another factor to consider is policy differences.
As stated, Sri Lanka implemented the banning of toxic pesticides to help curb the suicidal rates. However, as of 2019, India did not have a specific suicide prevention policy. It was only in 2022, that India implemented their first National Suicide Prevention Strategy (NSPS). Sadly, funding allocations continue to be low, making it challenging to push the initiatives.

- Another question to answer was how these causes of death compare with other countries in the world.
This can be accomplished by finding the global average number of deaths and comparing it to what is occurring in a particular country. By doing this, we are not only able to see which countries are fairing better but also where countries may be struggling, and the need for policy change.

```sql
--- To start lets get an idea of the global average for each cause of death
--- 1) Create a CTE with too gather global averages
WITH CTE_Global_average as
(
Select Causes,
AVG(total_deaths) as Globalavgdeaths
FROM #Cdeath1 c
group by Causes
)

Select 
c.Country,c.Causes,Sum(c.Total_deaths) as Total_deaths,
g.Globalavgdeaths,SUM(c.Total_Deaths) - g.Globalavgdeaths AS DifferenceFromGlobalAvg
FROM #Cdeath1 c

--- 2)Join the Global average CTE to our main table (Cdeath)
JOIN CTE_Global_average g ON c.Causes = g.Causes
---where C.Causes like '%alcohol%' 
---where C.Causes like '%Selfharm%'
---where C.Causes like '%drug%'
Group by c.Country,c.Causes,g.Globalavgdeaths
order by DifferencefromGlobalavg DESC
```
Using this script we can choose what cause of death we want to investigate with our global averages.
1) Alcohol use related deaths (Global averages)
   
![alcohol global](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/48fb0d2a-f278-4680-b256-1f4172c3707a)
![alcohol global 1](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/7c5b0f3b-d852-4ba2-945b-de183e4a1c19)

When looking at Alcohol associated deaths, countries such as Ukraine were higher on the list which aligns with my prior research. However, I was surprised to see countries such as Nigeria so high compared to our global average.

- But why is this the case? 
[In a study, I found that the prevalence of alcohol related deaths may be attributed to its high population but also to policies that permit the consumption of illicit and locally made alcoholic beverages sold freely within the country. As a result, policy changes have been recommended such as tax increases on alcohol and reduction of advertisements.](https://bmcpublichealth.biomedcentral.com/articles/10.1186/s12889-019-7139-9) However, the implementation of these policies has been challenging in Nigeria. Culturally, alcohol is seen as a celebratory drink and promoted as such. Additionally, other health challenges often take precedence, which also contributes to lower allocation of funds used on alcohol policy implementation.

2) Drug use related deaths (Global averages)
   
![drug global](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/181916e2-6ea2-4d53-b717-2781f5bae560)

Here I thought it would be interesting to explore drug use related deaths in Iran, which are disproportionately higher than the global average. With this information, I was able to see that drug related deaths are higher in Iran in comparison to larger continental regions such as Latin America and the Caribbean during the same period (1990–2019).

But I had more questions, as I previously stated culture and religion can often influence the way individuals govern their lives. The Middle East historically has lower numbers of suicidal deaths than other countries, but I wonder if there is a connection. One may surmise that due to cultural/religious pressures, drug use increases.

Geographically, Iran borders Pakistan and Afghanistan that also have rates higher than the global average. It is also this proximity to Afghanistan that is seen to be one of the drivers of the opioid crisis in Iran. Afghanistan is the largest producer of opium in the world. Resulting in Iran being a hot spot for drug trafficking.

 - So what's being done about this?
[Politically, Iran has placed strict laws on drug crimes, resulting in life imprisonment or death if caught with 30 grams of Heroin, Morphine, Cocaine, LSD, Methamphetamine, and other similar drugs.](https://www.hrw.org/news/2011/08/05/dont-praise-irans-war-drugs)Whilst lower amounts of funding are placed on harm reduction actions such as needle exchange programs and methadone treatment. This lack of funding could also be a contributor to the number we see today. The intersection of culture and politics in Iran shapes its approach to drug use. While cultural norms discourage substance abuse, strict political laws reflect a zero-tolerance stance. This divergence highlights the challenge of addressing drug-related issues, emphasizing the need to harmonize cultural beliefs with effective public health interventions.

3) Self Harm/Suicidal related deaths (Global averages)
   
 ![self harm global](https://github.com/Trav161/World_Cause_of_Death/assets/169755322/760367f4-d6ae-4598-bbfb-290baffd38ab)
  
Here I wanted to explore Japan's suicidal rates being one of the smallest countries on this list. According to my research, Japan have historically [honored the act of suicide](https://www.nippon.com/en/japan-topics/g02268/) in certain circumstances. [Additionally, there have been rising suicide rates among office workers and employees attributed to increased job pressures, longer work hours, and fewer holidays and sick days. This phenomenon, known as "karoshi" or "death by overwork," is prevalent in Japanese society. Alongside physical strain, mental stress from work can lead to "karojisatsu," or "overwork suicide."](https://www.wired.com/story/karoshi-japan-overwork-culture/)

- But what is being done to decrease the rate? 
Japan has implemented 2 policies to combat this issue. [The General Principles of Suicide Prevention Policy (GPSPP) focuses on holistic approaches to suicide prevention, encompassing mental health support, public awareness campaigns, and community engagement. In contrast, the Jisatsu Taisaku Kihonn Hou, or suicide prevention law in Japan, is a legislative measure aimed at promoting early detection, intervention, and support for individuals at risk of suicide, emphasizing collaboration between healthcare, education, and government sectors. Since their implementation in 2006, suicide rates within the country have reduced. Looking at the progress made with this policy could promote the implementation of similar renditions in other countries at increasing rates.](https://www.cambridge.org/core/journals/bjpsych-open/article/impact-of-the-japanese-governments-general-principles-of-suicide-prevention-policy-on-youth-suicide-from-2007-to-2022/316001A33A9A58E288369FD7C87A66AB)

### What did I learn:
---
Through this comprehensive analysis of global mortality trends, several key insights have emerged:

1) Complexity of Mortality Dynamics: Mortality patterns are influenced by a myriad of factors, including socio-economic conditions, cultural norms, and healthcare systems. Understanding these complexities is crucial for devising effective interventions.
   
2) Regional Disparities: Disparities in causes of death exist not only between countries but also within regions. Cultural norms, policy frameworks, and access to healthcare play pivotal roles in shaping these disparities.

3) Impact of Policy Interventions: Policy interventions can significantly impact mortality rates. Sri Lanka's success in reducing suicide rates post-pesticide regulation highlights the efficacy of targeted policy measures in addressing specific causes of death. 
   
4) Need for Holistic Approaches: Addressing complex mortality trends requires holistic approaches that integrate socio-economic, cultural, and healthcare dimensions. Japan's multifaceted suicide prevention strategies exemplify the effectiveness of comprehensive interventions.

5) Importance of Data Analysis: Data analysis, facilitated through SQL queries, unveils hidden trends and patterns in mortality data. The ability to manipulate and analyze large datasets empowers researchers to derive meaningful insights and inform evidence-based policy decisions.

### Conclusion:
---
In conclusion, this journey through global mortality trends has presented the intricate interplay of factors influencing causes of death worldwide. While progress has been made in understanding and addressing mortality disparities, persistent challenges remain. By harnessing the power of data analysis and adopting holistic approaches to public health, we can strive towards a future where preventable deaths are minimized, and well-being is prioritized across all regions and communities.

### Contribute 
- Question to the Reader: What strategies do you envision to address the underlying socio-economic and cultural determinants influencing causes of death across diverse regions?
- Feel free to use to the code to utilize different skills in SQL while learning more about health.
















  





