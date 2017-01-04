# An interpolation technique to vizualize spatial data optimized using the multiprocessing module in Python

In this post I will explain how to implement a spatial interpolation technique to measure spatial data measured at point locations.
Later I will show you how I optimized the code using the multiprocessing module in Python which will prove a significant improvement
if you are trying to interpolate a variable over the entire CONUS. 

Spatial interpolation is a simple process by which we derive a continuous field from point measurements in a given spatial domain. 
Ideally to begin with the interpolation technique we need to start out with two variables the resolution of our interpolated field 
and the number of point estimates we have to derive this field. 

Let us start out by defining our domain. We pick the Dallas Fort-Worth area in texas as the center of our domain and identify 
44 different sensors within the range radius (230 km) of the KFWS weather radar. To make things simple we are going to convert 
all lat long coordinates to a reference frame with respect to the KFWS radar. 

~~~~

def get_GPS_cartesian(IPWsites):
    
    lat0 = (np.pi/180.0)*DFW.KFWSlat
    long0 = (np.pi/180.0)*DFW.KFWSlong

    gpsX = np.zeros((IPWsites.shape[0],1))
    gpsY = np.zeros((IPWsites.shape[0],1))
    R = 6378.137

    for si in range(IPWsites.shape[0]):
        lat1 = (np.pi/180.0)*float(IPWsites[si,1])
        long1 = (np.pi/180.0)*float(IPWsites[si,2])
        # Get cartesian x distance from KFWS to GPS site
        gpsX[si] = R * np.arccos(np.cos(np.pi/2 - lat0) * np.cos(np.pi/2 - lat0) + 
                np.sin(np.pi/2-lat0) * np.sin(np.pi/2-lat0) * np.cos(long1-long0))
    
        # Get cartesian y distance from KFWS to GPS site
        gpsY[si] = R * np.arccos(np.cos(np.pi/2 - lat0) * np.cos(np.pi/2 - lat1) + 
        np.sin(np.pi/2 -lat0) * np.sin(np.pi/2 - lat1) * np.cos(long0 - long0))
    
        if (lat1 - lat0) > 0 and (long1 - long0 ) > 0:
            # First quadrant
            continue
        elif (lat1 - lat0) < 0 and (long1 - long0 ) > 0:
            gpsY[si] = - gpsY[si]
        elif (lat1 - lat0) < 0 and (long1 - long0 ) < 0:
            gpsY[si] = - gpsY[si]
            gpsX[si] = - gpsX[si]
        else:
            gpsX[si] = - gpsX[si]
        
    return gpsX,gpsY,IPWsites
    
~~~~
We now have the locations of all our sensors with respect to the KFWS radar. Okay so now let us go ahead and plot the sensors
along with the KFWS radar and the 230km range radius to have a visual understanding of where things lie. The code to generate 
get the plot is as follows. 

