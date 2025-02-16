STA380 Excercises
================
Katie Grant, Hannah Li, Trisha Paul, Puja Subramaniam

# Green Buildings

``` r
library(mosaic)
library(dplyr)
library(ggplot2)
setwd("/Users/pujas/OneDrive/Documents")
green =  read.csv("greenbuildings.csv")
summary(green)
```

*Clean Data:*

A dataset containing information on 7,894 rental properties across the
U.S. was used to determine if building a “green” building would be
economically advantageous for an Austin real estate developer. The data
was cleaned to remove all buildings with leasing rates under 10% and
those that were under a year old because these buildings may not have
established sound business practices yet.

``` r
#get data with leasing rates > 10% and age >0
data = green[green$leasing_rate > 10 & green$age > 0,]
#creating dummy variables
data$LEED <- as.factor(data$LEED)
data$Energystar <- as.factor(data$Energystar)
data$amenities <- as.factor(data$amenities)
data$green_rating = as.factor(data$green_rating)
data$class_a = as.factor(data$class_a)
data$class_b = as.factor(data$class_b)
data$net = as.factor(data$net)
data$cluster = as.factor(data$cluster)

#get green buildings and non-green buildings data
green_buildings = data[data$green_rating == 1,]
non_green_buildings = data[data$green_rating == 0,]
```

*Compare Rent Prices*

``` r
hist(green_buildings$Rent, 25)
```

![](figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
hist(non_green_buildings$Rent, 25)
```

![](figure-gfm/unnamed-chunk-3-2.png)<!-- -->

``` r
mean(green_buildings$Rent)
```

    ## [1] 30.03023

``` r
mean(non_green_buildings$Rent)
```

    ## [1] 28.42239

``` r
median(green_buildings$Rent)
```

    ## [1] 27.6

``` r
median(non_green_buildings$Rent)
```

    ## [1] 25

*Bootstrap the median*

``` r
#green
boot_g = do(2500)*{
    median(resample(green_buildings)$Rent)
}
head(boot_g)
```

    ##   result
    ## 1 27.930
    ## 2 27.615
    ## 3 27.500
    ## 4 28.400
    ## 5 28.285
    ## 6 27.600

``` r
qplot(boot_g$result, geom="histogram")+labs(x = "Rent per square foot", y = 'Frequency')+ggtitle("Median Rent for Green Buildings")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
confint(boot_g)
```

    ##     name lower upper level     method estimate
    ## 1 result 26.89  28.5  0.95 percentile     27.6

``` r
#non-green
boot_ng = do(2500)*{
    median(resample(non_green_buildings)$Rent)
}
head(boot_ng)
```

    ##   result
    ## 1  25.34
    ## 2  25.44
    ## 3  25.00
    ## 4  25.00
    ## 5  25.00
    ## 6  25.00

``` r
qplot(boot_ng$result, geom="histogram")+labs(x = "Rent per square foot", y = 'Frequency')+ggtitle("Median Rent for Non-Green Buildings")
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](figure-gfm/unnamed-chunk-4-2.png)<!-- -->

``` r
confint(boot_ng)
```

    ##     name    lower upper level     method estimate
    ## 1 result 24.77475 25.44  0.95 percentile       25

We first decided to compare rent rates to see how long it would take to
recuperate the loss from the 5% premium. The analyst took a single
median value to determine how long it would take to gain back that loss,
however this is not the most accurate way to measure rent rates. In
order to get a more accurate measure of median for rent, we bootstrapped
the median 2500 times and obtained a confidence interval of what the
rent price range was for green buildings. We then repeated the same
process for non-green buildings. In doing this, we noticed that
non-green building rent is around $24.74-$25.45 with 95% confidence and
green-building rent is around $26.89-$28.50 with 95% confidence; so
roughly a $2-$3 upcharge for tenants of green-buildings. This leads us
to believe that the owner will be able to regain their loss from the
premium cost in $5,000,000/$750,000 = 6.6 years at the earliest.

### Confounding Variables

There are several variables that may confound the relationship between
rent and green status. For example, a higher percentage of green
buildings have amenities than non-green buildings. Additionally,
green-buildings tend to be newer than non-green buildings and a higher
percentage have been renovated which may indicate that green buildings
have higher quality interiors. All of these factors may contribute to
green buildings having higher rents.

*Percent of buildings with amenities*

``` r
x1=data %>%
  group_by(green_rating) %>%
  summarize(amen_pct=sum(amenities==1)/n())
ggplot(data = x1) + 
   geom_bar(mapping = aes(x=green_rating, y=amen_pct), stat='identity',fill='#0c5712') + labs(x='Green Rating',y='% of Buildings with Amenities',title='Comparison of Amenities in Green and Non-Green Buildings') + theme(plot.title = element_text(hjust = 0.5))
```

![](figure-gfm/unnamed-chunk-5-1.png)<!-- -->

*Comparing age of green buildings to non-green
buildings*

``` r
ggplot(data, aes(age, fill = green_rating)) + geom_density(alpha = 0.2) + scale_fill_manual(values=c("blue", "green"))
```

![](figure-gfm/unnamed-chunk-6-1.png)<!-- -->

### Green-rating vs. leasing rate

Another potential advantage of building a green building is that it may
attract more tenants. While the average leasing rate is higher for green
rated buildings, the standard deviation shows that it is not
significantly different from the leasing rate of non-green buildings.

``` r
# Bar chart of green rating and leasing rate with standard error bars
greenleasing_summ = data %>%
  group_by(green_rating)  %>%  # group the data points by green rating
  summarize(leasing.mean = mean(leasing_rate), leasing.sd=sd(leasing_rate))  # calculate mean leasing rate
# Create plot
ggplot(greenleasing_summ, aes(x=green_rating, y=leasing.mean)) + 
  geom_bar(stat='identity',fill='#0c5712') + geom_errorbar(aes(ymin=leasing.mean-leasing.sd,ymax=leasing.mean+leasing.sd),size=1.5) + labs(title='Leasing Rate vs. Green Rating',y='Leasing Rate',x='Green Rating') + theme(plot.title = element_text(hjust = 0.5))
```

![](figure-gfm/unnamed-chunk-7-1.png)<!-- -->

### Conclusion

Ultimately, we do not agree with the analysis of the stats guru. In his
analysis, he failed to consider the confidence intervals associated with
the median rent for green and non-green buildings. After considering
these intervals, it is possible that the extra revenue generated by
green buildings is less than estimated. Additionally, he fails to
consider other variables that might contribute to high rents such as
amenities, and the age of the building. As a result of this, we
recommend that the real estate developer does not build a green
building. While it may be the socially responsible option, it will not
prove to be financially beneficial for the developer due to the
$5,000,000 premium for green certification. The owner will likely have
to add amenities and perform renovations to their property in order to
compete with the other green buildings, which will result in additional
costs .

