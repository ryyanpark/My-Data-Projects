# **My Baby's Development**

With the birth of our first child, my wife and I did not know what to expect. We were new to the game and we weren't sure if our daugther was growing healthy and strong. However, with apps like Hatchbaby, we were able to chart her growth, feeds, and dirty diapers. This data helped us to make sure she was on the right track!

## <u>Data Info</u> 
This data is from my daughter's first 5 months of life (March to early August). She was born 8lbs 3oz / 22in, which is heavy and long for her gender. She was born a big baby and continues to grow really well 👶🏻.

This dataset includes:
* Weight (lb,oz)
* Length (in)
* Feeding Amount (oz)
* Dirty Diaper (stool)

## <u>Questions To Answer:</u>
1. What was my baby's growth like (weight and length)?
2. Did the feeding amount affect her growth?
3. When did feedings become scheduled/consistent?
4. Did feeding amount affect dirty diaper frequency?

##### Initial Setup


```python
# Import Data & modules
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
from matplotlib import style
import datetime
from matplotlib.dates import DateFormatter
from matplotlib.pyplot import figure

df = pd.read_csv('./HatchBaby_updated')
df.head(3)
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
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
      <th>Unnamed: 9</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hidden Name</td>
      <td>03/05/2022 0:00 AM</td>
      <td>03/05/2022 0:00 AM</td>
      <td>Weight</td>
      <td>8.18</td>
      <td>0.8411</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Measured by Nurse</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Hidden Name</td>
      <td>03/05/2022 0:00 AM</td>
      <td>03/05/2022 0:00 AM</td>
      <td>Length</td>
      <td>21.5</td>
      <td>0.9983</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Measured by Nurse</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Hidden Name</td>
      <td>03/06/2022 0:00 AM</td>
      <td>03/06/2022 0:00 AM</td>
      <td>Weight</td>
      <td>7.875</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Measured by Nurse</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Number of Rows
len(df)
```




    1894




```python
# Data Types
df.dtypes
```




    Baby Name      object
    Start Time     object
    End Time       object
    Activity       object
    Amount         object
    Percentile    float64
    Duration      float64
    Info           object
    Notes          object
    Unnamed: 9    float64
    dtype: object




```python
# Check For Nulls
df.isnull().sum()
```




    Baby Name        1
    Start Time       0
    End Time         0
    Activity         0
    Amount           8
    Percentile    1864
    Duration       712
    Info           721
    Notes         1796
    Unnamed: 9    1894
    dtype: int64




```python
# Drop Unused Column
df2 = df.drop(columns = 'Unnamed: 9')
df2.head(1)
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
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hidden Name</td>
      <td>03/05/2022 0:00 AM</td>
      <td>03/05/2022 0:00 AM</td>
      <td>Weight</td>
      <td>8.18</td>
      <td>0.8411</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Measured by Nurse</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 'Activity' types
print(df2['Activity'].unique())
```

    ['Weight' 'Length' 'Feeding' 'Diaper' 'Sleep' 'Pump']



```python
# 'Amount' has non-numeric values; find these values
df2[df2['Amount'].isnull()]
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
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>504</th>
      <td>Hidden Name</td>
      <td>04/12/2022 17:05 PM</td>
      <td>04/12/2022 18:00 PM</td>
      <td>Sleep</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.916667</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>508</th>
      <td>Hidden Name</td>
      <td>04/12/2022 19:05 PM</td>
      <td>04/12/2022 20:18 PM</td>
      <td>Sleep</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.216667</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>863</th>
      <td>Hidden Name</td>
      <td>05/08/2022 15:25 PM</td>
      <td>05/08/2022 16:10 PM</td>
      <td>Sleep</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.750000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>891</th>
      <td>Hidden Name</td>
      <td>05/10/2022 10:51 AM</td>
      <td>05/10/2022 12:21 PM</td>
      <td>Sleep</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.500000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>906</th>
      <td>Hidden Name</td>
      <td>05/10/2022 23:55 PM</td>
      <td>05/11/2022 1:40 AM</td>
      <td>Sleep</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.750000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>907</th>
      <td>Hidden Name</td>
      <td>05/11/2022 2:15 AM</td>
      <td>05/11/2022 2:56 AM</td>
      <td>Sleep</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.683333</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>923</th>
      <td>Hidden Name</td>
      <td>05/12/2022 3:50 AM</td>
      <td>05/12/2022 5:25 AM</td>
      <td>Sleep</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.583333</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>924</th>
      <td>Hidden Name</td>
      <td>05/12/2022 6:10 AM</td>
      <td>05/12/2022 8:40 AM</td>
      <td>Sleep</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.500000</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# 'Amount' has non-numeric values; find these values
df_amount_nn = df2[df2['Amount'].str.match('\d+') == False]
print(df_amount_nn['Amount'].unique())
```

    ['Wet + Dirty' 'Wet' 'Dirty']



```python
# Add 'Date' Column
df2['Date'] = pd.to_datetime(df2['Start Time'], format = '%m/%d/%Y %H:%M %p').dt.floor('d')
#df2['Date'] = pd.to_datetime(df2['Start Time'], format = '%m/%d/%Y %H:%M %p').dt.date
```


```python
# Add 'Month' Column
df2['Month'] = pd.to_datetime(df2['Start Time'], format = '%m/%d/%Y %H:%M %p').dt.month
```


```python
# Add 'DOW' Column
df2['DOW #'] = pd.to_datetime(df2['Start Time'], format = '%m/%d/%Y %H:%M %p').dt.dayofweek
df2['DOW'] = pd.to_datetime(df2['Start Time'], format = '%m/%d/%Y %H:%M %p').dt.day_name()
```


