---
layout: page
title: Oracle Data Science Capstone project
subtitle: Electric Vehicle Present Discovery
bigimg: /img/pecanstreet.jpg
---

   <link rel="stylesheet" type="text/css" href="css/main.css" />

   <div id= "main">
		<div id="menubar">
			<ul id="menu">
			    <li><a href="https://monarch2018.github.io/ev_prediction/index.html">Overview</a></li>
			    <li><a href="https://monarch2018.github.io/ev_prediction/data/">Data</a></li>
			    <li><a href="https://monarch2018.github.io/ev_prediction/preprocessing/">Preprocessing</a></li>
			    <li class = "selected"><a href="https://monarch2018.github.io/ev_prediction/timeseries/">Time Series</a></li>
			    <li><a href="https://monarch2018.github.io/ev_prediction/baseline/">Baseline</a></li>
			    <li><a href="https://monarch2018.github.io/ev_prediction/prediction/">Prediction</a></li>
			</ul>
		</div>
	
   </div>

### Main Process (Deployment)
1. import libraries and methods 
2. import timeseries dataset
3. Family choose
4. Test Harness
4. Persistence
5. Data Anlysis
- Summary Statistics
- Line Plot
- Density Plot
- Box and Whisker Plots
6. ARIMA Models
- Manually Configured ARIMA
- Grid Search ARIMA Hyperparameters
- Review Residence Errors
- Box-Cox Transformed Dataset
7. Model Validation
- Finalize Model
- Validate Model

### 1. Import Library and datset
```
# Main packages and functions
from sklearn.metrics import mean_squared_error
from statsmodels.tsa.stattools import adfuller
from statsmodels.graphics.tsaplots import plot_acf
from statsmodels.graphics.tsaplots import plot_pacf
from statsmodels.tsa.arima_model import ARIMA
from statsmodels.graphics.gofplots import qqplot
from statsmodels.tsa.arima_model import ARIMAResults
from scipy.stats import boxcox
# Load timeseries dataset
timeseries = importfile(file_id = '1bx2sY8TGrJ7DVRoZ_yjXxp3tQgqn4Zix')
```

