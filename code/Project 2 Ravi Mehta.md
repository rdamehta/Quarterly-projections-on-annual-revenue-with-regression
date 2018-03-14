

# Forecasting Liquor Sales in the State of Iowa

![title](https://imgs.xkcd.com/comics/beer_2x.png)


```python
import pandas as pd

## Load the data into a DataFrame
df = pd.read_csv('../iowa_liquor_sales_proj_2.csv')

## Transform the dates if needed, e.g.
df["Date"] = pd.to_datetime(df["Date"], format="%m/%d/%Y")#big 'Y' for 4 digit date
```

    C:\Users\rdame\AppData\Local\Enthought\Canopy\edm\envs\User\lib\site-packages\IPython\core\interactiveshell.py:2728: DtypeWarning: Columns (6) have mixed types. Specify dtype option on import or set low_memory=False.
      interactivity=interactivity, compiler=compiler, result=result)
    


```python
df.sort_values(by = 'Date', inplace = True)
```

# Explore the data

Perform some exploratory statistical analysis and make some plots, such as histograms of transaction totals, bottles sold, etc.


```python
import seaborn as sns
import matplotlib.pyplot as plt
%matplotlib inline
df.info() #looks likesome Nulls in County and county number. lets use either city and or zipcode and see how much each
#city/zipcode sold
#change all prices into floats.
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2709552 entries, 695077 to 2709551
    Data columns (total 24 columns):
    Invoice/Item Number      object
    Date                     datetime64[ns]
    Store Number             int64
    Store Name               object
    Address                  object
    City                     object
    Zip Code                 object
    Store Location           object
    County Number            float64
    County                   object
    Category                 float64
    Category Name            object
    Vendor Number            int64
    Vendor Name              object
    Item Number              int64
    Item Description         object
    Pack                     int64
    Bottle Volume (ml)       int64
    State Bottle Cost        object
    State Bottle Retail      object
    Bottles Sold             int64
    Sale (Dollars)           object
    Volume Sold (Liters)     float64
    Volume Sold (Gallons)    float64
    dtypes: datetime64[ns](1), float64(4), int64(6), object(13)
    memory usage: 516.8+ MB
    


```python
#Sales is stored as a non numeric object, so lets get rid of the dollar sign and store it as a float
df['Sale (Dollars)'] = df['Sale (Dollars)'].str.replace('$', '').astype('float64')
df['State Bottle Cost'] = df['State Bottle Cost'].str.replace('$', '').astype('float64')
df['State Bottle Retail'] = df['State Bottle Retail'].str.replace('$', '').astype('float64')
```


```python
df.info() #confirming we price columns to floats
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2709552 entries, 695077 to 2709551
    Data columns (total 24 columns):
    Invoice/Item Number      object
    Date                     datetime64[ns]
    Store Number             int64
    Store Name               object
    Address                  object
    City                     object
    Zip Code                 object
    Store Location           object
    County Number            float64
    County                   object
    Category                 float64
    Category Name            object
    Vendor Number            int64
    Vendor Name              object
    Item Number              int64
    Item Description         object
    Pack                     int64
    Bottle Volume (ml)       int64
    State Bottle Cost        float64
    State Bottle Retail      float64
    Bottles Sold             int64
    Sale (Dollars)           float64
    Volume Sold (Liters)     float64
    Volume Sold (Gallons)    float64
    dtypes: datetime64[ns](1), float64(7), int64(6), object(10)
    memory usage: 516.8+ MB
    


```python
df['City'] = df['City'].map(lambda x: x.upper())
```


```python
# i saw some of the cities were mispelled, in the long run this probably wont make a difference but i renamed them anways.
df['City'] = df['City'].map(lambda x: x.replace('MT','MOUNT') if x[0] == 'M' else x)
df['City'] = df['City'].map(lambda x: x.replace('ST ','ST. ') if x[0] == 'S' else x)
df['City'] = df['City'].map(lambda x: x.replace('OTTUWMA','OTTUMWA') if x == 'OTTUWMA' else x)
df['City'] = df['City'].map(lambda x: x.replace('LEMARS','LE MARS') if x == 'LEMARS' else x)
df['City'] = df['City'].map(lambda x: x.replace('LECLAIRE','LE CLAIRE') if x == 'LECLAIRE' else x)
df['City'] = df['City'].map(lambda x: x.replace('GUTTENBURG','GUTTENBERG') if x == 'GUTTENBURG' else x)

```


```python
#but first lets sort by date and see over what timespan the data was collected
min(df.Date), max(df.Date) #January 5, 2015 to March 31, 2016
```




    (Timestamp('2015-01-05 00:00:00'), Timestamp('2016-03-31 00:00:00'))




```python
#lets plot the cities with the top 25 total sales
g = sns.factorplot(x= 'City', y = 'Sale (Dollars)',
                   kind = 'bar',
                   data = df.groupby('City')[['Sale (Dollars)']].sum().sort_values(by = 'Sale (Dollars)').reset_index().tail(25),
                   size = 5,
                   aspect = 3,
                   palette = None,)
g.set_xticklabels(rotation = 55)


```




    <seaborn.axisgrid.FacetGrid at 0x2b2ac5e6cf8>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_11_1.png)



