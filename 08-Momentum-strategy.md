# Momentum or directional trading 

In the last session, we covered the "Long samples test of efficient markets hypothesis (EMH) applied to portfolio". as you remember, we filter to get only those stocks for which the EMH fails, or the test suggests that it is an inefficient market, then we can make a prediction based on past information. 

The final goal (that we expect to cover in the final session) is to create a portfolio of filtered stocks and submit it to a trading platform, such as interactive brokers.

For this session, we will make another filter to get those stocks with high expected performance. We will apply a trading strategy called Momentum. That strategy consists of buying stocks when the instrument is trending up or selling when is down. The idea is that historical winners are expected to be winners and historical losers are expected to lose in the short run. The momentum effect is considered a market anomaly (see Cervantes, M., Montoya, M. Á., & Bernal, L.A. (2016)) 

Note. We also could apply a machine learning technique to predict stocks and make a filter to get the ones with better risk-reward expected performance, however, probably you covered that topic in the UF Algorithms and Data Analysis, as I did in the 2nd period. 




We will create a signal to buy or sell, based on variables created from past information (that is the reason why we made the EMH test). Finally, we will cover how to make the back-testing using historical data. 

    
Some of the code and methodology is based on Based on: Jeet and Vats Learning (2017). Other code is from my own authorship. 
  
First we read the data

```r
library(openxlsx)
library(quantmod)

data<-read.xlsx("dfx_2.xlsx")

# The following codes are to transform the data frame date into xts (as we did last session)
date<-data[,1]

# liminating the fits column
data<-data[,-1]

#Applying the xts function
datax<- xts(data,
         order.by = as.Date(date))

# Eliminating rows with missing values
datax<-na.omit(datax)

#Also we estimate the returns for each stock, as we did in last session
ret<-apply(datax,2,Delt)

# we lost the xts 
retx<- xts(ret,
         order.by = as.Date(date))
# Eliminating rows with missing values
retx<-na.omit(retx)
```


To back-test the momentum strategy, we have to divide the data set into two smaller data sets called in-sample and out-sample data sets. Something similar to the training and test in machine learning. 

we define four dates. in_sd defines the date by which the in-sample data starts, and in_ed the in-sample end date. Similarly, out_sd and out_ed are defined for the out-sample start and end dates. The dates are defined in order as our data is in time series format and we are interested in building a model on historical data which would be used on real-time data, that is, a data set which has dates later than historical data:

The in sample could 1-3 years and the out sample 1 year. 

```r
in_sd<- "2018-05-26"
in_ed<- "2021-05-26" 

out_sd<- "2021-05-27"
out_ed<- "2022-05-27"
```

To back-test the momentum strategy, we will divide the data set into two: in-sample and out-sample. Something like the training and test in machine learning. The back-testing is for proving the strategy performance before implementing it, without having to wait days, or months to see if it works. Also, it helps to control human bias towards parameter estimation. We should use in-sample data to back-test our strategy, estimate the optimal set of parameters, and evaluate its performance.


The in-sample data is for estimating the optimal set of parameters, and evaluate its performance. The optimal set of parameters must be applied on out-sample data to understand the generalization capacity of rules and parameters. If the performance on out-sample data is pretty similar to in-sample data, we assume the parameters and rule set have good generalization power and can be used for live trading.



We create the in and out subsample, for the price and for the return, for datax and retx. 

subset(data,
  +index(data)>= initial date &
  +index(data)<= final date)

```r
in_sd<- "2018-05-26"  # están en la 59
in_ed<- "2021-05-26" 
# in_sample of the price of the stocks
in_datax<- subset(datax,
  +index(datax)>= in_sd &
  +index(datax)<= in_ed)
# in_sample of the returns
in_ret<- subset(retx,
  +index(retx)>= in_sd &
  +index(retx)<= in_ed)

out_datax<- subset(datax,
  +index(datax)>= out_sd &
  +index(datax)<= out_ed)
# in_sample of the returns
out_ret<- subset(retx,
  +index(retx)>= out_sd &
  +index(retx)<= out_ed)
```


We will use moving average convergence divergence (MACD) and Bollinger band indicators to generate automated trading signals. 

