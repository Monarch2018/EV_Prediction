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
			    <li class = "selected"><a href="https://monarch2018.github.io/ev_prediction/preprocessing/">Preprocessing</a></li>
			    <li><a href="https://monarch2018.github.io/ev_prediction/timeseries/">Time Series</a></li>
			    <li><a href="https://monarch2018.github.io/ev_prediction/baseline/">Baseline</a></li>
			    <li><a href="https://monarch2018.github.io/ev_prediction/prediction/">Prediction</a></li>
			</ul>
		</div>
	
   </div>

### Merge Vehicle and nonvehicle dataset (1:1)

```
# Preprocess two raw datasets
novehicle = pd.read_csv(os.path.join("/content","novehicle.csv"), header = 0, keep_default_na = False)
vehicle = pd.read_csv(os.path.join("/content","metadata.csv"), header = 0, keep_default_na = False)
vehicle['label'] = '1'
novehicle['label'] = '0'
vehicle = vehicle.dropna(how = 'any', axis = 0)
novehicle = novehicle.dropna(how = 'any', axis = 0)
...
# Merge two datasets
capstone = vehicle.append(novehicle, ignore_index = True)
# Export new dataset to google drive and name it 'capstone'
from google.colab import drive
drive.mount('drive')
capstone.to_csv('capstone.csv')
!cp capstone.csv drive/My\ Drive/
...
# Check new dataset
Capstone = importfile(file_id = '1-1lVd9sPctCOOrpxJ8vpf7qf6zhU3EhN')
Capstone = pd.read_csv(os.path.join("/content","capstone.csv"), header = 0, keep_default_na = False)
Capstone.groupby('label').size()
```
![merge](/img/merge.png#merge)



### Dataport link

[Dataport](https://dataport.cloud/) hosts all the data collected via Pecan Street’s water and electricity research.

### University Access Database 
After sign up an account, I successfully got access to the University Access level database which is available to current faculty, staff, and students at a four-year postsecondary educational institution in the U.S. or equivalent-level institution in other nations. 
![university](/img/university.png#university)


### Or some code?

Some code might go here:

```
x <- 5 # Here's some R code
```

What if I just paste the HTML for a plotly plot?

We can do it with a line of markdown that looks like this (without the slashes - I haven't solved that problem just yet...):
```
\{\% include jupyter-basic_bar.html \%\}
```
{% include jupyter-basic_bar.html %}