```python
# Add 'Time' Column
df2['Time'] = pd.to_datetime(df2['Start Time'], format = '%m/%d/%Y %H:%M %p').dt.time
```


```python
# Add 'Hour' Column
df2['Hour(int)'] = pd.to_datetime(df2['Start Time'], format = '%m/%d/%Y %H:%M %p').dt.hour
df2['Hour(dt)'] = pd.to_datetime('2022-01-01') + pd.to_timedelta(df2['Hour(int)'], unit='H')
df2['Hour(dt)'] = pd.to_datetime(df2['Hour(dt)'])
```


```python
# Sort
df2 = df2.sort_values('Start Time', ascending=True)
df2.tail(5)
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
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
      <th>Date</th>
      <th>Month</th>
      <th>DOW #</th>
      <th>DOW</th>
      <th>Time</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1890</th>
      <td>Hidden Name</td>
      <td>08/06/2022 15:30 PM</td>
      <td>08/06/2022 15:30 PM</td>
      <td>Feeding</td>
      <td>5</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-08-06</td>
      <td>8</td>
      <td>5</td>
      <td>Saturday</td>
      <td>15:30:00</td>
      <td>15</td>
      <td>2022-01-01 15:00:00</td>
    </tr>
    <tr>
      <th>1892</th>
      <td>Hidden Name</td>
      <td>08/06/2022 18:44 PM</td>
      <td>08/06/2022 18:44 PM</td>
      <td>Feeding</td>
      <td>6</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-08-06</td>
      <td>8</td>
      <td>5</td>
      <td>Saturday</td>
      <td>18:44:00</td>
      <td>18</td>
      <td>2022-01-01 18:00:00</td>
    </tr>
    <tr>
      <th>1888</th>
      <td>Hidden Name</td>
      <td>08/06/2022 5:55 AM</td>
      <td>08/06/2022 5:55 AM</td>
      <td>Feeding</td>
      <td>4</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-08-06</td>
      <td>8</td>
      <td>5</td>
      <td>Saturday</td>
      <td>05:55:00</td>
      <td>5</td>
      <td>2022-01-01 05:00:00</td>
    </tr>
    <tr>
      <th>1887</th>
      <td>Hidden Name</td>
      <td>08/06/2022 6:44 AM</td>
      <td>08/06/2022 6:44 AM</td>
      <td>Feeding</td>
      <td>2</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-08-06</td>
      <td>8</td>
      <td>5</td>
      <td>Saturday</td>
      <td>06:44:00</td>
      <td>6</td>
      <td>2022-01-01 06:00:00</td>
    </tr>
    <tr>
      <th>1889</th>
      <td>Hidden Name</td>
      <td>08/06/2022 9:28 AM</td>
      <td>08/06/2022 9:28 AM</td>
      <td>Feeding</td>
      <td>5.8</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-08-06</td>
      <td>8</td>
      <td>5</td>
      <td>Saturday</td>
      <td>09:28:00</td>
      <td>9</td>
      <td>2022-01-01 09:00:00</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Set Style
plt.style.use('Solarize_Light2')
```

##### Weight Setup


```python
# Filter Weight; Drop Null Values from 'Amount'
df_weight = df2[df2['Activity'] == 'Weight']
df_weight = df_weight[df_weight['Amount'].notna()]
df_weight.head(3)
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
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
      <th>Date</th>
      <th>Month</th>
      <th>DOW #</th>
      <th>DOW</th>
      <th>Time</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hidden Name</td>
      <td>03/05/2022 0:00 AM</td>
      <td>03/05/2022 0:00 AM</td>
      <td>Weight</td>
      <td>8.18</td>
      <td>0.8411</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Measured by Nurse</td>
      <td>2022-03-05</td>
      <td>3</td>
      <td>5</td>
      <td>Saturday</td>
      <td>00:00:00</td>
      <td>0</td>
      <td>2022-01-01</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Hidden Name</td>
      <td>03/06/2022 0:00 AM</td>
      <td>03/06/2022 0:00 AM</td>
      <td>Weight</td>
      <td>7.875</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Measured by Nurse</td>
      <td>2022-03-06</td>
      <td>3</td>
      <td>6</td>
      <td>Sunday</td>
      <td>00:00:00</td>
      <td>0</td>
      <td>2022-01-01</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hidden Name</td>
      <td>03/09/2022 0:00 AM</td>
      <td>03/09/2022 0:00 AM</td>
      <td>Weight</td>
      <td>7.53</td>
      <td>0.5487</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Measured by Nurse</td>
      <td>2022-03-09</td>
      <td>3</td>
      <td>2</td>
      <td>Wednesday</td>
      <td>00:00:00</td>
      <td>0</td>
      <td>2022-01-01</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Convert 'Amount' to Float
df_weight['Amount'] = df_weight['Amount'].astype(float)
```

## Weight


```python
# Plot Weight
figure(figsize=(10,5))
plt.yticks(np.arange(0, (max(df_weight['Amount'])+1), 4))
plt.plot(df_weight['Date'], df_weight['Amount'])
plt.title('Weight')
plt.ylabel('Pounds')
```




    Text(0, 0.5, 'Pounds')




    
![png](output_24_1.png)
    



```python
# Plot Weight Percentile
figure(figsize=(10,5))
df_weight_p = df_weight[df_weight['Percentile'].notna()]
plt.plot(df_weight_p['Date'], df_weight_p['Percentile'])
plt.gca().xaxis.set_major_locator(mdates.MonthLocator(interval=1))
plt.yticks(np.arange(0, 1, .1))
plt.title('Weight Percentile')
```




    Text(0.5, 1.0, 'Weight Percentile')




    
![png](output_25_1.png)
    


