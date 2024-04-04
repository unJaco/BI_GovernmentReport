---
title: Country Data
---

```sql country_name
SELECT column1 as Country FROM translator WHERE column0 = '${params.country}'
```

Take a clooser look into <Value data={country_name}/>.

# Population Data

```sql population
SELECT TIME_PERIOD, Population FROM population_time WHERE REF_AREA LIKE '${params.country}'
```

```sql extreme_pop_change
WITH PopulationData AS (
    SELECT
        REF_AREA,
        TIME_PERIOD,
        Population,
        FIRST_VALUE(Population) OVER (PARTITION BY REF_AREA ORDER BY TIME_PERIOD) AS FirstPopulation,
        LAST_VALUE(Population) OVER (PARTITION BY REF_AREA ORDER BY TIME_PERIOD RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS LastPopulation
    FROM
        population_time
),
PercentageChanges AS (
    SELECT
        REF_AREA,
        ((LastPopulation - FirstPopulation) / CAST(FirstPopulation AS FLOAT)) * 100 AS PercentageChange
    FROM
        PopulationData
    GROUP BY
        REF_AREA, LastPopulation, FirstPopulation
),
AggregatedChanges AS (
    SELECT
        AVG(PercentageChange) AS AvgPercentageChange,
        MIN(PercentageChange) AS MinPercentageChange,
        MAX(PercentageChange) AS MaxPercentageChange
    FROM
        PercentageChanges
)
SELECT
    PC.REF_AREA,
    PC.PercentageChange,
    AC.AvgPercentageChange,
    AC.MinPercentageChange,
    AC.MaxPercentageChange,
    (PC.PercentageChange - AC.AvgPercentageChange) AS DELTA,
    0 AS INTERVAL
FROM
    PercentageChanges PC,
    AggregatedChanges AC
WHERE PC.REF_AREA = '${params.country}'
```

```sql demography
WITH MeasureValues AS (
    SELECT 'Y_LE4' AS MEASURE, Y_LE4 AS Value FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y5T9', Y5T9 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y10T14', Y10T14 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y15T19', Y15T19 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y20T24', Y20T24 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y25T29', Y25T29 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y30T34', Y30T34 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y35T39', Y35T39 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y40T44', Y40T44 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y45T49', Y45T49 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y50T54', Y50T54 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y55T59', Y55T59 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y60T64', Y60T64 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y65T69', Y65T69 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y70T74', Y70T74 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y75T79', Y75T79 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y80T84', Y80T84 FROM demography WHERE REF_AREA = '${params.country}'
    UNION ALL
    SELECT 'Y_GE85', Y_GE85 FROM demography WHERE REF_AREA = '${params.country}'   
)
SELECT mv.MEASURE, mv.Value
FROM MeasureValues mv
JOIN orderIndex oi ON mv.MEASURE = oi.Column
ORDER BY oi.Number
```

```sql total
SELECT Total_Sum, Percent_Under_20, Percent_Over_65 FROM demography WHERE REF_AREA = '${params.country}'
```

```sql le_time
SELECT TIME_PERIOD, Reference_Area, Life_Expectancy AS VALUE, 'Life Expectancy' AS MEASURE FROM le_time WHERE Reference_Area LIKE '${params.country}'
UNION ALL 
SELECT TIME_PERIOD, Reference_Area, Male AS VALUE, 'Male' AS MEASURE FROM le_time WHERE Reference_Area LIKE '${params.country}'
UNION ALL 
SELECT TIME_PERIOD, Reference_Area, Female AS VALUE, 'Female' AS MEASURE FROM le_time WHERE Reference_Area LIKE '${params.country}'
```


```sql wbs_time
SELECT TIME_PERIOD, REF_AREA, WBS, (WBSu5 / 100) AS WBSu5 FROM wbs_time WHERE REF_AREA LIKE '${params.country}'
```

Take a look into the development of the total population, the demography and the life expectancy.