# Flights at ABIA

``` r
library(mosaic)
library(tidyverse)
library(ggplot2)
library(knitr)
library(chron)
setwd("/Users/pujas/OneDrive/Documents")
abia = read.csv("ABIA.csv", header = T, stringsAsFactors = FALSE)
attach(abia)
#head(abia)
#names(abia)

#cleaning data by converting the integers to times.
DepTime <- substr(as.POSIXct(sprintf("%04.0f", DepTime), format='%H%M'), 12, 16)
CRSDepTime <- substr(as.POSIXct(sprintf("%04.0f", CRSDepTime), format='%H%M'), 12, 16)
ArrTime <- substr(as.POSIXct(sprintf("%04.0f", ArrTime), format='%H%M'), 12, 16)
CRSArrTime <- substr(as.POSIXct(sprintf("%04.0f", CRSArrTime), format='%H%M'), 12, 16)

# cleaning data by converting integers to actual days
DayOfWeek[DayOfWeek == 1] <- 'Monday'
DayOfWeek[DayOfWeek == 2] <- 'Tuesday'
DayOfWeek[DayOfWeek == 3] <- 'Wednesday'
DayOfWeek[DayOfWeek == 4] <- 'Thursday'
DayOfWeek[DayOfWeek == 5] <- 'Friday'
DayOfWeek[DayOfWeek == 6] <- 'Saturday'
DayOfWeek[DayOfWeek == 7] <- 'Sunday'

# cleaning data by converting integers to actual months.
Month[Month == 1] <- 'January'
Month[Month == 2] <- 'February'
Month[Month == 3] <- 'March'
Month[Month == 4] <- 'April'
Month[Month == 5] <- 'May'
Month[Month == 6] <- 'June'
Month[Month == 7] <- 'July'
Month[Month == 8] <- 'August'
Month[Month == 9] <- 'September'
Month[Month == 10] <- 'October'
Month[Month == 11] <- 'November'
Month[Month == 12] <- 'December'
```

Our goal was to see if we could ultimately recommend the best day and
carrier for individuals. First, we looked big picture to understand the
frequency of the flights coming in and out of Austin by month. It turns
out that there are more flights to/from Austin from May through July, as
many people go on vacation during summer months. March had a lot as
well, probably due to Spring Break and SXSW.

``` r
# plot for frequency of flights by month
ggplot(data=abia, aes(x=Month, fill=Month)) +
geom_bar(fill = "#AE3116") +
geom_text(stat='count', aes(label=..count..), vjust=-1) + 
scale_x_continuous(breaks = seq(1, 12, by = 1)) + ggtitle('Frequency of Flights by Month') + 
    labs(y="Number of Flights", x = "Month")
```

![](figure-gfm/unnamed-chunk-9-1.png)<!-- -->

Next, we zoomed in a little to see which days of the week had the most
flights, and the counts were pretty consistent Monday through Friday,
but Saturday and Sunday had significantly fewer flights. This means that
most people try to take full advantage of their weekend in a different
destination. This decrease during the weekend was something we wanted to
look into further.

``` r
# plot for frequency of flights by day
ggplot(data=abia, aes(x=DayOfWeek)) +
geom_bar(fill = "#AE3116", width = 0.7, ) +
geom_text(stat='count', aes(label=..count..), vjust=-1) + 
scale_x_continuous(breaks = seq(1, 12, by = 1)) + ggtitle('Frequency of Flights by Day') + 
    labs(y="Number of Flights", x = "Day of the Week")
```

![](figure-gfm/unnamed-chunk-10-1.png)<!-- --> Before
zooming into Saturday alone, we wanted to see how delays varied by day.
Arrival delays were interesting to see. Wednesday and Saturday had the
shortest arrival delays. Saturday stood out with the lowest arrival
delay of an average of 4 minutes. In general these averages are low
because for most of the year, planes arrived earlier than scheduled
arrival. This is likely because ABIA is a smaller airport, and is able
to get flights in and out according to schedule.

``` r
# plot for average arrival delay by day

abia1 <- subset(abia, abia$Origin == 'AUS')

library(dplyr)
summarise_at(group_by(abia1,DayOfWeek),vars(ArrDelay),funs(mean(.,na.rm=TRUE)))
```

    ## # A tibble: 7 x 2
    ##   DayOfWeek ArrDelay
    ##       <int>    <dbl>
    ## 1         1     6.66
    ## 2         2     6.01
    ## 3         3     4.27
    ## 4         4     6.95
    ## 5         5     8.09
    ## 6         6     4.00
    ## 7         7     5.80

``` r
v <- c("Mon","Tue","Wed","Thu","Fri","Sat","Sun")
t <- c(6.66,6.01,4.27,6.95,8.09,4.00, 5.80)

# Plot the line graph
plot(t, type = "o",col = "red", ylim=c(0,10), xlab = "Day", ylab = "Arrival Delay (in minutes)",
  main = "Average Plane Arrival Delay by Day")
```

![](figure-gfm/unnamed-chunk-11-1.png)<!-- --> Next, we
look at the average airtime by day, and notice again that Saturday
stands out. Flights out of AUS on Saturday have the longest flights, but
not by much: only an average of 5 minutes more than the other days.
However, even those few extra minutes could have significant costs
attached to them for both the airlines and airport.

``` r
#plot for average flight times by day
summarise_at(group_by(abia1,DayOfWeek),vars(AirTime),funs(mean(.,na.rm=TRUE)))
```

    ## # A tibble: 7 x 2
    ##   DayOfWeek AirTime
    ##       <int>   <dbl>
    ## 1         1    101.
    ## 2         2    101.
    ## 3         3    101.
    ## 4         4    101.
    ## 5         5    100.
    ## 6         6    105.
    ## 7         7    101.

``` r
v <- c("Mon","Tue","Wed","Thu","Fri","Sat","Sun")
t <- c(99.11365 ,99.41158   ,99.14433   ,99.55306   ,99.28342   ,103.75217, 99.27976)

# Plot the line grah
plot(t, type = "o",col = "red", ylim=c(90,105), xlab = "Day", ylab = "Airtime (in minutes)",
  main = "Average Airtime by Day")
```

