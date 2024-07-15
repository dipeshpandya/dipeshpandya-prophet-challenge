# prophet-challenge

The program is aimed to analyze [MercadoLibre](http://investor.mercadolibre.com/about-us) company's financial and user data to help the company grow and find out if the ability to predict search traffic can help to translate into the ability to successfully trade the stock.

The program is divided into (4) steps

* [Step 1](#step-1): Find unusual patterns in hourly Google search traffic

* [Step 2](#step-2): Mine the search traffic data for seasonality

* [Step 3](#step-3): Relate the search traffic to stock price patterns

* [Step 4](#step-4): Create a time series model with Prophet

## Step 1
### Find unusual patterns in hourly Google search traffic

1. At the outset, required libraries and dependencies are imported using following commands

        import pandas as pd
        from prophet import Prophet
        import datetime as dt
        import numpy as np
        %matplotlib inline
        import matplotlib.pyplot as plt

2. Data is imported to a dataframe. While importing data, its ensured that Date column is indexed by using `index_col='Date'` and  `dropna()` is applied to avoid any errors for blank data in the dataset. Using the display function, head and tail of the dataset is viewed together.

        df_mercado_trends = pd.read_csv(
            "https://static.bc-edx.com/ai/ail-v-1-0/m8/lms/datasets/google_hourly_search_trends.csv",
            index_col='Date',
            parse_dates=True
        ).dropna()
        display(df_mercado_trends.head())
        display(df_mercado_trends.tail())

3. Knowing that the company declares results in May, data for May 2020 is sliced to be analyzed against rest of the dataset.
4. Median of the search traffic for each month is arrived and compared with median search traffic for this specific month.
5. We can come to the conclusion that there is visible spike of 8% in this month against rest of the period

## Step 2
### Mine the search traffic data for seasonality

By using the `groupby` function, data is analyzed and plotted to see the seasonality by following time periods

* [hour of the day](/Images/hourly_traffic.png)

        hourly_traffic = df_mercado_trends.groupby(by=[df_mercado_trends.index.hour]).mean()

* [day of the week](/Images/daily_traffic.png)

        daily_traffic = df_mercado_trends.groupby(by=[df_mercado_trends.index.isocalendar().day]).mean()

* [week of the year](/Images/weekly_traffic.png). 

        weekyl_traffic = df_mercado_trends.groupby(by=[df_mercado_trends.index.isocalendar().week]).mean()

These charts indicate the peaks that happen in those respective periods and the peak in traffic can be correlated to results being declared in the month of May

## Step 3
### Identifying relationship between search traffic to stock price movements

We are now aware of the difficult market conditions that emerged in the first half of 2020 that had wide reaching impact. Though after the initial shock to global financial markets, e-commerce platforms experienced rise in new customers and thereby increased revenue increased. 

To get better understanding of stock price movement, stock prices dataframe for first half of 2020 (`2020-01` to `2020-06`) is sliced out and concatenated with search trends data for similar time frame.

        df_mercado_stock = pd.read_csv(
            "https://static.bc-edx.com/ai/ail-v-1-0/m8/lms/datasets/mercado_stock_price.csv",
            index_col="date",
            parse_dates=True
        ).dropna()
        display(df_mercado_stock.head())
        display(df_mercado_stock.tail())

*concatenating the frames together*

        mercado_stock_trends_df = pd.concat([df_mercado_stock, df_mercado_trends],axis=1).dropna()

Utilizing subplots parameters, a combined frame of two plots is created so that both graphs can be viewed on same x-axis

![](/Images/subplots.png)

first_half_df.plot(subplots=True)

To analyze the movement in search trends and volatility in stock price, 

New columns are added in the dataframe for 

Lagged search trends - by creating lag of one hour

        mercado_stock_trends_df1['Lagged Search Trends'] = mercado_stock_trends_df1['Search Trends'].shift(1)

stock volatility - exponentially weighted four-hour rolling average of the company’s [stock volatility](/Images/stock_volatality.png)

        mercado_stock_trends_df1['Stock Volatility'] = mercado_stock_trends_df['close'].pct_change().rolling(window=4).std()

Hourly Stock Return - percent change of the company's stock price on an hourly basis

        mercado_stock_trends_df1['Hourly Stock Return'] = mercado_stock_trends_df1['close'].pct_change()

To review time-series correlation between these three parameters, a [correlation table](/Images/correlation%20table.png) is constructed using following command

    mercado_stock_trends_df1[['Stock Volatility', 'Lagged Search Trends', 'Hourly Stock Return']].corr()

The time series correlation table enables identify predictability of relationship  between the lagged search traffic and the stock volatility or between the lagged search traffic and the hourly stock price returns

## Step 4
### Forecasting Trends

A time series model is produced to analyze and forecast patterns in the hourly search data by using a prebuilt procedure - [Prophet](http://facebook.github.io/prophet/)

Prophet model is stored to an object

        model = Prophet()
        
*Fitting the time series model*
        
        model.fit(df_mercado_trends_new)

*Creating a prediction for approx. (80 days) and storing to a dataframe*

        future_mercado_trends = model.make_future_dataframe(periods=2000, freq='H')

*plot_components function is used to visualize the forecast results by year, day, date, and hour*

![](/Images/components.png)

* The components chart reveals that the site is most popular at the midnight hour. 

* Tuesday is likely to receive most traffic followed by Wednesday and Thursday. 

* October is expected to have least amount traffic followed by peak beginning to form in November. The traffic is expected to peak in mid- December