<Tabs>
    <Tab label="Population Growth">
        <LineChart 
            data={population} 
            x=TIME_PERIOD 
            xFmt="####"
            y=Population
            yFmt=num1m
            yAxisTitle="Total Population"
            yScale=true
        />

        <BoxPlot 
            title='Population Growth since 1980'
            subtitle='compared with the OECD'
            yAxisTitle='Population Change in %'
            data={extreme_pop_change}
            name=REF_AREA
            midpoint=PercentageChange
            confidenceInterval=INTERVAL
            min=MinPercentageChange
            max=MaxPercentageChange
            swapXY=true
        >
            <ReferenceLine y=34.8 label="Average" hideValue=true/>
        </BoxPlot>
    </Tab>
    <Tab label="Demography">
        <FunnelChart 
            title='Demography in 2021'
            subtitle='from young to old'
            data={demography} 
            nameCol=MEASURE
            valueCol=Value
            valueFmt=num1m
            showPercent=true
            legend=false
            labelPosition=right
        />

        <BigValue 
            data={total}
            title='Total Population in 2021'
            value=Total_Sum
        />
        <BigValue 
            data={total}
            title='Percentage of Population under 20'
            value=Percent_Under_20
            fmt=pct1
        />
        <BigValue 
            data={total}
            title='Percentage of Population over 65'
            value=Percent_Over_65
            fmt=pct1
        />
    </Tab>

    <Tab label='Life Expectancy'>
        <LineChart 
            data={le_time} 
            x=TIME_PERIOD 
            xFmt="####"
            y=VALUE 
            series=MEASURE
            yAxisTitle="Life Expectancy in Years"
            yScale=true
        />
    </Tab>

    <Tab label='Well being'>
        <LineChart 
            data={wbs_time} 
            x=TIME_PERIOD 
            xFmt="####"
            y=WBS
            yAxisTitle="Average Well Being Score"
            y2=WBSu5
            y2Fmt=pct1
            y2AxisTitle="Percentage of Population with WBS under 5"
        />
    </Tab>
</Tabs>

<br>

# Economy

```sql mean_wage
SELECT 
    wt.TIME_PERIOD, 
    wt.Mean_Annual_Wages, 
    (wet.Low_Pay / 100) AS 'Low_Pay'
FROM 
    wage_time wt
LEFT JOIN 
    wage_extreme_time wet
ON 
    wt.TIME_PERIOD = wet.TIME_PERIOD AND
    wt.REF_AREA = wet.REF_AREA
WHERE 
    wt.REF_AREA LIKE '${params.country}' OR
    wet.REF_AREA LIKE '${params.country}'
```

```sql first
SELECT * FROM ${mean_wage} WHERE Low_Pay NOT NULL AND Mean_Annual_Wages NOT NULL LIMIT 1
```

```sql last
SELECT * FROM ${mean_wage} WHERE Low_Pay NOT NULL AND Mean_Annual_Wages NOT NULL ORDER BY TIME_PERIOD DESC LIMIT 1
```

```sql delta
SELECT 
    (SELECT Mean_Annual_Wages FROM ${mean_wage} WHERE Low_Pay NOT NULL AND Mean_Annual_Wages NOT NULL LIMIT 1) AS FIRST_W,
    (SELECT Mean_Annual_Wages FROM ${mean_wage} WHERE Low_Pay NOT NULL AND Mean_Annual_Wages NOT NULL ORDER BY TIME_PERIOD DESC LIMIT 1) AS LAST_W,
    (SELECT Low_Pay FROM ${mean_wage} WHERE Low_Pay NOT NULL AND Mean_Annual_Wages NOT NULL LIMIT 1) AS FIRST_L,
    (SELECT Low_Pay FROM ${mean_wage} WHERE Low_Pay NOT NULL AND Mean_Annual_Wages NOT NULL ORDER BY TIME_PERIOD DESC LIMIT 1) AS LAST_L
```

```sql delta_val
SELECT (LAST_W - FIRST_W) AS VAL_W, (LAST_L - FIRST_L) AS VAL_L  FROM ${delta}
```