![](figure-gfm/unnamed-chunk-12-1.png)<!-- --> Even
though Saturday had the longest flights, it still had one of the lowest
arrival delay with an average of 7.47 minutes. This means that most
likely, delays that occur on other days aren’t due to issues in the air,
but moreso on the ground when departing/landing. Saturday appears to
have, on average, the fewest delays, which is good for a passenger to
know, even though there are fewer flights on Saturday.

``` r
abia2 <- subset(abia, abia$Dest == 'AUS')

library(dplyr)
summarise_at(group_by(abia2,DayOfWeek),vars(DepDelay),funs(mean(.,na.rm=TRUE)))
```

    ## # A tibble: 7 x 2
    ##   DayOfWeek DepDelay
    ##       <int>    <dbl>
    ## 1         1    11.3 
    ## 2         2    10.1 
    ## 3         3     8.97
    ## 4         4    10.9 
    ## 5         5    13.2 
    ## 6         6     8.97
    ## 7         7    12.7

``` r
v <- c("Mon","Tue","Wed","Thu","Fri","Sat","Sun")
t <- c(11.3 ,10.1   ,8.97,10.9,13.2,8.97,12.7)

# Plot the bar chart.
plot(t, type = "o",col = "red", ylim=c(7,14), xlab = "Day", ylab = "Arrival Delay (in minutes)",
  main = "Average Plane Arrival Delay by Day")
```

![](figure-gfm/unnamed-chunk-13-1.png)<!-- --> Now that
Saturday has stood out on multiple occassions, it is time to dive deeper
into Saturday flights. We first looked at the frequency of flights by
the hour on Saturdays. 7AM and 11AM seem to be the most popular times at
the airport for departures. Knowing when the influx of people are during
various points in the day could help passengers know when to expect long
lines at security and at various businesses in the airport. It also
helps the Austin airport know when to have extra staff at the airport,
given that they know when to expect rush periods.

``` r
# plot for frequency of departures on Saturdays by hour
L = abia1$DayOfWeek == 6
df <-abia1[L,]
df$Hour = as.numeric(substr(df$DepTime, 1, nchar(df$DepTime)-2))
ggplot(data=df, aes(x=Hour)) +
geom_bar(fill = "#AE3116", width = 0.7) +
geom_text(stat='count', aes(label=..count..), vjust=-1) + 
scale_x_continuous(breaks = seq(1, 24, by = 1)) + ggtitle('Frequency of Saturday Departures by Hour') + 
    labs(y="Number of Flights", x = "Hour of the Day")
```

![](figure-gfm/unnamed-chunk-14-1.png)<!-- --> Now we
looked at arrivals by the hour at AUS on Saturdays. Most arrivals occur
at 11AM, 1PM, and 4PM. There was an interesting pattern of arrivals
between 11AM to 3PM as can be seen with the spike/dip pattern between
those hours. This makes sense since 11AM, 1PM, and 4PM are on average
more popular times of arrival, the airport has a limit of how many
planes can arrive at one time, and there are slight delays or early
arrivals/departures for most flights. So strategically scheduling less
arrivals following the peak hours helps control this inevitable
congestion. Again, this timing information could be valuable to the
airport for the same reasons mentioned above: staffing and airport
businesses.

``` r
# plot for frequency of arrivals by the hour
L = abia2$DayOfWeek == 6
df2 <-abia2[L,]
df2$Hour = as.numeric(substr(df2$ArrTime, 1, nchar(df2$ArrTime)-2))
ggplot(data=df2, aes(x=Hour)) +
geom_bar(fill = "#AE3116", width = 0.7) +
geom_text(stat='count', aes(label=..count..), vjust=-1) + 
scale_x_continuous(breaks = seq(1, 24, by = 1)) + ggtitle('Frequency of Saturday Arrivals by Hour') + 
    labs(y="Number of Flights", x = "Hour of the Day")
```

![](figure-gfm/unnamed-chunk-15-1.png)<!-- --> Lastly,
we looked at departure and arrival delays by the carriers on Saturdays.
We decided to look at this to see which carriers passengers should avoid
based on these delays. It is frustrating for passengers to have to wait
and be late to their destination. Also these delays can impact the
airport as well, since arriving or leaving too late can have a chain
reaction and negatively impact the other flights for the day. So
finally, we see that on Saturdays, MQ and US don’t have arrival or
delays (on average), while OH has, on average, the longest arrival and
departure delays; customers should be weary of this carrier before they
book a flight with this carrier.

``` r
# plot for arrivals and departures by carriers on Saturdays
L = abia$DayOfWeek == 6
df3 <-abia[L,]
summarise_at(group_by(df3,UniqueCarrier),vars(DepDelay),funs(mean(.,na.rm=TRUE)))
```

    ## # A tibble: 16 x 2
    ##    UniqueCarrier DepDelay
    ##    <chr>            <dbl>
    ##  1 9E              6.02  
    ##  2 AA              5.70  
    ##  3 B6             10.9   
    ##  4 CO              5.71  
    ##  5 DL             10.7   
    ##  6 EV              9.05  
    ##  7 F9              1.15  
    ##  8 MQ             -1.23  
    ##  9 NW             -0.875 
    ## 10 OH             13.3   
    ## 11 OO              9.06  
    ## 12 UA             13.9   
    ## 13 US             -0.0581
    ## 14 WN              7.87  
    ## 15 XE              8.18  
    ## 16 YV              9.87

``` r
summarise_at(group_by(df3,UniqueCarrier),vars(ArrDelay),funs(mean(.,na.rm=TRUE)))
```

    ## # A tibble: 16 x 2
    ##    UniqueCarrier ArrDelay
    ##    <chr>            <dbl>
    ##  1 9E               3.37 
    ##  2 AA               4.93 
    ##  3 B6               7.68 
    ##  4 CO               6.81 
    ##  5 DL              12.0  
    ##  6 EV               2.01 
    ##  7 F9              -0.920
    ##  8 MQ              -3.16 
    ##  9 NW               3    
    ## 10 OH              16.6  
    ## 11 OO               6.40 
    ## 12 UA              12.8  
    ## 13 US              -4.53 
    ## 14 WN               1.92 
    ## 15 XE               6.60 
    ## 16 YV               8.55

