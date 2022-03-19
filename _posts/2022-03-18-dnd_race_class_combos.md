---
title: "D&D Race/Class Combos in ~1 Million Character Sheets"
date: 2022-03-18
excerpt_separator: 
categories:
  - sql
  - pandas
  - dataviz
---


Last year, I scraped data from more than 1 million D&D character sheets from online sources, storing them in a SQLite database. The project was both for my own curiosity, and because I wanted to get some more hands-on practice using SQL. Just storing the data required learning how to use SQLAlchemy, which was worth the time on its own.

Anyways, now I have a big database of D&D characters, from which you can extract all sorts of fun insights. This notebook shows the most common race and class pairings. 


```python
import sqlite3

import pandas as pd
import numpy as np
import seaborn as sns
from matplotlib import pyplot as plt
```


```python
# Connect to the database
con = sqlite3.connect("../../db/main_11-30-21.db")
```

The query below extracts just the class and race from each character. Due to the way the data was coded, a case statement is necessary to disentangle subraces. It also filters out homebrew classes and class names that contain a parenthesis, which are usually unearthed arcana classes, designated by (UA).


```python
query = '''

SELECT 
    classes.charId AS charId,
    classes.classLevel,
    cd.className,
    CASE 
        WHEN r.isSubRace = 1 THEN
            baseRaceId
        ELSE
            r.fullName
        END raceName
        
FROM classes
    LEFT JOIN characterSheet cs ON classes.charId = cs.Id
    LEFT JOIN race r ON cs.raceId = r.Id
    LEFT JOIN classDefinition cd ON classes.classDefinitionId = cd.Id
WHERE 
    cs.raceID IS NOT NULL 
    AND r.isHomebrew = 0
    AND cd.isHomebrew = 0
    AND NOT raceName LIKE '%(%'
ORDER BY charId;

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
      <th>classLevel</th>
      <th>className</th>
      <th>raceName</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2510</td>
      <td>2</td>
      <td>Wizard</td>
      <td>Elf</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2778</td>
      <td>1</td>
      <td>Warlock</td>
      <td>Tiefling</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2806</td>
      <td>3</td>
      <td>Druid</td>
      <td>Genasi</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2903</td>
      <td>11</td>
      <td>Monk</td>
      <td>Elf</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2988</td>
      <td>1</td>
      <td>Monk</td>
      <td>Dragonborn</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1041035</th>
      <td>52414131</td>
      <td>6</td>
      <td>Paladin</td>
      <td>Half-Elf</td>
    </tr>
    <tr>
      <th>1041036</th>
      <td>52414131</td>
      <td>9</td>
      <td>Sorcerer</td>
      <td>Half-Elf</td>
    </tr>
    <tr>
      <th>1041037</th>
      <td>52414131</td>
      <td>5</td>
      <td>Warlock</td>
      <td>Half-Elf</td>
    </tr>
    <tr>
      <th>1041038</th>
      <td>52414159</td>
      <td>18</td>
      <td>Druid</td>
      <td>Warforged</td>
    </tr>
    <tr>
      <th>1041039</th>
      <td>52414174</td>
      <td>1</td>
      <td>Rogue</td>
      <td>Aarakocra</td>
    </tr>
  </tbody>
</table>
<p>1041040 rows × 4 columns</p>
</div>



There are more rows than unique characters. This is because of multiclassed characters, which get a row for each unique class. I'll leave the data as-is and count each unique class for each character a separate time. 


```python
len(df.charId.unique())
```




    928135



We'll limit ourselves to just the 12 most common races and classes, for a more manageable visualization.


```python
n = 12
class_filter = list(df.className.value_counts()[:n].index)
race_filter = list(df.raceName.value_counts()[:n].index)
```


```python
df = df[df.className.isin(class_filter) & df.raceName.isin(race_filter)]
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
      <th>classLevel</th>
      <th>className</th>
      <th>raceName</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2510</td>
      <td>2</td>
      <td>Wizard</td>
      <td>Elf</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2778</td>
      <td>1</td>
      <td>Warlock</td>
      <td>Tiefling</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2806</td>
      <td>3</td>
      <td>Druid</td>
      <td>Genasi</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2903</td>
      <td>11</td>
      <td>Monk</td>
      <td>Elf</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2988</td>
      <td>1</td>
      <td>Monk</td>
      <td>Dragonborn</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1041032</th>
      <td>52414003</td>
      <td>1</td>
      <td>Rogue</td>
      <td>Elf</td>
    </tr>
    <tr>
      <th>1041034</th>
      <td>52414055</td>
      <td>6</td>
      <td>Barbarian</td>
      <td>Dragonborn</td>
    </tr>
    <tr>
      <th>1041035</th>
      <td>52414131</td>
      <td>6</td>
      <td>Paladin</td>
      <td>Half-Elf</td>
    </tr>
    <tr>
      <th>1041036</th>
      <td>52414131</td>
      <td>9</td>
      <td>Sorcerer</td>
      <td>Half-Elf</td>
    </tr>
    <tr>
      <th>1041037</th>
      <td>52414131</td>
      <td>5</td>
      <td>Warlock</td>
      <td>Half-Elf</td>
    </tr>
  </tbody>
