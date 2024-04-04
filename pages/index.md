---
title: Country Data
---

Select a Country from the table to get a detailed look

```sql all_areas
SELECT 
    translator.column1 AS Country,
    bigdf.Reference_Area AS url
FROM 
    bigdf
INNER JOIN 
    translator 
ON 
    bigdf.Reference_Area = translator.column0
ORDER BY Country
```

<DataTable data="{all_areas}" search="true" rows=100 totalRow=true link=url/>