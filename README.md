## About


This open source tool is written as part of RACE for 2030, Curtailment and Network Voltage Analysis Study (CANVAS) project. The development of the open source tool is supported by funding from Digital Grid Futures Institute (DGFI), University of New South Wales (UNSW). 
This tool measures the amount of curtailed energy from a residential or commercial distributed energy resource such as distributed PV and/or battery energy storage system (BESS) via one of the following inverter power quality response modes (PQRM):
1. Tripping (inverter cease to operate during high voltage conditions)
2. V-VAr Response (high levels of VAr absorbtion and injection limits inverter maximum real power)
3. V-Watt Response (inverter linearly reduces its real power output as a function of voltage conditions)

The tool uses the time-series measurements of:
1. Voltage
2. Real power (D-PV/BESS inverter)
3. Reactive power (D-PV/BESS inverter)
4. Global horizontal irradiance (GHI)
5. Site information (dc and ac capacity of the inverter)

Through analysing the data mentioned above, this tool aims to answer the questions below:
1.	Does the D-PV inverter trip? If so, how often? 
2.	How much energy is energy is lost due to tripping curtailment in kWh/day?
3.	Does the D-PV inverter show V-VAr response?
4.	How much energy is lost due to V-VAr curtailment in kWh/day?
5.	Does the D-PV inverter show V-Watt response?
6.	How much energy is lost due to V-Watt curtailment in kWh/day?

This tool can benefit researchers and future projects which would like to understand and quantify DER curtailment due to different PQRMs.

## Getting Started