```python
#lets try and look at the best performing stores by city 
df.groupby(['City','Store Number'])[['Sale (Dollars)']].sum().sort_values(by = 'Sale (Dollars)', ascending = False)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Sale (Dollars)</th>
    </tr>
    <tr>
      <th>City</th>
      <th>Store Number</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="2" valign="top">DES MOINES</th>
      <th>2633</th>
      <td>12282646.26</td>
    </tr>
    <tr>
      <th>4829</th>
      <td>11085530.53</td>
    </tr>
    <tr>
      <th>IOWA CITY</th>
      <th>2512</th>
      <td>5206377.22</td>
    </tr>
    <tr>
      <th>CEDAR RAPIDS</th>
      <th>3385</th>
      <td>4759187.79</td>
    </tr>
    <tr>
      <th>WINDSOR HEIGHTS</th>
      <th>3420</th>
      <td>4018414.55</td>
    </tr>
    <tr>
      <th>BETTENDORF</th>
      <th>3952</th>
      <td>3768333.09</td>
    </tr>
    <tr>
      <th>WEST DES MOINES</th>
      <th>3814</th>
      <td>3365204.04</td>
    </tr>
    <tr>
      <th>CORALVILLE</th>
      <th>2670</th>
      <td>3015679.96</td>
    </tr>
    <tr>
      <th>CEDAR RAPIDS</th>
      <th>3773</th>
      <td>2948596.43</td>
    </tr>
    <tr>
      <th>DAVENPORT</th>
      <th>3354</th>
      <td>2920466.95</td>
    </tr>
    <tr>
      <th>MOUNT VERNON</th>
      <th>5102</th>
      <td>2557579.26</td>
    </tr>
    <tr>
      <th>DAVENPORT</th>
      <th>2625</th>
      <td>2399216.50</td>
    </tr>
    <tr>
      <th>SIOUX CITY</th>
      <th>3447</th>
      <td>2380193.90</td>
    </tr>
    <tr>
      <th>DUBUQUE</th>
      <th>4167</th>
      <td>2346190.58</td>
    </tr>
    <tr>
      <th>WATERLOO</th>
      <th>3494</th>
      <td>2189165.60</td>
    </tr>
    <tr>
      <th>AMES</th>
      <th>3524</th>
      <td>2048850.65</td>
    </tr>
    <tr>
      <th>COUNCIL BLUFFS</th>
      <th>2629</th>
      <td>2009175.89</td>
    </tr>
    <tr>
      <th>DAVENPORT</th>
      <th>2614</th>
      <td>2003270.50</td>
    </tr>
    <tr>
      <th>CARROLL</th>
      <th>2593</th>
      <td>1981636.04</td>
    </tr>
    <tr>
      <th>SIOUX CITY</th>
      <th>3820</th>
      <td>1978407.16</td>
    </tr>
    <tr>
      <th>WEST DES MOINES</th>
      <th>2648</th>
      <td>1977218.38</td>
    </tr>
    <tr>
      <th>CLINTON</th>
      <th>2616</th>
      <td>1947756.37</td>
    </tr>
    <tr>
      <th>WEST DES MOINES</th>
      <th>2619</th>
      <td>1939138.09</td>
    </tr>
    <tr>
      <th>URBANDALE</th>
      <th>2663</th>
      <td>1868749.31</td>
    </tr>
    <tr>
      <th>CORALVILLE</th>
      <th>4677</th>
      <td>1812906.19</td>
    </tr>
    <tr>
      <th>CEDAR FALLS</th>
      <th>2106</th>
      <td>1772173.90</td>
    </tr>
    <tr>
      <th>DES MOINES</th>
      <th>2561</th>
      <td>1750975.52</td>
    </tr>
    <tr>
      <th>COUNCIL BLUFFS</th>
      <th>4312</th>
      <td>1748177.94</td>
    </tr>
    <tr>
      <th>BURLINGTON</th>
      <th>2506</th>
      <td>1743498.14</td>
    </tr>
    <tr>
      <th>MARSHALLTOWN</th>
      <th>2544</th>
      <td>1722256.09</td>
    </tr>
    <tr>
      <th>...</th>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>DES MOINES</th>
      <th>5136</th>
      <td>5881.58</td>
    </tr>
    <tr>
      <th>GLIDDEN</th>
      <th>5205</th>
      <td>5716.18</td>
    </tr>
    <tr>
      <th>ROBINS</th>
      <th>5192</th>
      <td>4767.12</td>
    </tr>
    <tr>
      <th>NORWALK</th>
      <th>5229</th>
      <td>4717.13</td>
    </tr>
    <tr>
      <th>DAVENPORT</th>
      <th>9022</th>
      <td>4643.52</td>
    </tr>
    <tr>
      <th>WATERLOO</th>
      <th>5152</th>
      <td>4562.76</td>
    </tr>
    <tr>
      <th>DAVENPORT</th>
      <th>5130</th>
      <td>4474.96</td>
    </tr>
    <tr>
      <th>COUNCIL BLUFFS</th>
      <th>5195</th>
      <td>4319.69</td>
    </tr>
    <tr>
      <th>MONTROSE</th>
      <th>5213</th>
      <td>4220.56</td>
    </tr>
    <tr>
      <th>WEST BRANCH</th>
      <th>5240</th>
      <td>4207.66</td>
    </tr>
    <tr>
      <th>GILBERTVILLE</th>
      <th>5202</th>
      <td>4207.66</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">DUBUQUE</th>
      <th>5234</th>
      <td>4207.66</td>
    </tr>
    <tr>
      <th>5201</th>
      <td>3778.26</td>
    </tr>
    <tr>
      <th>COUNCIL BLUFFS</th>
      <th>5161</th>
      <td>3711.58</td>
    </tr>
    <tr>
      <th>IOWA CITY</th>
      <th>5053</th>
      <td>3694.38</td>
    </tr>
    <tr>
      <th>TABOR</th>
      <th>5223</th>
      <td>3086.25</td>
    </tr>
    <tr>
      <th>AUDUBON</th>
      <th>5232</th>
      <td>3086.25</td>
    </tr>
    <tr>
      <th>RUNNELLS</th>
      <th>5216</th>
      <td>3086.25</td>
    </tr>
    <tr>
      <th>PLEASANTVILLE</th>
      <th>5233</th>
      <td>3086.25</td>
    </tr>
    <tr>
      <th>GRISWOLD</th>
      <th>4990</th>
      <td>2884.66</td>
    </tr>
    <tr>
      <th>WAPELLO</th>
      <th>4121</th>
      <td>2846.31</td>
    </tr>
    <tr>
      <th>ANTHON</th>
      <th>5228</th>
      <td>2277.72</td>
    </tr>
    <tr>
      <th>ROCKWELL</th>
      <th>5247</th>
      <td>1853.71</td>
    </tr>
    <tr>
      <th>ATLANTIC</th>
      <th>5208</th>
      <td>1836.74</td>
    </tr>
    <tr>
      <th>BELLE PLAINE</th>
      <th>4059</th>
      <td>1813.33</td>
    </tr>
    <tr>
      <th>OKOBOJI</th>
      <th>4486</th>
      <td>1232.38</td>
    </tr>
    <tr>
      <th>URBANDALE</th>
      <th>4939</th>
      <td>1120.70</td>
    </tr>
    <tr>
      <th>COUNCIL BLUFFS</th>
      <th>5056</th>
      <td>973.58</td>
    </tr>
    <tr>
      <th>DES MOINES</th>
      <th>4567</th>
      <td>827.13</td>
    </tr>
    <tr>
      <th>CHARITON</th>
      <th>5218</th>
      <td>603.13</td>
    </tr>
  </tbody>
</table>
<p>1403 rows × 1 columns</p>
</div>




```python

