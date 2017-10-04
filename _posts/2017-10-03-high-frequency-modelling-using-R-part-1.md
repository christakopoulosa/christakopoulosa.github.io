---
layout: post
title:  "High frequency modelling using R part 1"
date:   2017-10-03
categories: jekyll update
---

<!-- Global Site Tag (gtag.js) - Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-107545335-1"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments)};
  gtag('js', new Date());

  gtag('config', 'UA-107545335-1');
</script>


In this series I will discuss some issues that I faced during writing my Master thesis on financial risk forecasting using high frequency data.

High frequency data are difficult to find. There are commercial sources but you need to pay fees which sometimes are overwhelming if you are not financially supported by your University or workplace. The idea is, the bigger the granularity of the data the greater the monetary cost. Looking for data to construct my dataset and conduct my research I found <a href="http://www.histdata.com/">http://www.histdata.com</a> which offers free tick data in monthly batches.

So the purpose of part 1 of this series is to guide you through the process of constructing a dataset using free data from histdata.com using R.

Firstly, we download the data from http://www.histdata.com/download-free-forex-data/?/ascii/tick-data-quotes. For my example I use the Euro currency against the Australian dollar (EUR/AUD) from January to March of 2015. You can expand the dataset according to your needs. However, please remember due to the sheer volume of the tick dataset you need ample computer power to perform these tasks. The least you should do is to switch to a x64 computer.

The dataset consists of a date stamp, a bid price, an offer price and the volume of each tick. The ticks are recorded on Eastern Standard Time (EST) without daylight savings adjustments.

{% highlight ruby %}
##Preliminary actions##
getwd()
setwd("C:/~") #set the Working Directory
Sys.setenv(TZ="EST") #Eastern Standard Time

##Raw data manipulation##
data1<-read.csv("EURAUD_201501.csv", header = FALSE, col.names = c("TIMESTAMP", "BID", "OFR", "VOL"), stringsAsFactors = FALSE)
data2<-read.csv("EURAUD_201502.csv", header = FALSE, col.names = c("TIMESTAMP", "BID", "OFR", "VOL"), stringsAsFactors = FALSE)
data3<-read.csv("EURAUD_201503.csv", header = FALSE, col.names = c("TIMESTAMP", "BID", "OFR", "VOL"), stringsAsFactors = FALSE)
total1<- rbind(data1, data2, data3) #bind the data to one file
{% endhighlight %}

The date stamp needs some tweaking

{% highlight ruby %}
total1$TIMESTAMP = sub( '(?<=.{11})', ':', total1$TIMESTAMP, perl=TRUE ) #create clear timestamps

total1$TIMESTAMP = sub( '(?<=.{14})', ':', total1$TIMESTAMP, perl=TRUE )

total1$TIMESTAMP = sub( '(?<=.{17})', '.', total1$TIMESTAMP, perl=TRUE )

head(total1, 5)
{% endhighlight %}

Then we have to convert our data to an xts object. Xts package is loaded automatically along with the high frequency package. Also, as we work with tick data even the milliseconds count so we have to inform R to take the milliseconds intro consideration explicitly.

{% highlight ruby %}
options(digits.secs = 3) #set the accuracy of tick-data to millieseconds

onexts=xts(total1[,c(2,3)], order.by = as.POSIXct(total1$TIMESTAMP, tz = "EST",

                                                  format = "%Y%m%d %H:%M:%OS")) #convert file to an xts object


We construct 5,10 and 15 minute intervals using the previous tick aggregation technique
{% endhighlight %}


{% highlight ruby %}
fivemin = aggregatets(onexts, FUN = "previoustick", on = "minutes", k = 5) #create 5 minute intervals

tenmin = aggregatets(onexts, FUN = "previoustick", on = "minutes", k = 10) #10 minute intervals

fifteenmin = aggregatets(onexts, FUN = "previoustick", on = "minutes", k = 15) #15 minute intervals

head(fivemin, 5)
{% endhighlight %}


And we create the final raw returns

{% highlight ruby %}

##Create the final raw returns##

fivemin_return = fivemin

fivemin_return$PRICE<-((log(fivemin_return$BID)+log(fivemin_return$OFR))/2) #logarithmic price

fivemin_return$DPRICE = diff(fivemin_return$PRICE, lag = 1) #logarithic returns
{% endhighlight %}


And the final plot

{% highlight ruby %}

##Plot##

plot(fivemin_returns[,1], main = "EUR/AUD Returns", auto.grid = FALSE, major.ticks = "months", minor.ticks = FALSE, col = "red")
{% endhighlight %}