</table>
<p>853527 rows × 4 columns</p>
</div>



To get counts of race-class pairings, pd.crosstab() can do the trick. Then I'll express each value as a proportion of the total, then sort the columns by column sum and rows by row sum.


```python
tab = pd.crosstab(df.className, df.raceName)
tab = tab / tab.sum(axis=0).sum()
tab = tab[tab.sum().sort_values(ascending=False).index] # order columns
tab = tab.loc[tab.sum(axis=1).sort_values(ascending=False).index] # order rows
tab
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
      <th>raceName</th>
      <th>Human</th>
      <th>Elf</th>
      <th>Half-Elf</th>
      <th>Tiefling</th>
      <th>Dragonborn</th>
      <th>Dwarf</th>
      <th>Halfling</th>
      <th>Half-Orc</th>
      <th>Aasimar</th>
      <th>Genasi</th>
      <th>Goliath</th>
      <th>Gnome</th>
    </tr>
    <tr>
      <th>className</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
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
      <th>Fighter</th>
      <td>0.046209</td>
      <td>0.013641</td>
      <td>0.007308</td>
      <td>0.004413</td>
      <td>0.012009</td>
      <td>0.012068</td>
      <td>0.003273</td>
      <td>0.009552</td>
      <td>0.003289</td>
      <td>0.004169</td>
      <td>0.008568</td>
      <td>0.002104</td>
    </tr>
    <tr>
      <th>Rogue</th>
      <td>0.026011</td>
      <td>0.027433</td>
      <td>0.013677</td>
      <td>0.010282</td>
      <td>0.004773</td>
      <td>0.002703</td>
      <td>0.014878</td>
      <td>0.002392</td>
      <td>0.002129</td>
      <td>0.004376</td>
      <td>0.001277</td>
      <td>0.005129</td>
    </tr>
    <tr>
      <th>Wizard</th>
      <td>0.023684</td>
      <td>0.021548</td>
      <td>0.007328</td>
      <td>0.005596</td>
      <td>0.004147</td>
      <td>0.002791</td>
      <td>0.002698</td>
      <td>0.001203</td>
      <td>0.002036</td>
      <td>0.003606</td>
      <td>0.000965</td>
      <td>0.010133</td>
    </tr>
    <tr>
      <th>Barbarian</th>
      <td>0.014192</td>
      <td>0.002970</td>
      <td>0.001577</td>
      <td>0.002313</td>
      <td>0.009702</td>
      <td>0.010622</td>
      <td>0.002471</td>
      <td>0.016702</td>
      <td>0.001907</td>
      <td>0.002701</td>
      <td>0.016979</td>
      <td>0.002000</td>
    </tr>
    <tr>
      <th>Cleric</th>
      <td>0.020795</td>
      <td>0.011510</td>
      <td>0.007037</td>
      <td>0.004647</td>
      <td>0.004334</td>
      <td>0.014367</td>
      <td>0.003281</td>
      <td>0.002573</td>
      <td>0.006887</td>
      <td>0.003699</td>
      <td>0.001943</td>
      <td>0.002587</td>
    </tr>
    <tr>
      <th>Paladin</th>
      <td>0.019423</td>
      <td>0.005600</td>
      <td>0.007600</td>
      <td>0.004853</td>
      <td>0.013997</td>
      <td>0.006096</td>
      <td>0.001425</td>
      <td>0.004592</td>
      <td>0.010450</td>
      <td>0.002214</td>
      <td>0.004360</td>
      <td>0.000935</td>
    </tr>
    <tr>
      <th>Warlock</th>
      <td>0.016376</td>
      <td>0.008866</td>
      <td>0.012703</td>
      <td>0.017114</td>
      <td>0.005292</td>
      <td>0.001707</td>
      <td>0.002131</td>
      <td>0.001964</td>
      <td>0.006857</td>
      <td>0.003223</td>
      <td>0.001224</td>
      <td>0.001947</td>
    </tr>
    <tr>
      <th>Sorcerer</th>
      <td>0.013764</td>
      <td>0.011463</td>
      <td>0.012260</td>
      <td>0.011004</td>
      <td>0.008352</td>
      <td>0.001740</td>
      <td>0.002132</td>
      <td>0.001201</td>
      <td>0.005480</td>
      <td>0.005096</td>
      <td>0.000941</td>
      <td>0.001823</td>
    </tr>
    <tr>
      <th>Bard</th>
      <td>0.014830</td>
      <td>0.008661</td>
      <td>0.015823</td>
      <td>0.009915</td>
      <td>0.003544</td>
      <td>0.002742</td>
      <td>0.006547</td>
      <td>0.002059</td>
      <td>0.003338</td>
      <td>0.002853</td>
      <td>0.001227</td>
      <td>0.003236</td>
    </tr>
    <tr>
      <th>Ranger</th>
      <td>0.015950</td>
      <td>0.025400</td>
      <td>0.008815</td>
      <td>0.003120</td>
      <td>0.003666</td>
      <td>0.002689</td>
      <td>0.003768</td>
      <td>0.002342</td>
      <td>0.001547</td>
      <td>0.002949</td>
      <td>0.001454</td>
      <td>0.002187</td>
    </tr>
    <tr>
      <th>Druid</th>
      <td>0.009741</td>
      <td>0.019025</td>
      <td>0.006246</td>
      <td>0.004015</td>
      <td>0.003322</td>
      <td>0.003475</td>
      <td>0.003380</td>
      <td>0.001959</td>
      <td>0.001846</td>
      <td>0.005482</td>
      <td>0.001582</td>
      <td>0.003496</td>
    </tr>
    <tr>
      <th>Monk</th>
      <td>0.015640</td>
      <td>0.010230</td>
      <td>0.003917</td>
      <td>0.002897</td>
      <td>0.003794</td>
      <td>0.002294</td>
      <td>0.003960</td>
      <td>0.002447</td>
      <td>0.002225</td>
      <td>0.004839</td>
      <td>0.002595</td>
      <td>0.001534</td>
    </tr>
  </tbody>
</table>
</div>



Now, let's make a heatmap.


```python
dims = (11.7, 8.27)
fig, ax = plt.subplots(figsize=dims)
cmap = sns.cm.rocket_r
heatmap = sns.heatmap(
    ax=ax, 
    data=tab.T,
    cmap=cmap 
)
title = ax.set_title('Race and Class Proportions in ~1 Million D&D Characters')
ylabel = ax.set_ylabel('Race')
xlabel = ax.set_xlabel('Class')