``` r
d <- data.frame(column1=rep(c("9E","AA","B6", "CO","DL","EV","F9","MQ","NW","OH","OO","UA","US","WN","XE","YV"), each=2), 
                column2=rep(c("DepDelay", "ArrDelay"), 2), 
                column3=c(6.02194357, 3.3710692, 5.69703215, 4.9265070, 10.91718750 , 7.6755486, 5.70562293, 6.8066298  , 10.69841270, 12.0079681, 9.04901961   , 2.0098039, 1.14878893 , -0.9204152, -1.22560976, -3.1585366, -0.87500000, 3.0000000, 13.28176796  ,16.5888889, 9.06288032, 6.3963415, 13.87553648, 12.7931034, -0.05813953,-4.5290698,7.87417568,1.9197466,8.18257261,6.6008316,9.87347561,8.5460123))
require(lattice)

barchart(column3 ~ column1, groups=column2, d, auto.key = list(columns = 2), main='Delay Times on Saturdays for Various Carriers', xlab='Carriers',ylab='Delays (in minutes)', ylim = c(0 , 20))
```

![](figure-gfm/unnamed-chunk-16-1.png)<!-- --> In
conclusion, Saturday flights incited some interesting findings at ABIA.
The lowest number of flights and longest lenght flights were on
Saturdays. It also had the shortest delays for both arrivals and
departures compared to the rest of the week, which is good to note for
customers who are looking for the best days to fly. And finally, based
on this data, OH carrier should be avoided since it notably has the
longest delays.

# Portfolio Modeling

``` r
library(mosaic)
library(quantmod)
library(foreach)
```

### PORTFOLIO \#1: Aggressive

``` r
rm(list=ls())
set.seed(512)
mystocks1 = c("VTI", "QQQ", "IVV", "SHYG", "HYG")
myprices = getSymbols(mystocks1, from = "2014-01-01")

# creating new object with adjusted values for each ticker
for(ticker in mystocks1) {
    expr = paste0(ticker, "a = adjustOHLC(", ticker, ")")
    eval(parse(text=expr))
}

all_returns = cbind(ClCl(VTIa),
                                ClCl(QQQa),
                                ClCl(IVVa),
                                ClCl(SHYGa),
                                ClCl(HYGa))
all_returns = as.matrix(na.omit(all_returns))

#looping over four trading weeks
total_wealth = 100000
weights = c(0.25, 0.25, 0.25, 0.125, 0.125)
holdings = weights * total_wealth
n_days = 20
wealthtracker = rep(0, n_days) 
for(today in 1:n_days) {
    return.today = resample(all_returns, 1, orig.ids=FALSE)
    holdings = holdings + holdings*return.today
    total_wealth = sum(holdings)
    wealthtracker[today] = total_wealth
}
total_wealth
```

    ## [1] 98259.41

``` r
plot(wealthtracker, type='l')
```

![](figure-gfm/unnamed-chunk-18-1.png)<!-- -->

``` r
# creating histogram to get VaR
initial_wealth = 100000
sim1 = foreach(i=1:5000, .combine='rbind') %do% {
    total_wealth = initial_wealth
    weights = c(0.25, 0.25, 0.25, 0.125, 0.125)
    holdings = weights * total_wealth
    n_days = 20
    wealthtracker = rep(0, n_days)
    for(today in 1:n_days) {
        return.today = resample(all_returns, 1, orig.ids=FALSE)
        holdings = holdings + holdings*return.today
        total_wealth = sum(holdings)
        wealthtracker[today] = total_wealth
    }
    wealthtracker
}

# Profit/loss
mean(sim1[,n_days])
```

    ## [1] 100905.3

``` r
hist(sim1[,n_days]- initial_wealth, breaks=30, main="Profit/Loss for Portfolio #1" )
```

![](figure-gfm/unnamed-chunk-19-1.png)<!-- -->

``` r
h = (sim1[,n_days] - initial_wealth)
quantile(h, 0.05)
```

    ##       5% 
    ## -4452.19

For the first portfolio, we decided to make it a high growth portfolio
by picking three large capital growth equities ETFs to make up 75% of
the portfolio, and two high yield bonds ETFs to comprise the other 25%
of the portfolio. We structured it this way instead of having 100% of
the high growth stocks because the bonds act as a buffer since there is
lower risk with bonds. So we wanted to be aggressive, but not too
aggressive. We selected Vanguard Total Stock Market ETF (VTI), iShares
Core S\&P 500 ETF (IVV), Invesco QQQ (QQQ), ISHARES TR/0-5 YR HIGH YIELD
CORP B (SHYG), and iShares iBoxx $ High Yid Corp Bond (HYG). We decided
to pick these five ETFs based on the total assets each had, and the best
daily change rate. The value at risk at the 5% level is a loss of
$4520.20. This means that if there is no trading over the period, there
is a 5% chance that this portfolio will fall in value by over $4520.
This high amount is expected since this is a high growth portfolio;
since there is more of a return on this portfolio, there is also a
higher risk.

### PORTFOLIO \#2: Conservative

``` r
rm(list=ls())
set.seed(512)
mystocks2 = c("SHV", "SHY", "VGSH", "BOND", "VTI", "VXUS")
myprices = getSymbols(mystocks2, from = "2014-01-14")
```

    ## pausing 1 second between requests for more than 5 symbols
    ## pausing 1 second between requests for more than 5 symbols

``` r
for(ticker in mystocks2) {
  expr = paste0(ticker, "a = adjustOHLC(", ticker, ")")
  eval(parse(text=expr))
}
all_returns = cbind(ClCl(SHVa),
                    ClCl(SHYa),
                    ClCl(VGSHa),
                    ClCl(BONDa),
                    ClCl(VTIa),
                    ClCl(VXUSa))
head(all_returns)
```

    ##                ClCl.SHVa     ClCl.SHYa    ClCl.VGSHa    ClCl.BONDa
    ## 2014-01-14            NA            NA            NA            NA
    ## 2014-01-15  0.000000e+00 -0.0003553542 -0.0011501807 -0.0023687701
    ## 2014-01-16 -9.072109e-05  0.0002370542  0.0006580194  0.0037990407
    ## 2014-01-17  9.072932e-05  0.0000000000 -0.0001644254  0.0028384520
    ## 2014-01-21 -9.072109e-05 -0.0001184931  0.0008220816 -0.0009434758
    ## 2014-01-22  0.000000e+00 -0.0004739455 -0.0003285855 -0.0027387006
    ##                ClCl.VTIa    ClCl.VXUSa
    ## 2014-01-14            NA            NA
    ## 2014-01-15  0.0050172887  0.0021193833
    ## 2014-01-16 -0.0007280291  0.0001923092
    ## 2014-01-17 -0.0038509887 -0.0036524029
    ## 2014-01-21  0.0033434333  0.0036657919
    ## 2014-01-22  0.0018744143  0.0030757401

