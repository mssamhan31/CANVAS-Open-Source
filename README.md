## About

This Solar Curtailment project is a tool to detect curtailment and measure the amount of energy curtailed from a residential or commercial PV System site in three modes:
1. Tripping (the inverter stops operating in the high voltage condition)
2. V-VAr Response (VAr absorbtion and injection of inverter limits the maximum real power in the high voltage condition)
3. V-Watt Response (The inverter limits the maximum real power production in the high voltage condition)

Given the historical time-series data of ghi, voltage, real power, reactive power, and site information like maximum ac capacity of the inverter, this tool aims to give outputs:
1.	Does the solar inverter trip?
2.	How much is the curtailment due to tripping curtailment in kWh/day?
3.	Does the solar inverter show V-VAr response?
4.	How much is the curtailment due to V-VAr response in kWh/day?
5.	Does the solar inverter show V-Watt response?
6.	How much is the curtailment due to V-Watt curtailment in kWh/day?

This tool will benefit anyone who wants to study PV-Curtailment, and the improved understanding of curtailment could lead to higher levels of PV System integration. 

## Getting Started
This project runs completely in python with common libraries using Jupyter Notebook. Hence, it is recommended to run the program in Jupyter Notebook as well.The package contains only one script, which is solarcurtailment. For quick start,
1. Download all files (exluding those in monthly D-PV data folder) from [this link](https://unsw-my.sharepoint.com/:f:/g/personal/z5404477_ad_unsw_edu_au/EvguTkYy48RGiXaQE5aP1l4B2OriyWIwqvi29mUL_ReKDw?e=4ceZec) .
2. Install the package in the terminal by using this command "pip install -i https://test.pypi.org/simple/ trialsamhan1" . This should make solarcurtailment module available to be imported. 
3. Try running the [test script](https://github.com/mssamhan31/Solar-Curtailment/blob/Package-Implementation/test/test_solarcurtailment.ipynb), which basically input the module solarcurtailment then deploy it for some samples. Make sure to adjust the file_path variable. 

The dataset information are available in [this link](https://github.com/mssamhan31/Solar-Curtailment/blob/main/documentations/solar%20curtailment%20dataset%20information.docx). 

## Tool Use Demonstration
### Input
There are two inputs for this tool: a time-series D-PV data of a certain date of a certain site, and GHI data of a certain date. The detail explanation of the dataset is given in the solar curtailment dataset information in documentations folder.

D-PV Time Series Data for a certain date and certain site:
![Input Data](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/input_data.PNG?raw=true)  

GHI Data for a certain date:
![Input GHI](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/input_ghi.PNG?raw=true)  

From the input dataset, the tool processes and obtain 4 outputs:

### Output 1. Summary Table
![Output 1](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/output_summary.PNG?raw=true)  
This summary table shows whether the date is a clear sky day or not, how much is the total energy generated in that day, how much is the expected energy generated without curtailment, estimation method used to calculate the expected energy generation, and most importantly:
1.	Tripping response and the amount of curtailment it makes
2.	V-VAr response and the amount of curtailment it makes
3.	V-Watt response and the amount of curtailment it makes.  
To illustrate the summary table, we visualize the data into 3 plots below:

### Output 2. GHI Plot
![Output 2](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/output_ghi.png?raw=true)  
This GHI plot shows the irradiance value of the certain day.

### Output 3. Scatter plot of real power, reactive power, and power factor vs voltage
![Output 3](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/output_scatter.png?raw=true)  
The real power and reactive power is normalized by VA rating of the inverter, so the maximum value of all quantities here is 1. For a site with VVAr response, we expect a scatter of reactive power to be not zero at high voltages. For a site with VWatt response, we expect a scatter of real power to reduce linearly in high voltages.

### Output 4. Line plot of real power, reactive power, expected real power, power limit, and voltage vs time
![Output 4](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/output_lineplot.png?raw=true)  

## High Level Explanation of How The Algorithm Works
### Clear Sky Day Determination
We judge whether a date is a clear sky day or not based on two criterias:
1.	The average change of GHI between two consecutive time is lower than a certain threshold (which suggests there is no (or minimum) sudden change in GHI. Sudden change in the GHI means there is a cloud.
2.	The maximum GHI value is higher than a certain threshold, which must be true if there is no cloud.

### Energy Generated Calculation
To calculate the energy generation, we use the D-PV time series data and use these steps:
1.	Resample the power data into hourly basis using the mean, so the power value (watt) is basically the same with energy value (Wh)
2.	Take the sum of all the power data in that day
3.	Divide it by 1000 to convert the unit into kWh.

### Expected Energy Generated Calculation
We essentially use similar process with the energy generated calculation, but we used power_expected values instead of power value. The power_expected values are obtained using estimation method below:

### Estimation Method
To obtain the power_expected value in the D-PV time series data, we use these logics:
If it is a clear sky day: Use polyfit estimation
If it is not a clear sky day and there is tripping curtailment: Use linear estimation
If it is not a clear sky day and there is no tripping curtailment: Do not estimate, because we cannot be sure whether the curtailment is due to vvar, vwatt, or merely cloud. 

### Linear Estimation
This method is only used in a non clear sky day with tripping curtailment. Major steps:
1.	Filter the D-PV time series data into times between sunrise and sunset
2.	Detect the zero power values, which must be tripping event
3.	Detect the ramping down power values before zero values and ramping up power values after zero values, and consider them as tripping
4.	Detect starting point and end point for each tripping point
5.	Use points obtained from step 4 to make a linear equation of each tripping point, and use the linear equation to get the estimated power value without curtailment in every tripping timestamps
6.	For times other than the tripping event, leave the power expected to be the same with the actual power. 

### Polyfit Estimation
Necessary steps include:
1.	Filter the D-PV time series data into times between sunrise and sunset
2.	Filter out curtailed power values because we want the estimation fits the actual power without curtailment
3.	Filter to include only decreasing gradient data. Since we expect a perfect parabolic curve, the gradient should always decrease
4.	Convert the timestamp from datetime object into numerical values for fitting
5.	Fit the power values & numerical timestamp values using a polyfit with degree = 2 (quadratic function)
6.	Obtain the values of the expected power by the polyfit for all timestamps, including outside of the times used for the fitting

### Tripping Detection
The method is described already in the linear estimation section. We basically say there is a tripping curtailment if there are zero values between the sunrise and sunset time. 

### Tripping Curtailment Calculation
For a tripping case in a non clear sky day, using the linear estimation, we calculate the energy generation expected. The amount of curtailment is equal to the energy generation expected minus energy generated. For a tripping case in a clear sky day, we just replace the linear estimation by the polyfit estimation.

### V-VAr Response Detection
If the V-VAr response of an inverter is not enabled, we expect the reactive power is always zero and the power factor is always 1. So, we judge a site is a V-VAr site if the reactive power of the day has at least a value more than a certain threshold. In our tool, we use 100 as the threshold, meaning if there is reactive power value more than 100 VAr we judge it as V-VAr response enabled. 

### V-VAr Curtailment Calculation
Unlike tripping, where tripping site must have energy curtailment, V-VAr enabled site can have zero curtailment. This is because the power limit due to V-VAr response is still higher than the power generated by the PV system in a given GHI value. To calculate the energy curtailed due to VVAr:
1.	For clear sky day, we use polyfit to estimate the power generated without curtailment. 
2.	For a non clear sky day, we multiply ghi data, dc_cap, and eff_system to estimate the PV-system power production.   

After that, we calculate the difference between the power production and the expected power production. The difference is then filtered to positive value only then multiplied by its duration and converted to kWh to get the energy curtailed. We include only positive value because it does not make sense to have actual power higher than the expected power. It is worth to note, however, that the amount of energy curtailed in a non clear sky day are most likely overestimated. This is because no one can sure whether the curtailment is due to V-VAr response or due to cloud.

### V-Watt Response Detection
In a V-Watt enabled site, the real power limit value will decrease linearly with increasing voltage. The voltage value where the real power limit start decrease, however, can vary from 235-255 V according to AS/NZS 4777 2020. It will stop decreasing exactly at 265 V, where the real power limit is 20% the ac capacity of the inverter. That is why we need to check the scatter plot of power with voltage, whether it matches one of the possible V-Watt curve, with voltage threshold can vary from 235-255 V. The preliminary steps:
1.	If it is not a clear sky day, it is inconclusive
2.	Else we check the polyfit quality. If the polyfit quality is not good enough, it is inconclusive as well
3.	Else we check whether the dataset contain points where the voltage is more 235. If not, it is inconclusive because no points to be checked  
If it passes the preliminary steps, meaning it is a clear sky day with good polyfit quality and available overvoltage points, we then check the VWatt response. For each of the possible VWatt curve (from 235-255 V threshold voltage), we check the actual data with these steps:
1.	Map each voltage over the threshold voltage into the real power limit according to the standards. This is done simply by forming a linear equation where we know two points. The first point is the real power limit is 100% when the voltage is the threshold voltage. The second point is the real power limit is 20% when the voltage is 265 V.
2.	In the actual dataset, we filter the data only into where the expected power without curtailment obtained from polyfit estimate is higher than the real power limit obtained from the previous step. The filtered data is called suspect data.
3.	Then, we count the percentage of datapoints from step 2 which lie in the buffer range of the VWatt curve from step 1. The buffer value we use is 150 watt.   

After finish looping through all possible threshold voltage, we decide which threshold voltage gives the highest percentage. We decide a certain site is a V-Watt enabled site only if the highest percentage count is higher than a percentage threshold, 84% and the number of actual point lying in the buffer is more than a count threshold, 30%. 
If the above criteria is not satisfied, and the maximum voltage of the suspect data is less than 255, we say that it is inconclusive due to insufficient data points. This is because the possibility of it being a V-Watt enabled site, but the voltage threshold value is higher than the maximum available voltage datapoints. Other than that, we say that there is no V-Watt site. 

### V-Watt Curtailment Calculation
For a V-Watt enabled site, the curtailed energy is equal to the expected energy generated subtracted by the actual energy generated. The expected energy generated and the actual energy generated are calculate by the time-series power data and time-series expected power data using the method mentioned below. 

### Sample File Creation
The raw time series D-PV data we have from Solar Analytics is a monthly data with 500 sites mixed into a file. We want, however, to analyze a specific site for a certain date. So, for testing purpose, we create sample simply by filtering the data for a certain day and certain site. For convenience purpose, we also process the time series D-PV data by converting the time from UTC time into local time (Adelaide GMT + 9:30). 
Similary, the GHI data we have is a monthly data. So, we filter it into certain date only. We also process the ghi data by adding a timestamp column by combining some columns like year, month ,day, hour, and minute information. 

## Tool Limitation & Notes
1. For the power scatter plot, the reactive power is sometime seen above zero in the VVAr site. It should be worth to note that these reactive power should have been negative, because the inverter is absorving reactive power from the grid in the daytime. 
2. In the power lineplot in VVAr site, we sometime see the power limit due to vvar response is below the actual power generated. This is attributed to the fact that we do not have the actual va data of the inverter, and use the ac capacity data of the inverter to estimate the va capacity. This is an underestimation, so the power limit is lower than the actual value.

## Some Related Articles and Papers
1. https://greenreview.com.au/energy/rooftop-solar-pv-curtailment-raises-fairness-concerns/
2. https://theconversation.com/solar-curtailment-is-emerging-as-a-new-challenge-to-overcome-as-australia-dashes-for-rooftop-solar-172152
3. https://www.racefor2030.com.au/wp-content/uploads/2021/11/CANVAS-Succinct-Final-Report_11.11.21.pdf

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for instruction to contribute.

## Project Partners
The project partners are AGL, SAPN, Solar Analytics, and UNSW.

## Authors

* **Naomi M Stringer** - *Tripping Algorithm*
* **Baran Yildiz** - *VVAr Algorithm*
* **Tim Klymenko** - *VWatt Algorithm*
* **M. Syahman Samhan** - *Merging, Open Source Implementation, Debugging*

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details. 

## Contact
Email m.samhan@student.unsw.edu.au.