The rate of weight gain seems to be more or less consistent. However, there was a short burst sometime in early May before returning to its normal trajectory. We can later check if there was also an increase in feeding during this time, which may have caused the quick weight gain.

My daughter also had jaundice shortly after birth, which is reflected in the weight drop in March. We supplemented her feeding with formula and she has been doing great since! We may have been slightly overfeeding her due to the jaundice scare, causing her to jump to a very high weight percentile. This will be further explored later.

##### Length Setup


```python
# Filter Length; Drop Null Values from 'Amount'
df_len = df2[df2['Activity'] == 'Length']
df_len = df_len[df_len['Amount'].notna()]
df_len.head(3)
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
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
      <th>Date</th>
      <th>Month</th>
      <th>DOW #</th>
      <th>DOW</th>
      <th>Time</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Hidden Name</td>
      <td>03/05/2022 0:00 AM</td>
      <td>03/05/2022 0:00 AM</td>
      <td>Length</td>
      <td>21.5</td>
      <td>0.9983</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Measured by Nurse</td>
      <td>2022-03-05</td>
      <td>3</td>
      <td>5</td>
      <td>Saturday</td>
      <td>00:00:00</td>
      <td>0</td>
      <td>2022-01-01</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Hidden Name</td>
      <td>03/09/2022 0:00 AM</td>
      <td>03/09/2022 0:00 AM</td>
      <td>Length</td>
      <td>21.3</td>
      <td>0.9884</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Measured by Nurse</td>
      <td>2022-03-09</td>
      <td>3</td>
      <td>2</td>
      <td>Wednesday</td>
      <td>00:00:00</td>
      <td>0</td>
      <td>2022-01-01</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Hidden Name</td>
      <td>03/24/2022 0:00 AM</td>
      <td>03/24/2022 0:00 AM</td>
      <td>Length</td>
      <td>22.2</td>
      <td>0.9884</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Measured by Nurse</td>
      <td>2022-03-24</td>
      <td>3</td>
      <td>3</td>
      <td>Thursday</td>
      <td>00:00:00</td>
      <td>0</td>
      <td>2022-01-01</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Convert 'Amount' to Float
df_len['Amount'] = df_len['Amount'].astype(float)
```

## Length


```python
# Plot-Length
figure(figsize=(10,5))
plt.plot(df_len['Date'], df_len['Amount'])
plt.yticks(np.arange(20, (max(df_len['Amount'])+1), 2))
plt.gca().xaxis.set_major_locator(mdates.MonthLocator(interval=1))
plt.title('Length')
plt.ylabel('Inches')
```




    Text(0, 0.5, 'Inches')




    
![png](output_31_1.png)
    



```python
# Plot-Length Percentile
figure(figsize=(10,5))
df_len_p = df_len[df_len['Percentile'].notna()]
plt.plot(df_len_p['Date'], df_len_p['Percentile'])
plt.gca().xaxis.set_major_locator(mdates.MonthLocator(interval=1))
plt.yticks(np.arange(0, 1, .1))
plt.title('Length Percentile')
```




    Text(0.5, 1.0, 'Length Percentile')




    
![png](output_32_1.png)
    


Her rate of length increase seems to be consistent overall, except for the slight March dip, which coincided with the weight drop during jaundice, and another small dip in July. However, this may be negligible as these dips are only off by fractions of an inch. I don't think she actually got shorter. It may be due to the difficulty of getting precise measurements when the baby is squirming and different people (several nurses, me) doing the measuring.

<u>Overall, she seems to always be in the high 90th percentile in length since birth. Weight was lagging due to the jaundice, but it caught up eventually within a few months.</u> With both weight and length percentiles about the same, this seems to be a good sign that her growth is going really well!

##### Feeding Setup


```python
# Filter Feeding; Drop Null Values
df_feed = df2[df2['Activity'] == 'Feeding']
df_feed = df_feed[df_feed['Amount'].notna()]
df_feed.head(3)
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
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
      <th>Date</th>
      <th>Month</th>
      <th>DOW #</th>
      <th>DOW</th>
      <th>Time</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>18</th>
      <td>Hidden Name</td>
      <td>03/13/2022 0:56 AM</td>
      <td>03/13/2022 0:56 AM</td>
      <td>Feeding</td>
      <td>2</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-03-13</td>
      <td>3</td>
      <td>6</td>
      <td>Sunday</td>
      <td>00:56:00</td>
      <td>0</td>
      <td>2022-01-01 00:00:00</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Hidden Name</td>
      <td>03/13/2022 11:45 AM</td>
      <td>03/13/2022 11:45 AM</td>
      <td>Feeding</td>
      <td>2.5</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-03-13</td>
      <td>3</td>
      <td>6</td>
      <td>Sunday</td>
      <td>11:45:00</td>
      <td>11</td>
      <td>2022-01-01 11:00:00</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Hidden Name</td>
      <td>03/13/2022 14:45 PM</td>
      <td>03/13/2022 14:45 PM</td>
      <td>Feeding</td>
      <td>2.5</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-03-13</td>
      <td>3</td>
      <td>6</td>
      <td>Sunday</td>
      <td>14:45:00</td>
      <td>14</td>
      <td>2022-01-01 14:00:00</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Convert 'Amount' to Float
df_feed['Amount'] = df_feed['Amount'].astype(float)
```


```python
# Group & Sum by Day
df_feed_sum_day = df_feed['Amount'].groupby(df_feed['Date']).sum().reset_index(name ='Day Feed Amount')
```

## Feeding

#### *-Sum by Day*


