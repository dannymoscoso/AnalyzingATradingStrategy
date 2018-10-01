
# How Well Did My Bot Trade?



For this analysis we only use a few great python3 libraries: pandas, plotly,requests, and datetime<br>
Lets import all of our libraries and sign into plotly


```python
import pandas as pd
import plotly
import plotly.plotly as py
import plotly.offline as py
import plotly.graph_objs as go
import requests
from datetime import datetime, timedelta
#plotly.tools.set_credentials_file(username='dannymoscoso05', api_key='OfRkZGSqKlcgLkBPG6rY')
#plotly.tools.set_config_file(world_readable=True,)
```

The first step is load our data into pandas and fix the dates


```python
df = pd.read_csv('trans01.csv')
df['Date'] = pd.to_datetime(df['Date'])
```

This dataset was taken from my Oanda account transaction history. All these trades were made with a python script I wrote to apply a simple technical analysis strategy, more about this trading strategy can be found here: 
https://tradingstrategyguides.com/parabolic-sar-moving-average-trade-strategy/.
The script was able to make trades based on this strategy, 
however it made a lot more losing trades than winning ones,
lets explore how well it performed.



```python
df.head()
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
      <th>Ticket</th>
      <th>Date</th>
      <th>Timezone</th>
      <th>Transaction</th>
      <th>Details</th>
      <th>Instrument</th>
      <th>Price</th>
      <th>Units</th>
      <th>Direction</th>
      <th>Spread Cost</th>
      <th>Stop Loss</th>
      <th>Take Profit</th>
      <th>Trailing Stop</th>
      <th>Financing</th>
      <th>Commission</th>
      <th>P/L</th>
      <th>Amount</th>
      <th>Balance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>6312</td>
      <td>2018-08-29 17:25:11</td>
      <td>EDT</td>
      <td>ORDER_FILL</td>
      <td>MARKET_ORDER_TRADE_CLOSE</td>
      <td>USD/CAD</td>
      <td>1.29076</td>
      <td>650.0</td>
      <td>Sell</td>
      <td>0.1755</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>-0.0001</td>
      <td>0.0</td>
      <td>-0.2210</td>
      <td>NaN</td>
      <td>81.79</td>
    </tr>
    <tr>
      <th>1</th>
      <td>6313</td>
      <td>2018-08-29 17:25:11</td>
      <td>EDT</td>
      <td>ORDER_CANCEL</td>
      <td>LINKED_TRADE_CLOSED</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6314</td>
      <td>2018-08-29 17:25:11</td>
      <td>EDT</td>
      <td>ORDER_CANCEL</td>
      <td>LINKED_TRADE_CLOSED</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6315</td>
      <td>2018-08-29 17:25:14</td>
      <td>EDT</td>
      <td>MARKET_ORDER</td>
      <td>TRADE_CLOSE</td>
      <td>USD/JPY</td>
      <td>NaN</td>
      <td>650.0</td>
      <td>Sell</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6316</td>
      <td>2018-08-29 17:25:14</td>
      <td>EDT</td>
      <td>ORDER_FILL</td>
      <td>MARKET_ORDER_TRADE_CLOSE</td>
      <td>USD/JPY</td>
      <td>111.67100</td>
      <td>650.0</td>
      <td>Sell</td>
      <td>0.1240</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0019</td>
      <td>0.0</td>
      <td>0.2854</td>
      <td>NaN</td>
      <td>82.08</td>
    </tr>
  </tbody>
</table>
</div>



Since there is a lot going on in this transaction history, I just want to filter out the trades had an account balance.


```python
DateAndBalance = df.loc[df['Balance'] > 0, ['Date','Balance']]
```

This is what our new dataframe looks like:


```python
DateAndBalance.head()
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
      <th>Date</th>
      <th>Balance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2018-08-29 17:25:11</td>
      <td>81.79</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018-08-29 17:25:14</td>
      <td>82.08</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2018-08-29 17:25:45</td>
      <td>82.08</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2018-08-29 17:48:27</td>
      <td>81.70</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2018-08-30 02:56:16</td>
      <td>81.70</td>
    </tr>
  </tbody>
</table>
</div>




