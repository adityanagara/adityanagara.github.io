---
layout: post
title: Engineering a Nowcasting System using Open Data Sources
comments: true 
redirect_from: "/2016/02/28/Vaisala-Open-Weather-Data-Challenge/"
permalink: jekyll-github-pages-poole 
---
This is the solution page of the vaisala open weather data challenge. 

The basic idea is to measure the amount water vapor in the atmosphere which is a precursor to severe storms(cite). Water vapor is a variable that evolves over time and space very quickly. The existing methods to measure atmospheric water vapor are not very effective. Some of the existing methods include radiosondes, water vapor radiometry and the GOES satellite and GPS meteorology. All of these techniques have drawbacks except GPS meteorology. \\

As signals propagate from GNSS satellites to a ground based receiver they travel an excess path length due to refraction in the atmosphere. This excesses path length that the signal travels causes an error in the position that is calculated from the GNSS signals. For geodic applications this error needs to be account for and discarded. However for meteorologists this error due to refraction is directly proportional to the amount of water vapor in the atmosphere. Hence this error is useful in meteorology to measure atmospheric water vapor. \\

The GPS-Meteorology technique thus provides applications in different domains one geology and the other meteorology. Due to this there application in two domains there has been a growing number of GNSS stations spawning around the world. These GNSS stations operate continuously collecting pseudo range and carrier phase (distance travelled by signal from satellite to receiver) from satellites anywhere between 14-20 GNSS satellites (GPS, GLONASS, Galelio) at 30s intervals. The data for over 1000 GNSS stations are available online for the last 5 years (http://sopac.ucsd.edu/,  http://www.suominet.ucar.edu/, http://www.ngs.noaa.gov/). We propose to use the raw GPS carrier phase and pseudo range data from these repositories. \\

In order to calculate the precipitable water at a GNSS station we require surface meteorological observations. Most GNSS stations do not have co located meteorological sensors and thus we need to use sensors from other networks.\\