``` r
all_returns = as.matrix(na.omit(all_returns))
return.today = resample(all_returns, 1, orig.ids=FALSE)
total_wealth = 100000
weights = c(0.2, 0.2, 0.2, 0.2, 0.15, 0.05)
holdings = weights * total_wealth
n_days = 20
wealthtracker = rep(0, n_days) # Set up a placeholder to track total wealth
for(today in 1:n_days) {
  return.today = resample(all_returns, 1, orig.ids=FALSE)
  holdings = holdings + holdings*return.today
  total_wealth = sum(holdings)
  wealthtracker[today] = total_wealth
}
total_wealth
```

    ## [1] 100787.7

``` r
plot(wealthtracker, type='l')
```

![](figure-gfm/unnamed-chunk-20-1.png)<!-- -->

``` r
# Now simulate many different possible scenarios  
initial_wealth = 100000
sim1 = foreach(i=1:5000, .combine='rbind') %do% {
  total_wealth = initial_wealth
  weights = c(0.2, 0.2, 0.2, 0.2, 0.15, 0.05)
  holdings = weights * total_wealth
  n_days = 20
  wealthtracker = rep(0, n_days)
  for(today in 1:n_days) {
    return.today = resample(all_returns, 1, orig.ids=FALSE)
    holdings = holdings + holdings*return.today
    total_wealth = sum(holdings)
    wealthtracker[today] = total_wealth
  }
  wealthtracker
}

# Profit/loss
mean(sim1[,n_days])
```

    ## [1] 100251.3

``` r
hist(sim1[,n_days]- initial_wealth, breaks=30, main="Profit/Loss for Portfolio #2")
```

![](figure-gfm/unnamed-chunk-21-1.png)<!-- -->

``` r
h = (sim1[,n_days] - initial_wealth)
quantile(h, 0.05)
```

    ##        5% 
    ## -933.2883

For the second portfolio, we took a more conservative approach, with 80%
made up of government bond ETFs and 20% made up of stock ETFs. These
ETF’s include the iShares Short Treasury Bond ETF (SHV), iShares 1-3
Year Treasury Bond ETF (SHY), Vanguard Short-Term Treasury ETF (VGSH),
PIMCO Active Bond Exchange-Traded Fund (BOND), Vanguard Total Stock
Market ETF (VTI), and Vanguard Total International Stock ETF (VXUS). We
picked government bonds because they are known to be low-risk
investments since the issuing government backs them. We paired it with
two large growth ETFs to slightly increase its risk. As a result, the
value at risk at the 5% level is a loss of -$935.28. This is
considerably lower than the previous example because this is a
conservative, low risk portfolio, so it is understandable that the risk
of loss would be lower.

### PORTFOLIO \#3: Diversified

``` r
rm(list=ls())
set.seed(512)
mystocks3 = c("AOR", "FXE", "LQD", "VHT", "USO", "IYW", "DBC", "VFH", "INDA", "VNQ")
myprices = getSymbols(mystocks3, from = "2014-01-14")
```

    ## pausing 1 second between requests for more than 5 symbols
    ## pausing 1 second between requests for more than 5 symbols
    ## pausing 1 second between requests for more than 5 symbols
    ## pausing 1 second between requests for more than 5 symbols
    ## pausing 1 second between requests for more than 5 symbols
    ## pausing 1 second between requests for more than 5 symbols

``` r
for(ticker in mystocks3) {
  expr = paste0(ticker, "a = adjustOHLC(", ticker, ")")
  eval(parse(text=expr))
}

all_returns = cbind(ClCl(AORa),
                    ClCl(FXEa),
                    ClCl(LQDa),
                    ClCl(VHTa),
                    ClCl(USOa),
                    ClCl(IYWa),
                    ClCl(DBCa),
                    ClCl(VFHa),
                    ClCl(INDAa),
                    ClCl(VNQa))
head(all_returns)
```

    ##                ClCl.AORa     ClCl.FXEa     ClCl.LQDa     ClCl.VHTa
    ## 2014-01-14            NA            NA            NA            NA
    ## 2014-01-15  0.0025980253 -0.0058418548  0.0002608382  9.552096e-05
    ## 2014-01-16 -0.0007773776  0.0013388351  0.0023469836  2.960256e-03
    ## 2014-01-17 -0.0010373703 -0.0070568789  0.0011274304 -3.808531e-04
    ## 2014-01-21  0.0018172378  0.0024687814 -0.0013860273  6.191075e-03
    ## 2014-01-22  0.0005182949 -0.0007463134 -0.0024288602  3.786539e-04
    ##               ClCl.USOa     ClCl.IYWa    ClCl.DBCa    ClCl.VFHa
    ## 2014-01-14           NA            NA           NA           NA
    ## 2014-01-15  0.022094400  0.0126425449  0.002816901  0.010105570
    ## 2014-01-16 -0.003553391 -0.0004458923 -0.001605177 -0.004891107
    ## 2014-01-17  0.001188618 -0.0076948256  0.001205828 -0.002680943
    ## 2014-01-21  0.008904809  0.0048325577  0.002408631  0.002688150
    ## 2014-01-22  0.016769608 -0.0002237445  0.004805807  0.002010724
    ##              ClCl.INDAa    ClCl.VNQa
    ## 2014-01-14           NA           NA
    ## 2014-01-15  0.006865913  0.007557436
    ## 2014-01-16 -0.003610108  0.001800105
    ## 2014-01-17 -0.013687601 -0.005840072
    ## 2014-01-21 -0.001224531  0.009037491
    ## 2014-01-22  0.012259951  0.004329019

``` r
all_returns = as.matrix(na.omit(all_returns))


# simulating a random day
return.today = resample(all_returns, 1, orig.ids=FALSE)

# Now loop over four trading weeks
total_wealth = 100000
weights = c(0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1,0.1)
holdings = weights * total_wealth
n_days = 20
wealthtracker = rep(0, n_days) # Set up a placeholder to track total wealth
for(today in 1:n_days) {
  return.today = resample(all_returns, 1, orig.ids=FALSE)
  holdings = holdings + holdings*return.today
  total_wealth = sum(holdings)
  wealthtracker[today] = total_wealth
}
total_wealth
```

    ## [1] 102065.4

``` r
plot(wealthtracker, type='l')
```

![](figure-gfm/unnamed-chunk-22-1.png)<!-- -->

