# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program
   
### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases).

### Goals
In this project, our goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. We will be provided the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also, the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 50 m/s^3.

#### Model for generating path.

We generate new way points from the current position of the can using the frenet coordinates. We use the current frenet coordinate of the car and extrapolate the path for next 90 meters. 

```C++
  vector<double> next_wp0 = getXY(car_s+30,(2+4*lane),map_waypoints_s,map_waypoints_x,map_waypoints_y);
  vector<double> next_wp1 = getXY(car_s+60,(2+4*lane),map_waypoints_s,map_waypoints_x,map_waypoints_y);
  vector<double> next_wp2 = getXY(car_s+90,(2+4*lane),map_waypoints_s,map_waypoints_x,map_waypoints_y);
```


Then, we convert those coordinates to global x,y coordinate. We then convert it to the local coordinate of the car, to fit a spline in the way points.
Then we convert the points that are lying on the spline as the points the car should follow.

```C++
   for(int i=0; i<ptsx.size(); i++){
          double shift_x = ptsx[i] - ref_x;
        double shift_y = ptsy[i] - ref_y;
        ptsx[i] = (shift_x*cos(0-ref_yaw) - shift_y*sin(0-ref_yaw));
        ptsy[i]= (shift_x*sin(0-ref_yaw)+ shift_y*cos(0-ref_yaw));
    }
```

In order to have a smooth transition to the car, we need to merge the previous ways generated by a spline( at t-1 of the simulator, which is not yet processed). 

```C++
       for(int i = 0; i < previous_path_x.size(); i++)
                    {
                        next_x_vals.push_back(previous_path_x[i]);
                        next_y_vals.push_back(previous_path_y[i]);

                    }
```


#### Lane Change and maintaining speed.


We use state based lane change model. We use the fused sensor data from the sensors to predict the vehicle. We use the current position of the other vehicles and their velocity predict the future location at time (t+1)

```C++
    \\ The prev_size is the size of the way points that was not covered by the current iteration of the simulator.
     check_car_s+= ((double)prev_size *.02*check_speed_front);
```

We have 6 states in the model.

a. Maintain lane and speed
b. Maintain lane increase speed.
c. Maintain lane decrease speed.
d. Start lane change left
e. Start lane change right
f. Finished lane change 

Maintain lane and speed:
This is the default state of the vehicle. In this state, vehicle maintains the speed and lane. A vehicle continues to be in this state if the vehicle is already at the maximum speed limit and there is no vehicle in front of it (at 30 m or more in future(t+1)).

Maintain lane and increase speed:
In this state, vehicle increases the speed (accelerates) and maintains lane. A vehicle continues to be in this state if the vehicle is below the speed limit and there is no vehicle in front of it (at 30 m or more).

Maintain lane decrease speed:
In this state, vehicle decreases the speed (decelerate) and maintains lane. A vehicle continues to be in this state if the vehicle there is a vehicle in front of it (at 30 m or less). This state can lead to a start lane change left or right.

Start lane change left and start lane change right:
This is performed when the lane to left / right, has no vehicle in front of it with distance 40 m or less in intended lane(left / right lane) and no vehicle behind it 20 m less. Once the car has entered lane change, it cannot reenter this state until the lane change complete state is achieved.

 Finished lane change :
 This state can be reached only from 'start lane change left' and 'start lane change right' state. This state indicates the lane change has been completed.

 ### Conculsion

 The car was able to successfully drive the entire track without any incidents. There are several improvements that can be performed in the project. We could make car prefer the left lane change or right lane change by comparing the speed of the car in front of the intented lane. 
 Also, We would like to experiment with jerk minimizing trajectory(JMT).