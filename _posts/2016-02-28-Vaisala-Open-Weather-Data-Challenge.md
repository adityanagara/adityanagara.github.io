---
layout: post
title: Engineering a Nowcasting System using Open Data Sources
comments: true 
redirect_from: "/2016/02/28/Vaisala-Open-Weather-Data-Challenge/"
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

#Data and Software
## Open Source Data
Our nowcasting system which we term as "Deep Nowcaster" uses data from the following open data sources and software. 

1. GNSS data: The GNSS data from all GNSS stations operated world wide are available from BGS [CORS](ftp://geodesy.noaa.gov/cors/) (Continuously Operated 
Reference Stations), [SOPAC](ftp://garner.ucsd.edu/pub/rinex/) (Scripps Orbit and Permanent Array Center) and [IGS](ftp://igscb.jpl.nasa.gov/pub/station/). 

2. NOAA NEXRAD reflectivity products: Our data pipeline includes pulling data from the NOAA NEXRAD reflectivity database
from [Amazon S3](https://aws.amazon.com/noaa-big-data/nexrad/). 

3. Meteorological data: Since the GNSS stations do not have co-located meteorological sensore we use a meterological network called
the [ASOS (Automates Surface Observing Systems)](http://www.nws.noaa.gov/asos/). This being the crucial ingredient in calculating
high resolution water vapor measurements from our network og GNSS stations were not openly available online after a thorough search. 
We finally got the data for the ASOS stations from UCAR COSMIC(Constellation Observing System for Meteorology, Ionosphere, and Climate) program
where Dr. Teresa Van Hove gave us surface meterological observations for the entire year of 2014 from over a 100 ASOS stations. 
We use this data to and interpolate it to our closest GNSS station. 

## Open Source Software

<iframe width="560" height="315" src="https://www.youtube.com/embed/DK_0jXPuIr0" frameborder="0" allowfullscreen></iframe>

with a **youtube** video embedded