``` r
# Now simulate many different possible scenarios  
initial_wealth = 100000
sim1 = foreach(i=1:5000, .combine='rbind') %do% {
  total_wealth = initial_wealth
  weights = c(0.1, 0.1, 0.1, 0.1, 0.1, 0.1, 0.1,0.1,0.1,0.1)
  holdings = weights * total_wealth
  n_days = 20
  wealthtracker = rep(0, n_days)
  for(today in 1:n_days) {
    return.today = resample(all_returns, 1, orig.ids=FALSE)
    holdings = holdings + holdings*return.today
    total_wealth = sum(holdings)
    wealthtracker[today] = total_wealth
  }
  wealthtracker
}

# Profit/loss
mean(sim1[,n_days])
```

    ## [1] 100345.4

``` r
hist(sim1[,n_days]- initial_wealth, breaks=30, main="Profit/Loss for Portfolio #3")
```

![](figure-gfm/unnamed-chunk-23-1.png)<!-- -->

``` r
h = (sim1[,n_days] - initial_wealth)
quantile(h, 0.05)
```

    ##       5% 
    ## -4101.95

The last portfolio was comprised of ten ETF’s that were all very
different to have a diversified portfolio since research has shown that
those are more likely to have a higher return. The ETF’s range from oil
& gas to health/biotech to international equity ETF’s. Each of them were
given a 10% weight in the portfolio. The value at risk at the 5% level
is a potential loss of $4101.95. This high amount is expected because
each ETF within the portfolio carries various amounts of risk, so when
they are put together into one, it is a risky profile overall. But the
outlook four week outlook looked better for this portfolio than the
previous two as the investment is predicted to increase over time.

# Market Segmentation

``` r
rm(list=ls())
library(flexclust)
library(ggplot2)
setwd("/Users/pujas/OneDrive/Documents")
social = read.csv("social_marketing.csv", header = TRUE, stringsAsFactors = FALSE, row.names = 1)

library(ggcorrplot)
cormat <- round(cor(social), 2)
ggcorrplot(cormat, hc.order = TRUE)
```

![](figure-gfm/unnamed-chunk-24-1.png)<!-- --> Hard to
read the correlation plot, but we see that there’s definitely some
correlation present between words

### PRE-PROCESSING

*Data cleaning:* We started by eliminating tags that Twitter already
tries to weed out. This includes spam, chatter, inappropriate adult
content. We also removed uncategorized information, given that it would
be hard to place its importance. All of this information potentially may
have been generated by bots, so we want to ensure that the marketing
team does not focus on irrelevant features.

``` r
# Remove chatter, uncategorized, spam, and adult
social = social[,c(-1,-5,-35,-36)]

# First normalize phrase counts to phrase frequencies.
Z = social/rowSums(social)
#gives us frequencies of the data used versus the actual counts

#standardize Z
Z_std = scale(Z, center=TRUE, scale=FALSE)
plot(Z_std)
```

![](figure-gfm/unnamed-chunk-25-1.png)<!-- -->

### ANALYSIS

``` r
# PCA
pc = prcomp(Z, scale=TRUE)
#PC1 accounts for ~8% of the variation in the model, PC2 for ~7%, etc.

pr_var <-  pc$sdev ^ 2
pve <- pr_var / sum(pr_var)
plot(pve, xlab = "Principal Component", ylab = "Proportion of Variance Explained", ylim = c(0,1), type = 'b')
```

![](figure-gfm/unnamed-chunk-26-1.png)<!-- -->

``` r
#x axis shows # of principal components, y axis shows proportion of variance 
loadings = pc$rotation
```

Loadings describe the weight given to each raw variable in calculating
the new PC

Positive values contribute positively to the component. Negative values
mean negative relationship. Larger numbers mean stronger relationship

``` r
scores = pc$x
plot(pc, main='Variance by PC')
```

![](figure-gfm/unnamed-chunk-27-1.png)<!-- -->

Looking at the plot, we see that the ‘knee’ of the graph happens at
around 7, so we will look at 7 PC components

``` r
pc7 = prcomp(Z, scale=TRUE, rank=7) #stops after 7 prinicipal components
loadings = pc7$rotation
#PC1: 
#highest: sports_fandom, food, family, health_nutrition, cooking, religion, parenting, personal fitness
scores = pc7$x

barplot(pc7$rotation[,1], las=3)
```

![](figure-gfm/unnamed-chunk-28-1.png)<!-- -->

Next , we want to see how the individual PCs are loaded on the original
variables

``` r
# The top words associated with each component
o1 = order(loadings[,1], decreasing=TRUE)
o2 = order(loadings[,2], decreasing=TRUE)
o3 = order(loadings[,3], decreasing=TRUE)
o4 = order(loadings[,4], decreasing=TRUE)
o5 = order(loadings[,5], decreasing=TRUE)
o6 = order(loadings[,6], decreasing=TRUE)
o7 = order(loadings[,7], decreasing=TRUE)
```

We then break down Twitter users into general groups, based on the
tweets they posted.

``` r
# Return the top 5 variables with the highest loadings for each PC
colnames(Z)[tail(o1,4)]
```

    ## [1] "fashion"          "personal_fitness" "health_nutrition"
    ## [4] "cooking"

This group of individuals are likely very health conscious. They eat
clean, like to look up recipes for meal prep Sunday, and are committed
to working out daily. Thus, the best way for NutrientH20 to reach them
is by playing up the ‘health’ aspect of the brand - talk about its
health benefits, how it can be used in the kitchen, how the electrolytes
in the product help improve workouts, etc.

``` r
colnames(Z)[tail(o2,4)]
```

    ## [1] "tv_film"     "college_uni" "travel"      "politics"

This group most likely comprises of college students who are actively
engaged with their community. When they’re not studying, they’re likely
Netflix bingers. They also follow politics, given that they are now of
voting age. They also are likely to be travelling during their breaks,
whether for fun or to see family. The best way for Nutrient H20 to reach
this market is by perhaps creating a student discount, advertising how
the product is a great product for taking to class, to have while
chilling on the couch, or while travelling.

``` r
colnames(Z)[tail(o3,4)]
```

    ## [1] "personal_fitness" "health_nutrition" "news"            
    ## [4] "politics"

This group probably includes young professionals - no longer broke
college students, they’re spending more time focusing on their health
and fitness. They’re also very interested in current events, and follow
the news so that they know what their coworkers are talking about. The
best way to reach this market is to advertise on online news platforms,
like the New York Times, since these are the types of sites this group
probably frequenting the most to get information about everything they
need.

``` r
colnames(Z)[tail(o4,4)]
```

    ## [1] "politics" "cooking"  "beauty"   "fashion"

This group of individuals also actively follows beauty and fashion
blogs, and love to try new recipes from their cookbooks, but they’re
equally active in politics as well. A great way to reach out to them is
by also engaging with them through various blogs, both political and
personal.

