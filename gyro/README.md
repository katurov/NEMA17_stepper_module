# The Gyro module #

The idea of this code is to determine necessary length for each table's legs to make table fully horisontal. The part interrested to me is *Trigonometry* where I can use values from gyro to calculate hights of each point.

As soon as I know the delta of Z-position (hight) for leg I can move it. Great.

## RAW and Rad gyro's values ##

[Source library by Kristian Lauszus, TKJ Electronics](https://github.com/TKJElectronics/Example-Sketch-for-IMU-including-Kalman-filter)

I've used this library as a source and made no changes (yet). This have to be told:

Values ```kalAngleX``` and ```kalAngleY``` in this version are in radians (degree in original code), which is usable to calculate sinuses.

*TODO*: Calculate offset and add it to initialisation procedure

## Kalman filtering ##

[Source library by Kristian Lauszus, TKJ Electronics](https://github.com/TKJElectronics/KalmanFilter)

This library already included in MPU6050 library by Kristian, so here are no problem to start using it, all code are in thwo parts: initial filling of values and updating it with new ones. In second case we have to give time as a paramater. Initialization: ```kalmanX.setAngle(roll)``` also used if ```roll``` gots out of border; ```kalAngleX = kalmanX.getAngle(roll, gyroXrate, dt)``` for any other case, where ```dt``` is time in seconds from previous call. 

*TODO*: well, it is use value of seconds, but called 10 times per second, also it uses ```micros()``` instead of ```millis()``` with no reason.

## Trigonometry ##

Lets suppose a table as a rectangle with points A, B, C, and D in the centers of legs (no matter how bit the table is):
```
A ----- B
|       |
|       |
|       |
|       |
D ----- C
```

The gyro have two vectors: X and Y, so in our case Y arrowed from D to A and X from D to C:
```
A ----- B
| ^     |
| | Y   |
| |     |
| --> X |
D ----- C
```

Suppose we have Y = 0.32 (sorry for this graphics):
```
    A
   /|
  / |
 /  |
D----
```

If D have hight = 0.00 (ground level), then A will have hight equal to lenght of DA * sin(abs(Y)), where Y in radians. If length of DA is in mm, so hight of A will be in mm too. 

BUT we have two vectors which affects to third "Z-vector" for point. To calculate it we have to use this algorithm:
1. For easy calculations we have to find "ground" point: if X is positive then D-C line is down; if Y is positive then B-C line is down; if X is positive and Y is positive, when C is the lowest point of the table
2. As C is the lowest, lets look on the projection: from side C-D, and C-B. Using formula from above we will have hights of D and B (length from C to point multyplied by sin of the angle).
3. Fourth point is the easiest one from this moment. Vector addition makes it easy: hight of A is summ of hights D and B.

Now we have all four points. You can get it via serial just send "L" for it (don't forget to upload code).

## Usage ##

Suppose we have coordinates:
```
A : 0.00
B : 1.00
C : 1.00
D : 2.00
```
So we have to hold A-leg, move B and C legs with constant speed, and move D with speed * 2.