MACD and Bollinger band indicators are calculated using the following two lines of code. I used the same parameter values in both of these functions; however, you can use the parameters which you think are best for your dataset. The output variable macd contains the MACD indicator and its signal value; however, the output variable bb contains the lower band, average, upper band, and percentage Bollinger band:

Here is where we use the 26 days lags of the ArchTest function. We are generating a MACD for 26 lags, implying that we could use the information of the past 26 days to predict a signal for buying or selling a stock. 

We are going to star with a single stock, the one of the first column, to understand what we are doing. After we will apply the procedure to all the stocks. 

We apply it for the first stock price, in_sample[,1] 
MACD(data, nFast = , nSlow =  , nSig = ,maType="SMA")   

usually nfast is 12 and nSlow 26 and nsig 9.

BBands(data, n = , maType="SMA", sd = )
usually n=20 and se=2

```r
macd<-MACD(in_datax[,3] , nFast =12 , nSlow =26  , nSig =9 ,maType="SMA")

bb<-BBands(in_datax[,3], n = 20, maType="SMA", sd = 2)
```


Now we create the variable signal and initializes it with NULL. In the second line, I generated a buy signal (1) when dji is above the upper Bollinger band and the macd value is above its macd-signal value; a sell signal (-1) when dji is down the lower Bollinger band and macd is less than its macd-signal value; and out of market when the signal is 0:

signal <- ifelse(in_data[,col]> bb[,'up'] &macd[,'macd'] >macd[,'signal'],1,ifelse(in_sample[,col]< bb[,'dn'] &macd[,'macd'] <macd[,'signal'],-1,0))
where "col" is the column number


```r
signal <- ifelse(in_datax[,1]> bb[,'up'] & macd[,'macd'] >macd[,'signal'],1,ifelse(in_datax[,1]< bb[,'dn'] &macd[,'macd'] <macd[,'signal'],-1,0))
plot(signal[,1])
```

<img src="08-Momentum-strategy_files/figure-html/unnamed-chunk-5-1.png" width="90%" style="display: block; margin: auto;" />


 We can modify this signal generation mechanism and use any other criterion. We haven't included any transaction cost and slippage cost to calculate its performance as none of the strategies are directly for trading. 



To estimate the strategy return, we will use the return and the previous day signal. I like assuming that, if is a buying signal, we buy today and sell it tomorrow, making a one day profit. 
 
in_return[,col]*(stats::lag(signal[,col]))
where "col" is the column number

```r
strat_ret<-in_ret[,1]*(stats::lag(signal[,1]))
```

We will use the package PerformanceAnalytics to calculate strategy performance such as annual return.
Estimating Returns 

Annualized geometric return

$$ geometric\ return =\prod_{i=1}^{n} (1+HPR)^{scale/n}$$
where $\prod_{i=1}^{n} (1+HPR)$ is the product of  (1+HPR). Also,  n is the number of observations, scale is number of periods in a year (daily scale = 252, monthly scale = 12, quarterly scale = 4) and HPR is the Holding Period Return:

$$HPR=  \frac{Price_{t} - Price_{t-1}}{Price_{t-1}}$$



Return.annualized(data,geometric = T,scale= )


```r
library(PerformanceAnalytics)
# The strategy return signal 
ret_annual<-Return.annualized(strat_ret,geometric = T,scale= 252)
ret_annual
#>                   AAPL.Close
#> Annualized Return -0.3863758

sd<-StdDev.annualized(strat_ret,scale=252)

# Sharpe ratio
ret_annual/sd
#>                   AAPL.Close
#> Annualized Return  -1.077952
```


Annualized SD

$$ Std\ Dev.annualized  = (variance(HPR)*252)^{0.5}$$

StdDev.annualized(x,scale=)




Assuming that the risk free rate is zero, estimate the Sharp Ratio.




## Bibliography
Cervantes, M., Montoya, M. Á., & Bernal, L.A. (2016). Effect of the Business Cycle on Investment Strategies: Evidence from Mexico. Revista Mexicana de Economía y Finanzas, 11(2).

Based on:  Jeet, P and  VatsLearning, P (2017). Quantitative Finance with R. Packt Publishing. 