```sql gdp_time
SELECT TIME_PERIOD, GDP, (GDP_Growth_Rate / 100) AS GDP_Growth_Rate FROM gdp_time WHERE REF_AREA = '${params.country}'
```

<Tabs>
    <Tab label="GDP">
        <LineChart 
            title='Gross Domestic Product'
            subtitle='with Growth Rate'
            data={gdp_time} 
            x=TIME_PERIOD 
            xFmt=####
            xAxisLabels=true
            y=GDP
            yAxisTitle="GDP (PPP converted) in billions"
            y2=GDP_Growth_Rate
            y2SeriesType=line
            y2Fmt=pct1
            y2Scale=false
            y2AxisTitle="GDP Grwoth Rate"
        />
    </Tab>
    <Tab label="Wages">
        <LineChart 
            title='Annunal Average Wage'
            subtitle='with Low Incidence Pay'
            data={mean_wage} 
            x=TIME_PERIOD 
            xFmt=####
            xAxisLabels=true
            showAllLabels=true
            y=Mean_Annual_Wages
            yFmt=num1k
            y2=Low_Pay
            y2SeriesType=line
            y2Fmt=pct1
            y2Scale=false
            yScale=true
        />

        In the time period from <Value data={first} column=TIME_PERIOD fmt=####/> to <Value data={last} column=TIME_PERIOD fmt=####/> the annual average wage increased by <Value data={delta_val} column=VAL_W fmt=usd1k/>.

        In the same period the percentage of people working for less than two-thirds of the gross median earnings of all full-time workers changed by <Value data={delta_val} column=VAL_L fmt=pct1/>.

    </Tab>
</Tabs>

# Trust in the Government


The average percent of people who trust their National Government in the OECD is only **<Value data={trust_avg}/>**. 

Take a look into the trust metrics of <Value data={country_name}/> and compare them with the other OECD countries.

<script>

$: trust_ng_min = Math.min(...grouped_data.filter(item => item.MEASURE === 'Trust in National Government' && item.VALUE !== null).map(item => item.VALUE));
$: trust_ng_max = Math.max(...grouped_data.filter(item => item.MEASURE === 'Trust in National Government' && item.VALUE !== null).map(item => item.VALUE));
$: trust_ng_avg = grouped_data.filter(item => item.MEASURE === 'Trust in National Government' && item.VALUE !== null).reduce((acc, item, _, array) => acc + item.VALUE / array.length, 0);

$: trust_cl_min = Math.min(...grouped_data.filter(item => item.MEASURE === 'Trust in Court and Legal System' && item.VALUE !== null).map(item => item.VALUE));
$: trust_cl_max = Math.max(...grouped_data.filter(item => item.MEASURE === 'Trust in Court and Legal System' && item.VALUE !== null).map(item => item.VALUE));
$: trust_cl_avg = grouped_data.filter(item => item.MEASURE === 'Trust in Court and Legal System' && item.VALUE !== null).reduce((acc, item, _, array) => acc + item.VALUE / array.length, 0);

$: trust_le_min = Math.min(...grouped_data.filter(item => item.MEASURE === 'Trust in Legislature' && item.VALUE !== null).map(item => item.VALUE));
$: trust_le_max = Math.max(...grouped_data.filter(item => item.MEASURE === 'Trust in Legislature' && item.VALUE !== null).map(item => item.VALUE));
$: trust_le_avg = grouped_data.filter(item => item.MEASURE === 'Trust in Legislature' && item.VALUE !== null).reduce((acc, item, _, array) => acc + item.VALUE / array.length, 0);

let correlation_value;
$: correlation_value = value.find(element => true)?.Value

</script>

