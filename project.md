### 1. What are the total number of countries involved in deforestation? 

#### __SOLUTION__: 

- USING SUBQUERIES

```sql
SELECT TOP 10 country_code AS COUNTRIES_INVOLVED 
FROM
(SELECT DISTINCT country_code 
FROM forest_area
WHERE forest_area_sqkm IS NOT NULL
) AS COUNTRY_COUNT;

 -- CTE

WITH COUNTRY_COUNT 
AS
(SELECT DISTINCT country_code 
FROM forest_area
WHERE forest_area_sqkm IS NOT NULL
) 
SELECT TOP 10 country_code AS countries_involved
FROM COUNTRY_COUNT ;
```

#### __RESULT__

| countries_involved |
| ------ |
| Aruba  |
| Afghanistan  |
|Albania  |
Andorra  |
| United Arab Emirates  |
| Argentina |
| Armenia  |
| American Samoa  |
| Antigua and Barbuda |

#### __INSIGHTS__

There are a total of 208 countris involved in deforestation.

### 2.  Show the income groups of countries having total area ranging from 75,000 to 150,000 square meter?

#### __SOLUTION__: 

```sql
SELECT TOP 10
R.COUNTRY_NAME, ROUND(TOTAL_AREA_SQ_MI,1) AS SELECTED_AREA,  INCOME_GROUP 
FROM LAND_AREA LA 
JOIN FOREST_AREA FA ON LA.COUNTRY_CODE = FA.COUNTRY_CODE
JOIN REGIONS R ON LA.COUNTRY_CODE = R.COUNTRY_CODE
WHERE TOTAL_AREA_SQ_MI BETWEEN 75000 AND 150000 
GROUP BY  R.COUNTRY_NAME, INCOME_GROUP, TOTAL_AREA_SQ_MI ;
```
#### __RESULT__

| COUNTRY_NAME	| SELECTED AREA	 | INCOME_GROUP |
|--------------|---------------|-------------|
|Burkina Faso	| 105637.1	| Low income|
|Belarus	| 78373.8	| Upper middle income |
|Belarus	| 78369.9	| Upper middle income |
|Belarus	| 78368	| Upper middle income |
|Belarus	| 78342.9 | Upper middle income |
|Belarus	| 78343.6	| Upper middle income |
|Belarus	| 78316.6	| Upper middle income |
|Belarus	| 78305	| Upper middle income |
|Belarus	| 78312.7	| Upper middle income |
|Belarus	| 78339.8	| Upper middle income |

#### __INSIGHTS__

A total of 10 countries have forest area between 75000 and 150000 amd majority of the income group belong to upper middle income and low income.

### 3. Calculate average area in square miles for countries in the 'upper middle income region'. Compare the result with the rest of the income categories.

#### __SOLUTION__:

```sql
SELECT INCOME_GROUP, ROUND(AVG(Total_area_sq_mi), 0) as Avg_TA_Sq_mi
FROM LAND_AREA LA 
JOIN  REGIONS R ON LA.COUNTRY_CODE = R.COUNTRY_CODE
WHERE INCOME_GROUP = 'upper middle income'
GROUP BY INCOME_GROUP;
```

#### __RESULT__

|INCOME_GROUP |	Avg_TA_Sq_mi |
|-------------|--------------|
|Upper middle income |	383277 |

- Comparing the result to other income group

```sql
SELECT INCOME_GROUP, ROUND(AVG(TOTAL_AREA_SQ_MI),0) AS AVG_AREA
FROM LAND_AREA LA
JOIN REGIONS R ON LA.COUNTRY_CODE = R.COUNTRY_CODE
WHERE INCOME_GROUP <> 'upper middle income'
GROUP BY INCOME_GROUP;
```
 #### __RESULT__
 
|INCOME_GROUP|	Total_FA_sqkm|
|------------|---------------|
Low income	|106410401
Lower middle income	|160393377
UNSPECIFED	|1093577960
Upper middle income |	537631841


#### __INSIGHTS__

The average  area in sq miles is 38,3277 for countries in the upper middle income group while other income groups are significantly higher.

### 4. Determine the total forest area in square km for countries in the 'high income' group. Compare result with the rest of the income categories.

#### __SOLUTION__: 

```sql
SELECT INCOME_GROUP, round(SUM(FOREST_AREA_SQKM), 0) as Total_FA_sqkm FROM FOREST_AREA FA 
JOIN  REGIONS R ON FA.COUNTRY_CODE = R.COUNTRY_CODE
WHERE INCOME_GROUP = 'High income'
GROUP BY INCOME_GROUP;
```
 #### __RESULT__

|INCOME_GROUP |	Total_FA_sqkm|
|-------------|--------------|
|High income	|280145167|


- Comparing the result to other income group

```sql
SELECT INCOME_GROUP, round(SUM(FOREST_AREA_SQKM), 0) AS Total_FA_sqkm
FROM FOREST_AREA FA
JOIN REGIONS R ON FA.COUNTRY_CODE = R.COUNTRY_CODE
WHERE INCOME_GROUP <> 'High income'
GROUP BY INCOME_GROUP;
```
 #### __RESULT__
 
| INCOME_GROUP |	Total_FA_sqkm|
|--------------|---------------|
|Low income	|106410401|
Lower middle income	| 160393377
UNSPECIFED	|1093577960
|Upper middle income|	537631841 |

#### __INSIGHTS__

The average  area in sq miles is 280,145,167 for countries in the High income group is significantly higher than other income groups.


### 5. Show countries from each region(continent) having the highest total forest areas.

#### __SOLUTION__: 

