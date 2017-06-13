# Visualize currency-unhedged performance of a domestic vs foreign asset

This article demonstrates how a user can use Eikon Scripting Python API to quickly prototype and plot custom analytics which are not readily available within Eikon. The example shown here is one of the many use case, that a Quants/Analysts can employ to analyze financial data. 

### Eikon Scripting API
Thomson Reuters provides a python library for accessing market data. This Eikon library is available from python packages and can be installed using pip package manager. The library itself depends on **Eikon Scripting Proxy**, which is an application running in the local network which connects to Eikon back end system in cloud. The Proxy runs with a user's Eikon credentials, and provides a web interface to data, like: 

* End of day and intraday prices
* News
* Symbology conversion
* Reference data
* Streaming market price (coming soon), etc..

These services can be consumed either in a pure web manner (sending HTTP post request to the proxy) or using the provided python library. The python API calls invoke the underlying web methods and marshal JSON data into python usable data types (Pandas DataFrame).

### Use cases
The scripting API can be used to retrieve any of the rich, value added data point which are available within Eikon. The real power of API is when inhouse-data is combined with pricing and analytics sourced from Eikon. Some sample use cases are:

* Price a portfolio and determine net risk exposure
* Regression analysis of a trading strategy
* Build custom dashboards by combining powerful D3 visuals
* Combine government stats dataset with Thomson Reuters sourced data points
	- or simply -
* Visualize the currency-unhedged performance of a domestic vs foreign asset
	
### Application Sample
The goal of this article sample is to demonstrate how easy it is to perform a simple analysis and plot it on a chart. This what-if analysis compares the return of the a foreign asset against a domestic asset, if same dollar amount was invested in either one; taking into account the currency fluctuation rate during the comparison period.
The sample accomplishes this by getting the trade currency of both the assets, and then requesting the calculated cross rate for this currency pair. Performance data for both the assets is requested, which is first scaled using the currency cross rate data and then normalized. The normalized data can then be plotted on a line chart.

### Dependencies
To try the code provided in this article, a setup and familiarity with following technologies is required:  
Python language  
JSON  
Pandas Data Analysis library  
Jupyter (iPython) Notebooks  
Cufflinks and Plotly Charting library  

		
### Implementation
Following steps are implemented in this sample:

1. Define a domestic and foreign instruments and a time range
2. Get the market cap currency for both instruments

	```python
	data,err = ek.get_data([instr1, instr2], "TR.CompanyMarketCap.Currency")
	```

3. Create a currency cross-rate RIC and get historical data for the given time range

	```python
	crossRIC = data['Currency'][1] + data['Currency'][0] + '=X'
	curr = ek.get_timeseries([crossRIC], fields='CLOSE', start_date=start_date, end_date=end_date)
	```

4. Get historical data (CLOSE Price) for both instruments
5. Scale foreign asset using the currency data

	```python
	perf = pd.concat([perf1, perf2, perf2 * curr], axis=1)
	label3 = 'Adjusted ' + instr2;
	perf.columns = [instr1, instr2, label3]
	# drop invalid entries from dataframe
	perf = perf.dropna()
	```

6. Normalize the data for plotting

	```python
	perf_norm = perf * 100 / (perf[instr1][0], perf[instr2][0], perf[label3][0])
	```

7. Plot a double line chart

	```python
	perf_norm.iplot()
	```

See attached iPython notebook for complete working sample	
	
### Related samples
Thomson Reuters [github samples repository](https://github.com/TR-API-Samples) has many more samples and snippets of code showing other uses of Eikon scripting API. Just search for python or Eikon within repositories.

### Reference
[Eikon Scripting API](https://developers.thomsonreuters.com/eikon-apis/eikon-web-and-scripting-apis-limited-access)   
[Pandas library](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.html)   
[Plotly Charting](https://plot.ly/)   