fig, ax = plt.subplots(figsize = (12,8))
ax.set_xlim(xmin = 0, xmax = 50)
plt.xkcd()
sns.distplot(df.groupby('Store Number')['Bottles Sold'].mean(), norm_hist=False, kde = False, hist_kws=dict(edgecolor="k", linewidth=2))
plt.title('# of Bottles Sold by Store')
plt.ylabel('Count')
```




    <matplotlib.text.Text at 0x2b2ab813a90>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_13_1.png)



```python
df.groupby('Store Number')['Bottles Sold'].mean().mean()
```




    10.604507536204743




```python
#price of bottles
fig, ax = plt.subplots(figsize = (12,8))
ax.set_xlim(xmin = 0, xmax = 40)
sns.distplot(df.groupby('Store Number')['State Bottle Retail'].mean(), hist_kws= dict(edgecolor="k", linewidth=2, color = 'yellow'), kde = True, norm_hist = False)
plt.title('Avg Retail Price of Bottles')
plt.ylabel('Count')
plt.xlabel('Retail Price of Bottle for Each Store')
```




    <matplotlib.text.Text at 0x2b2cfad97f0>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_15_1.png)



```python
#Cost of bottles
fig, ax = plt.subplots(figsize = (12,8))
ax.set_xlim(xmin = 0, xmax = 40)
sns.distplot(df.groupby('Store Number')['State Bottle Cost'].mean(), hist_kws=dict(edgecolor="k", linewidth=2, color = 'deepskyblue'), kde = True, norm_hist = False)
plt.title('Cost of Bottles for Store')
plt.ylabel('Count')
plt.xlabel('Each Stores Bottle Cost')
```




    <matplotlib.text.Text at 0x2b2cfbefb70>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_16_1.png)



```python

fig, ax = plt.subplots(figsize = (12,8))
#c1, c2, c3 = sns.color_palette("Set1", 3)
ax.set_xlim(xmin = 0, xmax = 40)
plt.xkcd()
sns.distplot(df.groupby('Store Number')['State Bottle Retail'].mean(),
             hist_kws={"histtype": "stepfilled", "color": "yellow"},
             norm_hist = False,
             kde = True,
             ax = ax,
             label = 'Retail Price')

sns.distplot(df.groupby('Store Number')['State Bottle Cost'].mean(),
             hist_kws={"histtype": "stepfilled", "color": "deepskyblue", 'alpha': .25},
             norm_hist = False,
             kde = True,
             ax = ax,
             label = 'Cost for Store')

plt.title('Retail Price and Cost of Bottles')
plt.ylabel('Density')
plt.xlabel('Each Stores Average Price')
plt.legend()
```




    <matplotlib.legend.Legend at 0x2b2d40024e0>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_17_1.png)



```python
df['State Bottle Retail'].mean(), df['State Bottle Cost'].mean()
```




    (14.740117521272886, 9.8162094287173645)




```python
fig, ax = plt.subplots(figsize = (24,16))
df['Category Name'].value_counts().head(25).plot(kind = 'bar', color = 'lightblue', rot = 75, fontsize = 16)

