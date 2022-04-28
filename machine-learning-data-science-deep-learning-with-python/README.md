# Machine Learning, Data Science and Deep Learning with Python

### Anaconda : 
It is a Python environment made for scientific computing, data science and machine learning.

Install Anaconda.
Install other libraries through Anaconda :
```
$ conda install pydotplus
$ conda install tensorflow
```

## Python reminder :
Tuples are like list but they are immutable. They are handy for functionnal programming.

### Pandas
Pandas is a Python library that makes handling tabular data easier.

Pandas DataFrame creation :
```python
	df = pd.read_csv("PastHires.csv")
```

* head() : is a handy way to visualize what is loaded. It is possible to pass it an integer to see some specific number of rows at the beginning of the DataFrame.<br/>
* tail() : to show the last rows of the DataFrame.<br/>
* shape : the "shape" of your DataFrame. This is just its dimensions. How many rows for how many columns.<br/>
* len(df) : gives the number of rows in a DataFrame.<br/>
* columns : to show the array of the column names.

A serie is 1d array, for example : 
```python
	df['age']
```

A value is a single cell extracted from the serie :
```python 
	df['age'][5]
```

To extract more the one column from a DataFrame :
```python 
	df[['age'], 'incoume']]
```

To sort the DataFrame according to a specific column values :
```python
	df.sort_values(['years_experience'])
```

To break down the number of unique values in a given column into a Series :
```python
	df['Level of Education'].value_counts()
		BS     7
		PhD    4
		MS     2
```

Pandas even makes it easy to plot a Series or DataFrame, just call plot():
```python
	degree_counts.plot(kind='bar')
```

### NumPy
"NumPy arrays" are multi-dimensional array objects. They are very common in data science. It is easy to create a Pandas DataFrame from a NumPy array, and Pandas DataFrames can be cast as NumPy arrays.

### Scikit_Learn
It is a machine learning library. It generally takes NumPy arrays as its input.

**To summarize :**<br/> 
So, a typical thing to do is to load, clean, and manipulate your input data using Pandas. Then convert your Pandas DataFrame into a NumPy array as it is being passed into some Scikit_Learn function. That conversion can often happen automatically.

### Matplot
https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.plot.html

Tell the Jupyter that we want to view all the results as part of the notebook within the browser :
```python
	%matplotlib inline 
```

Display a plot and save to a file :
```python
	plt.plot(x, norm.pdf(x))
	plt.savefig('MyPlot.png', format='png')
```

We can play with the axes :
```python 
	axes = plt.axes()
	axes.set_xlim([-5, 5])
	axes.set_xticks([-5, -4, -3, -2, -1, 0, 1, 2, 3, 4, 5])
```

Or add a grid :
```python
	axes.grid()
```

Or change Line Types and Colors :
```python
	plt.plot(x, norm.pdf(x), 'b-')
	plt.plot(x, norm.pdf(x, 1.0, 0.5), 'r:')
```

Labeling Axes and Adding a Legend :
```python
	plt.xlabel('Greebles')
	plt.ylabel('Probability')
	plt.legend(['Sneetches', 'Gacks'], loc=4)
```

Pie charts :
```python
	plt.rcdefaults()

	values = [12, 55, 4, 32, 14]
	colors = ['r', 'g', 'b', 'c', 'm']
	explode = [0, 0, 0.2, 0, 0]
	labels = ['India', 'United States', 'Russia', 'China', 'Europe']
	plt.pie(values, colors= colors, labels=labels, explode = explode)	
```

Bar charts :
```python
	plt.bar(range(0,5), values, color= colors)
```

Scatter Plot Diagrams, Histograms, Box & Whisker Plots, etc.

### Seaborn

It is basically a more complete version of matplotlib. It is a visualization library that sits on top of matplotlib and make it prettier to look. It also has different kinds of charts and graphs.

## Statistics and Probability Refresher, and Python Practice.

### Types of data :

* Numerical : Represents some quantitative measurement. There are two types of numerical data :
    1. Discrete data : Integer based, often counts of some event.
    2. Continuous data : Has an infinite number of possible values.

* Categorical :
Quantitative data that has no inherent mathematical meaning. For example : Gender, Y/N, Race, Product category, etc.

* Ordinal :
A mixture of numerical and categorical data. It is categorical data that has mathematical meaning. For example : movie ratings (rating 1 is worst than 5, it is possible to calculate average rating, etc.).

### Mean 
Is the other name for average : Sum / number of samples

### Median 
Sort the values and take the value at the midpoint. If there is an even number of samples, take the average of the two in the middle.
Median is less susceptible to outliers than the mean. Before talking about the mean, it is interessting to have a look on vvalues distribution.

### Mode
The most common value in a data set. It is not relevant to continuous numerical data.

### Variation
Variance measures how "spread-out" the data is.
Variance is simply the average of the squared differences from the mean. Squared values removes the negative values and gices more weight to the outliers.

### Population Vs. Sample
When working on sample of data instead of an entire data set (the entire population) : To calculate sample variance, for N samples divide the squared variances by N-1 instead of N.<br/>

![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_001.png)

### Standard Deviation 
It is the square root of the variance.<br/>
It is used as a way to identify outliers. Data points that lie more than one standard deviation from the mean can be considered unusual.<br/>
We can say how extreme a data point is by talking about "how many sigmas" away from the mean it is.

For a mean of 4.4 and a standard deviation of 2.24 we can consider that values outside of the interval [mean - standard deviation, mean + standard deviation] are outliers.