```sql
SELECT * FROM (
SELECT R.COUNTRY_NAME, REGION, round(SUM(FOREST_AREA_SQKM), 0) AS TOTAL_FOREST_AREA, DENSE_RANK() OVER(PARTITION BY REGION ORDER BY round(SUM(FOREST_AREA_SQKM), 0) DESC) AS FRANK
FROM FOREST_AREA FA 
JOIN  REGIONS R ON FA.COUNTRY_CODE = R.COUNTRY_CODE 
GROUP BY R.COUNTRY_NAME, R.REGION
) AS RANKFOREST
WHERE FRANK = 1
ORDER BY TOTAL_FOREST_AREA DESC;

-- USING CTE'S 
WITH RankForest AS (
    SELECT
        R.COUNTRY_NAME,
        R.REGION,
        round(SUM(FOREST_AREA_SQKM), 0) AS TOTAL_FOREST_AREA,
        RANK() OVER (PARTITION BY R.REGION ORDER BY round(SUM(FOREST_AREA_SQKM), 0) DESC) AS FRANK
    FROM
        FOREST_AREA FA
    JOIN
        REGIONS R ON FA.COUNTRY_CODE = R.COUNTRY_CODE
		GROUP BY REGION, R.COUNTRY_NAME)
SELECT *
FROM RankForest
WHERE FRANK = 1
ORDER BY TOTAL_FOREST_AREA DESC;
```

#### __RESULT__

|COUNTRY_NAME |	REGION |	TOTAL_FOREST_AREA |	FRANK|
|-------------|--------|----------------------|------|
World|	World|	1093577960|	1
Russian Federation |	Europe & Central Asia|	218980155|	1
Brazil|	Latin America & Caribbean|	139155605|	1
Canada|	North America|	93866359|	1
China|	East Asia & Pacific|	49948755|	1
Congo, Dem. Rep.|	Sub-Saharan Africa|	42204997|	1
India	|South Asia|	18124909|	1
Iran, Islamic Rep.|	Middle East & North Africa|	2695485	|1

#### __INSIGHTS__

This shows the top countries by forest area in each region .

### 6. What is the trend of forest area change over the years globally?

#### __SOLUTION__: 

```sql
SELECT top 10 year, round(SUM(FOREST_AREA_SQKM), 0) as total_forest_area
FROM forest_area
GROUP BY year
ORDER BY year ;
```
#### __RESULT__

|year |	total_forest_area |
|-----|----------------|
1990|	82016473
1991|	81874076
1992|	81731982
1993|	81736539
1994|	81592850
1995|	81449164
1996|	81305475
1997|	81161786
1998|	81018096
1999|	80874407

#### __INSIGHTS__

### 7. Are there any regions where deforestation is more prevalent among lower-income countries?

#### __SOLUTION__: 

```sql
 SELECT R.region, R.INCOME_GROUP, ROUND(SUM(F.forest_area_sqkm), 0) AS total_forest_area
FROM regions R
JOIN forest_area F ON R.country_code = F.country_code
WHERE INCOME_GROUP IN ('Low income', 'Lower middle income')
GROUP BY R.region, R.income_group
ORDER BY R.region, total_forest_area DESC;
```

#### __RESULT__

|REGION|	INCOME_GROUP|	total_forest_area|
|------|----------------|-------------------|
East Asia & Pacific|	Lower middle income	|62678820
East Asia & Pacific	|Low income|	1769250
Europe & Central Asia|	Lower middle income	|4490954
Europe & Central Asia|	Low income	|110672
Latin America & Caribbean|	Lower middle income	|18627701
Latin America & Caribbean|	Low income|	28727
Middle East & North Africa	|Lower middle income|	1681524
Middle East & North Africa|	Low income	|268330
South Asia	|Lower middle income	|20350215
South Asia	|Low income	|1430975
Sub-Saharan Africa|	Low income	|102802447
Sub-Saharan Africa|	Lower middle income|	52564163

#### __INSIGHTS__

The query showed 12 regions where defrostation was more prevalent among low income countries.  

### 8. How has the total forest area changed over the years for each income group?

#### __SOLUTION__: 

```sql
 SELECT R.income_group, ROUND(SUM(F.forest_area_sqkm), 0) AS total_forest_area
FROM regions R
JOIN forest_area F ON R.country_code = F.country_code
GROUP BY R.income_group
ORDER BY R.income_group desc; 
```

#### __RESULT__

|income_group|	total_forest_area|
|------------|-----------------|
Upper middle income |	537631841
UNSPECIFED| 	1093577960
Lower middle income	| 160393377
Low income	| 106410401
High income	| 280145167

#### __INSIGHTS__

This shows the change in the forest area for each income group, This grouped it into all the availble income groups, for all the years for each group. showing the trend
on how the forest area has changed over the years.

### 9. Bottom 5 countries with their regions by forest area

#### __SOLUTION__: 

```sql
SELECT TOP 5 R.country_name, region, ROUND(forest_area_sqkm,1) AS Forest_area_rounded
FROM regions R
JOIN forest_area F ON R.country_code = F.country_code
WHERE forest_area_sqkm IS NOT NULL
GROUP BY R.country_name, region, ROUND(forest_area_sqkm,1) 
ORDER BY Forest_area_rounded; 
```
#### __RESULT__

|country_name|	region |	Forest_area_rounded|
|----------|-----------|-------------------|
Faroe Islands|	Europe & Central Asia|	0.8
Bahrain	|Middle East & North Africa	|2.2
Greenland|	Europe & Central Asia|	2.2
Bahrain	|Middle East & North Africa|	2.3
Bahrain|	Middle East & North Africa	|2.5

#### __INSIGHTS__

This shows the bottom 5 country and their forest area with faroe islands being the smallest..

Thank you. :smile

































