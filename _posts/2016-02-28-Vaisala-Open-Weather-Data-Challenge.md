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
 
 
