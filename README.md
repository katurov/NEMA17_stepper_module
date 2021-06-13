# NEMA17 stepper module

The idea of this project is to create a module of NEMA17 driven by 1-wire interface. This will allow me to create projects with 4 or 6 drives without complicated arhitecture. Using 1-wire makes possible to use attiny 13 or 85 which looks just fine.

## Motor wiring

I'm using L293D as a driver for motor, by this common use schematics ([source](https://forum.arduino.cc/t/bipolar-stepper-motor-fails-at-proper-voltage/185105)):
![L293D for NEMA17](https://github.com/katurov/NEMA17_stepper_module/blob/main/images/ed46e21a44e0ca7a3de016f1e88d1e2e93e5245b.gif?raw=true)

My NEMA17 is color-coded this way (pin on schema - color of wire):
1. is Yellow or Black
2. is Green
3. is Red
4. is Blue

## Driving itself

We all have an example of ["Stepper" lib on arduino](https://www.arduino.cc/en/reference/stepper), but after I've tried to use it, it was too noizy! So I used this idea to build own control called 'half-step' which makes this movement fast, powerful and silent. The idea ([source](https://create.arduino.cc/projecthub/voske65/arduino-half-step-stepper-motor-driver-l298n-24df69)) looks like so:

![power diagram](https://github.com/katurov/NEMA17_stepper_module/raw/main/images/halfstep__VYLC4petD0.jpg) ![coils states](https://github.com/katurov/NEMA17_stepper_module/raw/main/images/halfstep1_o6zNxipXxz.jpg)

Firstly translate sequence into byte array. As we have one motor, only four bits are used (this case mapped to ports 8-9-10-11 on Arduino):
```cpp
byte steps[8] = {
  B00001001,
  B00001000,
  B00001010,
  B00000010,
  B00000110,
  B00000100,
  B00000101,
  B00000001,
};
```
When later use it in a loop. NEMA17 have 200 steps per full rotation (1.8 degree per step: 360 / 1.8) from datasheet, so far half-step will use 400 steps per full rotation (0.8 degree per step: 360 / (1.8/2)). This loop is: 50 * 8 = 400; means one full rotation (counterclockwise in this case):
```cpp
for ( j = 0; j < 50; j ++) {
  for ( i = 0; i < 8; i ++ ) {
    PORTB = steps[i];
    delayMicroseconds(650);
  }
}
```

Where `PORTB` is byte from [port manupulation](https://www.arduino.cc/en/Reference/PortManipulation) of Arduino.