```


![png](/assets/images/03-22/heatmap.png)


There's a lot of fun information packed in this little graphic. For example, human fighters make up almost 5% -- 1 in 20 -- of all D&D characters in the dataset. Islands of heat show pairings that are especially popular: elf rogue, goliath barbarian, tiefling warlock.

Also, some races seem to have greater class diversity than others. Goliath are often barbarians and halflings are often rogues, but genasi seem more "spread out" among a variety of classes (probably due to the fact that they have a variety of subrace options). 

An easy way to express this quantitatively is with the Gini index. In effect, we can measure the internal "inequality" of the population of each race. 

There are a million Gini implementations out there, but I'll use a concise [one I found on stackexchange](https://stackoverflow.com/questions/48999542/more-efficient-weighted-gini-coefficient-in-python/48999797#48999797).


```python
def gini(x, w=None):
    # Array indexing requires reset indexes.
    x = pd.Series(x).reset_index(drop=True)
    if w is None:
        w = np.ones_like(x)
    w = pd.Series(w).reset_index(drop=True)
    n = x.size
    wxsum = sum(w * x)
    wsum = sum(w)
    sxw = np.argsort(x)
    sx = x[sxw] * w[sxw]
    sw = w[sxw]
    pxi = np.cumsum(sx) / wxsum
    pci = np.cumsum(sw) / wsum
    g = 0.0
    for i in np.arange(1, n):
        g = g + pxi.iloc[i] * pci.iloc[i - 1] - pci.iloc[i] * pxi.iloc[i - 1]
    return g
```

Now we can just apply this along each column, and sort. 


```python
race_gini = tab.apply(gini).sort_values()

dims=(10,7)
fig, ax = plt.subplots(figsize=dims)
sns.set_style('darkgrid')
sns.set_color_codes('muted')
ax = sns.barplot(
    x=race_gini, 
    y=race_gini.index,
    orient='h',
    color='b'
)
ax.set_title('Class-wise Gini Coefficient of D&D Races', fontsize=16)
ylabel = ax.set_ylabel('Race', fontsize=14)
```




    Text(0, 0.5, 'Race')



![png](/assets/images/03-22/gini.png)


There you have it: D&D races sorted by class diversity. Genasi and humans have the most class variety, whereas half-orcs and goliaths are more homogenous. 

The Goliath spot is not so surprising, though going into this I would have guessed tieflings would be more homogenous. I guess warlocks don't dominate the ranks of tieflings as much as barbarians do for goliaths.