plt.xlabel('# of Bottles')
plt.ylabel('Type of Liquor')
plt.title('Quantity of Liquor Sold by Type')
```




    <matplotlib.text.Text at 0x2b2d4097cf8>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_19_1.png)



```python
df.groupby('Category Name')[['State Bottle Retail']].mean().sort_values(by = 'State Bottle Retail', ascending = False)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>State Bottle Retail</th>
    </tr>
    <tr>
      <th>Category Name</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>CORN WHISKIES</th>
      <td>269.082906</td>
    </tr>
    <tr>
      <th>HIGH PROOF BEER - AMERICAN</th>
      <td>142.760000</td>
    </tr>
    <tr>
      <th>JAPANESE WHISKY</th>
      <td>52.702248</td>
    </tr>
    <tr>
      <th>AMARETTO - IMPORTED</th>
      <td>46.700000</td>
    </tr>
    <tr>
      <th>SINGLE MALT SCOTCH</th>
      <td>46.012337</td>
    </tr>
    <tr>
      <th>SINGLE BARREL BOURBON WHISKIES</th>
      <td>43.859528</td>
    </tr>
    <tr>
      <th>BOTTLED IN BOND BOURBON</th>
      <td>28.206263</td>
    </tr>
    <tr>
      <th>STRAIGHT RYE WHISKIES</th>
      <td>25.947272</td>
    </tr>
    <tr>
      <th>IRISH WHISKIES</th>
      <td>25.481093</td>
    </tr>
    <tr>
      <th>SCOTCH WHISKIES</th>
      <td>25.477772</td>
    </tr>
    <tr>
      <th>IMPORTED DRY GINS</th>
      <td>22.570780</td>
    </tr>
    <tr>
      <th>IMPORTED GRAPE BRANDIES</th>
      <td>21.830706</td>
    </tr>
    <tr>
      <th>DECANTERS &amp; SPECIALTY PACKAGES</th>
      <td>21.343763</td>
    </tr>
    <tr>
      <th>TENNESSEE WHISKIES</th>
      <td>21.135274</td>
    </tr>
    <tr>
      <th>MISC. IMPORTED CORDIALS &amp; LIQUEURS</th>
      <td>21.125908</td>
    </tr>
    <tr>
      <th>TEQUILA</th>
      <td>20.658319</td>
    </tr>
    <tr>
      <th>IMPORTED VODKA</th>
      <td>20.094232</td>
    </tr>
    <tr>
      <th>IMPORTED AMARETTO</th>
      <td>20.016626</td>
    </tr>
    <tr>
      <th>SCHNAPPS - IMPORTED</th>
      <td>18.720000</td>
    </tr>
    <tr>
      <th>MISCELLANEOUS  BRANDIES</th>
      <td>18.249110</td>
    </tr>
    <tr>
      <th>CREAM LIQUEURS</th>
      <td>17.914352</td>
    </tr>
    <tr>
      <th>WHISKEY LIQUEUR</th>
      <td>17.852488</td>
    </tr>
    <tr>
      <th>STRAIGHT BOURBON WHISKIES</th>
      <td>17.794761</td>
    </tr>
    <tr>
      <th>IMPORTED VODKA - MISC</th>
      <td>17.426485</td>
    </tr>
    <tr>
      <th>JAMAICA RUM</th>
      <td>16.391365</td>
    </tr>
    <tr>
      <th>IMPORTED SCHNAPPS</th>
      <td>16.178381</td>
    </tr>
    <tr>
      <th>COFFEE LIQUEURS</th>
      <td>15.942514</td>
    </tr>
    <tr>
      <th>LOW PROOF VODKA</th>
      <td>15.688201</td>
    </tr>
    <tr>
      <th>DISTILLED SPIRITS SPECIALTY</th>
      <td>15.324125</td>
    </tr>
    <tr>
      <th>BARBADOS RUM</th>
      <td>15.227719</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
    </tr>
    <tr>
      <th>IMPORTED VODKA - CHERRY</th>
      <td>10.580000</td>
    </tr>
    <tr>
      <th>FLAVORED GINS</th>
      <td>10.551435</td>
    </tr>
    <tr>
      <th>MISCELLANEOUS SCHNAPPS</th>
      <td>10.302014</td>
    </tr>
    <tr>
      <th>BLENDED WHISKIES</th>
      <td>10.234614</td>
    </tr>
    <tr>
      <th>APPLE SCHNAPPS</th>
      <td>10.149394</td>
    </tr>
    <tr>
      <th>PEACH SCHNAPPS</th>
      <td>9.859197</td>
    </tr>
    <tr>
      <th>VODKA 80 PROOF</th>
      <td>9.462945</td>
    </tr>
    <tr>
      <th>BUTTERSCOTCH SCHNAPPS</th>
      <td>9.458378</td>
    </tr>
    <tr>
      <th>ROOT BEER SCHNAPPS</th>
      <td>9.368027</td>
    </tr>
    <tr>
      <th>BLACKBERRY BRANDIES</th>
      <td>9.116332</td>
    </tr>
    <tr>
      <th>AMERICAN DRY GINS</th>
      <td>8.896308</td>
    </tr>
    <tr>
      <th>100 PROOF VODKA</th>
      <td>8.808843</td>
    </tr>
    <tr>
      <th>RASPBERRY SCHNAPPS</th>
      <td>8.628365</td>
    </tr>
    <tr>
      <th>STRAWBERRY SCHNAPPS</th>
      <td>8.243275</td>
    </tr>
    <tr>
      <th>AMERICAN GRAPE BRANDIES</th>
      <td>8.136450</td>
    </tr>
    <tr>
      <th>APRICOT BRANDIES</th>
      <td>8.058605</td>
    </tr>
    <tr>
      <th>CHERRY BRANDIES</th>
      <td>7.916450</td>
    </tr>
    <tr>
      <th>TROPICAL FRUIT SCHNAPPS</th>
      <td>7.564541</td>
    </tr>
    <tr>
      <th>AMERICAN SLOE GINS</th>
      <td>7.554810</td>
    </tr>
    <tr>
      <th>SPEARMINT SCHNAPPS</th>
      <td>7.500000</td>
    </tr>
    <tr>
      <th>PEPPERMINT SCHNAPPS</th>
      <td>7.498497</td>
    </tr>
    <tr>
      <th>PEACH BRANDIES</th>
      <td>7.207693</td>
    </tr>
    <tr>
      <th>WHITE CREME DE MENTHE</th>
      <td>7.127856</td>
    </tr>
    <tr>
      <th>ANISETTE</th>
      <td>7.111266</td>
    </tr>
    <tr>
      <th>DARK CREME DE CACAO</th>
      <td>7.098593</td>
    </tr>
    <tr>
      <th>WHITE CREME DE CACAO</th>
      <td>7.034384</td>
    </tr>
    <tr>
      <th>GREEN CREME DE MENTHE</th>
      <td>7.033439</td>
    </tr>
    <tr>
      <th>AMERICAN AMARETTO</th>
      <td>6.835315</td>
    </tr>
    <tr>
      <th>CREME DE ALMOND</th>
      <td>6.808717</td>
    </tr>
    <tr>
      <th>TRIPLE SEC</th>
      <td>4.312756</td>
    </tr>
  </tbody>
</table>
<p>73 rows × 1 columns</p>
</div>




```python
fig, ax = plt.subplots(figsize = (18,12))
df[df['Category Name'] == 'VODKA 80 PROOF']['Item Description'].value_counts().head(10).plot(kind = 'barh', color = 'aqua', fontsize = 24)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x2b2a301ec88>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_21_1.png)



```python
fig, ax = plt.subplots(figsize = (18,12))
df[df['Category Name'] == 'CANADIAN WHISKIES']['Item Description'].value_counts().head(10).plot(kind = 'barh', color = 'gold', fontsize = 24)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x2b2d4133c88>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_22_1.png)



```python
fig, ax = plt.subplots(figsize = (18,12))
df[df['Category Name'] == 'STRAIGHT BOURBON WHISKIES']['Item Description'].value_counts().head(10).plot(kind = 'barh', color = 'purple', fontsize = 24)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x2b2a37898d0>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_23_1.png)


# Record your findings

Be sure to write out anything observations from your exploratory analysis.



```python
# top 5 stores with highest total sales are:
# DES MOINES 	2633 	12282646.26
#             4829 	11085530.53
# IOWA CITY 	2512 	5206377.22
# CEDAR RAPIDS 	3385 	4759187.79
# WINDSOR HEIGHTS 	3420 	4018414.55


# The average number of bottles sold by store per transaction was 10 bottles, there was a heavy positive skew though

# average retail cost was 14.7 - averate store cost 9.8 = 4.9$ margins on each bottle sold

# vodka 80 proof, canadian whiskeys, and straight bourbon whiskies were the most popular type of liquors
# Haweye vodka, black velvet, and Jim bean were the top sellers in those respective categories

```

# Mine the data

Now you are ready to compute the variables you will use for your regression from the data. For example, you may want to compute total sales per store from Jan to March of 2015, mean price per bottle, etc. Refer to the readme for more ideas appropriate to your scenario.

Pandas is your friend for this task. Take a look at the operations here for ideas on how to make the best use of pandas and feel free to search for blog and Stack Overflow posts to help you group data by certain variables and compute sums, means, etc. You may find it useful to create a new data frame to house this summary data.
 



```python
#for our geographic location lets just use data from 2015 since new stores will have sales
df_2015 = df.loc[df['Date'].dt.year == 2015]

```


```python
target = df_2015.groupby('Store Number')[['Sale (Dollars)']].sum().sort_values(by = 'Sale (Dollars)',ascending = False).reset_index()
target.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Store Number</th>
      <th>Sale (Dollars)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2633</td>
      <td>9839393.08</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4829</td>
      <td>8742779.31</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2512</td>
      <td>4155665.47</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3385</td>
      <td>3947176.01</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3420</td>
      <td>3422351.55</td>
    </tr>
  </tbody>
</table>
</div>