``` r
colnames(Z)[tail(o5,4)]
```

    ## [1] "news"    "beauty"  "cooking" "fashion"

These are the fashion bloggers and Instagram influencers (or wanna be
influencers). They’re likely to follow all the fashion trends, know what
vegetable is now ‘in’, and all of the makeup tips. Given that they’re on
social media often, mainly to focus on these hashtags, the best way for
NutrientH20 to reach them is advertising on Instagram, Facebook,
Snapchat, and Twitter.

``` r
colnames(Z)[tail(o6,4)]
```

    ## [1] "crafts"  "travel"  "art"     "tv_film"

This is the DIYer - the people who love to try making their friends
handmade gifts, pick up quaint gifts when they travel, and love watching
TV channels like HGTV. They live at the craft store, and therefore
NutrientH20 could think about ways to advertise how their packaging
could be reused for various craft projects (perhaps the bottle could
become a
    terrarium).

``` r
colnames(Z)[tail(o7,4)]
```

    ## [1] "sports_playing" "dating"         "travel"         "computers"

This group is similar to the young professional, but probably includes
more of the male clientele. They probably work in tech, might be
travelling regularly for work, follow the NFL/NBA, and amidst all this,
are trying to date. NutrientH20 can focus on this group by also focusing
on various online platforms, including tech, sports, and dating blogs.

## Summary

Based on the results of our Principal Component Analysis, there are a
variety of ways that NutrientH20 can better reach their target market.
We could see early on that there were correlations between what their
social media followers were talking about - people after all, have
similar interests\! We were able to create 7 different groups based on
Tweet topics that we want NutrientH20 to target to best reach their
customers.

We do see some overlap between groups, which is expected - people have
similar interests after all. Of course, this helps us in our marketing
plan, because we can assume that certain marketing strategies will
perform better, as there are more individuals who will view them.

# Author Attribution

``` r
library(tm) 
library(magrittr)
library(slam)
library(proxy)
library(tidyverse)
library(glmnet)
```

*Read all files in the training set*

``` r
# Read plain text docs in English
readerPlain = function(fname){
  readPlain(elem=list(content=readLines(fname)), 
            id=fname, language='en') }
# Load files from all 50 authors
setwd("/Users/pujas/OneDrive/Documents")
files_train = Sys.glob('ReutersC50/C50train/*/*.txt')
class_labels = c(rep('aaron', 50), rep('alan', 50), rep('alexander', 50), 
                 rep('benjamin', 50), rep('bernard', 50), rep('brad', 50), 
                 rep('darren', 50), rep('david', 50), rep('edna', 50), 
                 rep('eric', 50), rep('fumiko', 50), rep('graham', 50), 
                 rep('heather', 50), rep('jane', 50), rep('jan', 50), 
                 rep('jim', 50), rep('joe', 50), rep('john', 50), 
                 rep('jonathan', 50), rep('jo', 50), rep('karl', 50), 
                 rep('keith', 50), rep('kevind', 50), rep('kevinm', 50), 
                 rep('kirstin', 50), rep('kourosh', 50), rep('lydia', 50), 
                 rep('lynne', 50), rep('lynnley', 50), rep('marcel', 50), 
                 rep('mark', 50),rep('martin', 50), rep('matthew', 50), 
                 rep('michael', 50), rep('mure', 50), rep('nick', 50), 
                 rep('patricia', 50), rep('peter', 50), rep('pierre', 50), 
                 rep('robin', 50), rep('roger', 50), rep('samuel', 50), 
                 rep('sarah', 50), rep('scott', 50), rep('simon', 50), 
                 rep('tan', 50), rep('therese', 50), rep('tim', 50), 
                 rep('todd', 50), rep('william', 50))
docs_train = lapply(files_train, readerPlain)
```

*Tokenization/ Pre-processing*

``` r
corpus_train = Corpus(VectorSource(docs_train))
corpus_train = tm_map(corpus_train, content_transformer(tolower))
corpus_train = tm_map(corpus_train, content_transformer(removeNumbers))
corpus_train = tm_map(corpus_train, content_transformer(removePunctuation))
corpus_train = tm_map(corpus_train, content_transformer(stripWhitespace))
corpus_train = tm_map(corpus_train, content_transformer(removeWords), stopwords("en"))
```

Pre-processing was carried out on the text in the articles so the words
could be processed by a computer. All of the letters were changed to
lowercase, and numbers, punctuation, extra white spaces, and stop words
were removed.

*Create Document Term Matrix*

``` r
DTM_train = DocumentTermMatrix(corpus_train)
DTM_train = removeSparseTerms(DTM_train, 0.92)
DTM_asmatrix = as.matrix(DTM_train)
```

A document-term matrix was created to count the frequency of a word in
each document. Sparse terms were removed from the dataset because they
may introduce noise. Terms were considered sparse if they appeared in
less than 92% of the documents. Several values between 90 and 100% were
tested to determine what percentage should be used as the threshold for
sparse terms. 92% was found to be the value that lead to the highest
test accuracy without a great reduction in the number of terms.

*Create test corpus, tokenization, preprocessing*

``` r
setwd("/Users/pujas/OneDrive/Documents")
files_test = Sys.glob('ReutersC50/C50test/*/*.txt')
docs_test = lapply(files_test, readerPlain)

corpus_test = Corpus(VectorSource(docs_test))
corpus_test = tm_map(corpus_test, content_transformer(tolower))
corpus_test = tm_map(corpus_test, content_transformer(removeNumbers))
corpus_test = tm_map(corpus_test, content_transformer(removePunctuation))
corpus_test = tm_map(corpus_test, content_transformer(stripWhitespace))
corpus_test = tm_map(corpus_test, content_transformer(removeWords), stopwords("en"))

DTM_test = DocumentTermMatrix(corpus_test)

#Terms(DTM_train)
#Terms(DTM_test)
#summary(Terms(DTM_test) %in% Terms(DTM_train))

#DTM_test2 = DocumentTermMatrix(corpus_test,
#                               control = list(dictionary=Terms(DTM_train)))
DTM_test2 = removeSparseTerms(DTM_test, 0.92)
DTM_testasmatrix = as.matrix(DTM_test2)
#summary(Terms(DTM_test2) %in% Terms(DTM_train))
```

The test data was pre-processed using the same steps used on the
training data.

### Random forest

``` r
library(randomForest)
```

    ## Warning: package 'randomForest' was built under R version 3.6.1

