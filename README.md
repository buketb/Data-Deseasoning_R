# Data-Deseasoning_R
Deseasoning the Corona data. The decomposition of the trend and cycle.

library(readxl)
library(openxlsx)
library(PerformanceAnalytics)
library(zoo)
require(xts)
require(zoo)
library(ggplot2)
library(tidyquant)
library(mFilter)

data<-read.xlsx("data.xlsx")

#date, daily, 7da(7-days-average), logd (log of daily), dummies

#date format:

data$date= as.Date(data$date, origin = "1899-12-30")

#data as xts:

xtsdata <- xts(data[,2:10], order.by = data$date, frequency = 1)

#deviation between daily data and 7-days-average:
dev <- cbind(date=data$date, dev=as.data.frame(data[,2]-data[,3]))

#deviation as xts:

xtsdev <- xts(dev[,2], order.by = dev$date, frequency = 1)

#FIGURE-1:

par(mfrow=c(2,1),mar=c(3,3,2,1),cex=.8)

plot(xtsdata[,1],lwd=2,main = "Daily new Corona cases and moving 7-day average",
     ylim=c(-1000,30000))
     
lines(xtsdata[,2],col="blue",lwd=2)

addLegend("top",
       c("Daily","7-day-average"),
       fill=c("black","blue"),cex=.8)
       
plot(xtsdev[,1],lwd=2,main = "Deviation between daily and 7-day-average",
     ylim = c(-5000,14000))
     
     
#FIGURE-2:
#OLS:

model <- lm(logd~ Mo+Tu+We+Th+Fr+Sa, data= xtsdata)

summary(model)


ut <- model$residuals

vt <- exp(ut)

#vt is the exp(ut), where ut is the residulas from the ls resgression.

#Seasonally adjusted series:

'Once you have run the regression, compute the residuals u(t) and 
use v(t)=exp(u(t)) as seasonally adjusted series. You will find that 
this series does not fit the original series as you subtracted the mean. 
Therefore, compute the mean of the original series and add a constant 
to v(t) such that it has the same mean.'


sas <- vt + 2

#I added the mean value

sas <- as.data.frame(cbind(date=xtsdata$date, sas))


sas <- cbind(date=data$date, sas)

#data as xts:

xtssas <- xts(sas[,2],order.by=sas$date, frequency = 1)



#create day-effects:

dayeff<-read.xlsx("dayeff.xlsx")

#date format:

dayeff$date= as.Date(dayeff$date, origin = "1899-12-30")

#dayeff is the day effect series


dayeff <- xts(dayeff[,2], order.by = dayeff[,1],frequency =1)

dayeff =dayeff/1000000


dayeff <- dayeff - mean(dayeff)


par(mfrow=c(2,1),mar=c(3,3,2,1),cex=.8)

plot(xtsdata[,3],lwd=2,main = "Log of daily new Corona cases and weekday-adjusted series",
     ylim = c(-3,10))
     
lines(xtssas,col="blue",lwd=2)

addLegend("bottomright", c("Daily","Seasonally adj."),fill=c("black","blue"),cex=.8)


plot(dayeff,lwd=2,main = "The day-effect from the dummy regression", ylim = c(-1,1))
     
     
     
#FIGURE-3:


#hp-filter:

tsdataf <- hpfilter(xtsdata[,3], freq=1600, drift=F)

cycle <- as.data.frame(tsdataf$cycle)


cycle <- cbind( date=data$date, cycle)


#cycle from hp-filter as xts:

xtscyc <- xts(cycle[,2],order.by=cycle$date, frequency = 1)


trend <- as.data.frame(tsdataf$trend)


trend <- cbind( date=data$date, trend)


#cycle from hp-filter as xts:

xtstrend <- xts(trend[,2],order.by=trend$date, frequency = 1)


par(mfrow=c(2,1),mar=c(3,3,2,1),cex=.8)

plot(xtsdata[,3],lwd=2,main = "Log of daily new Corona cases and HP-filtered series",
     ylim= c(-1,7))
     
lines(xtstrend,col="blue",lwd=2)

addLegend("bottomright", c("Daily","HP-filtered"),fill=c("black","blue"),cex=.8)



plot(xtscyc,lwd=2,main = "", ylim = c(-0.5,0.5))