```python
#sum sales from jan to march 2015
mask = (df_2015['Date'] >= '2015-01-05') & (df_2015['Date'] <= '2015-03-31')
X = df_2015[mask].groupby('Store Number')[['Sale (Dollars)']].sum().reset_index()
X.rename(columns = {'Sale (Dollars)':'Jan to March Sales'}, inplace = True)
```


```python
X.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Store Number</th>
      <th>Jan to March Sales</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2106</td>
      <td>337166.53</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2113</td>
      <td>22351.86</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2130</td>
      <td>277764.46</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2152</td>
      <td>16805.11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2178</td>
      <td>54411.42</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(target), len(X)
```




    (1283, 1283)




```python
#before we add features, lets see why the length of our target and length of X are different
target = target[target['Store Number'].isin(X['Store Number'])]
```


```python
#adding average bottle retail, average bottle cost, and total volume sold features to my X (predictors)
bottles_sum = df.groupby('Store Number')[['Bottles Sold']].mean().reset_index()
X = X.merge( bottles_sum, how = 'left', on = ('Store Number'))
X = X.merge((df.groupby('Store Number')[['State Bottle Retail']].mean().reset_index()),
            how = 'left', on = ('Store Number'))
X = X.merge((df.groupby('Store Number')[['State Bottle Cost']].mean().reset_index()),
            how = 'left', on = ('Store Number'))
X =X.merge((df.groupby('Store Number')[['Volume Sold (Liters)']].mean().reset_index()),
           how = 'left', on = ('Store Number'))
```


```python
#renaming columns
X.rename(columns = {'State Bottle Cost':'Avg Bottle Cost',
                    'State Bottle Retail':'Avg Bottle Retail',
                    'Volume Sold (Liters)':'Avg Volume Sold (Liters)',
                   'Bottles Sold':'Avg Bottles Sold'},
         inplace = True)
```


```python
X.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Store Number</th>
      <th>Jan to March Sales</th>
      <th>Avg Bottles Sold</th>
      <th>Avg Bottle Retail</th>
      <th>Avg Bottle Cost</th>
      <th>Avg Volume Sold (Liters)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2106</td>
      <td>337166.53</td>
      <td>19.514209</td>
      <td>16.126917</td>
      <td>10.744967</td>
      <td>18.355608</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2113</td>
      <td>22351.86</td>
      <td>4.667047</td>
      <td>15.963609</td>
      <td>10.632919</td>
      <td>4.623090</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2130</td>
      <td>277764.46</td>
      <td>18.523256</td>
      <td>15.386269</td>
      <td>10.251927</td>
      <td>16.787416</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2152</td>
      <td>16805.11</td>
      <td>4.024110</td>
      <td>13.139489</td>
      <td>8.742015</td>
      <td>4.195431</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2178</td>
      <td>54411.42</td>
      <td>7.677192</td>
      <td>15.127811</td>
      <td>10.068841</td>
      <td>8.070549</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2190</td>
      <td>255939.81</td>
      <td>8.285090</td>
      <td>18.142489</td>
      <td>12.086029</td>
      <td>5.056179</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2191</td>
      <td>319020.69</td>
      <td>13.863643</td>
      <td>17.125039</td>
      <td>11.411090</td>
      <td>14.230320</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2200</td>
      <td>45340.33</td>
      <td>4.001452</td>
      <td>17.425620</td>
      <td>11.606667</td>
      <td>4.477271</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2205</td>
      <td>57849.23</td>
      <td>6.048682</td>
      <td>15.036204</td>
      <td>10.013212</td>
      <td>5.043834</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2228</td>
      <td>51031.04</td>
      <td>5.333907</td>
      <td>15.000341</td>
      <td>9.989567</td>
      <td>5.354428</td>
    </tr>
  </tbody>
</table>
</div>



# Refine the data

Look for any statistical relationships, correlations, or other relevant properties of the dataset.


```python
fig, ax = plt.subplots(figsize = (18,12))
sns.set(font_scale=3)
sns.regplot(x = 'Jan to March Sales', y = 'Sale (Dollars)', data = X.merge(target, how = 'left', on = ('Store Number')))
```




    <matplotlib.axes._subplots.AxesSubplot at 0x2b31c195b38>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_37_1.png)



```python
import numpy as np #still cant believe i hadnt imported it before
mean_corr = X.merge(target, how = 'left', on = ('Store Number')).corr()

# Set the default matplotlib figure size:
fig, ax = plt.subplots(figsize=(12,8))

# Generate a mask for the upper triangle (taken from seaborn example gallery)
#mask = np.zeros_like(mean_corr, dtype=np.bool)
#mask[np.triu_indices_from(mask)] = True

# Plot the heatmap with seaborn.
# Assign the matplotlib axis the function returns. This will let us resize the labels.
ax = sns.heatmap(mean_corr, ax=ax)

# Resize the labels.
ax.set_xticklabels(ax.xaxis.get_ticklabels(), fontsize=14, rotation = 'vertical')
ax.set_yticklabels(ax.yaxis.get_ticklabels(), fontsize=14)

# If you put plt.show() at the bottom, it prevents those useless printouts from matplotlib.
plt.show()
```


![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_38_0.png)



```python
mean_corr
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Store Number</th>
      <th>Jan to March Sales</th>
      <th>Avg Bottles Sold</th>
      <th>Avg Bottle Retail</th>
      <th>Avg Bottle Cost</th>
      <th>Avg Volume Sold (Liters)</th>
      <th>Sale (Dollars)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Store Number</th>
      <td>1.000000</td>
      <td>-0.331292</td>
      <td>0.242105</td>
      <td>-0.243479</td>
      <td>-0.243026</td>
      <td>0.036661</td>
      <td>-0.345518</td>
    </tr>
    <tr>
      <th>Jan to March Sales</th>
      <td>-0.331292</td>
      <td>1.000000</td>
      <td>0.268871</td>
      <td>0.291059</td>
      <td>0.291266</td>
      <td>0.374776</td>
      <td>0.990398</td>
    </tr>
    <tr>
      <th>Avg Bottles Sold</th>
      <td>0.242105</td>
      <td>0.268871</td>
      <td>1.000000</td>
      <td>0.029339</td>
      <td>0.029801</td>
      <td>0.832051</td>
      <td>0.281944</td>
    </tr>
    <tr>
      <th>Avg Bottle Retail</th>
      <td>-0.243479</td>
      <td>0.291059</td>
      <td>0.029339</td>
      <td>1.000000</td>
      <td>0.999996</td>
      <td>0.306177</td>
      <td>0.310664</td>
    </tr>
    <tr>
      <th>Avg Bottle Cost</th>
      <td>-0.243026</td>
      <td>0.291266</td>
      <td>0.029801</td>
      <td>0.999996</td>
      <td>1.000000</td>
      <td>0.305964</td>
      <td>0.310872</td>
    </tr>
    <tr>
      <th>Avg Volume Sold (Liters)</th>
      <td>0.036661</td>
      <td>0.374776</td>
      <td>0.832051</td>
      <td>0.306177</td>
      <td>0.305964</td>
      <td>1.000000</td>
      <td>0.395640</td>
    </tr>
    <tr>
      <th>Sale (Dollars)</th>
      <td>-0.345518</td>
      <td>0.990398</td>
      <td>0.281944</td>
      <td>0.310664</td>
      <td>0.310872</td>
      <td>0.395640</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#this takes a while to run, it just visualizes the above correlation matrix