```sql df_data
SELECT
    TIME_PERIOD AS "TIME PERIOD",
    REF_AREA,
    Satisfaction_with_Democracy AS "Satisfaction with Democracy",
    Trust_in_Court_and_Legal_System AS "Trust in Court and Legal System",
    Trust_in_Legislature AS "Trust in Legislature",
    Trust_in_National_Government,
    Gender_Equality_in_Parliament AS "Gender Equality in Parliament",
    Gender_equality_in_Senior_Management_Positions AS "Gender equality in Senior Management Positions",
    Percent_Under_20 AS "Percent Under 20",
    Percent_Over_65 AS "Percent Over 65",
    Life_Expectancy AS "Life Expectancy",
    Population AS "Population",
    Mean_Annual_Wages AS "Mean Annual Wages",
    Low_Pay AS "Low Pay",
    WBS AS "WBS",
    WBSu5 AS "WBSu5",
    GDP_Growth_Rate AS "GDP Growth Rate"
FROM faktentabelle
```

```sql trust
SELECT * FROM ng
```

```sql tust_m_v
SELECT Countries, 'High or moderately high trust' AS Measure, "High or moderately high trust" AS Value, 1 AS OrderCol FROM ng
UNION ALL
SELECT Countries, 'Neutral' AS Measure, "Neutral" AS Value, 2 AS OrderCol FROM ng
UNION ALL
SELECT Countries, 'Low or no trust' AS Measure, "Low or no trust" AS Value, 3 AS OrderCol FROM ng
UNION ALL
SELECT Countries, 'Do not know' AS Measure, "Don't know" AS Value, 4 AS OrderCol FROM ng
ORDER BY OrderCol, Countries
```

```sql trust_avg
SELECT AVG(Value) AS avg_trust FROM ${tust_m_v} WHERE Measure LIKE 'High or moderately high trust'
```

## Comparison of Trust in National Government

A comparison of the Trust in National Goverment accross different OECD Countries.

<BarChart
    data={tust_m_v} 
    x=Countries 
    y=Value
    series=Measure
    sort=false
    type=stacked100
    yAxisTitle='Trust in National Government'
    xAxisTitle='Country'>

</BarChart>


## Country Data Deep Dive
```sql grouped_data
SELECT "Reference_Area", 'Trust in National Government' AS MEASURE, "Trust_in_National_Government" AS VALUE
FROM bigdf 
UNION ALL
SELECT "Reference_Area", 'Trust in Court and Legal System' AS MEASURE, "Trust_in_Court_and_Legal_System" AS VALUE
FROM bigdf 
UNION ALL
SELECT "Reference_Area", 'Trust in Legislature' AS MEASURE, "Trust_in_Legislature" AS VALUE
FROM bigdf
ORDER BY "Reference_Area", MEASURE
```

```sql grouped_country_data
SELECT "Reference_Area", MEASURE, VALUE, ${trust_ng_min} AS MIN, ${trust_ng_max} AS MAX, ${trust_ng_avg} AS AVG,0 as INTERVAL, 1 as OrderCol FROM ${grouped_data} WHERE "Reference_Area" LIKE '${params.country}' AND MEASURE LIKE 'Trust_in_National_Government'
UNION ALL
SELECT "Reference_Area", MEASURE, VALUE, ${trust_cl_min} AS MIN, ${trust_cl_max} AS MAX, ${trust_cl_avg} AS AVG,0 as INTERVAL, 2 as OrderCol FROM ${grouped_data} WHERE "Reference_Area" LIKE '${params.country}' AND MEASURE LIKE 'Trust_in_Court_and_Legal_System'
UNION ALL
SELECT "Reference_Area", MEASURE, VALUE, ${trust_le_min} AS MIN, ${trust_le_max} AS MAX, ${trust_le_avg} AS AVG,0 as INTERVAL, 3 as OrderCol FROM ${grouped_data} WHERE "Reference_Area" LIKE '${params.country}' AND MEASURE LIKE 'Trust_in_Legislature'
ORDER BY OrderCol
```

Take a look at two more metrics which measure the trust of the population into the court and legal system and the current legilature.

<BoxPlot 
    title={params.country}
    subtitle='Trust in National Government, Court and Legal System and Legislature'
    data={grouped_country_data}
    name=MEASURE
    midpoint=VALUE
    confidenceInterval=INTERVAL
    min=MIN
    max=MAX
    yAxisTitle='Score between 0-100'
    xAxisTitle='Metric'
    swapXY=true
    showAllXAxisLabels=true
