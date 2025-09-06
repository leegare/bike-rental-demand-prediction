# Project3
## Summary: 
This study examines the determinants of public bike-sharing demand using the Capital Bikeshare dataset (2011–2012) enriched with weather data. Statistical analyses reveal significant differences in ridership patterns across weekdays and weekends, as well as between casual and registered users. Weather, particularly perceived temperature, emerges as a dominant predictor of demand, exhibiting a strong correlation with ridership volume. Regression and time-series models are applied to forecast demand, with linear regression (using perceived temperature) outperforming ARIMA and manual sinusoidal models in predictive accuracy on test data. Findings underscore the importance of integrating temporal and meteorological factors in forecasting systems to enhance resource allocation, optimize bike distribution, and mitigate service shortages within urban mobility networks.

## Introduction and Goal: 

If you aspire to be environmentally friendly, avoid your city's traffic and your city has a public bicycle sharing system like the Velib in France, the Santander Cycles in London or the Capital Bikeshare in Washington D.C. then you as a responsible Earth-dweller may have been confronted with two problems: not finding a bike in your nearest bike station and not finding a free spot at your destination's station. To answer this issue, the bike company needs to know the variability of the demand throughout time. 

This project analyzes the statistical measures of the demand for rental bikes as a function of time and the weather. 

## Questions for Exploration

1. What's the distribution of the demand for rental bikes throughout time? Is the demand higher on weekdays than the weekends? On any normal day, is there a pattern in the bike usage over the course of time? 

2. Is there a difference in usage between a casual and a registered user? (Are they statistically different?)

3. How influential is the weather with both users? Will a nice day be more enticing to use the bike over a rainy day? Does the wind or temperature discourage people from taking the bike? 

4. Can the temperature felt be characterized by a combination of wind speed, humidity, and official temperature? 

5. Can we predict the count of bikers given the time, day, and weather? 

# Acquire the data

The dataset is taken from Kaggle's Bike Sharing Dataset, containing data of rental bikes from Capital Bikeshare for the period between the years of 2011 and 2012. This data is segmented by: 
- Date (segmented in year, month, and day) and time represented by the hour of use. 
- The number of bikes in use by the hour.  
- The user type: Registered and Casual (independent events). 
- Whether the day is a holiday or not, a working day or a weekend. 


Moreover, the weather information from freemeteo.com is merged with the dataframe, adding 5 columns: 
- A brief categorical description of the sky (clear, scattered clouds, light rain, heavy snow or rain)
- Temperature in Celsius normalized by a factor of 41, 
- Feels-like temperature (*atemp*) in Celsius normalized by a factor of 50, 
- Normalized humidity by a factor of 100 and 
- Normalized windspeed by a factor of 67




# PreProcessing

The following operations will prepare and refine the data for the analysis:

1. Parse e.g. convert strings to timestamps and extract date & time components to columns
2. Cast to the adequate data types
3. Handle missing or incomplete data
>There are some missing values; ideally, a filling function would consider the missing hourly values as well as the trend of the non-missing values throughout the remaining hours of the day. So an interpolation from the previous hour and the next seems appropriate. 

4. Normalize the number of bikes per day 
> The demand distribution is right-skewed with several outliers. To correct this, these values will be normalized in function of the total demand per day. 

5. Create downsampled Dataframes 
>Group the hourly data into daily data and daily data into weekly data. The daily sampling behaves more like a normal distribution than the hourly distribution. Without outliers on the total and registered type count, these two distributions seem pretty normal. 

## What is the distribution of the demand for rental bikes throughout the day? Is there a significant difference between the weekdays and the weekends? 

The purpose of this analysis is to see how the demand for bikes fluctuates throughout the week and the weekends, and thus develop a suitable model for the supply chain to keep the demand balanced.

![**Figure . % Q1WeekendVsWeekday**](reports/img/Q1WeekendVsWeekday.png)

A **t-Test** shows a p-value lower than the significance level (0.05), so **the null hypothesis is rejected**, which means if we were to construct a 95% confidence interval for the sample, it would not capture the weekday distribution mean. ***Therefore, the demand on weekends is different (statistically significant) than on weekdays, and we can notice this in the figure below, where the demand distribution is graphed by the hour.***

![**Figure . % Q1WeekendVsWeekdayTS**](reports/img/Q1WeekendVsWeekdayTS.png)

On the weekends, bikers start their day around 9 am and then it goes incrementally til 1 pm to then begin a smooth decline in the late night hours and reach an inactivity state around 5 am-6 am. 
Activity on weekdays appears to have 2 peaks, one at 9am and the other at 6pm. The comparison made by the t-test is noticeable, so we can appreciate how the demand is different between these 2 groups of the week. 

## Is there a difference in usage between a casual and a registered user?

The figure below shows a comparison of the distributions of the number of bikes in use by registered vs casual users in function of:
1. the time of day 
2. and whether it's a holiday/weekend or just a normal weekday. 

![**Figure . % Q2CasualVsRegisteredTS**](reports/img/Q2CasualVsRegisteredTS.png)

Figures 1, 3, and 4 show a similar distribution. Figure 1 represents the average usage for the casual user on a weekday peaking at 5pm, whereas figure 3 shows the casual user on a weekend having a high demand in the afternoon, and Figure 4 shows the registered users on a weekend also demanding high volumes of bikes starting at noon and carrying on in throughout the rest of the evening. 

It's important to consider that in the distribution of the demand's mean in the weekdays graph, the influence of the casual users was not big enough to modify the shape of the distribution because the user type ratio is slightly higher than 6:1. 

We can notice that registered users are more intense over the weekends, starting to bike earlier than their fellow casual users. So it is interesting to see if these 2 proportions are different from each other in terms of bike usage. For that, another 2-sample t-test will answer our question.  