sns.set()
sns.pairplot(X.merge(target, how = 'left', on = ('Store Number')))
```




    <seaborn.axisgrid.PairGrid at 0x2b31e34b0f0>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_40_1.png)


# Build your models

Using scikit-learn or statsmodels, build the necessary models for your scenario. Evaluate model fit.


```python
target.sort_values(by = 'Store Number', inplace = True)
```


```python
X.sort_values(by = 'Store Number', inplace = True)
```


```python
X.drop('Store Number', axis = 'columns', inplace = True)
```


```python
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
scaler.fit(X)
Xs = scaler.transform(X)
```


```python
from sklearn.linear_model import LinearRegression, Ridge, Lasso, RidgeCV, LassoCV
from sklearn.model_selection import cross_val_score, cross_val_predict, train_test_split


X_train, X_test, y_train, y_test = train_test_split(Xs,
                                                    target['Sale (Dollars)'],
                                                    test_size = .25,
                                                    random_state = 42)

lr = LinearRegression()

scores = cross_val_score(lr, X_train, y_train, cv = 10)
scores.mean()

```




    0.96593097488672408




```python
lr.fit(X_train, y_train)
lr_predictions = lr.predict(X_test) #im gonna plot this vs y_test
lr.score(X_test, y_test)

```




    0.97790081290698494




```python
alpha_range = 10.**np.arange(-2, 3)
ridgecv = RidgeCV(alphas = alpha_range, scoring = 'neg_mean_squared_error')
ridgecv.fit(X_train,y_train)
alpha = ridgecv.alpha_
print(alpha)
```

    10.0
    


```python
ridge = Ridge(alpha = alpha)#using the alpha we found using RidgeCV
ridge_scores = cross_val_score(ridge, X_train, y_train, cv = 10)
ridge_scores.mean()
```




    0.96509871690047522




```python
ridge.fit(X_train, y_train)
ridge_predictions = ridge.predict(X_test)
ridge.score(X_test, y_test)
#ridge was pretty good, ,a fraction of a percent better
```




    0.98005592441245293



# Plot your results

Again make sure that you record any valuable information. For example, in the tax scenario, did you find the sales from the first three months of the year to be a good predictor of the total sales for the year? Plot the predictions versus the true values and discuss the successes and limitations of your models


```python
fig, ax = plt.subplots(figsize = (12,8))
ax.scatter(lr_predictions, y_test, label = 'linear regression predictions')
ax.scatter(ridge_predictions, y_test, color = 'red', alpha = .5, marker = '+', label = 'ridge regression predictions')
plt.legend()
```




    <matplotlib.legend.Legend at 0x2b31fe75358>




![png](Project%202%20Ravi%20Mehta_files/Project%202%20Ravi%20Mehta_52_1.png)



```python
# lets throw in 2016 data and predict the 2016 sales
#adding average bottle retail, average bottle cost, and total volume sold features to my X (predictors)
#remember though OUR MODEL WAS TRAINED ON FULL YEAR OF SUMS, SO we divide by 4 for the sum columns.
df_2016 = df.loc[df['Date'].dt.year == 2016]

X_2016 = df_2016.groupby('Store Number')[['Sale (Dollars)']].sum().reset_index()
X_2016.rename(columns = {'Sale (Dollars)':'Jan to March Sales'}, inplace = True)

X_2016 = X_2016.merge( (df_2016.groupby('Store Number')[['Bottles Sold']].mean()).reset_index(),
            how = 'left', on = ('Store Number'))
X_2016 = X_2016.merge((df_2016.groupby('Store Number')[['State Bottle Retail']].mean().reset_index()),
            how = 'left', on = ('Store Number'))
X_2016 = X_2016.merge((df_2016.groupby('Store Number')[['State Bottle Cost']].mean().reset_index()),
            how = 'left', on = ('Store Number'))
X_2016 = X_2016.merge( (df_2016.groupby('Store Number')[['Volume Sold (Liters)']].mean()).reset_index(),
           how = 'left', on = ('Store Number'))