```python
trace0 = go.Scatter(
    x = DateAndBalance.Date.tolist(),
    y = DateAndBalance.Balance.tolist(),
    mode = 'lines+markers',
    marker={'color': 'purple'}, 
    name='1st Trace'
)

data = [trace0]
layout=go.Layout(title="Balance of Account", xaxis={'title':'Date Time'}, yaxis={'title':'Balance'})
figure=go.Figure(data=data,layout=layout)
#py.iplot(figure, filename='Balance August29-30')
py.plot(figure, filename='Balance August29-30.html',auto_open=False)
```




    'file:///home/dannymoscoso05/focus/Balance August29-30.html'







```python
df.tail(5)
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
      <th>Ticket</th>
      <th>Date</th>
      <th>Timezone</th>
      <th>Transaction</th>
      <th>Details</th>
      <th>Instrument</th>
      <th>Price</th>
      <th>Units</th>
      <th>Direction</th>
      <th>Spread Cost</th>
      <th>Stop Loss</th>
      <th>Take Profit</th>
      <th>Trailing Stop</th>
      <th>Financing</th>
      <th>Commission</th>
      <th>P/L</th>
      <th>Amount</th>
      <th>Balance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>86</th>
      <td>6398</td>
      <td>2018-08-30 14:08:07</td>
      <td>EDT</td>
      <td>ORDER_CANCEL</td>
      <td>LINKED_TRADE_CLOSED</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>87</th>
      <td>6399</td>
      <td>2018-08-30 14:11:29</td>
      <td>EDT</td>
      <td>MARKET_ORDER</td>
      <td>CLIENT_ORDER</td>
      <td>EUR/USD</td>
      <td>NaN</td>
      <td>375.0</td>
      <td>Buy</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>88</th>
      <td>6400</td>
      <td>2018-08-30 14:11:29</td>
      <td>EDT</td>
      <td>ORDER_FILL</td>
      <td>MARKET_ORDER</td>
      <td>EUR/USD</td>
      <td>1.16686</td>
      <td>375.0</td>
      <td>Buy</td>
      <td>0.0316</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>81.36</td>
    </tr>
    <tr>
      <th>89</th>
      <td>6401</td>
      <td>2018-08-30 14:11:30</td>
      <td>EDT</td>
      <td>TAKE_PROFIT_ORDER</td>
      <td>CLIENT_ORDER</td>
      <td>NaN</td>
      <td>1.16801</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>90</th>
      <td>6402</td>
      <td>2018-08-30 14:11:30</td>
      <td>EDT</td>
      <td>STOP_LOSS_ORDER</td>
      <td>CLIENT_ORDER</td>
      <td>NaN</td>
      <td>1.16551</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
Losses = df.loc[df['P/L'] < 0, ['Date','Instrument','P/L','Balance']]
Profits = df.loc[df['P/L'] > 0, ['Date','Instrument','P/L','Balance']]
print('Lost Trades: '+str(len(Losses)))
print('Winning Trades: '+str(len(Profits)))
```

    Lost Trades: 8
    Winning Trades: 7



```python
Losses['Instrument'].value_counts()
```




    EUR/USD    6
    USD/CAD    2
    Name: Instrument, dtype: int64




```python
Profits['Instrument'].value_counts()
```




    EUR/USD    6
    USD/JPY    1
    Name: Instrument, dtype: int64




```python
pairprofits =  Profits['Instrument'].value_counts()
pairlosses =  Profits['Instrument'].value_counts()
pairs =  Profits['Instrument'].value_counts().index.tolist()
```


```python
profitloss = []
for x in pairs:
    numoflosses = len(Losses.loc[Losses['Instrument'] == x,])
    numofwins = len(Profits.loc[Profits['Instrument'] == x,])
    profitloss.append([x,numoflosses,numofwins])
profitloss.sort(key=lambda x: -x[2])
profitloss
```




    [['EUR/USD', 6, 6], ['USD/JPY', 0, 1]]




```python
fromtimeg = Profits.Date.tolist()[0] - timedelta(hours=6)
fromtimeg = fromtimeg.isoformat().replace(':','%3A') 
tog = Profits.Date.tolist()[0] 
tog = tog.isoformat().replace(':','%3A') 
```