```python
# Plot-Feeding Amount By Day
figure(figsize=(10,5))
plt.yticks(np.arange(0, (max(df_feed_sum_day['Day Feed Amount'])+1), 4))
plt.plot(df_feed_sum_day['Date'], df_feed_sum_day['Day Feed Amount'])
plt.title('Feeding Amount Per Day')
plt.ylabel('Ounces')
```




    Text(0, 0.5, 'Ounces')




    
![png](output_40_1.png)
    


There was a jump of about 10oz in May, which coincides with the slight increase in the rate of weight gain previously mentioned. The feeding amount increased once again in June, until a drop in July to previous levels. This drop was when the doctor told us we needed to slow down on the feeding. <u>This increase in feeding from May to early July is also reflected in the increase in weight during the same time. We can safely conclude that feeding amount strongly affected her weight at the very least, which in turn most likely affected her overall growth.</u>

##### *-Sum by Hour Setup*


```python
# Group & Sum by Hour-Bar
df_feed_sum_hr_bar = df_feed.groupby(['Hour(dt)']).agg({'Amount': ['sum']}).reset_index()
df_feed_sum_hr_bar.sort_values(by=('Amount','sum'),ascending=False).head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>Hour(dt)</th>
      <th>Amount</th>
    </tr>
    <tr>
      <th></th>
      <th></th>
      <th>sum</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>18</th>
      <td>2022-01-01 18:00:00</td>
      <td>291.40</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2022-01-01 12:00:00</td>
      <td>267.70</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2022-01-01 16:00:00</td>
      <td>247.60</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2022-01-01 06:00:00</td>
      <td>242.30</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2022-01-01 15:00:00</td>
      <td>241.75</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2022-01-01 14:00:00</td>
      <td>239.20</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2022-01-01 19:00:00</td>
      <td>237.10</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2022-01-01 09:00:00</td>
      <td>229.80</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2022-01-01 17:00:00</td>
      <td>224.20</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2022-01-01 10:00:00</td>
      <td>216.50</td>
    </tr>
  </tbody>
</table>
</div>



#### *-Sum by Hour*


```python
# Bar-Feeding Amount By Hr
fig, ax_fs_h = plt.subplots(figsize=(10,5))

x_fs_h = df_feed_sum_hr_bar['Hour(dt)']
y_fs_h = df_feed_sum_hr_bar['Amount']['sum']
ax_fs_h.bar(x_fs_h, y_fs_h, width=0.02)

hours = mdates.HourLocator(interval = 2)
hh_mm = mdates.DateFormatter('%H:%M')
ax_fs_h.xaxis.set_major_locator(hours)
ax_fs_h.xaxis.set_major_formatter(hh_mm)

ax_fs_h.set_title('Total Feeding Amount Per Hour')
ax_fs_h.set_ylabel('Ounces')

ax_fs_h.set_xlim(datetime.datetime(2021, 12, 31, 23, 0), datetime.datetime(2022, 1, 1, 23, 59))
#plt.setp(ax_fs_h.get_xticklabels()[-1], visible=False)

fig.autofmt_xdate()
fig.tight_layout()
```


    
![png](output_45_0.png)
    


Top ten hours for total amount were:
1. 6 PM
2. 12 PM
3. 4 PM
4. 6 <u>***AM***</u>
5. 3 PM
6. 2 PM
7. 7 PM
8. 9 <u>***AM***</u>
9. 5 PM
10. 10 <u>***AM***</u>

##### *-Different View*


```python
# Group & Sum by Hour-Plot
df_feed_sum_hr_plot = df_feed['Amount'].groupby(df_feed['Hour(int)']).sum().reset_index(name ='Hour Feed Amount')
df_feed_sum_hr_plot.head()
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
      <th>Hour(int)</th>
      <th>Hour Feed Amount</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>83.3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>71.6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>73.7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>96.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>85.5</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Plot-Feeding Amount by Time
plt.yticks(np.arange(0, (max(df_feed_sum_hr_plot['Hour Feed Amount'])+20), 50))
plt.xticks(np.arange(0, 23, 2))
plt.plot(df_feed_sum_hr_plot['Hour(int)'], df_feed_sum_hr_plot['Hour Feed Amount'])
plt.title('Total Feeding Amount Per Hour')
plt.ylabel('Ounces')
plt.xlabel('Hour')
plt.gcf().autofmt_xdate()
plt.gcf().tight_layout()
```


    
![png](output_49_0.png)
    


##### *-Avg by Hr Setup*


```python
# Group & Avg by Hour
df_feed_avg_hr = df_feed['Amount'].groupby(df_feed['Hour(int)']).mean().reset_index(name ='Hour Feed Avg')

df_feed_avg_hr.sort_values(by='Hour Feed Avg', ascending=False).head(10)
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
      <th>Hour(int)</th>
      <th>Hour Feed Avg</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>12</th>
      <td>12</td>
      <td>4.182812</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>4.108333</td>
    </tr>
    <tr>
      <th>23</th>
      <td>23</td>
      <td>3.997917</td>
    </tr>
    <tr>
      <th>16</th>
      <td>16</td>
      <td>3.993548</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13</td>
      <td>3.967925</td>
    </tr>
    <tr>
      <th>18</th>
      <td>18</td>
      <td>3.937838</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>3.908065</td>
    </tr>
    <tr>
      <th>15</th>
      <td>15</td>
      <td>3.837302</td>
    </tr>
    <tr>
      <th>14</th>
      <td>14</td>
      <td>3.796825</td>
    </tr>
    <tr>
      <th>17</th>
      <td>17</td>
      <td>3.736667</td>
    </tr>
  </tbody>
</table>
</div>



#### *-Avg by Hr*


