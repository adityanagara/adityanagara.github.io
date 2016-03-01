---
layout: post
title: Engineering a Nowcasting System using Open Data Sources
comments: true 
redirect_from: "/2016/02/28/Vaisala-Open-Weather-Data-Challenge"
permalink: vaisala-weather-data 
---
# Introduction
Water vapor plays an important role in the atmosphere as it holds water in the vapor form which is
capable of traveling both horizontally and vertically over a short span of time. This movement of water 
vapor thus greatly affect the weather in a vary short time scale (the next few hours). Thus by measuring this quantity
we can make better predictions on the weather in the near future. 

If this is the case why are people not making use of water vapor measurements for better predicting the weather? The answer 
is that the amount of water vapor at any given time and at a given location is very difficult to measure. We thus believe
a weather forecasting system with accurate measurements of water vapor at high temporal and spatial resolutions
would benefit any weather forecasting/nowcasting system. The current methods to measure water vapor are as follows

 1. Radiosondes: A radiosonde is a battery powered telemetry instrument package with sensors for sampling various atmospheric variables as it is 
carried up by a balloon from ground launch to between 20-30 km altitude (http://www.wrh.noaa.gov/rev/tour/UA/introduction.php). 
While radiosondes have the advantage that they can measure the vertical distribution of water vapor, they have the distinct disadvantages that they are 
typically only launched two times a day (at 0000 and 1200 UTC) and from only a handful of locations 
(the entire CONUS is covered by a mere 90 radiosonde launch sites). 
 
 2. Radiometers
 
 3. Satellites
 
 4. GPS-Meteorology

Our solution fo the Vaisala Open Weather data challenge is focused on using open source GPS data from a dense network 
of GPS stations along with in-situ surface meterological observations to obtain a quantity called IPW or Integrated 
Precipitable Water Vapor. 

Using water vapor our idea is to develop a nowcasting system which takes GPS data, surface meterological data and NEXRAD
reflectivity data to generate precipitation nowcasts 1 hour into the future. 

# Data and Software
Our goal is to develop a nowcasting system which predicts precipitation fields 1 hour into the future. The nowcasting system uses
precipitable water measured by a network of GNSS stations and as a proxy for precipitation we use radar reflectivity products from 
a NEXRAD radar. Archived precipitable water vapor and reflectivity are trained using a Random Forest Machine Learning Classifier 
on storm cases from 2014 which occurred in the DFW Texas region. The following section will discuss the data sources and software
used in our nowcasting system. 

## Open Source Data
Since our nowcasting system uses precipitable water vapor along with precipitation fields (radar reflectivity) we use the 
following data sources to get the data for the entire year of 2014. 

1. GNSS(Global Navigation Satellite Systems) data: High precision GNSS signals from dual frequency receivers which operate continuously
are in the form of RINEX (Receiver Independent Exchange format) files. These files contain pseudo range and carrier phase data 
from each satellite in view at 30s intervals. For over a 1000 stations world wide RINEX files can be obtained from FTP servers maintained
by three major organizations NOAA NGS [CORS](ftp://geodesy.noaa.gov/cors/)(Continuously Operated Reference Stations),
[SOPAC](ftp://garner.ucsd.edu/pub/rinex/) (Scripps Orbit and Permanent Array Center), [IGS](ftp://igscb.jpl.nasa.gov/pub/station/) 
(International GNSS Services).<br>  
**Real time data:** The above mentioned servers also provide realtime data of the RINEX files for select stations. 

2. Meteorological data: Since most of the GNSS stations in our study do not have co-located weather stations we interpolate
the meterological observations from a network of weather stations operated by NOAA [ASOS](http://www.nws.noaa.gov/asos/)(Automates Surface Observing Systems). 
These stations record pressure, relative humidity and temperature in 30 minute intervals. After extensively searching the web for
the meteorological data from ASOS stations in the 30 minute intervals, we obtained the data for the entire year of 2014 from 
Dr. Teresa Van Hove from [UCAR COSMIC](http://www.suominet.ucar.edu/index.html) (Constellation Observing System for Meteorology, Ionosphere, and Climate)
program. We downloaded the meteorological data for the year 2014 for select stations from a UCAR server from a link provided by
Dr. Teresa Van Hove.<br> 
**Real time data:** The realtime feed from the ASOS stations is distributed thru the LDM protocol. Dr. Teresa also provided us with
a LDM link so that we can get a feed of the real time ASOS data (suomildm1.cosmic.ucar.edu).  

2. NOAA NEXRAD reflectivity products: We obtain the level III radar reflectivity products from the [NOAA NCDC](http://www.ncdc.noaa.gov/data-access/radar-data)
archive.<br> 
**Real time data:** For real time operation we are developing a script which pulls the level II reflectivity products from
NOAA NEXRAD reflectivity database on [Amazon S3](https://aws.amazon.com/noaa-big-data/nexrad/). 

## Open Source Software

We use freely available software to process and analyse our data. 

1. GAMIT (GPS Analysis at MIT): The [GAMIT](http://www-gpsg.mit.edu/~simon/gtgk/) software package takes in the 
input RINEX observation(carrier phase and pseudo range data) and meterological files and processes the IPW for each station. 

2. Python: The development of our nowcasting system is in python and we use the machine learning library 
[scikit-learn](http://scikit-learn.org/stable/). 

# Data Pipeline

The data set we will use for our nowcasting experiments comes from the Dallas-Fort-Worth (DFW) region. 
Part of the infamous U.S. "tornado alley", DFW spring and summer weather is dominated by convective thunderstorms
that move in lines generally from west to east through the region. We choose the DFW region because we understand
its climatology (through CASA's more than 15 years of operating networks of weather radars in tornado alley, 
first in Oklahoma and now in DFW) and because the DFW region has a high density of GPS receivers and
weather stations whose data are publicly available on-line for our use.

We take as the center of our domain the NWS KFWS NEXRAD radar in Fort-Worth Texas.
Within the 230 km coverage range of the radar we identified 44 Regional Reference Points, i.e., 
high performance dual-frequency GPS receivers. We found these stations by parsing through [GNSS station log files](ftp://geodesy.noaa.gov/cors/station_log/)
and identified the stations which are within the 230 km range radius ot the radar. Further we found out that most of these GNSS receivers are operated by the [Texas Dept. of Transportation (TxDOT)](http://www.txdot.gov/inside-txdot/division/information-technology/gps.html),
and were deployed to provide precise position information for Geodetic studies. As such these GNSS receivers do not have 
collocated weather stations. For the weather data (surface temperature, pressure, and relative humidity) 
required for IPW estimation, we used data from the network of Automated Surface Observation Stations (ASOS) 
operated by NOAA NWS. We find the closest ASOS station to each GNSS station and interpolate the meteorological
variables from the MSL of the ASOS station to the MSL of the GNSS station as suggested by [(Bai et al 2003)](http://www.sage.unsw.edu.au/wang/jgps/v2n2/v2n2pB.pdf). 
The near real-time water vapor system operated by [UCAR](http://www.suominet.ucar.edu/index.html) follows a similar principle. 
The figure below shows a map of all the sensors used in this study with respect to the KFWS radar in Dallas Fort Worth. 

{:.center}
![Figure 1. Sensor map in the Dallas Fort-Worth area Texas](/pictures/GPS_ASOS_Locations.png)

The following figure summarizes the data pipeline.
{:.center}
![Figure 1. Data pipeline to realize this nowcasting algorithm in realtime](/pictures/data_pipeline.png)
# Preliminary Experiments

From the data sources listed above we obtain the GNSS RINEX files for all 44 stations, met observations from 40 ASOS stations
for the entire year of 2014. We then feed the RINEX observation files (containing pseudo range and carrier phase) and RINEX
met data (containing pressure, temperature and relative humidity for each station) to the GAMIT software to obtain 30 minute 
intervals of precipitable water vapor measurements for each station for the entire year of 2014. 

We then normalize the precipitable water vapor with respect to each station and month to remove height dependencies in measurements
and seasonal variation in precipitable water vapor. We then select 23 days from the storm season (May-August) in 2014 for our
training/test set for our machine learning experiments. We choose days which have normalized precipitable water vapor values
exceeding 2 standard deviations from normal and term these days as "Weather Anomalies". 

We use the Multiquadric Interpolation technique suggested by (Tabios et al. 1985) to interpolate the point measurements of normalized precipitable
water vapor to a field spaning 300 by 300 km centered on the KFWS radar with a resolution of 3km. The reflectivity fields are
also converted from its native polar coordinates to cartesian coordinates to match the precipitable water fields. The following 
video shows the reflectivity fields overlapped over the normalized precipitable water fields for a storm case on May 8th 2014 UTC. 

{:.center} 
<iframe width="560" height="315" src="https://www.youtube.com/embed/2qXhBIHlfaM" frameborder="0" allowfullscreen></iframe>

Our initial machine learning experiment is to learn a discriminative machine learning random forest classifier to predict rain
or n rain 1 hour into the future for each pixel based on the last 4 time steps of precipitable water vapor and reflectivity foelds
at 30 minute intervals. We further average the precipitable water and reflectivity field at each time step to obtain a feature
vector of size 8 for each prediction. 

The random forest is trained and tested using a K-fold cross validation technique from the above mentioned 23 days. We perform 3
sets of experiments as follows: 
1. Use precipitable water features only
2. Use reflectivity features only
3. Use both precipitable water and reflectivity features. 

The results of the above three experiments are evaluated using a precision recall curve as shown below. 

![Figure 1. Nowcasting Schematic](/pictures/random_forest_average_precision.png)

As seen by the precision recall curves we can observe that the average precision score (area under the curve) evaluated by
varying the decision probability for the IPW and reflectivity feature is the highest. The random forest classifier thus performs
best when IPW features are concatenated with the reflectivity features. The following table shows additional metrics for our 
nowcasting algorithm. 

|     | IPW  | Refl.| IPW + Refl.  |
|-----| ---- |:----:| ------------:|
|POD  | 0.17 | 0.38 |0.56          |
|FAR  | 0.23 | 0.29 |0.15          |
|CSI  | 0.16 | 0.33 |0.51          |

# Nowcasting Algorithm
Our nowcasting algorithm currently predicts a binary value for each pixel indicating rain or no rain where rain is defined as 
anything which exceeds 24 dbZ. We use the last 4 time steps of average NIPW field and average reflectivity field as features
to our nowcasting algorithm. The figure below shows a schematic of our noecasting algorithm. 

![Figure 1. Nowcasting Schematic](/pictures/nowcast_over_time.png)

The above mentioned algorithm can be summarized with the following concept of operation.  

# CONOPS

1. Get GPS data for current time epoch.

2. Get ASOS data for current time epoch.

3. Get NEXRAD reflectivity data for current time epoch.

4. Map met data from ASOS stations to each GPS station and put in RINEX format (involves choosing closest ASOS station, transforming the data from ASOS station height to GPS station height, and putting the met data, surface pressure, surface temperature, and surface relative humidity, in RINEX format).

5. Run GAMIT to get an IPW value for each GPS station.

6. FOR REAL-TIME OPERATION, COULD MAINTAIN RUNNING AVERAGE/VARIANCE OF IPW AT EACH STATION

7. Normalize IPW value (either with a running multi-day average or using historical monthly average) to eliminate height and other effects to get normalized IPW (NIPW) value for each GPS station.

8. Use geospatial interpolation to interpolate NIPW values from each GPS station to the grid that defines the precipitation prediction domain.

9. Map the NEXRAD reflectivity data from its polar coordinates to the same precipitation prediction grid.

10. For each grid point
  1. Calculate average NIPW for past 4 time epochs in 33 by 33 region around grid point.

  2. Ditto for reflectivity.

  3. Feed the above 8-vector into the appropriate trained random forest classifer to get the precipitation probability for the grid point for 1-hour in the future.

  4. If probability exceeds a given threshold, set the grid point to 1 (will rain next hour), otherwise set it to 0 (won’t rain next hour).

  5. FOR REAL-TIME OPERATION COULD UPDATE LEARNING BASED ON PERFORMANCE DURING LAST EPOCH
11. End

12. Display predicted precipitation field (currently a binary field showing where dBZ is expected to exceed 24dBZ).




