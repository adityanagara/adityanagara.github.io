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
(International GNSS Services).  
**Real time data:** The above mentioned servers also provide realtime data of the RINEX files for select stations. 

2. Meteorological data: Since most of the GNSS stations in our study do not have co-located weather stations we interpolate
the meterological observations from a network of weather stations operated by NOAA [ASOS](http://www.nws.noaa.gov/asos/)(Automates Surface Observing Systems). 
These stations record pressure, relative humidity and temperature in 30 minute intervals. After extensively searching the web for
the meteorological data from ASOS stations in the 30 minute intervals, we obtained the data for the entire year of 2014 from 
Dr. Teresa Van Hove from [UCAR COSMIC](http://www.suominet.ucar.edu/index.html)(Constellation Observing System for Meteorology, Ionosphere, and Climate)
program. We downloaded the meteorological data for the year 2014 for select stations from a UCAR server from a link provided by
Dr. Teresa Van Hove. 
**real time data** The realtime feed from the ASOS stations is distributed thru the LDM protocol. Dr. Teresa also provided us with
a LDM link so that we can get a feed of the real time ASOS data (suomildm1.cosmic.ucar.edu).  

2. NOAA NEXRAD reflectivity products: We obtain the level III radar reflectivity products from the [NOAA NCDC](http://www.ncdc.noaa.gov/data-access/radar-data)
archive. 
**real time data** For real time operation we are developing a script which pulls the level II reflectivity products from
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

We take as the center of our region the NWS KFWS NEXRAD radar in Fort-Worth Texas.
Within the 230 km coverage range of the radar we identified 44 Regional Reference Points, i.e., 
high performance dual-frequency GPS receivers. We found these stations by parsing through [GNSS station log files](ftp://geodesy.noaa.gov/cors/station_log/). 
Further we found out that these GNSS receivers are operated by the [Texas Dept. of Transportation (TxDOT)](http://www.txdot.gov/inside-txdot/division/information-technology/gps.html),
and were deployed to provide precise position information for Geodetic studies. As such these GPS receivers do not have 
collocated weather stations. For the weather data (surface temperature, pressure, and relative humidity) 
required for IPW estimation, we used data from the network of Automated Surface Observation Stations (ASOS) 
operated by NOAA NWS. We find the closest ASOS station to each GNSS station and interpolate the meteorological
variables from the MSL of the ASOS station to the MSL of the GNSS station. 
Figure 1 shows the relative locations of the GNSS receivers and ASOS stations within the 230 km 
coverage range of the KFWS radar. 

{:.center}
![Figure 1. Sensor map in the Dallas Fort-Worth area Texas](/pictures/GPS_ASOS_Locations.png)

The following figure summarizes the data pipeline.
{:.center}
![Figure 1. Data pipeline to realize this nowcasting algorithm in realtime](/pictures/data_pipeline.png)

We processes IPW data for the entire year of 2014. We obtain the met values from the ASOS stations in 30 minute intervals from 
Dr. Teresa Van Hove from UCAR COSMIC. We got in touch with Dr. Van Hove through an e mail and requested for the met values
for the year 2014. We were also able to get an LDM feed which gives us the real-time met values from each station. 

Our first analysis was to plot a histogram for each station for each month to look for any seasonal variation of IPW. We
found that IPW varies measured at each height and across the year where there is a higher monthly mean of IPW for the warmer
spring and summer months and a lower mean for the winter months. We thus normalized the IPW values with respect to each station
and each month to obtain a quantity which we term as NIPW (Normalized Integrated Water Vapor). 

Once normalizing we look for correlations between NIPW and reflectivity fields. We plot the reflectivity fields over the NIPW
fields for storm cases in 2014. The following movie shows the May 8th storm in the Dallas Fort-Worth area. 

The following video shows a storm case in the Dallas-Fort worth area on May 8th 2014.
{:.center} 
<iframe width="560" height="315" src="https://www.youtube.com/embed/2qXhBIHlfaM" frameborder="0" allowfullscreen></iframe>

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