```python
# Bar-Avg Feeding Amount Per Hr
figure(figsize=(10,5))
plt.yticks(np.arange(0, (max(df_feed_avg_hr['Hour Feed Avg'])+1), 1))
plt.xticks(np.arange(0, 23, 2))
plt.bar(df_feed_avg_hr['Hour(int)'], df_feed_avg_hr['Hour Feed Avg'])
plt.title('Avg Feeding Amount Per Hour')
plt.ylabel('Ounces')
plt.xlabel('Hour')
```




    Text(0.5, 0, 'Hour')




    
![png](output_53_1.png)
    


Top ten hours for average amount were:
1. 12 PM
2. 7 <u>***AM***</u>
3. 11 PM
4. 4 PM
5. 1 PM
6. 6 PM
7. 6 <u>***AM***</u>
8. 3 PM
9. 2 PM
10. 5 PM

12 PM, 4 PM, and 6 PM are in the top six of both total and average amount, but it may be better to analyze these times within separate months. The feeding amount and number of feedings changed each month, and aggregating by month will give us a better picture of favored times.

##### *-Sum by Hour/Month Setup*


```python
# Group & Avg by Month, Hour
df_feed_sum_mth_hr = df_feed.groupby(['Month', 'Hour(int)', 'Hour(dt)']).agg({'Amount': ['sum']})
df_feed_sum_mth_hr = df_feed_sum_mth_hr.reset_index()
df_feed_sum_mth_hr.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>Month</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
      <th>Amount</th>
    </tr>
    <tr>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>sum</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3</td>
      <td>0</td>
      <td>2022-01-01 00:00:00</td>
      <td>24.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>1</td>
      <td>2022-01-01 01:00:00</td>
      <td>9.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2</td>
      <td>2022-01-01 02:00:00</td>
      <td>14.7</td>
    </tr>
  </tbody>
</table>
</div>



#### *-Sum by Hour/Month*


```python
# Bar-Total Feeding Amount by Hour/Month
fig, [[ax1, ax2, ax3], [ax4, ax5, ax6]] = plt.subplots(2, 3, figsize=(25,10), sharey=True)

list_of_axes = [ax1, ax2, ax3, ax4, ax5, ax6]
n = 3
month_name = ['March', 'April', 'May', 'June', 'July', 'August']
i = 0

for ax in list_of_axes:
    df_feed_sum_mth_hr_m = df_feed_sum_mth_hr[df_feed_sum_mth_hr['Month'] == n]
    x = df_feed_sum_mth_hr_m['Hour(dt)']
    y = df_feed_sum_mth_hr_m['Amount']['sum']
    ax.bar(x, y, width=0.03, align='center')
    ax.title.set_text(month_name[i])
    n = n+1
    i = i+1

for ax in list_of_axes:
    hours = mdates.HourLocator(interval=2)
    hh_mm = mdates.DateFormatter('%H:%M')
    ax.xaxis.set_major_locator(hours)
    ax.xaxis.set_major_formatter(hh_mm)
    ax.set_xlim([datetime.datetime(2021,12,31,23,0), datetime.datetime(2022,1,1,23,59)])
    plt.setp(ax.get_xticklabels(), rotation=45)  

ax1.set_ylabel('Ounces', fontsize=16)
ax4.set_ylabel('Ounces', fontsize=16)
fig.suptitle('Total Feeding Amount Per Hour & Month', fontsize=24)

fig.tight_layout()
```


    
![png](output_59_0.png)
    


Total Feeding Amount - Top 3 Hours by Month:

- March: 2 PM, 6 <u>***AM***</u>, 11 <u>***AM***</u>
- April: 5 PM, 3 PM, 3 <u>***AM***</u>
- May: 4 PM,  1 PM, 6 PM
- June: 7 PM, 12 PM, 2 PM
- July: 6 PM, 7 <u>***AM***</u>, 3 PM
- August: 12 PM, 6 <u>***AM***</u>, 9 <u>***AM***</u>

##### *-Rank*

##### *-Avg by Hour/Month Setup*


```python
df_feed_sum_mth_hr[df_feed_sum_mth_hr['Month'] == 8].sort_values(by=('Amount','sum'),ascending=False).head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>Month</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
      <th>Amount</th>
    </tr>
    <tr>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>sum</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>119</th>
      <td>8</td>
      <td>12</td>
      <td>2022-01-01 12:00:00</td>
      <td>28.75</td>
    </tr>
    <tr>
      <th>115</th>
      <td>8</td>
      <td>6</td>
      <td>2022-01-01 06:00:00</td>
      <td>26.00</td>
    </tr>
    <tr>
      <th>117</th>
      <td>8</td>
      <td>9</td>
      <td>2022-01-01 09:00:00</td>
      <td>25.80</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Group & Avg by Month, Hour
df_feed_avg_mth_hr = df_feed.groupby(['Month', 'Hour(int)', 'Hour(dt)']).agg({'Amount': ['mean']})
df_feed_avg_mth_hr = df_feed_avg_mth_hr.reset_index()

df_feed_avg_mth_hr.head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>Month</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
      <th>Amount</th>
    </tr>
    <tr>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3</td>
      <td>0</td>
      <td>2022-01-01 00:00:00</td>
      <td>2.227273</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>1</td>
      <td>2022-01-01 01:00:00</td>
      <td>2.250000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>2</td>
      <td>2022-01-01 02:00:00</td>
      <td>2.100000</td>
    </tr>
  </tbody>
</table>
</div>



#### *-Avg by Hour/Month*