/>

<br>

### Set that into perspective

```sql comparison_values
SELECT 
"Trust_in_National_Government", ${trust_ng_avg} AS NG_AVG, ("Trust_in_National_Government" - NG_AVG) AS NG_DELTA, "Trust_in_Court_and_Legal_System", ${trust_cl_avg} as CL_AVG, ("Trust_in_Court_and_Legal_System" - CL_AVG) AS CL_DELTA, "Trust_in_Legislature", ${trust_le_avg} as LE_AVG, ("Trust_in_Legislature" - LE_AVG) AS LE_DELTA FROM bigdf WHERE "Reference_Area" LIKE '${params.country}'
```

<BigValue 
  data={comparison_values}
  title='Trust in National Government'
  value=Trust_in_National_Government
  comparison=NG_DELTA
  comparisonTitle="compared to avg"
/>

<BigValue 
  data={comparison_values}
  title='Trust in Court and legal system'
  value=Trust_in_Court_and_Legal_System
  comparison=CL_DELTA
  comparisonTitle="compared to avg"
/>

<BigValue 
  data={comparison_values}
  title='Trust in Legislature'
  value=Trust_in_Legislature
  comparison=LE_DELTA
  comparisonTitle="compared to avg"
/>


## Take a look into Correlations

You want to improve the _Trust in National Governemnt_? Therefore you need to understand how _Trust in National Governemnt_ correlates to other metrices.


```sql correlation_heatmap
SELECT 'Satisfaction with Democracy' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Satisfaction_with_Democracy'
UNION ALL
SELECT 'Trust in Court and Legal System' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Trust_in_Court_and_Legal_System'
UNION ALL
SELECT 'Trust in Legislature' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Trust_in_Legislature'
UNION ALL
SELECT 'Gender Equality in Parliament' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Gender_Equality_in_Parliament'
UNION ALL
SELECT 'Gender equality in Senior Management Positions' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Gender_equality_in_Senior_Management_Positions'
UNION ALL
SELECT 'Well Being Score (WBS)' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'WBS'
UNION ALL
SELECT 'Percentage of Population with WBS under 5' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'WBSu5'
UNION ALL
SELECT 'Life Expectancy' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Life_Expectancy'
UNION ALL
SELECT 'Percent of Population Under 20' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Percent_Under_20'
UNION ALL
SELECT 'Percent of Population Over 65' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Percent_Over_65'
UNION ALL
SELECT 'Population' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Population'
UNION ALL
SELECT 'Mean Annual Wage' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Mean_Annual_Wages'
UNION ALL
SELECT 'Pecentage of People who work for less than 2/3 of median wage' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'Low_Pay'
UNION ALL
SELECT 'GDP Growth Rate' AS MEASURE, "Trust_in_National_Government" AS Correlation, 'Trust in National Government' AS COR_TO FROM cor WHERE MEASURE = 'GDP_Growth_Rate'

ORDER BY Correlation DESC
```

<Heatmap 
    title='Trust in National Government'
    subtitle='and how it correlates'
    data={correlation_heatmap} 
    x=COR_TO
    y=MEASURE
    value=Correlation
    min=-1
    max=1
    valueLabels=false
    colorPalette={['red', 'white', 'red']}
    rightPadding=200
    leftPadding=0
/>

A high correlation means that changes when the selected metric increases, _Trust in National Government_ tend to increase as well.

A low correlation means changes in the selected metric don't affect the _Trust in National Government_ as much.

## Correlation Deep Dive

To understand how the _Trust in National Governement_ correlates with other metrices select a metric in the dropdown. You can compare the line of the metric with the bars of the _Trust in National Government_.

