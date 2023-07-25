# Functime Benchmark
## TLDR
Functime's AutoML time series forecasting does not understand how to globally forecast for time series data. Missing vital features to test such as scaling and differencing, the procedure produces terrible results with little hope of obtaining anything particularly useful even if ran until the end of time. Thus it should not be marketed as an AutoML solution (one of my pet peeves) and should not be used unless supplied with feature-rich exogenous data.

No hate to the team (in fact they have been very helpful with my questions!), just to the AutoML method!

## Introduction
With the new functime being released there have been a ton of benchmarking of the speed but as of yet no tests on the built in 'AutoML' method. These notebooks will test the auto LightGBM procedure on the M4 weekly and yearly datasets. I did run into some issues with datetime for the package so I did not add calendar features via the pipeline and I would argue that I shouldn't have to anyway due to it being 'AutoML'. Furthermore, there were errors due to test sizes and lag lengths which restricted some of the values to what I would consider suboptimal testing strategies. 

Either way, I don't think these ultimately change the outcome too much...the team still has some work to do...

For the benchmark, we will compare functime with a package I whipped up awhile ago for automated forecasting that is wrapped around mlforecast which turns out to be far more sophisticated (I guess). For the record, I don't really recommend putting séance into a prod environment but it typically gives you what you want out of an AutoML solution - reasonable results with minimal effort. It tests differencing, scaling types, seasonality, trend-basis functions, using the ID as LightGBM's categorical feature along with LightGBM's model parameters. 

## Weekly Datasets
The run can be found in a notebook in the directory but here are the results:

| Model | Run Time (sec) | SMAPE | M4 Rank |
| -------- | ------- | ------- | ------- |
| functime | 3545 | 11.59 | 51 |
| séance (Restricted) | 2592 | 7.32 | 11 |
| Naive Model | - | 9.161 | 39 |

I just want to reiterate that I don't care too much about the run time, as long as the methods have enough time to produce something reasonable. After I saw these results I took a deeper look at functime's method and it only tunes for the lags and LightGBM's parameters. **This is model abuse**. The tree will have to work SO HARD to produce forecasts without any scaling. Some datasets will work fine in this setup but most of the time you will need differencing or scaling to make a global model work in a pure time series context. This is a misunderstanding of how these models work. 

The performance is significantly worse than the naive method. All of this makes sense when we look at the plots of some of the forecasts, the tree isn't given enough to fit differing magnitudes (this is why scaling is so important). This leads to forecasts that CAN'T even overfit our test set even if we optimized for it! 

You may notice the 'Restricted' tag with séance, that is because we have to use suboptimal settings due to functime errors, these errors are ignored by séance (for better or worse) but an unrestricted model will get a 6.9 SMAPE on the dataset using a longer test size. Either way, the difference in accuracy is night and day.

## Yearly Datasets
I chose to do yearly as well to deal with the seasonality issue I was having with functime, this test should be more 'fair' for functime since it really is only testing the more 'sophisticated' features of séance. An additional wrinkle is that tree models can sometime struggle vs other time series methods in straight up yearly data, at least in my experience. This can be overcome slightly with some good feature engineering which is something that, as I have mentioned, is not being done by functime. Anyway, here are the results:

| Model | Run Time (sec) | SMAPE | M4 Rank |
| -------- | ------- | ------- | ------- |
| functime | 5077 | 19.41 | 55 |
| séance (Restricted) | 8740 | 13.49 | 4 |
| Naive Model | - | 16.31 | 40 |

Functime still performs miserably, and there is nothing that will change that. You can't give a tree some lagged values and poke it. You need to give it features for it to partition and scaled target values so it can more easily average across time series of different magnitudes. 

## Conclusion
The Functime team needs to address these major issues before they can call this an 'AutoML' solution. It's pretty simple, which is why the lack of the features is so concerning. Let's test out seasonality (easiest with fourier features and a given seasonal period IMO), scaling (a robust boxcox, minmax, standard, log etc.), differencing (just 1 difference, much more than that gets harder to tune IMO), using an id variable, and throw in some linear basis functions to allow for more dynamics. Then throw it in a black box and watch as some magic comes out. 

These will get you most of the way, and don't call the method 'AutoML' until you have them!