```python
# Bar-Avg Feeding Amount by Hour/Month
fig, [[ax1, ax2, ax3], [ax4, ax5, ax6]] = plt.subplots(2, 3, figsize=(25,10), sharey=True)

list_of_axes = [ax1, ax2, ax3, ax4, ax5, ax6]
n = 3
month_name = ['March', 'April', 'May', 'June', 'July', 'August']
i = 0

for ax in list_of_axes:
    df_feed_avg_mth_hr_m = df_feed_avg_mth_hr[df_feed_avg_mth_hr['Month'] == n]
    x = df_feed_avg_mth_hr_m['Hour(dt)']
    y = df_feed_avg_mth_hr_m['Amount']['mean']
    ax.bar(x, y, width=0.03)
    ax.title.set_text(month_name[i])
    n = n+1
    i = i+1

for ax in list_of_axes:
    hours = mdates.HourLocator(interval=2)
    hh_mm = mdates.DateFormatter('%H:%M')
    ax.xaxis.set_major_locator(hours)
    ax.xaxis.set_major_formatter(hh_mm)
    ax.set_xlim([datetime.datetime(2021,12,31,23,0), datetime.datetime(2022,1,1,23,59)])
    plt.setp(ax.get_xticklabels(), rotation=45)
    
ax1.set_ylabel('Ounces', fontsize=16)
ax4.set_ylabel('Ounces', fontsize=16)
fig.suptitle('Avg Feeding Amount Per Hour & Month', fontsize=24)

fig.tight_layout()
```


    
![png](output_66_0.png)
    


Total Feeding Amount - Top 3 Hours by Month:

- March: 11 PM, 8 PM, 11 <u>***AM***</u>
- April: 1 PM, 5 <u>***AM***</u>, 1 <u>***AM***</u>
- May: 3 <u>***AM***</u>,  5 PM, 12 <u>***AM***</u>
- June: 12 <u>***AM***</u>, 12 PM, 11 <u>***AM***</u>
- July: 5 <u>***AM***</u>, 9 PM, 11 PM
- August: 1 PM, 4 PM, 3 PM

For the first few months, the average feeding times seem to be spread out more or less evenly. However, as June rolls around, a couple late night feeding hours are completely dropped while some hours are slowly being established. At this point, we're slowly starting to get a rhythm and an idea of when she likes to feed. By August, we can see that most of the late night feedings have dropped. Sleep training had been completed and she started sleeping through the night! Hooray! <u>A much more consistent feeding schedule has finally appeared by 5 months.</u>

##### *-Rank*


```python
df_feed_avg_mth_hr[df_feed_avg_mth_hr['Month'] == 8].sort_values(by=('Amount','mean'),ascending=False).head(3)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>Month</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
      <th>Amount</th>
    </tr>
    <tr>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>120</th>
      <td>8</td>
      <td>13</td>
      <td>2022-01-01 13:00:00</td>
      <td>6.000</td>
    </tr>
    <tr>
      <th>123</th>
      <td>8</td>
      <td>16</td>
      <td>2022-01-01 16:00:00</td>
      <td>6.000</td>
    </tr>
    <tr>
      <th>122</th>
      <td>8</td>
      <td>15</td>
      <td>2022-01-01 15:00:00</td>
      <td>5.425</td>
    </tr>
  </tbody>
</table>
</div>



#### *Number of Feedings Per Day-Setup*


```python
df_feed.head(3)
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
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
      <th>Date</th>
      <th>Month</th>
      <th>DOW #</th>
      <th>DOW</th>
      <th>Time</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>18</th>
      <td>Hidden Name</td>
      <td>03/13/2022 0:56 AM</td>
      <td>03/13/2022 0:56 AM</td>
      <td>Feeding</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-03-13</td>
      <td>3</td>
      <td>6</td>
      <td>Sunday</td>
      <td>00:56:00</td>
      <td>0</td>
      <td>2022-01-01 00:00:00</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Hidden Name</td>
      <td>03/13/2022 11:45 AM</td>
      <td>03/13/2022 11:45 AM</td>
      <td>Feeding</td>
      <td>2.5</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-03-13</td>
      <td>3</td>
      <td>6</td>
      <td>Sunday</td>
      <td>11:45:00</td>
      <td>11</td>
      <td>2022-01-01 11:00:00</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Hidden Name</td>
      <td>03/13/2022 14:45 PM</td>
      <td>03/13/2022 14:45 PM</td>
      <td>Feeding</td>
      <td>2.5</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-03-13</td>
      <td>3</td>
      <td>6</td>
      <td>Sunday</td>
      <td>14:45:00</td>
      <td>14</td>
      <td>2022-01-01 14:00:00</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_feed.groupby(['Date','Hour(int)']).agg('count').tail(10)
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
      <th></th>
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
      <th>Month</th>
      <th>DOW #</th>
      <th>DOW</th>
      <th>Time</th>
      <th>Hour(dt)</th>
    </tr>
    <tr>
      <th>Date</th>
      <th>Hour(int)</th>
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
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">2022-08-05</th>
      <th>6</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th rowspan="6" valign="top">2022-08-06</th>
      <th>5</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_num_feed = df_feed.groupby(['Date','Hour(int)']).agg('count')
