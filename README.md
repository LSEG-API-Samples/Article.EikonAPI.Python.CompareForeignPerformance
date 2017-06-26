# Visualize currency-unhedged performance of a domestic vs foreign asset

This article demonstrates how a user can use Eikon Scripting Python API to quickly prototype and plot custom analytics which are not readily available within Eikon. The example shown here is one of the many use case, that a Quants/Analysts can employ to analyze financial data. 

### Eikon Scripting API
Thomson Reuters provides a python library for accessing market data. This Eikon library depends on **Eikon Scripting Proxy**, which runs with a user's Eikon credentials, and provides a web interface to data, like End of day/ intraday prices, News, Symbology etc.

For more information on Scripting API, please visit [Thomson Reuters Developers Portal](https://developers.thomsonreuters.com/eikon-apis/eikon-web-and-scripting-apis-limited-access).

### Use cases
The scripting API can be used to retrieve any of the rich, value added data point which are available within Eikon. The real power of API is when inhouse-data is combined with pricing and analytics sourced from Eikon. Some sample use cases are:

* Price a portfolio and determine net risk exposure
* Regression analysis of a trading strategy
* Build custom dashboards by combining powerful D3 visuals
* Combine government stats dataset with Thomson Reuters sourced data points
	- or simply -
* Visualize the currency-unhedged performance of a domestic vs foreign asset
	
### Application Sample
The goal of this article sample is to demonstrate how easy it is to perform a simple analysis and plot it on a chart. This what-if analysis compares the return of a foreign asset against a domestic asset, as if same dollar amount was invested in either one; taking into account the currency fluctuation rate during the comparison period. The term domestic is used loosely here and this approach can be used to compare any two assets, trading in any currencies. The sample accomplishes this by getting the trade currency of both the assets, and then requesting the calculated cross rate for this currency pair. Performance data for both the assets is requested, which is first scaled using the currency cross rate data and then normalized. The normalized data can then be plotted on a line chart. 

The accompanying sample compares the performance of a Canadian bank - TD (TD.TO), trading on Toronto Stock Exchange in Canadian Dollars with British bank - Barclays (BARC.L), trading on London Stock Exchange in Pound Sterling.

### Dependencies
To try the code provided in this article, a setup and familiarity with following technologies is required:  
Python language  
JSON  
Pandas Data Analysis library  
Jupyter (iPython) Notebooks  
Plotly (or Cufflinks) Charting library  
The code provided with this article was tested using python (3.6.1), eikon (0.1.7), numpy (1.12.1), pandas (0.20.1) and plotly (1.12.9)  
		
### Implementation
Following steps are executed to achieve the comparison chart:

1. Define a domestic and foreign instruments and a time range for comparison. Since the performance data is normalized, any one of the two instruments can be considered as base (domestic) and other as foreign, and it will produce the same output.

2. Get the market cap currency for both instruments. The Currency field in Eikon datafield "TR.CompanyMarketCap" is used here.

	```python
	data,err = ek.get_data([instr1, instr2], "TR.CompanyMarketCap.Currency")
	```

3. Create a currency cross-rate RIC and get historical data for the given time range. The cross rate is calculated by Thomson Reuters if no indicated rate exists for the currency pair. This RIC is formed by appending =X to both currencies. For e.g. Calculated cross rate RIC for GBP vs CAD would be "GBPCAD=X".

	```python
	crossRIC = data['Currency'][1] + data['Currency'][0] + '=X'
	curr = ek.get_timeseries([crossRIC], fields='CLOSE', start_date=start_date, end_date=end_date)
	```

4. Get historical data (CLOSE Price) for both instruments for requested time range.

5. Scale the foreign asset using the currency cross rate we retrieved earlier. 

	```python
	# create a single DataFrame with both data series (useful if using cufflinks to plot instead of plotly)
	perf = pd.concat([perf1, perf2 * curr], axis=1)
	perf.columns = [instr1, instr2]
	# drop invalid entries from dataframe
	perf = perf.dropna()
	```

6. Rebase the data to the start time period, so that initial investment in both instruments shows up as 100%.

	```python
	# rebase both timeseries to 100% at start date
	perf_rb = perf * 100 / (perf[instr1][0], perf[instr2][0])
	```

7. Plot a double line chart. The two chart lines will show the % growth/loss of a unit of investment on both instruments.

	```python
	# using offline plotly for charting
	offline.iplot([{
		'x': perf_rb.index,
		'y': perf_rb[col],
		'name': col
	}  for col in perf_rb.columns])	
	```

The code sample above breaks the implementation into multiple steps for clarification and to show as an example, how various data and timeseries can be retrieved from Eikon. A simple implementation of this code would be:
	
	```python
	response, error = ek.get_data(instruments=[instr1, instr2], fields=['TR.ClosePrice.Date', 'TR.ClosePrice.Value'], parameters={'Curn’:'CAD','SDate':start_date,'EDate':end_date,'Frq':'D'})
	df = response.pivot_table(values='Close Price', index=['Date', 'Instrument']).unstack('Instrument').dropna()
	rebased = df.apply(lambda series: series/series[0]*100)

	offline.iplot([{
		'x': rebased.index,
		'y': rebased[column],
		'name': column[1]
	}  for column in rebased.columns])
	```
	
### Output
The resulting chart shows that an equal dollar amount invested in TD in Jan 2015 and withdrawn Jan 2017 would net 20% gain vs 16% loss in Barclays, when factoring in the currency movement.
![Sample Chart](pic.png)
	
See the attached iPython notebook for complete working sample.	
	
### Related samples
Thomson Reuters [github samples repository](https://github.com/TR-API-Samples) has many more samples and snippets of code showing other uses of Eikon scripting API. Just search for python or EikonAPI within repositories.

### Reference
[Eikon Scripting API](https://developers.thomsonreuters.com/eikon-apis/eikon-web-and-scripting-apis-limited-access)   
[Pandas library](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html)   
[Plotly Charting](https://plot.ly/)   
