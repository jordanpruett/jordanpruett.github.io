---
title: "SQL Practice With D&D Data"
date: 2022-03-23
categories:
  - sql
  - dataviz
---

Below are some SQL challenges I made for myself as practice, using a database I built from public D&D Beyond character sheets. 

I tried to do as much of the data manipulation in SQL as possible, though I ran the queries through pandas for ease of inspection/visualization.


```python
import sqlite3

import pandas as pd
from matplotlib import pyplot as plt
import seaborn as sns
```


```python
# Connect to the database
con = sqlite3.connect("../../db/main_11-30-21.db")
```

The first one is pretty simple. This one counts all feats selected by all characters. The only tricky thing is that you have to do a tiny bit of entity resolution, since some feats include optional features in the name: "Resilience (Constitution)" and "Resilience (Wisdom)" should normalize to "Resilience." I accomplished this with a subquery.


```python
query = '''

SELECT
    feats_clean.featName,
    COUNT (feats_clean.featName) as featCount
FROM (
    SELECT
        CASE
            WHEN INSTR(fd.featName, ' -') != 0
                THEN SUBSTR(fd.featName, 1, INSTR(fd.featName, ' -') - 1)
            WHEN INSTR(fd.featName, ' (') != 0
                THEN SUBSTR(fd.featName, 1, INSTR(fd.featName, ' (') - 1)
            ELSE
                fd.featName
            END featName 
    FROM feats
        LEFT JOIN featDefinition fd ON feats.featDefinitionId = fd.Id
    WHERE 
        fd.isHomebrew = 0
) feats_clean
GROUP BY featName
ORDER BY featCount DESC
;
'''
```

Let's display the top 10 most commonly picked feats. The only thing here that suprises me is that Grappler is #2. It's widely considered bad, but apparently ~31,000 characters didn't get the memo.


```python
df = pd.read_sql_query(query, con)
df.iloc[0:10]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>featName</th>
      <th>featCount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>War Caster</td>
      <td>32555</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Grappler</td>
      <td>31612</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Tough</td>
      <td>22710</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sharpshooter</td>
      <td>16776</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Alert</td>
      <td>16656</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Lucky</td>
      <td>15860</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Observant</td>
      <td>15686</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Resilient</td>
      <td>15638</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Magic Initiate</td>
      <td>14927</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Sentinel</td>
      <td>14334</td>
    </tr>
  </tbody>
</table>
</div>



This next one calculates the total value in gold pieces of all of a character's currencies and inventory items. It also pulls their tier of play, based on their character levels. The idea was to see how closely tier-wise wealth averages resemble the estimates provided in the Dungeon Masters' Guide.

Total character level is not a stored value and had to be generated from class levels, which matters in the case of multiclass characters.

Total gold value is not a stored value and has to be calculated from the converted value of all currencies plus the total cost of all inventory items.


```python
query = '''

SELECT
    summed_levels.charId,
    CASE
        WHEN summed_levels.charLevel BETWEEN 1 AND 4 THEN 1
        WHEN summed_levels.charLevel BETWEEN 5 AND 10 THEN 2
        WHEN summed_levels.charLevel BETWEEN 11 AND 16 THEN 3
        WHEN summed_levels.charLevel BETWEEN 17 AND 20 THEN 4
        ELSE 5
        END tier,
    0.01*(c.cp) + 0.1*(c.sp) + 0.5*(c.ep) + c.gp + 10*(c.pp) AS gold,
    i.total_cost AS inventory_value
FROM (
    SELECT
        charId,
        SUM(classLevel) AS charLevel
    FROM 
        classes
    GROUP BY
        charId
) summed_levels
LEFT JOIN
    currencies c ON c.charId = summed_levels.charId
LEFT JOIN
    (
        SELECT
            charId,
            SUM(iDef.cost * quantity) AS total_cost
        FROM 
            inventory
        LEFT JOIN 
            itemDefinition iDef ON iDef.id = inventory.itemDefinitionId
        GROUP BY 
            charId
    ) i ON i.charId = summed_levels.charId
;
'''
```


```python
df = pd.read_sql_query(query, con)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>charId</th>
      <th>tier</th>
      <th>gold</th>
      <th>inventory_value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2510</td>
      <td>1</td>
      <td>130.0</td>
      <td>155.97</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2778</td>
      <td>1</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2806</td>
      <td>1</td>
      <td>135.0</td>
      <td>134.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2903</td>
      <td>3</td>
      <td>0.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2988</td>
      <td>1</td>
      <td>7.6</td>
      <td>0.20</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>987217</th>
      <td>52414013</td>
      <td>1</td>
      <td>0.0</td>
      <td>100.41</td>
    </tr>
    <tr>
      <th>987218</th>
      <td>52414055</td>
      <td>2</td>
      <td>27.0</td>
      <td>57.50</td>
    </tr>
    <tr>
      <th>987219</th>
      <td>52414131</td>
      <td>4</td>
      <td>10.0</td>
      <td>162.10</td>
    </tr>
    <tr>
      <th>987220</th>
      <td>52414159</td>
      <td>4</td>
      <td>0.0</td>
      <td>30.20</td>
    </tr>
    <tr>
      <th>987221</th>
      <td>52414174</td>
      <td>1</td>
      <td>0.0</td>
      <td>300.00</td>
    </tr>
  </tbody>
