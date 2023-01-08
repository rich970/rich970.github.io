---
title: An simple app built using streamlit to calculate hiking times for walks. 
image: 
---  

<iframe src="https://rich970-hiking-trip-app-hiking-trip-app-0f5ced.streamlit.app/" style="border: none;"></iframe>



The core process is summarised below though:

```python
import pandas as pd
import numpy as np
import datetime as dt
import matplotlib.pyplot as plt

plt.close('all')

av_pace = 4.8 #km/hr - average pace
hike_distance = 32 # km - total hike distance
rest_interval = 5 # km - distance interval between rests
standard_rest = 15 # mins - normal rest time
lunch_rest = 45 # mins - lunch rest
t_start = dt.datetime(2000, 1, 1, hour=7, minute=40) #  start time for hike

# initialise some datetime constants
t_sunrise = dt.datetime(2000, 1, 1, hour=6,minute=30)
t_sunset= dt.datetime(2000, 1, 1, hour=17,minute=30)

# Modify rest interval to nearest value which has integer division with av_pace (km/min)
rest_interval = (av_pace/60) * np.round(rest_interval/(av_pace/60))

############# MAIN SIMULATION CODE ##############
# calculate rests and total hike time:
n_rests = (hike_distance-1)//rest_interval
total_rest = (dt.timedelta(minutes=n_rests*standard_rest)
              + dt.timedelta(minutes=(lunch_rest - standard_rest)))
                         
total_hiketime = dt.timedelta(hours=(hike_distance/av_pace))

# calculate arrival time
t_arrival = t_start + total_hiketime + total_rest

print('Departing at: ', t_start.time())
print('Average pace: {0} km/hr'.format(av_pace))
print('Resting every {0} km, results in {1} rests totalling {2} (HH:MM:SS) '.format(
    rest_interval, n_rests, total_rest))

print('Total hike time (HH:MM:SS): ', total_hiketime)
effective_pace = np.round(3600*hike_distance/(total_hiketime.seconds + total_rest.seconds), 2)
print('Effective pace (inc\' rests): {0} km/hr'.format(effective_pace))
print('Estimated arrival time: ', t_arrival.time())

# Calculate progress:
d = [0]
i = 1
rest_time = 0
t = np.arange(t_sunrise, t_sunset+dt.timedelta(minutes=1), dt.timedelta(minutes=1))
#t = np.array([dt.datetime(2000, 1, 1, x, 0) for x in range(24)])
for t_sim in t[:-1]:
    d_sim = d[i-1]
    # Create the 'rest' flag based on the current distance:
    rest = (abs(d_sim - rest_interval*(np.round(d_sim/rest_interval))) < 1e-5) 
    
    # set the length of rest depending on current simulation time
    if (t_sim > dt.datetime(2000, 1, 1, hour=12) ) and (t_sim < dt.datetime(2000, 1, 1, hour=13, minute=30)):
        rest_period = lunch_rest
    else: rest_period = standard_rest
    
    
    if t_sim < t_start:
        d.append(0)
        
    elif (t_sim < t_arrival) and (t_sim > t_start):
        # are we resting?
        if rest and (d_sim > 0) and (rest_time < rest_period):
            rest_time += 1
            d.append(d[i-1])
        # are we restarting after a rest?
        elif rest_time == rest_period:
            rest_time = 0
            d.append(d_sim + av_pace/60)
        # otherwise we are in a normal hiking window
        else: d.append(d_sim + av_pace/60)
        
    else:
        d.append(d_sim)

    i+=1 

df = pd.DataFrame(index = t, data = {'distance' : d})
df.plot()
plt.xlabel('Time of day')
plt.ylabel('Distance travelled [km]')

```