This project runs completely in python with common libraries using Jupyter Notebook (or user's preferred integrated development environment -IDE).
All the required raw data samples are available in [this link](https://unsw-my.sharepoint.com/:f:/g/personal/z5404477_ad_unsw_edu_au/EvguTkYy48RGiXaQE5aP1l4B2OriyWIwqvi29mUL_ReKDw?e=4ceZec). 
The dataset information are available in [this link](https://github.com/mssamhan31/Solar-Curtailment/blob/main/documentations/solar%20curtailment%20dataset%20information.docx). 
The main script is available in [this link](https://github.com/mssamhan31/Solar-Curtailment/blob/main/src/SolA%20Curtailment%20Daily%20Analysis.ipynb).

## Demonstration of the tool use
Currently, the tool can only be demonstrated for D-PV systems as the BESS dataset is confidential as per the non-disclosure agreement (NDA) between project partners. The authors are working to obtain BESS data samples that can be used for the demonstration of the tool.

### Input
There are two inputs for this tool: 
a) Time-series D-PV data from a certain site and a date, and 
b) GHI data of a certain date. 
The detailed explanations of the datasets are provided in the 'solar curtailment dataset information.docx' under the documentations folder.

Sample images for the format of D-PV and GHI data can be seen via the images below:
![Input Data](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/input_data.PNG?raw=true)  

GHI Data for a certain date: ===================@SAMHAN: This GHI image you show is very detailed. I think we only usem mean GHI with the relevant time-stamp data. Could you please confirm this and change the example image accordingly?
![Input GHI](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/input_ghi.PNG?raw=true)  

Via using the input data, the tool produces 4 main outputs:

### Output 1. Summary Table
![Output 1](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/output_summary.PNG?raw=true)  
This summary table shows whether the date is a clear sky day or not, how much is the measured total energy generated in that day, how much is the expected energy generated without curtailment, estimation method used to calculate the expected energy generation, and most importantly:
1.	Tripping response and the associated tripping curtailment
2.	V-VAr response and the associated V-VAr curtailment 
3.	V-Watt response and the associated V-Watt curtailment  
To illustrate the summary table, we visualize the data into 3 plots below:

### Output 2. GHI Plot
![Output 2](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/output_ghi.png?raw=true)  
This GHI plot shows the irradiance of a certain day.

### Output 3. Scatter plot of real power, reactive power, and power factor vs voltage
![Output 3](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/output_scatter.png?raw=true)  
The real power and reactive power is normalized by VA rating of the inverter, so the maximum value is 1. For a site with V-VAr response, inverter is expected to absorb/inject VAR according to it's respective V-VAr curve.
For a site with V-Watt response, we expect a scatter of real power to reduce linearly in high voltages (as seen in the image with voltages equal or greater than 251V).

### Output 4. Line plot of real power, reactive power, expected real power, power limit, and voltage vs time
![Output 4](https://github.com/mssamhan31/Solar-Curtailment/blob/main/image/output_lineplot.png?raw=true) ===================@SAMHAN, can you please update the image with V-Watt rather than VW (to be more clear...)

## High Level Explanation of How The Algorithm Works
### Clear Sky Day Determination
We judge whether a date is a clear sky day or not based on two criterias:
1.	The average change of GHI between two consecutive time is lower than a certain threshold which suggests there is no or minimum sudden change in GHI. Sudden change in the GHI means there is a cloud cover.
2.	The maximum GHI value is higher than a certain threshold, which must be true if there is no cloud.

### Daily Energy Generated Calculation
To calculate the energy generation, we use the D-PV time series data and use these steps: 

================================== @ Samhan, this is not correct! We should be calculating Wh energy for every 5 mins and then summing up all 5 minutely values within the day. Converting 5 minutely data to hourly resolution will significantly decrease the value of our research and curtailment algorithm as it is much coarser than 5 minutely values. I don't think this is true for any of the curtailment algorithms. I have corrected as below:

1.	Calculate every 5 minutely energy value in Wh.
2.	Sum all the 5 minutely energy values within the day (288 data points).
3.	Divide it by 1000 to convert the unit into kWh/day.

### Expected Energy Generated Calculation
We essentially use similar process with the energy generated calculation, but we use power_expected values instead of real power value. The power_expected values are obtained using estimation method below:

### Estimation Method
To obtain the power_expected value in the D-PV time series data, we use these logics:
If it is a clear sky day: Use polyfit estimation (see below):
If it is not a clear sky day and there is tripping curtailment: Use linear estimation (see below):
If it is not a clear sky day and there is no tripping curtailment: Do not estimate, because we cannot be sure whether the reduction in power is due to v-var/v-watt curtailment or cloud cover. 

### Linear Estimation
This method is only used in a non clear sky day with tripping curtailment. Major steps:
1.	Filter the D-PV time series data into times between sunrise and sunset
2.	Detect the zero power values, which indicates a tripping event
3.	Detect the ramping down power values before zero values and ramping up power values after zero values, and consider them as tripping
4.	Detect starting point and end point for each tripping point
5.	Use points obtained from step 4 to make a linear equation of each tripping point, and use the linear equation to get the estimated power value without curtailment in every tripping timestamps
6.	For times other than the tripping event, leave the power expected to be the same with the actual power. 

### Polyfit Estimation
This method is only used in a clear-sky day condition. Necessary steps include:
1.	Filter the D-PV time series data into times between sunrise and sunset
2.	Filter out curtailed power values because we want the estimation fits the actual power without curtailment (D-PV is expected to generate a parabolic curve in clear sky-day conditions. This validated through observing the system throughout the year and confirm that it is not exposed to regular shading conditions)
3.	Filter to include only decreasing gradient data. Since we expect a perfect parabolic curve, the gradient should always decrease =================@Samhan, this is not very clear, please be more clear with what do you mean...
4.	Convert the timestamp from datetime object into numerical values for fitting
5.	Fit the power values & numerical timestamp values using a polyfit with degree = 2 (quadratic function)
6.	Obtain the values of the expected power by the polyfit for all timestamps, including outside of the times used for the fitting

### Tripping Detection
The method is described already in the linear estimation section. We basically say there is a tripping curtailment if there are zero values (sudden drop to zero from a non-zero value) between the sunrise and sunset time. 

### Tripping Curtailed Energy Calculation
For a tripping case in a non clear sky day, using the linear estimation, we calculate the energy generation expected. The amount of curtailment is equal to the energy generation expected minus energy generated. For a tripping case in a clear sky day, we just replace the linear estimation by the polyfit estimation.

### V-VAr Response Detection
If the V-VAr response of an inverter is not enabled, we expect the reactive power to be always zero and the power factor is always 1. For sites that shows VAr response, we compare their V-VAr scatter plots against the benchmark AS-NZS 4777.2 2015 and AS-NZS 4777.2 2020 V-VAr curves to decide on their V-VAR curve. We use 100 VAr as a threshold to take into account various glitches and inaccuracies in the monitoring device and circuit (i.e. VAr>100 for an inverter to be considered to absrob/inject any VArs). Please note that, due to some monitoring set-up errors, the raw VAr data needed to be divided by 60 in order to find the actual 5 minutely values

======================@ SAMHAN this point above is not correct. If an inverter shows more than 100 VAr it shows that it absorbs or injects some VAr but this doesn't guarantee it shows V-VAr response (VAr doesn't have to be dependent on Voltage just because it is greater than 100 VAr) I corrected this part accordingly...

### V-VAr Curtailment Calculation
Unlike tripping, where tripping site must have energy curtailment, V-VAr enabled site can have zero curtailment. This is because the real power of the inverter may not be limited in the presence of VAr and it depends on the magnitude of absorbed/injected VArs. For example for an inverter with 5 kVA limit, absorbtion of 3 kVAr leaves 4 kW real power capacity and energy is only curtailed when inverter can generate more than 4 kW which is calculated based on the GHI (i.e. expected energy generation method above). To calculate the energy curtailed due to VVAr:
1.	For clear sky day, we use polyfit to estimate the power generated without curtailment. 
2.	For a non clear sky day, we multiply ghi data, dc_cap, and eff_system to estimate the PV-system power production.   
3.  We filter out instances where inverter isn't absorbing or injecting VArs (there won't be V-VAr curtailment when there is no VAr).
4.  For the remaining instances we compare the real vs. expected generation
5.  We calculate the difference between the power production and the expected power production and if there is any discrepancy, we double check with VAr values to confirm V-VAr curtailment. It is worth to note, however, that the amount of energy curtailed in a non clear sky day are most likely overestimated. This is because no one can sure whether the curtailment is due to V-VAr response or due to cloud.

### V-Watt Response Detection
In a V-Watt enabled site, the real power limit value will decrease linearly with increasing voltage. The voltage threshold value where the real power starts decreasing can vary from 235-255 V according to AS/NZS 4777 2020. It will stop decreasing exactly at 265 V, where the real power limit is 20% the ac capacity of the inverter (after this voltage, inverter must trip and cease to operate). That is why we need to check the scatter plot of power with voltage, whether it matches one of the possible V-Watt curve, as the voltage threshold can vary from 235-255 V. The preliminary steps for V-Watt response detection are:
1.	If it is not a clear sky day, it is inconclusive
2.	Else we check the polyfit quality. If the polyfit quality is not good enough, it is inconclusive as well
3.	Else we check whether the dataset contain points where the voltage is more 235. If not, it is inconclusive because there are no points to be checked for V-Watt response
If it passes these preliminary steps, meaning it is a clear sky day with good polyfit quality and available overvoltage points, we then check the V-Watt response. For each of the possible V-Watt curve (from 235-255 V threshold voltage), we check the actual data with these steps:
1.	Map each voltage over the threshold voltage into the real power limit according to the standards. This is done simply by forming a linear equation where we know two points. The first point is when the real power limit is 100% and when the voltage is the threshold voltage. The second point is when the real power limit is 20% and when the voltage is 265 V.
2.	In the actual dataset, we filter the data only into where the expected power without curtailment obtained from polyfit estimate is higher than the real power limit obtained from the previous step. The filtered data is called suspect data. 
=================@ Samhan, this 2. point above is not very clear here! Can you try to explain this again?
3.	Then, we count the percentage of datapoints from step 2 which lie in the buffer range of the V-Watt curve from step 1. The buffer value we use is 150 watt.   
================@ Samhan, this point is not clear either. We need to use better terms to explain these...
4. We loop through all possible voltage thresholds, we decide which threshold voltage gives the highest percentage.
================@Samhan (what do you mean by highest percentage, be more clear and specific)

We decide a certain site is a V-Watt enabled site only if the highest percentage count is higher than a percentage threshold, 84% and the number of actual point lying in the buffer is more than a count threshold, 30%. 
If the above criteria is not satisfied, and the maximum voltage of the suspect data is less than 255, we say that it is inconclusive due to insufficient data points. This is because the possibility of it being a V-Watt enabled site, but the voltage threshold value is higher than the maximum available voltage datapoints. Other than that, we say that there is no V-Watt site. 


================@ Samhan Please re-write this V-Watt response detection section as it is very difficult to understand. Try to use simpler sentences and terms. We can discuss further if you have any difficulty.

### V-Watt Curtailment Calculation
For a V-Watt enabled site, the curtailed energy is equal to the expected energy generated subtracted by the actual energy generated. The expected energy generated and the actual energy generated are calculate by the time-series power data and time-series expected power data using the Expected Energy Generation Method by polyfit estimation mentioned below. 

### Sample File Creation
The raw time series D-PV data is from Solar Analytics which consists of a monthly data with 500 sites mixed into a file. In this tool, we analyze a specific site for a certain date. So, for testing purpose, we create sample simply by filtering the data for a certain day and certain site. 
For convenience, we also process the time series D-PV data by converting the time from UTC time into local time (Adelaide GMT + 9:30). 
Similary, the GHI data we have is a monthly data. So, we filter it into certain dates for the sample analysis period. We also process the ghi data by adding a timestamp column by combining some columns like year, month ,day, hour, and minute information. 

## Tool Limitation & Notes

1. The tool is currently limited to measuring one curtailment mode at a time. Tripping curtailment can't co-exist with other modes however, V-VAr and V-Watt can operate simultaneously. In the studied sites, majority didn't have V-VAr mode enabled so this limitation didn't impact our results. However, as more inverters start complying with both modes, analysis of simultaneous operations of these modes are necessary and this is a primary future research objective.

2. The data-set doesn't include the actual VA capacity of the inverter, and therefore, we used the AC capacity as VA capacity. For some sites, this was an underestimation, as their VA capacity calculated from the real and reactive power exceeded the inverter's AC capacity. This may have resulted in over-estimation of V-VAr curtailment for some inverters as they had higher VA capacity in reality than their assumed VA capacity (AC capacity) in our study.   


## Some Related Articles and Papers
1. https://www.pv-magazine-australia.com/2021/05/24/unsw-digs-the-data-how-much-solar-energy-is-lost-through-automated-inverter-settings/
2. https://www.abc.net.au/news/science/2022-02-16/solar-how-is-it-affected-by-renewable-energy-curtailment/100830738
3. https://greenreview.com.au/energy/rooftop-solar-pv-curtailment-raises-fairness-concerns/
4. https://theconversation.com/solar-curtailment-is-emerging-as-a-new-challenge-to-overcome-as-australia-dashes-for-rooftop-solar-172152
5. https://www.racefor2030.com.au/wp-content/uploads/2021/11/CANVAS-Succinct-Final-Report_11.11.21.pdf

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for instructions to contribute to this open-source tool.

## Project Partners
The project partners of CANVAS are: AGL, SA Power Networks (SAPN), Solar Analytics, and University of New South Wales (UNSW).

## Authors

* **Naomi M Stringer** - *Tripping Algorithm*
* **Baran Yildiz** - *VVAr Algorithm* & Lead Chief Investigator
* **Tim Klymenko** - *VWatt Algorithm*
* **M. Syahman Samhan** - *Merging of algorithms, Open Source Development & Implementation, Debugging*

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details. 

## Contact
Email baran.yildiz@unsw.edu.au