### Probability Density Function (PDF)
It gives the probabilty of a data point falling within some given range of given value.

![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_002.0.png)

When talking about discrete data it is about Probability Mass Function. It works better with charts.
![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_002.1.png)

### Data Distributions

* **Uniform Distribution :** It is a flat, constant probability of a value occurring within a given range.
![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_003.png)

- **Normal / Gaussian Distribution :** The major number of data is around the mean.<br/>
![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_004.png)
![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_005.png)

- Exponential PDF / Power Law :** Values fall off in an exponential manner.<br/>
![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_006.png)

- **Binomial Probability Mass Function :** It deals with discrete.<br/>
![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_007.png)

- **Poisson Probability Mass Function :** If we have some information about the average number of things that happen in some period of time. Can be used to use predictions.<br/>
![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_008.png)

### Percentiles
In a data set, what is the point at which X% values are less than that value ?

### Moments
It is wa way to measure the shape of data distribution. A quantitative measures of the shape of a probability density function.
1. The first moment is the mean.
2. The second moment is the variance. 
3. The third moment is the skew. It is basically a measure of how lopsided is the distribution.
A distribution with a longer tail on the left will be skewed left, and have a negative skew.
4. The fourth moment is kurtosis. How thick is the tail, and how sharp is the peak, compared to a normal distribution.
For example : Higher peaks have higher kurtosis.

### Covariance and Correlation
It measures how two variables vary in tandem from their means.
![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_009.png)

**How to measure Covariance ?**<br/>
Think of the data sets for the two variables as high-dimensional vectors.<br/>
Convert these vectors of variances from the mean.<br/>
Take the dot product (cosine of the angle between them) of the two vectors.<br/>
Divide by the sample size.

**How to interpret Covariance ?**<br/>
A small covariance, close to 0, means there is not much correlation between the two variables.<br/>
And a large covariance, far from 0 (could be negative for inverse relationships) means there is a correlation. But how large is "large" ? 

That is where correlation comes in !<br/>
Divide the covariance by the standard deviations of both variables, and that normalizes things.<br/>
So a correlation of -1 means a perfect inverse correlation.<br/>
A correlation of 0 means there is NO correlation.<br/>
A correlation of 1 means a perfect correlation.

BUT correlation does not imply causation !<br/>
Only a controlled, randomized experiment can give insights on causation.<br/>
Use correlation to decide what experiments to conduct.

### Conditional Probabilty
Having two events that depend on each other, what is the probability that both will occur ?<br/>
Notation P(A,B) is the probability of A and B both occuring.<br/>
P(B|A) is the probability of B given that A has occured.<br/>
So : P(B|A) = P(A,B) / P(A)

Example :<br/>
![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_010.png)

If A and B are independent, then P(A|B) would be the same as P(A).

### Bayes Theorem
P(A|B) = P(A)P(B|A) / P(B)<br/>
The probability of A given B, is the probability of A times the probability of B given A over the probability of B.<br/>
So : The probability of something that depends on B depends very much on the base probability of B and A.

Example :<br/>
![Machine Learning](img/screenshot_2021-05-15_machine_learning_in_python_(data_science_and_deep_learning)_012.png)

## Predictive models

### Linear Regression

Fit a line to a data set of observations and use this line to predict unobserved values.<br/>
No idea why it is called regression.

It usually uses "least squares" : It minimizes the squared-root between each point and the line.

Slope intercept equation of the line : 	
```
    y = (slop * x) + intercept
```

The slope is the correlation between the two variables times the standard deviation in Y, all divided by the standard deviation in X.<br/>
The intercept is the mean of Y minus the clope times the mean of X.

How does it work ?<br/>
Least squares minimizes the sum of squared errors.<br/>
The line represents the maximum likelihood of the observed data when thinking in terms of probabilities : Maximum likelihood estimation.

Gradient Descent is an alternate method to least squares.<br/>
It is more expensive in terms of calculation. It iterates to find the line that best follows the contours defined by the data.<br/>
Can make sense when dealing with 3D data.

To measure how well the line fits to data : Measure error with r-squared (coefficient of determination).<br/>
```
	r-squared = 1 - (sum of squared errors / sum of squared variation from mean)
```

Result interpretation : O is bad, 1 is good (all of the variance is captured).

### Polynomial Regression

Not all relationships are linear, sometimes data might not really be appropriate for a straight line. That is where polynomial regression comes in.<br/>
* **Linear formula :** y = (m * x) + b is a 'first order' or 'first degree' polynomial, as the power of x is 1 !<br/>
* **Second order polynomial :** y = ax² + bx + c<br/>
* **Third order :** y = ax3 + bx² + cx + d

Highr orders produce more complex curves : More degrees is not always better.<br/>
So : BEWARE OVERFITTING !!!<br/>
Visualize the data first to see how complex of a curve there might really be.<br/>
Visualize the fit : is the curve going out of its way to accomodate outliers ?

### Multiple Regression

It is a regression that takes more than one variable into account.<br/>
What if more than one variable influences the one we are interested in ?<br/>
Example : Predicting a price for a car based on its many attributes (body style, brand, mileage, etc.).

So it is necessary to have a coefficient for each factor. The coefficients imply how important is each factor (if data is all normalized).<br/>
For example : price = alpha + (beta1 * mileage) + (beta2 * age) + (beta3 * doors)<br/>
It needs to assume that the different factors are not themselves dependent on each other.<br/>
Get rid of factors that do not matter !