```


```python
df_test = df_num_feed.groupby(level=0).count()['Baby Name']
df_test.loc['2022-04-01':'2022-04-15']
```




    Date
    2022-04-01    12
    2022-04-02    11
    2022-04-03    14
    2022-04-04    14
    2022-04-05    11
    2022-04-06    14
    2022-04-07    10
    2022-04-08    13
    2022-04-09    13
    2022-04-10    10
    2022-04-11    14
    2022-04-12     9
    2022-04-13     9
    2022-04-14    10
    2022-04-15     9
    Name: Baby Name, dtype: int64




```python
df_num_feed.loc['2022-04-01':'2022-04-15'].head(13)
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
      <th></th>
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
      <th>Month</th>
      <th>DOW #</th>
      <th>DOW</th>
      <th>Time</th>
      <th>Hour(dt)</th>
    </tr>
    <tr>
      <th>Date</th>
      <th>Hour(int)</th>
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
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="12" valign="top">2022-04-01</th>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2022-04-02</th>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_feed[(df_feed['Date']=='2022-04-01')]
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
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
      <th>Date</th>
      <th>Month</th>
      <th>DOW #</th>
      <th>DOW</th>
      <th>Time</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>307</th>
      <td>Hidden Name</td>
      <td>04/01/2022 0:10 AM</td>
      <td>04/01/2022 0:20 AM</td>
      <td>Feeding</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>10.0</td>
      <td>Left</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>00:10:00</td>
      <td>0</td>
      <td>2022-01-01 00:00:00</td>
    </tr>
    <tr>
      <th>314</th>
      <td>Hidden Name</td>
      <td>04/01/2022 12:00 PM</td>
      <td>04/01/2022 12:00 PM</td>
      <td>Feeding</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>12:00:00</td>
      <td>12</td>
      <td>2022-01-01 12:00:00</td>
    </tr>
    <tr>
      <th>315</th>
      <td>Hidden Name</td>
      <td>04/01/2022 14:44 PM</td>
      <td>04/01/2022 14:44 PM</td>
      <td>Feeding</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>14:44:00</td>
      <td>14</td>
      <td>2022-01-01 14:00:00</td>
    </tr>
    <tr>
      <th>317</th>
      <td>Hidden Name</td>
      <td>04/01/2022 15:40 PM</td>
      <td>04/01/2022 15:48 PM</td>
      <td>Feeding</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>8.0</td>
      <td>Left</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>15:40:00</td>
      <td>15</td>
      <td>2022-01-01 15:00:00</td>
    </tr>
    <tr>
      <th>319</th>
      <td>Hidden Name</td>
      <td>04/01/2022 18:45 PM</td>
      <td>04/01/2022 18:45 PM</td>
      <td>Feeding</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>18:45:00</td>
      <td>18</td>
      <td>2022-01-01 18:00:00</td>
    </tr>
    <tr>
      <th>322</th>
      <td>Hidden Name</td>
      <td>04/01/2022 20:20 PM</td>
      <td>04/01/2022 20:36 PM</td>
      <td>Feeding</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>16.0</td>
      <td>Both</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>20:20:00</td>
      <td>20</td>
      <td>2022-01-01 20:00:00</td>
    </tr>
    <tr>
      <th>320</th>
      <td>Hidden Name</td>
      <td>04/01/2022 20:30 PM</td>
      <td>04/01/2022 20:30 PM</td>
      <td>Feeding</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>20:30:00</td>
      <td>20</td>
      <td>2022-01-01 20:00:00</td>
    </tr>
    <tr>
      <th>323</th>
      <td>Hidden Name</td>
      <td>04/01/2022 21:30 PM</td>
      <td>04/01/2022 21:30 PM</td>
      <td>Feeding</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>21:30:00</td>
      <td>21</td>
      <td>2022-01-01 21:00:00</td>
    </tr>
    <tr>
      <th>324</th>
      <td>Hidden Name</td>
      <td>04/01/2022 21:45 PM</td>
      <td>04/01/2022 21:45 PM</td>
      <td>Feeding</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>21:45:00</td>
      <td>21</td>
      <td>2022-01-01 21:00:00</td>
    </tr>
    <tr>
      <th>325</th>
      <td>Hidden Name</td>
      <td>04/01/2022 22:00 PM</td>
      <td>04/01/2022 22:00 PM</td>
      <td>Feeding</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>22:00:00</td>
      <td>22</td>
      <td>2022-01-01 22:00:00</td>
    </tr>
    <tr>
      <th>326</th>
      <td>Hidden Name</td>
      <td>04/01/2022 22:40 PM</td>
      <td>04/01/2022 22:40 PM</td>
      <td>Feeding</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>22:40:00</td>
      <td>22</td>
      <td>2022-01-01 22:00:00</td>
    </tr>
    <tr>
      <th>308</th>
      <td>Hidden Name</td>
      <td>04/01/2022 2:42 AM</td>
      <td>04/01/2022 2:42 AM</td>
      <td>Feeding</td>
      <td>2.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>02:42:00</td>
      <td>2</td>
      <td>2022-01-01 02:00:00</td>
    </tr>
    <tr>
      <th>310</th>
      <td>Hidden Name</td>
      <td>04/01/2022 5:52 AM</td>
      <td>04/01/2022 5:52 AM</td>
      <td>Feeding</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>05:52:00</td>
      <td>5</td>
      <td>2022-01-01 05:00:00</td>
    </tr>
    <tr>
      <th>311</th>
      <td>Hidden Name</td>
      <td>04/01/2022 8:00 AM</td>
      <td>04/01/2022 8:10 AM</td>
      <td>Feeding</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>10.0</td>
      <td>Left</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>08:00:00</td>
      <td>8</td>
      <td>2022-01-01 08:00:00</td>
    </tr>
    <tr>
      <th>312</th>
      <td>Hidden Name</td>
      <td>04/01/2022 9:29 AM</td>
      <td>04/01/2022 9:29 AM</td>
      <td>Feeding</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>Bottle</td>
      <td>NaN</td>
      <td>2022-04-01</td>
      <td>4</td>
      <td>4</td>
      <td>Friday</td>
      <td>09:29:00</td>
      <td>9</td>
      <td>2022-01-01 09:00:00</td>
    </tr>
  </tbody>
</table>
</div>



#### *-Number of Feedings Per Day*


```python
# Plot-Feeding Amount by Time
x = df_num_feed.groupby(level=0).count().index
y = df_num_feed.groupby(level=0).count()['Baby Name']

figure(figsize=(10,5))
plt.yticks(np.arange(0, (max(y)+2), 2))
plt.plot(x,y)
plt.title('Number of Feedings Per Day')
plt.ylabel('Number of Feedings')
plt.gcf().tight_layout()
```


    
![png](output_79_0.png)
    