X_2016
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Store Number</th>
      <th>Jan to March Sales</th>
      <th>Bottles Sold</th>
      <th>State Bottle Retail</th>
      <th>State Bottle Cost</th>
      <th>Volume Sold (Liters)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2106</td>
      <td>337804.05</td>
      <td>19.206672</td>
      <td>15.718779</td>
      <td>10.475720</td>
      <td>18.126892</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2113</td>
      <td>21736.63</td>
      <td>4.333333</td>
      <td>15.965471</td>
      <td>10.638728</td>
      <td>4.091781</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2130</td>
      <td>306942.27</td>
      <td>19.087584</td>
      <td>15.243263</td>
      <td>10.159480</td>
      <td>17.490616</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2152</td>
      <td>13752.24</td>
      <td>3.524430</td>
      <td>14.272020</td>
      <td>9.500684</td>
      <td>3.724821</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2178</td>
      <td>58939.90</td>
      <td>7.577629</td>
      <td>15.487229</td>
      <td>10.315693</td>
      <td>7.690551</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2190</td>
      <td>332979.03</td>
      <td>7.991111</td>
      <td>19.315638</td>
      <td>12.872968</td>
      <td>5.266067</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2191</td>
      <td>302592.88</td>
      <td>12.606711</td>
      <td>17.174584</td>
      <td>11.446128</td>
      <td>12.866262</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2200</td>
      <td>55315.23</td>
      <td>4.128587</td>
      <td>17.303486</td>
      <td>11.529957</td>
      <td>4.567014</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2228</td>
      <td>42398.57</td>
      <td>5.489552</td>
      <td>15.053836</td>
      <td>10.029000</td>
      <td>5.212881</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2233</td>
      <td>56241.57</td>
      <td>8.524896</td>
      <td>18.831618</td>
      <td>12.547220</td>
      <td>8.989896</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2238</td>
      <td>63272.07</td>
      <td>35.886792</td>
      <td>24.640189</td>
      <td>16.425189</td>
      <td>30.599057</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2248</td>
      <td>137416.93</td>
      <td>8.542800</td>
      <td>20.861984</td>
      <td>13.902649</td>
      <td>6.220393</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2285</td>
      <td>158700.11</td>
      <td>14.524702</td>
      <td>22.595571</td>
      <td>15.061158</td>
      <td>12.334446</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2290</td>
      <td>125731.51</td>
      <td>3.651503</td>
      <td>15.691851</td>
      <td>10.456689</td>
      <td>3.301911</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2327</td>
      <td>21834.49</td>
      <td>3.898420</td>
      <td>16.039932</td>
      <td>10.684673</td>
      <td>3.710767</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2353</td>
      <td>76339.35</td>
      <td>8.173355</td>
      <td>15.808716</td>
      <td>10.532006</td>
      <td>9.628250</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2413</td>
      <td>204038.87</td>
      <td>12.400930</td>
      <td>16.410363</td>
      <td>10.934009</td>
      <td>12.206651</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2445</td>
      <td>15689.80</td>
      <td>4.030812</td>
      <td>14.251849</td>
      <td>9.498543</td>
      <td>4.042101</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2448</td>
      <td>48545.69</td>
      <td>3.374743</td>
      <td>15.045267</td>
      <td>10.025903</td>
      <td>3.266961</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2459</td>
      <td>16320.74</td>
      <td>5.145749</td>
      <td>13.033765</td>
      <td>8.676154</td>
      <td>5.515466</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2460</td>
      <td>69206.03</td>
      <td>6.815190</td>
      <td>15.102949</td>
      <td>10.060722</td>
      <td>6.873962</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2465</td>
      <td>53182.55</td>
      <td>4.735065</td>
      <td>18.926584</td>
      <td>12.612468</td>
      <td>4.319610</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2475</td>
      <td>30402.43</td>
      <td>9.237209</td>
      <td>16.901395</td>
      <td>11.256372</td>
      <td>12.896605</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2478</td>
      <td>47217.40</td>
      <td>11.764977</td>
      <td>22.648018</td>
      <td>15.097005</td>
      <td>11.078341</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2498</td>
      <td>6367.76</td>
      <td>4.352941</td>
      <td>10.703856</td>
      <td>7.129216</td>
      <td>3.629477</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2500</td>
      <td>329252.09</td>
      <td>6.925632</td>
      <td>15.128327</td>
      <td>10.080509</td>
      <td>6.276105</td>
    </tr>
    <tr>
      <th>26</th>
      <td>2501</td>
      <td>332976.05</td>
      <td>8.569969</td>
      <td>15.543994</td>
      <td>10.355642</td>
      <td>7.788243</td>
    </tr>
    <tr>
      <th>27</th>
      <td>2502</td>
      <td>234926.02</td>
      <td>9.174132</td>
      <td>16.518203</td>
      <td>11.006702</td>
      <td>9.346232</td>
    </tr>
    <tr>
      <th>28</th>
      <td>2505</td>
      <td>151934.45</td>
      <td>7.903409</td>
      <td>14.012237</td>
      <td>9.336740</td>
      <td>8.161946</td>
    </tr>
    <tr>
      <th>29</th>
      <td>2506</td>
      <td>314068.92</td>
      <td>9.797591</td>
      <td>16.113562</td>
      <td>10.736776</td>
      <td>9.628388</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1290</th>
      <td>5212</td>
      <td>4845.15</td>
      <td>4.899281</td>
      <td>8.898921</td>
      <td>5.931079</td>
      <td>2.437482</td>
    </tr>
    <tr>
      <th>1291</th>
      <td>5213</td>
      <td>3540.58</td>
      <td>5.462366</td>
      <td>9.881075</td>
      <td>6.584301</td>
      <td>3.160323</td>
    </tr>
    <tr>
      <th>1292</th>
      <td>5214</td>
      <td>15029.94</td>
      <td>9.411765</td>
      <td>13.627143</td>
      <td>9.079916</td>
      <td>6.523529</td>
    </tr>
    <tr>
      <th>1293</th>
      <td>5215</td>
      <td>25903.47</td>
      <td>4.211462</td>
      <td>15.061561</td>
      <td>10.034783</td>
      <td>3.537648</td>
    </tr>
    <tr>
      <th>1294</th>
      <td>5216</td>
      <td>3086.25</td>
      <td>5.680851</td>
      <td>12.070213</td>
      <td>8.045106</td>
      <td>4.030426</td>
    </tr>
    <tr>
      <th>1295</th>
      <td>5217</td>
      <td>6726.96</td>
      <td>7.642857</td>
      <td>12.294405</td>
      <td>8.192024</td>
      <td>5.101548</td>
    </tr>
    <tr>
      <th>1296</th>
      <td>5218</td>
      <td>603.13</td>
      <td>1.838710</td>
      <td>12.114839</td>
      <td>8.075484</td>
      <td>1.387097</td>
    </tr>
    <tr>
      <th>1297</th>
      <td>5220</td>
      <td>14837.15</td>
      <td>2.714004</td>
      <td>11.964734</td>
      <td>7.974122</td>
      <td>1.961637</td>
    </tr>
    <tr>
      <th>1298</th>
      <td>5222</td>
      <td>88979.97</td>
      <td>25.076712</td>
      <td>14.339425</td>
      <td>9.558219</td>
      <td>9.297096</td>
    </tr>
    <tr>
      <th>1299</th>
      <td>5223</td>
      <td>3086.25</td>
      <td>5.680851</td>
      <td>12.070213</td>
      <td>8.045106</td>
      <td>4.030426</td>
    </tr>
    <tr>
      <th>1300</th>
      <td>5224</td>
      <td>53152.89</td>
      <td>15.576408</td>
      <td>14.485094</td>
      <td>9.654853</td>
      <td>6.980912</td>
    </tr>
    <tr>
      <th>1301</th>
      <td>5225</td>
      <td>9185.23</td>
      <td>6.111111</td>
      <td>14.769524</td>
      <td>9.843492</td>
      <td>3.661667</td>
    </tr>
    <tr>
      <th>1302</th>
      <td>5226</td>
      <td>18114.80</td>
      <td>19.162162</td>
      <td>11.079932</td>
      <td>7.385000</td>
      <td>7.053041</td>
    </tr>
    <tr>
      <th>1303</th>
      <td>5227</td>
      <td>20700.99</td>
      <td>9.061688</td>
      <td>13.370390</td>
      <td>8.912305</td>
      <td>3.638344</td>
    </tr>
    <tr>
      <th>1304</th>
      <td>5228</td>
      <td>2277.72</td>
      <td>14.000000</td>
      <td>13.396667</td>
      <td>8.929333</td>
      <td>10.740000</td>
    </tr>
    <tr>
      <th>1305</th>
      <td>5229</td>
      <td>4717.13</td>
      <td>5.267606</td>
      <td>12.864507</td>
      <td>8.574507</td>
      <td>3.757042</td>
    </tr>
    <tr>
      <th>1306</th>
      <td>5230</td>
      <td>28241.17</td>
      <td>5.610790</td>
      <td>13.860867</td>
      <td>9.238613</td>
      <td>2.689037</td>
    </tr>
    <tr>
      <th>1307</th>
      <td>5232</td>
      <td>3086.25</td>
      <td>5.680851</td>
      <td>12.070213</td>
      <td>8.045106</td>
      <td>4.030426</td>
    </tr>
    <tr>
      <th>1308</th>
      <td>5233</td>
      <td>3086.25</td>
      <td>5.680851</td>
      <td>12.070213</td>
      <td>8.045106</td>
      <td>4.030426</td>
    </tr>
    <tr>
      <th>1309</th>
      <td>5234</td>
      <td>4207.66</td>
      <td>5.583333</td>
      <td>13.002000</td>
      <td>8.666333</td>
      <td>4.080167</td>
    </tr>
    <tr>
      <th>1310</th>
      <td>5236</td>
      <td>47337.36</td>
      <td>5.938069</td>
      <td>16.607887</td>
      <td>11.070182</td>
      <td>5.903133</td>
    </tr>
    <tr>
      <th>1311</th>
      <td>5237</td>
      <td>9301.44</td>
      <td>15.000000</td>
      <td>17.163167</td>
      <td>11.440833</td>
      <td>8.070000</td>
    </tr>
    <tr>
      <th>1312</th>
      <td>5240</td>
      <td>4207.66</td>
      <td>5.583333</td>
      <td>13.002000</td>
      <td>8.666333</td>
      <td>4.080167</td>
    </tr>
    <tr>
      <th>1313</th>
      <td>5247</td>
      <td>1853.71</td>
      <td>3.260000</td>
      <td>13.631000</td>
      <td>9.079400</td>
      <td>3.325000</td>
    </tr>
    <tr>
      <th>1314</th>
      <td>9001</td>
      <td>22355.76</td>
      <td>24.222222</td>
      <td>28.082222</td>
      <td>18.720000</td>
      <td>17.250000</td>
    </tr>
    <tr>
      <th>1315</th>
      <td>9002</td>
      <td>28283.60</td>
      <td>21.651515</td>
      <td>19.884091</td>
      <td>13.253333</td>
      <td>12.414697</td>
    </tr>
    <tr>
      <th>1316</th>
      <td>9010</td>
      <td>697.44</td>
      <td>12.000000</td>
      <td>19.373333</td>
      <td>12.913333</td>
      <td>9.000000</td>
    </tr>
    <tr>
      <th>1317</th>
      <td>9013</td>
      <td>6191.34</td>
      <td>29.117647</td>
      <td>12.823529</td>
      <td>8.547059</td>
      <td>22.720588</td>
    </tr>
    <tr>
      <th>1318</th>
      <td>9022</td>
      <td>651.42</td>
      <td>42.000000</td>
      <td>15.510000</td>
      <td>10.340000</td>
      <td>31.500000</td>
    </tr>
    <tr>
      <th>1319</th>
      <td>9023</td>
      <td>1266.72</td>
      <td>24.000000</td>
      <td>26.390000</td>
      <td>17.590000</td>
      <td>18.000000</td>
    </tr>
  </tbody>