![**Figure . % Q2CasualVsRegisteredTTest**](reports/img/Q2CasualVsRegisteredTTest.png)

The p-value is so small that the null hypothesis can be rejected and thus ***both groups are not identical but statistically different***

## Can the temperature felt be characterized by a combination of wind speed, humidity, and official temperature? 

The scatter plot of the hourly records looks a bit noisy, and with some outliers around the "feels like" temperature of 0.25. A resampling showed an improvement in the visibility of the possible relationship. 

![**Figure . % Q3WeatherPairPlot**](reports/img/Q3WeatherPairPlot.png)

Unfortunately, the wind speed and the humidity show no relationship with other factors that can be deduced at a simple glance.
However, the temperature is highly correlated (0.99) with the temperature felt. 

To find the optimal number and best combination of predictors, the **cross-validation** method threw the model with the least RSS and 3 predictors, where the summary is shown below:

![**Figure . % Q3summary**](reports/img/Q3summary.png)

The p-value for windspeed is not small enough, so we can consider temp and humidity as the best predictors for the temperature felt. The figure below shows the residual plot with a mean of zero and fairly homoscedastic 

![**Figure . % Q3ResidualPlot**](reports/img/Q3ResidualPlot.png)

Residuals do not contradict the linear assumption. 
The quantile-quantile plot of the residuals shows that they are normally distributed, making the model a good fit. The zone of interest in the q-q plot is the middle, so the extreme points on the sides can be ignored. 

![**Figure . % Q3TS**](reports/img/Q3TS.png)

## 4. How influential is the weather on both users? 

Will a nice day be more enticing to use the bike over a rainy day? Does the wind and/or temperature discourage people from taking the bike?
In the figure below, the total demand is displayed throughout the 2 years with a sampling period of a week. The demand seems to fluctuate in cycles, seasons, and even has a positive trend. 

From the figure below, we can see that the temperature measured and the temperature felt have a strong correlation with the demand for bikes. 

![**Figure . % Q4TS**](reports/img/Q4TS.png)

The count seems to correlate with the temperature and the feels-like temperature. During the 2 years, the cycles of the temperature seem to follow those of the demand. 

Furthermore, it can be seen that almost every change in humidity, there is an opposite change in demand...

Similarly to the previous question, an exhaustive search for the optimal combination of predictors took place, and the cross-validation method showed the best results (lowest RSS) with a single predictor model using temperature-felt. The temperature felt can therefore be considered as our main weather descriptor, as seen earlier, it's a variable that is determined by a somewhat linear relationship with temperature, measured windspeed, and humidity. 

The figure below shows the method implemented to determine the optimal number of predictors with the lowest value of RSS, AIC, and BIC, and the highest value of the adjusted squared. 

![**Figure . % Q4CVBestNoPredictors**](reports/img/Q4CVBestNoPredictors.png)

The residual plot and quantile plot below show somewhat normally distributed residuals with equal variance 

![**Figure . % Q4CVResidualPlot**](reports/img/Q4CVResidualPlot.png)

![**Figure . % Q4CVqqPlot**](reports/img/Q4CVqqPlot.png)

**Furthermore** the correlation can be improved by adding polynomial degrees to the temperature felt slope. Below is shown how, after the 3rd polynomial degree, the squared correlation coefficient is reaching a value of 0.45

![**Figure . % Q4Polynomials**](reports/img/Q4Polynomials.png)

## 5. On any normal day, is there a pattern of usage over the course of time? Can we predict the count of bikers given the time, day, and weather?

I will use the weekly average to reduce the seasonality and trend of the total demand. Then it will go through a smoothing process of 2 days average, and the result is shown below:

![**Figure . % Q57DayRollingMean**](reports/img/Q57DayRollingMean.png)

Using the weekly sampled data, I tested forecasting the demand with the aid of the conventional models, and the most promising ones were: 
1. The manual modeled by a customized sinusoidal signal. 
2. A regression model with temperature felt as its predictor. 
3. A 101 ARIMA model. 

![**Figure . % Q5AllPredictions**](reports/img/Q5AllPredictions.png)

The ARIMA model has the same RMSE as a linear model characterized by the trend of the signal. The differenced signal resulted in a stationary signal created by subtracting the demand with a Simple Exponential Smoothing model, as it showed a high fidelity following the demand. However, the best model with the lowest RMSE is the manual sine model.

And their RMSE: 
![**Figure . % Q5RMSEResults**](reports/img/Q5RMSEResults.png)

Looks like the sine model yields the best results. Let's see if it applies to the test dataset. I will now forecast the 5 remaining months in the test set using the 3 best models. 

![**Figure . % Q5Test**](reports/img/Q5Test.png)

The best model for the training set is not the best for the test set. In this testing phase looks like the linear regression model has the best prediction. Recalling the predictor used for this model was the temperature felt. 
The ARIMA model didn't fall behind, predicting better than the manual model sine. 

![**Figure . % Q5TestResults**](reports/img/Q5TestResults.png)

## Conclusion and further work

The manual model showed pretty good results on the training set; however, the results were different on the test set, where the regression model got the lowest RMSE value of 0.15. 
The parameters of the manual model are not optimal, which means an exhaustive search for the lowest RMSE can be done in future work to beat the linear regression model’s RMSE.
This study shows how the temperature is a decisive descriptor of the number of bikes rented, and knowing and understanding weather patterns over time can save bike rental companies from problems related to the number of bikes available.

There is also a pattern in the daily demand; adding day of week as a categorical variable could improve the model accuracy because the loss rate will have been minimized compared to a linear average on all entities.

