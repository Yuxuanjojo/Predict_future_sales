# Predict Future Sales

This is the final project for "How to win a data science competition" Coursera course, also a competition on Kaggle. According to the competition, I am supposed to predict the sale for a series of Russian stores (data was provided by one of the largest Russian software firms - 1C Company). 

In the dataset, dataset introduction are as follows:

* sales_train.csv - the training set. Daily historical data from January 2013 to October 2015.
* test.csv - the test set. You need to forecast the sales for these shops for November 2015.
* sample_submission.csv - a sample submission file in the correct format.
* items.csv - supplemental information about the items/products.
* item_categories.csv - supplemental information about the items categories.
* shops.csv- supplemental information about the shops.

The introduction of each column are as follow:

* ID: an Id that represents a (Shop, Item) tuple within the test set
* shop_id: unique identifier of a shop
* item_id: unique identifier of a product
* item_category_id: unique identifier of item category
* item_cnt_day: number of products sold. You are predicting a monthly amount of this measure
* item_price: current price of an item
* date: date in format dd/mm/yyyy
* date_block_num: a consecutive month number, used for convenience. January 2013 is 0, February 2013 is 1,..., October 2015 is 33
* item_name: name of item
* shop_name: name of shop
* item_category_name: name of item category
 
Check the dataset:
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/dataset_review1.png?raw=true" width="400px">    
</p>  

<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/dataset_review2.png?raw=true" width="400px">    
</p>  

Here is the summary of the top 20 item_category_id with the most products.
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/product_summary.png?raw=true" width="400px">    
</p>  

As I am going to analyze the sale for the whole stores, I am going to group by the product, which is “date_block_num” in “sales_train.csv” and calculate the sale value of each month by multiple “item_cnt_day” and “item_price”, then sum them. As a result we got a time series for sales value. Here is the visualized Sale value time series.
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/sale_value_ts.png?raw=true" width="400px">    
</p>  
To predict a time series, I consider the ARIMA model. So the mind map should be like:

<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/flowchart.png?raw=true" width="250px">    
</p>  

**ADF test** is used to check whether the time series is stable. When the test statistic value is smaller than critical value (5%), there would be 95% possibility the time series is stationary. If not, take **differential processing** to deal with the time series to get a new time series, after once or several times differential processing until the new time series is stationary. Here, adfuller.() is a meaningful method for us to check whether this is a stationary time series.     

<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/ADF_test.png?raw=true" width="400px">    
</p>  

When the time series is stationary, I need a **white noise test** to ensure this time series is reasonable to analyze. Generally, we can check the **mean/standard deviation**, if they change over the time, that means this is not a white noise. the method .mean(), .std() helped me. Here is the trend of mean and Standard Deviation.
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/mean&SD.png?raw=true" width="400px">    
</p> 

Obviously, the mean and standard deviation changed over time, thus this is not a white noise time series.
After differential processing and white noise test, there is a stationary non-white-noise time series. Then we can check the **ACF** and **PACF**, according to the cut off and tail out who finds the best value of **p** and **q**. However, according to the plot, it can be a bit confusing, so I use smt.graphics.plot_acf() and visualize it to find the best order for p and q. Here is the plot of ACF and PACF for AR model(1,0), (2,0) and MA model (0,1), (0,2). Here it is:
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/AR-1.png?raw=true" width="400px">    
</p> 
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/AR-2.png?raw=true" width="400px">    
</p> 
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/MA-1.png?raw=true" width="400px">    
</p> 
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/MA-2.png?raw=true" width="400px">    
</p> 

- AR(1) process has ACF tailing out and PACF cutting off at lag=1, 
- AR(2) process has ACF tailing out and PACF cutting off at lag=2; 
- MA(1) process has ACF cut off at lag=1 and PACF tailing out, 
- MA(2) process has ACF cut off at lag=2 and PACF tailing out. 

According to the plot, it can be a bit confusing, so I use asmt.ARMA().fit() to fit the model and use a loop to find the best order to fit the ARMA model by comparing the AIC which got by tmp_mdl.aic.

<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/code_find_arma.png?raw=true" width="400px">    
</p> 

Then I got the ARMA model. 
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/ARMA(2.2).png?raw=true" width="400px">    
</p> 

Next step of getting a model is to predict. I chose library **Prophet** from **fbprophet**, which is an easy tool to predict the time series. Here is the result. 
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/predict_value.png?raw=true" width="300px">    
</p> 
<p align="center">
  <img src="https://github.com/Yuxuanjojo/Predict_future_sales/blob/main/img_predict/predict_plot.png?raw=true" width="450px">    
</p> 