</table>
<p>1320 rows × 6 columns</p>
</div>




```python
X_2016.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1320 entries, 0 to 1319
    Data columns (total 6 columns):
    Store Number            1320 non-null int64
    Jan to March Sales      1320 non-null float64
    Bottles Sold            1320 non-null float64
    State Bottle Retail     1320 non-null float64
    State Bottle Cost       1320 non-null float64
    Volume Sold (Liters)    1320 non-null float64
    dtypes: float64(5), int64(1)
    memory usage: 72.2 KB
    


```python
#dropping stores from our predictor matrix, storing stores in a stores_2016 dataframe which we'll use later to display predicted sales
stores_2016 = X_2016.loc[:,['Store Number']].copy()
X_2016.drop('Store Number', axis = 'columns', inplace = True)
```


```python
Xs_2016 = scaler.transform(X_2016)

predictions_2016 = lr.predict(Xs_2016)
```


```python
stores_2016['2016 Sales'] = predictions_2016
```


```python
stores_2016.head(20)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Store Number</th>
      <th>2016 Sales</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2106</td>
      <td>1.500007e+06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2113</td>
      <td>9.939981e+04</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2130</td>
      <td>1.365142e+06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2152</td>
      <td>4.937831e+04</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2178</td>
      <td>2.648661e+05</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2190</td>
      <td>1.434505e+06</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2191</td>
      <td>1.335979e+06</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2200</td>
      <td>2.466910e+05</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2228</td>
      <td>1.875647e+05</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2233</td>
      <td>2.653044e+05</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2238</td>
      <td>3.705093e+05</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2248</td>
      <td>6.016665e+05</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2285</td>
      <td>7.201480e+05</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2290</td>
      <td>5.417762e+05</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2327</td>
      <td>9.313216e+04</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2353</td>
      <td>3.525462e+05</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2413</td>
      <td>9.059889e+05</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2445</td>
      <td>7.540099e+04</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2448</td>
      <td>2.115764e+05</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2459</td>
      <td>6.697522e+04</td>
    </tr>
  </tbody>
</table>
</div>



# Present the Results

Present your conclusions and results. If you have more than one interesting model feel free to include more than one along with a discussion. Use your work in this notebook to prepare your write-up.


```python
predictions_2016.mean() - target['Sale (Dollars)'].mean()
```




    9215.637315893895




```python
#on average the difference in yearly sales for iowa liquor stores from 2016 to 2015 was 9,215, 
#thats a $9,215 per store on average increase in yearly sales in 2016, very little difference bewteen my Ridge Regression and
# and linear regression models, most likely because jan - March sales were so heavily correlated with total Sales.
```


```python

```


```python

```