```python
pair = 'USD_CAD'

req = requests.get('https://api-fxpractice.oanda.com/v3/instruments/'+pair+'/candles?price=M&granularity=M1&from='+fromtimeg+'&to='+tog,headers=headers)
get = req.json()
timestamp=[]

o=[]
h=[]
l=[]
c=[]

for x in get['candles']:
    timestamp.append(x['time'])
    o.append(float(x['mid']['o']))
    h.append(float(x['mid']['h']))
    l.append(float(x['mid']['l']))
    c.append(float(x['mid']['c']))

df = pd.DataFrame(
{"timestamp" : timestamp,
"open" : o,
"high" : h,
"low" : l,
"close" : c,
},)
df['timestamp'] = pd.to_datetime(df['timestamp'])
```


```python
len(df)
```




    347




```python
trace0 = go.Scatter(
    x = df.timestamp.tolist(),
    y = df.close.tolist(),
    mode = 'lines+markers',
    marker={'color': 'blue'}, 
    name='1st Trace'

)

data = [trace0]
layout=go.Layout(title="Exploring Price", xaxis={'title':'Date Time'}, yaxis={'title':'Close Price'})
figure=go.Figure(data=data,layout=layout)
py.iplot(figure, filename='Close')
```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~dannymoscoso05/10.embed" height="525px" width="100%"></iframe>




```python
Losses
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
      <th>Date</th>
      <th>Instrument</th>
      <th>P/L</th>
      <th>Balance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2018-08-29 17:25:11</td>
      <td>USD/CAD</td>
      <td>-0.2210</td>
      <td>81.79</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2018-08-29 17:48:27</td>
      <td>USD/CAD</td>
      <td>-0.3770</td>
      <td>81.70</td>
    </tr>
    <tr>
      <th>25</th>
      <td>2018-08-30 03:58:05</td>
      <td>EUR/USD</td>
      <td>-0.6687</td>
      <td>81.59</td>
    </tr>
    <tr>
      <th>37</th>
      <td>2018-08-30 05:34:16</td>
      <td>EUR/USD</td>
      <td>-0.6256</td>
      <td>81.57</td>
    </tr>
    <tr>
      <th>49</th>
      <td>2018-08-30 07:12:16</td>
      <td>EUR/USD</td>
      <td>-0.6593</td>
      <td>81.47</td>
    </tr>
    <tr>
      <th>67</th>
      <td>2018-08-30 10:42:43</td>
      <td>EUR/USD</td>
      <td>-0.6229</td>
      <td>82.07</td>
    </tr>
    <tr>
      <th>73</th>
      <td>2018-08-30 11:24:56</td>
      <td>EUR/USD</td>
      <td>-0.6624</td>
      <td>81.41</td>
    </tr>
    <tr>
      <th>79</th>
      <td>2018-08-30 13:07:18</td>
      <td>EUR/USD</td>
      <td>-0.6181</td>
      <td>80.79</td>
    </tr>
  </tbody>
</table>
</div>




```python
fromtimeg = Losses.Date.tolist()[0] - timedelta(hours=6)
fromtimeg = fromtimeg.isoformat().replace(':','%3A') 
tog = Profits.Date.tolist()[0] 
tog = tog.isoformat().replace(':','%3A') 

pair = 'GBP_USD'

req = requests.get('https://api-fxpractice.oanda.com/v3/instruments/'+pair+'/candles?price=M&granularity=M1&from='+fromtimeg+'&to='+tog,headers=headers)
get = req.json()
timestamp=[]

o=[]
h=[]
l=[]
c=[]

for x in get['candles']:
    timestamp.append(x['time'])
    o.append(float(x['mid']['o']))
    h.append(float(x['mid']['h']))
    l.append(float(x['mid']['l']))
    c.append(float(x['mid']['c']))

df = pd.DataFrame(
{"timestamp" : timestamp,
"open" : o,
"high" : h,
"low" : l,
"close" : c,
},)
df['timestamp'] = pd.to_datetime(df['timestamp'])


trace0 = go.Scatter(
    x = df.timestamp.tolist(),
    y = df.close.tolist(),
    mode = 'lines+markers',
    marker={'color': 'blue'}, 
    name='1st Trace'

)

data = [trace0]
layout=go.Layout(title="Exploring Price", xaxis={'title':'Date Time'}, yaxis={'title':'Close Price'})
figure=go.Figure(data=data,layout=layout)
py.iplot(figure, filename='Close')
```




<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~dannymoscoso05/10.embed" height="525px" width="100%"></iframe>