```sql col_names
SELECT
  'Satisfaction_with_Democracy' AS Original, 'Satisfaction with Democracy' AS Modified
UNION ALL
SELECT
  'Trust_in_Court_and_Legal_System', 'Trust in Court and Legal System'
UNION ALL
SELECT
  'Trust_in_Legislature', 'Trust in Legislature'
UNION ALL
SELECT
  'Trust_in_National_Government', 'Trust in National Government'
UNION ALL
SELECT
  'Gender_Equality_in_Parliament', 'Gender Equality in Parliament'
UNION ALL
SELECT
  'Gender_equality_in_Senior_Management_Positions', 'Gender equality in Senior Management Positions'
UNION ALL
SELECT
  'Percent_Under_20', 'Percent Under 20'
UNION ALL
SELECT
  'Percent_Over_65', 'Percent Over 65'
UNION ALL
SELECT
  'Life_Expectancy', 'Life Expectancy'
UNION ALL
SELECT
  'Population', 'Population'
UNION ALL
SELECT
  'Mean_Annual_Wages', 'Mean Annual Wages'
UNION ALL
SELECT
  'Low_Pay', 'Low Pay'
UNION ALL
SELECT
  'WBS', 'WBS'
UNION ALL
SELECT
  'WBSu5', 'WBSu5'
UNION ALL
SELECT
  'GDP_Growth_Rate', 'GDP Growth Rate'

```

```sql cor_data
SELECT REF_AREA, Trust_in_National_Government, "${inputs.cor_val.value}" FROM ${df_data} WHERE Trust_in_National_Government IS NOT NULL AND "${inputs.cor_val.value}" IS NOT NULL
```

<Dropdown
    title='Select a Metric'
    name=cor_val
    data={col_names}
    value=Modified
/>

<LineChart 
    title='Trust in National Governement'
    subtitle='compared to {inputs.cor_val.value}'
    data={cor_data} 
    x=REF_AREA 
    xAxisLabels=true
    showAllLabels=true
    y={inputs.cor_val.value.replace(/_/g, " ")}
    step=false
    lineType=dotted
    y2='Trust_in_National_Government'
    y2SeriesType=bar
/>

```sql correlation
SELECT 
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'Satisfaction_with_Democracy') AS 'Satisfaction with Democracy',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'Trust_in_Court_and_Legal_System') AS 'Trust in Court and Legal System',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'Trust_in_Legislature') AS 'Trust in Legislature',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'Trust_in_National_Government') AS 'Trust in National Government',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'Gender_Equality_in_Parliament') AS 'Gender Equality in Parliament',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'Gender_equality_in_Senior_Management_Positions') AS 'Gender equality in Senior Management Positions',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'WBS') AS 'WBS',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'WBSu5') AS 'WBSu5',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'Life_Expectancy') AS 'Life Expectancy',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'Perc_of_Populaition_with_WBS_under_5') AS 'Perc of Population with WBS<5',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'Percent_Under_20') AS 'Percent Under 20',
  (SELECT ("Trust_in_National_Government") FROM cor WHERE MEASURE = 'Percent_Over_65') AS 'Percent Over 65',
```

```sql value
SELECT "${inputs.cor_val.value}" AS Value FROM ${correlation}
```

The correlation between **Trust in National Government** and **{inputs.cor_val.value}** is <Value data={value}/>
<ExplainButton metric={inputs.cor_val.value} value={correlation_value}/>

<br>

{#if params.country == 'FRA'}

# Key Takeaways

1. **Well being**: average well being slighlty decreased and percentage of people with a well being of 5 or less out of ten nearly doubled since 2018
2. **Life Expectancy**: since 2019 the life expectancy slighly sunk (0.7 years); Woman live nearly 6 years longer
3. **GDP**: in the corona crisis our GDP shrunk, but it is recovering good
4. **Wages**: Although the Average Wage is rising, the percentage of people working for less than 2/3 of the median is stagnating
5. **Trust in Governemnt**: OECD Average is bad, but we are even worse
6. **Court and Legaly System**:very low trust

### Actions

1. Start campaign to improve trust in the legal system
2. Improve the well being - focus on the people who are already medium 
3. Strengthen the Government with strong females

{/if}