### 2. Deploy function which can analyze and predict among 76 families
```
from pandas import read_csv
df = read_csv('timeseries.csv', index_col=2, header = 0, parse_dates=True, squeeze=True)
df.drop(['Unnamed: 0'], axis = 1, inplace = True)

daily_summary = pd.DataFrame()
series = pd.DataFrame()
ids = df.dataid.unique().tolist()

for id in ids:
  daily_summary_id = df[df['dataid'] == id]
  daily_summary['dataid'] = daily_summary_id.dataid.resample('D').max()
  daily_summary['use'] = daily_summary_id.use.resample('D').sum()
  daily_summary['label'] = daily_summary_id.label.resample('D').max()
  series = series.append(daily_summary)
series.dropna(inplace = True)

print('There are totally 76 families, please enter which family you want to predict:')
x = int(input('Enter family number:'))
series = series[series['dataid'] == ids[x-1]]['use']
print("Number %d family's dataid is %d"%(x,ids[x-1]))
```
eg:
![family](/img/family.png#family)

### 3. Test Harness

- Define a Validation Dataset
- Deploy a Method for Model Evaluation  

```
split_point = len(series) - 68
dataset, validation = series[0:split_point], series[split_point:]
print('Dataset %d, Validation %d' % (len(dataset), len(validation))) 
dataset.to_csv('dataset.csv')
validation.to_csv('validation.csv')
```
Dataset 297, Validation 68

### 4. Persistence

```
# prepare data
X = series.values
X = X.astype('float32')
train_size = int(len(X) * 0.50)
train, test = X[0:train_size], X[train_size:]
# walk-forward validation
history = [x for x in train]
predictions = list()
for i in range(len(test)):
  # predict
  yhat = history[-1]
  predictions.append(yhat)
  # observation
  obs = test[i]
  history.append(obs)
  print('>Predicted=%.3f, Expected=%3.f' % (yhat, obs))
# report performance
rmse = sqrt(mean_squared_error(test, predictions)) 
print('RMSE: %.3f' % rmse)
```
...

![persistence](/img/persistence.png#persistence)

**Conclusion:** Running the test harness prints the prediction and observation for each iteration of the test dataset. The code ends by printing the RMSE for the model. In this case, the persistence model achieved an RMSE of 12.701. This means that on average, the model was wrong by about 12 electrical usage for each prediction made.

### 5. Data Analysis

- Summary Statistics

```
print(series.describe())
```
![describe](/img/describe.png#describe)

**Conclusion:** The large spread in this series will likely make highly accurate predictions difficult if it is caused by random fluctuation (e.g. not systematic).

- Area Plot
{% include elec.html %}

- Histgram
{% include hist.html %}

**Conclusion:** 
1. The distribution is not Gaussian.
2. The distribution is left shifted and may be exponential or a double Gaussian.

- Box & Whisker Plot
{% include box.html %}

**Conclusion:** The observations suggest that the month-to-month fluctuations may not be systematic and hard to model.

### 6. ARIMA Model
#### I will approach this in four steps:
- Developing a manually configured ARIMA model.
- Using a grid search of ARIMA to find an optimized model.
- Analysis of forecast residual errors to evaluate any bias in the model.
- Explore improvements to the model using power transforms.

#### 6.1 Manually Configured ARIMA

```
# create a differenced time series
def difference(dataset):
  diff = list()
  for i in range(1, len(dataset)):
    value = dataset[i] - dataset[i - 1]
    diff.append(value)
  return Series(diff) 
X = series.values
# difference data
stationary = difference(X)
stationary.index = series.index[1:]
# check if stationary
result = adfuller(stationary) 
print('ADF Statistic: %f' % result[0]) 
print('p-value: %f' % result[1]) 
print('Critical Values:')
for key, value in result[4].items():
  print('\t%s: %.3f' % (key, value)) # save
stationary.to_csv('stationary.csv')
```
Output: ADF Statistic: -7.041553    p-value: 0.000000    Critical Values: 1%: -3.449, 5%: -2.870, 10%: -2.571.

**Conclusion:** Running the example outputs the result of a statistical significance test of whether the 1-lag differenced series is stationary. The results show that the test statistic value -8.763759 is smaller than the critical value at 5% of -2.872. This suggests that we can reject the null hypothesis with a significance level of less than 5%.

#### 6.2 Autocorrelation Function (ACF) and Partial Autocorrelation Function (PACF) plots

```
pyplot.figure(1)
pyplot.subplot(211)
plot_acf(series, lags=50, ax=pyplot.gca())
pyplot.subplots_adjust(left=None, bottom=None, right=None, top=1.5, wspace=None, hspace=0.25)
pyplot.figure(1)
pyplot.subplot(212)
plot_pacf(series, lags=50, ax=pyplot.gca())
pyplot.show()
```
![acf](/img/acf.png#acf)

**Conclusion:** This quick analysis suggests an ARIMA(15,1,2) on the raw data may be a good starting point. The model can be simplified to ARIMA(0,1,2).

After doing ACF and PACF, the RMSE reduced from 12.701 to 10.716. 

#### 6.3 Grid Search ARIMA Hyperparameters

- p: 0 to 4.
- d: 0 to 2.
- q: 0 to 4.

```
# evaluate combinations of p, d and q values for an ARIMA model
def evaluate_models(dataset, p_values, d_values, q_values): 
  dataset = dataset.astype('float32')
  best_score, best_cfg = float("inf"), None
  for p in p_values:
      for d in d_values:
        for q in q_values:
          order = (p,d,q)
          try:
            rmse = evaluate_arima_model(dataset, order)
            if rmse < best_score:
              best_score, best_cfg = rmse, order 
            print('ARIMA%s RMSE=%.3f' % (order,rmse))
          except:
            continue
  print('Best ARIMA%s RMSE=%.3f' % (best_cfg, best_score))
  
# evaluate parameters
p_values = range(0, 5)
d_values = range(0, 3)
q_values = range(0, 5)
warnings.filterwarnings("ignore")
evaluate_models(series.values, p_values, d_values, q_values)
```
![grid](/img/grid.png#grid)

#### 6.4 Box-Cox Transformed Dataset

```
X = series.values
transformed, lam = boxcox(X)
print('Lambda: %f' % lam)
pyplot.subplots_adjust(left=None, bottom=None, right=None, top=2, wspace=None, hspace=0.25)
pyplot.figure(1)
# line plot
pyplot.subplot(311)
pyplot.plot(transformed)
# histogram
pyplot.subplot(312)
pyplot.hist(transformed)
# q-q plot
pyplot.subplot(313)
qqplot(transformed, line='r', ax=pyplot.gca()) 
pyplot.show()      
```

![cox](/img/cox.png#cox)

```
for i in range(len(test)):
  # transform
  transformed, lam = boxcox(history)
  if lam < -5:
    transformed, lam = history, 1
  # predict
  model = ARIMA(transformed, order=(1,1,1))
  model_fit = model.fit(disp=0)
  yhat = model_fit.forecast()[0]
```
The result shows that RMSE is 10.476, again it improves the accuracy!

### 7 Model Validation
#### 7.1 Finalize Model

```
# monkey patch around bug in ARIMA class
def __getnewargs__(self):
  return ((self.endog),(self.k_lags, self.k_diff, self.k_ma))
ARIMA.__getnewargs__ = __getnewargs__

# prepare data
X = series.values
X = X.astype('float32')
# transform data
transformed, lam = boxcox(X)
# fit model
model = ARIMA(transformed, order=(1,1,1)) 
model_fit = model.fit(disp=0)
# save model
model_fit.save('model.pkl') 
numpy.save('model_lambda.npy', [lam])
```

#### 7.2 Validation Model

```
# load model
model_fit = ARIMAResults.load('model.pkl')
lam = numpy.load('model_lambda.npy')
```

![best](/img/best.png#best)