``` r
colnames(DTM_asmatrix) = make.names(colnames(DTM_asmatrix))
df <- data.frame(Author = class_labels, DTM_asmatrix)

colnames(DTM_testasmatrix) = make.names(colnames(DTM_testasmatrix))
df_test <- data.frame(Author = class_labels, DTM_testasmatrix)

rf = randomForest(Author~.,data=df)
CM = table(predict(rf), df_test$Author)
accuracy = (sum(diag(CM)))/sum(CM)
```

Random forest was run on the training articles and used to predict the
author of the test articles. The document-term matrix was converted to a
data frame and a column was added providing the author of that article.
Random forest had an accuracy of 76.16%.

# Association Rule Mining

``` r
library(tidyverse)
library(tidyr)
library(arules)
library(arulesViz)
library(dplyr)
# Load data as transactions
setwd("/Users/pujas/OneDrive/Documents")
groceries = read.transactions('groceries.txt',sep=',',header=FALSE)
str(groceries)
```

    ## Formal class 'transactions' [package "arules"] with 3 slots
    ##   ..@ data       :Formal class 'ngCMatrix' [package "Matrix"] with 5 slots
    ##   .. .. ..@ i       : int [1:43367] 29 88 118 132 33 157 167 166 38 91 ...
    ##   .. .. ..@ p       : int [1:9836] 0 4 7 8 12 16 21 22 27 28 ...
    ##   .. .. ..@ Dim     : int [1:2] 169 9835
    ##   .. .. ..@ Dimnames:List of 2
    ##   .. .. .. ..$ : NULL
    ##   .. .. .. ..$ : NULL
    ##   .. .. ..@ factors : list()
    ##   ..@ itemInfo   :'data.frame':  169 obs. of  1 variable:
    ##   .. ..$ labels: chr [1:169] "abrasive cleaner" "artif. sweetener" "baby cosmetics" "baby food" ...
    ##   ..@ itemsetInfo:'data.frame':  0 obs. of  0 variables

``` r
summary(groceries)
```

    ## transactions as itemMatrix in sparse format with
    ##  9835 rows (elements/itemsets/transactions) and
    ##  169 columns (items) and a density of 0.02609146 
    ## 
    ## most frequent items:
    ##       whole milk other vegetables       rolls/buns             soda 
    ##             2513             1903             1809             1715 
    ##           yogurt          (Other) 
    ##             1372            34055 
    ## 
    ## element (itemset/transaction) length distribution:
    ## sizes
    ##    1    2    3    4    5    6    7    8    9   10   11   12   13   14   15 
    ## 2159 1643 1299 1005  855  645  545  438  350  246  182  117   78   77   55 
    ##   16   17   18   19   20   21   22   23   24   26   27   28   29   32 
    ##   46   29   14   14    9   11    4    6    1    1    1    1    3    1 
    ## 
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   1.000   2.000   3.000   4.409   6.000  32.000 
    ## 
    ## includes extended item information - examples:
    ##             labels
    ## 1 abrasive cleaner
    ## 2 artif. sweetener
    ## 3   baby cosmetics

*Get association
rules*

``` r
groceryrules = apriori(groceries, parameter=list(support=.005, confidence=.1, maxlen=4,minlen=2))
```

    ## Apriori
    ## 
    ## Parameter specification:
    ##  confidence minval smax arem  aval originalSupport maxtime support minlen
    ##         0.1    0.1    1 none FALSE            TRUE       5   0.005      2
    ##  maxlen target   ext
    ##       4  rules FALSE
    ## 
    ## Algorithmic control:
    ##  filter tree heap memopt load sort verbose
    ##     0.1 TRUE TRUE  FALSE TRUE    2    TRUE
    ## 
    ## Absolute minimum support count: 49 
    ## 
    ## set item appearances ...[0 item(s)] done [0.00s].
    ## set transactions ...[169 item(s), 9835 transaction(s)] done [0.01s].
    ## sorting and recoding items ... [120 item(s)] done [0.00s].
    ## creating transaction tree ... done [0.01s].
    ## checking subsets of size 1 2 3 4

    ## Warning in apriori(groceries, parameter = list(support = 0.005, confidence
    ## = 0.1, : Mining stopped (maxlen reached). Only patterns up to a length of 4
    ## returned!

    ##  done [0.01s].
    ## writing ... [1574 rule(s)] done [0.00s].
    ## creating S4 object  ... done [0.00s].

``` r
arules::inspect(groceryrules)
```

The initial thresholds for support and confidence were low so the
apriori function would return a lot of rules. Higher thresholds are
applied below to identify the most important rules.

*Look at
subsets*

``` r
arules::inspect(subset(groceryrules, subset=lift > 2.5 & confidence > 0.4))
```


A lift value of 2.5 indicates that the selected rules are 2.5 times
better than random guessing. A confidence value of 0.5 was used so that
only rules that had high likelihoods of being true given the factors on
the left hand side of the rule would be used. These thresholds allowed
us to identify the most important rules.

*Make plots*

``` r
# plot all the rules in (support, confidence) space
plot(groceryrules)
```

    ## To reduce overplotting, jitter is added! Use jitter = 0 to prevent jitter.

![](figure-gfm/unnamed-chunk-46-1.png)<!-- -->

``` r
# swap the axes and color scales
plot(groceryrules, measure = c("support", "lift"), shading = "confidence")
```

    ## To reduce overplotting, jitter is added! Use jitter = 0 to prevent jitter.

![](figure-gfm/unnamed-chunk-46-2.png)<!-- -->

``` r
# "two key" plot: coloring is by size (order) of item set
plot(groceryrules, method='two-key plot')
```

    ## To reduce overplotting, jitter is added! Use jitter = 0 to prevent jitter.

![](figure-gfm/unnamed-chunk-46-3.png)<!-- -->

``` r
# graph-based visualization
sub1 = subset(groceryrules, subset=confidence > 0.5 & lift > 2.5)
plot(sub1, method='graph')
```

![](figure-gfm/unnamed-chunk-46-4.png)<!-- -->

``` r
# export
saveAsGraph(sub1,file="stronggroceryrules.graphml")
```

*Gephi plots*

``` r
library("png")
pp <- readPNG("grocery_network2.png")
plot.new() 
rasterImage(pp,0,0,1,1)
```

![](figure-gfm/unnamed-chunk-47-1.png)<!-- --> 

The network shown sizes nodes based on their betweenness centrality. The
colors show the modularity classes. This network tells us that
vegetables and whole milk are highly influential across the whole
network. The modularity classes highlights the items that are most
commonly purchased together. The unlabeled nodes represent itemsets with
more than one item.