</table>
<p>987222 rows × 4 columns</p>
</div>



That one was useful as a coding exercise, but I realized after browsing the data that these numbers probably aren't very meaningful. It seems like most people aren't tracking wealth very closely on these sheets - lots of characters have no gold and total inventory value of 0. Many of these characters are theory-crafts that haven't actually seen play, so they don't have items or money.

And even if we exclude these characters, the median total wealth is much lower than I'd expect (though note that this doesn't include magic items, which don't have set prices in 5th edition).


```python
df['inventory_value'] = df.inventory_value.fillna(0)
df['wealth'] = df.gold + df.inventory_value
df[(df.gold != 0) & (df.inventory_value != 0)].groupby('tier').wealth.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
    <tr>
      <th>tier</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>308188.0</td>
      <td>3.999601e+06</td>
      <td>2.741318e+08</td>
      <td>0.35</td>
      <td>103.72</td>
      <td>154.500</td>
      <td>282.6000</td>
      <td>2.493229e+10</td>
    </tr>
    <tr>
      <th>2</th>
      <td>207866.0</td>
      <td>1.834248e+06</td>
      <td>1.868466e+08</td>
      <td>0.21</td>
      <td>125.20</td>
      <td>240.750</td>
      <td>1081.3000</td>
      <td>2.493229e+10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>44546.0</td>
      <td>8.459603e+06</td>
      <td>4.193392e+08</td>
      <td>1.63</td>
      <td>123.50</td>
      <td>340.005</td>
      <td>1747.4575</td>
      <td>2.493229e+10</td>
    </tr>
    <tr>
      <th>4</th>
      <td>34418.0</td>
      <td>1.531096e+10</td>
      <td>2.830436e+12</td>
      <td>1.02</td>
      <td>94.50</td>
      <td>168.245</td>
      <td>1141.8000</td>
      <td>5.251050e+14</td>
    </tr>
  </tbody>
</table>
</div>



These numbers are... suspicious. Median wealth at tier 4 is only 168 gp? I inspected some of the tier 4 characters and it looks like they are either theory-crafts, without items, or character sheets that only list magic items (which have no price). Ah well - limitation of the data.

But the outliers are really something. Who is this character with over 6 billion gp in wealth?


```python
df[df.wealth == df.wealth.max()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>charId</th>
      <th>tier</th>
      <th>gold</th>
      <th>inventory_value</th>
      <th>wealth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>974732</th>
      <td>51974698</td>
      <td>4</td>
      <td>2.493229e+10</td>
      <td>5.250801e+14</td>
      <td>5.251050e+14</td>
    </tr>
  </tbody>
</table>
</div>



Alright, let's end with something a little more interesting. This query gets the highest magic item rarity of each character. It requires multiple nested queries since, first of all, rarity has to be converted to a numeric value in order to get the "highest," and because, as before, character level has to be inferred.


```python
query = '''


SELECT
    summed_levels.charId,
    summed_levels.charLevel,
    char_max_rarity.max_rarity
FROM (
    SELECT
        charId,
        SUM(classLevel) AS charLevel
    FROM 
        classes
    GROUP BY
        charId
) summed_levels
LEFT JOIN
    (
        SELECT
            charId,
            MAX(rarity) AS max_rarity
        FROM
            (SELECT
                charId,
                CASE
                    WHEN iDef.rarity = 'Uncommon' THEN 1
                    WHEN iDef.rarity = 'Rare' THEN 2
                    WHEN iDef.rarity = 'Very Rare' THEN 3
                    WHEN iDef.rarity = 'Legendary' THEN 4
                    WHEN iDef.rarity = 'Artifact' THEN 5
                    ELSE 0
                    END rarity     
            FROM 
                inventory
            LEFT JOIN 
                itemDefinition iDef ON iDef.id = inventory.itemDefinitionId
            ) inv_numeric_rarity 
        GROUP BY
            charId
    ) char_max_rarity ON char_max_rarity.charId = summed_levels.charId
WHERE 
    char_max_rarity.max_rarity > 0
AND
    summed_levels.charLevel BETWEEN 1 AND 20
;

'''
```


```python
df = pd.read_sql_query(query, con)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>charId</th>
      <th>charLevel</th>
      <th>max_rarity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>4534</td>
      <td>20</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4565</td>
      <td>20</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4668</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4952</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5505</td>
      <td>1</td>
      <td>4</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>271635</th>
      <td>52413585</td>
      <td>20</td>
      <td>3</td>
    </tr>
    <tr>
      <th>271636</th>
      <td>52413597</td>
      <td>20</td>
      <td>4</td>
    </tr>
    <tr>
      <th>271637</th>
      <td>52414003</td>
      <td>9</td>
      <td>1</td>
    </tr>
    <tr>
      <th>271638</th>
      <td>52414131</td>
      <td>20</td>
      <td>4</td>
    </tr>
    <tr>
      <th>271639</th>
      <td>52414159</td>
      <td>18</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
<p>271640 rows × 3 columns</p>
</div>



Let's make a box plot and see what the distribution by tier looks like.


```python
import matplotlib.ticker as ticker

sns.set_style("darkgrid")
dims = (10.7, 7.27)
fig, ax = plt.subplots(figsize=dims)
ax = sns.boxplot(
    x='max_rarity', 
    y='charLevel', 
    data=df,
    ax=ax
)
ax.set_xticklabels([
    'Uncommon',
    'Rare',
    'Very Rare',
    'Legendary',
    'Artifact'
], fontsize=12)
ax.set_ylabel('character level', fontsize=12)
ax.set_xlabel('')
ax.set_title('Highest Magic Item Rarity by Character Level', fontsize=14)
ax.yaxis.set_major_locator(ticker.MultipleLocator(2))
ax.yaxis.set_major_formatter(ticker.ScalarFormatter())
```


![png](/assets/images/03-22/highest_rarity.png)


Now we're talking. A couple interesting things to note:

* The spread of the distribution increases as you move up the rarity ladder. Most characters with only an uncommon item are between levels 3 and 7, but characters who have at least a legendary- or artifact-rarity item are all sorts of levels.
* Artifact rarity actually has a lower median character level than legendary. I'm assuming this is because DMs like to drop an artifact into play a little earlier for plot reasons: call it the "one ring" effect (this theory is supported by the fact that it disappears if I remove homebrew items)

If you enjoyed this, consider checking out my other posts. I plan to do more analysis with this data in the near future.