Since we recorded bottle and breastfeedings separately, this will give us a larger number of feedings than reality. For example, we may have given the bottle initially, but if she was still hungry, she was given more through breastfeeding. Even though this should be a single feeding, it was recorded twice because the app cannot combine different types of feeding methods. Therefore, I made these into single feedings by unifying them to each hour.

Even with the adjustment, the data shows that there were a large number feedings per day in the first month or so. Newborns can only drink a few ounces at a time, hence more feedings per day, but this was also due to our inexperience. If she was crying, we were unsure if she was still hungry, sleepy, uncomfortable, or something else. This caused many moments where we mistook her cries as feeding cues and attempted to feed her, causing more feeding records. Eventually, we got the hang of it! As she grew, she could drink more per feeding and the number of feedings stabilized to something manageable and consistent.

#### Dirty Diapers Setup


```python
# Filter Diapers, Dirty
df_diapers = df2[df2['Activity']=='Diaper']
df_drty_diapers = df_diapers[df_diapers['Amount'].str.contains('Dirty')]
df_drty_diapers.tail(3)
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
      <th>Baby Name</th>
      <th>Start Time</th>
      <th>End Time</th>
      <th>Activity</th>
      <th>Amount</th>
      <th>Percentile</th>
      <th>Duration</th>
      <th>Info</th>
      <th>Notes</th>
      <th>Date</th>
      <th>Month</th>
      <th>DOW #</th>
      <th>DOW</th>
      <th>Time</th>
      <th>Hour(int)</th>
      <th>Hour(dt)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1852</th>
      <td>Hidden Name</td>
      <td>07/31/2022 14:23 PM</td>
      <td>07/31/2022 14:23 PM</td>
      <td>Diaper</td>
      <td>Dirty</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2022-07-31</td>
      <td>7</td>
      <td>6</td>
      <td>Sunday</td>
      <td>14:23:00</td>
      <td>14</td>
      <td>2022-01-01 14:00:00</td>
    </tr>
    <tr>
      <th>1868</th>
      <td>Hidden Name</td>
      <td>08/03/2022 8:44 AM</td>
      <td>08/03/2022 8:44 AM</td>
      <td>Diaper</td>
      <td>Dirty</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2022-08-03</td>
      <td>8</td>
      <td>2</td>
      <td>Wednesday</td>
      <td>08:44:00</td>
      <td>8</td>
      <td>2022-01-01 08:00:00</td>
    </tr>
    <tr>
      <th>1886</th>
      <td>Hidden Name</td>
      <td>08/05/2022 17:20 PM</td>
      <td>08/05/2022 17:20 PM</td>
      <td>Diaper</td>
      <td>Dirty</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2022-08-05</td>
      <td>8</td>
      <td>4</td>
      <td>Friday</td>
      <td>17:20:00</td>
      <td>17</td>
      <td>2022-01-01 17:00:00</td>
    </tr>
  </tbody>
</table>
</div>



## Dirty Diapers


```python
# Bar-Total Dirty Diapers by DOW
figure(figsize=(10,5))
ax = pd.value_counts(df_drty_diapers['DOW']).plot.bar()
ax.set_title('Total Dirty Diapers By DOW', fontsize=22)
ax.set_ylabel('Diapers')
ax.tick_params(axis='x', rotation=45)
```


    
![png](output_84_0.png)
    



```python
# Plot-Dirty Diaper Frequency
df_drty_diapers_plot = df_drty_diapers.copy()
df_drty_diapers_plot['Amount'] = df_drty_diapers['Amount'].replace('Wet + Dirty', 'Dirty')

x = df_drty_diapers_plot['Date']
y = df_drty_diapers_plot['Amount']

fig, ax_dd = plt.subplots(1,1, figsize=(25,5))

plt.xlim([datetime.date(2022, 3, 5), datetime.date(2022, 8, 6)])
plt.tick_params(axis='x', rotation=45)
plt.gca().xaxis.set_major_locator(mdates.DayLocator(interval=3))
ax_dd.xaxis.set_major_formatter(mdates.DateFormatter('%m/%d'))
ax_dd.set_title('Dirty Diaper Frequency', fontsize=20)
ax_dd = plt.scatter(x,y)
```


    
![png](output_85_0.png)
    


##### Dirty Diaper Totals by Month


```python
df_drty_diapers_mth = df_drty_diapers_plot.groupby('Month')
df_drty_diapers_mth['Amount'].count()
```




    Month
    3    41
    4    31
    5    37
    6    18
    7    18
    8     2
    Name: Amount, dtype: int64



##### Estimate Dirty Diaper Frequency by Month


```python
df_drty_diapers_mth['Amount'].count().div(31)
```




    Month
    3    1.322581
    4    1.000000
    5    1.193548
    6    0.580645
    7    0.580645
    8    0.064516
    Name: Amount, dtype: float64



Our daughter's first dirty diaper came on March 13, more than a week after birth. It was certainly relieving to finally have that first dirty diaper, and it became consistent after that. However, it seems that as she got older, she had less frequent dirty diapers. She went from averaging at least one dirty diaper a day to only one dirty diaper about every other day. <u>It does seem that around the time we started to feed her more (May), she slowly started to have less dirty diapers.</u> Could this be due to other factors such as feeding more formula rather than breastmilk? Further analysis is required, but that's for another day!

## Conclusion

Overall, I am so glad that my daughter is growing healthy and strong. From this analysis, it looks like we are on the right track raising our child. Thanks for coming along on this journey with me!
