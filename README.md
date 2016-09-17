# ArduinoTapTempo
An Arduino library that times consecutive button presses to calculate a tempo. Corrects for missed beats and can reset phase with single taps.

## How to install

In the Arduino IDE, go to Sketch > Include libraries > Manage libraries, and search for ResponsiveAnalogInput. You can also just use the files directly from the src folder.

Look at the example in the examples folder for an idea on how to use it in your own projects. The source files are also heavily commented, so check those out if you want fine control of the library's behaviour.

## How to use

```c++
// include the ArduinoTapTempo library
#include <ArduinoTapTempo.h>

// define the pin you want to attach your tap button to
const int BUTTON_PIN = 5;

// make an ArduinoTapTempo object
ArduinoTapTempo tapTempo;

void setup() {
  // begin serial so we can see the state of the tempo object through the serial monitor
  Serial.begin(9600);

  // setup your button as required by your project
  pinMode(BUTTON_PIN, INPUT);
  digitalWrite(BUTTON_PIN, HIGH);
}

void loop() {
  // get the state of the button
  boolean buttonDown = digitalRead(BUTTON_PIN) == LOW;
  
  // update ArduinoTapTempo
  tapTempo.update(buttonDown);

  // print out the beats per minute
  Serial.print("bpm: ");
  Serial.println(tapTempo.getBPM());
}
```

## Basic methods

```c++
void update(bool buttonDown);
```
Call this each time you read your button state. Accepts a boolean indicating if the tap button is down.
```c++
float getBPM();
```
Returns the number of beats per minute.
```c++
unsigned long getBeatLength();
```
Returns the length of the beat in milliseconds.
```c++
float beatProgress();
```
Returns a float from 0.0 to 1.0 indicating the percent through the current beat.
```c++
bool onBeat();
```
Returns true if a beat has occured since the last update(). Not accurate enough for timing in music but useful for LEDs or other indicators. Note that this may return true in rapid succession while tapping and setting the tempo to a slightly slower rate than the current tempo, as the beat may occur and shortly afterward a new tap triggers a new beat.
```c++
unsigned long getLastTapTime();
```
Returns the time of the last tap in milliseconds since the program started.


## Other methods (settings)

### Chains

ArduinoTapTempo calls groups of taps 'chains', and calculates the tempo by averaging a number of taps in a chain. Chains are active for a short while after each most recent tap, and while they are active they will count new taps as part of the current chain and average the timing of new taps together with previous taps. The chain is reset if a tap occurs once the chain is no longer active, and the tempo calculation will then be based off 2 or more taps in the new chain.

```c++
bool isChainActive(unsigned long ms /*optional*/);
```
Returns true if the current tap chain is still accepting new taps to fine tune the tempo.
```c++
void resetTapChain(unsigned long ms /*optional*/);
```
Resets the current chain of taps and sets the start of the bar to the current time. This is called internally when a tap occurs and the current chain is no longer active
```c++
void setBeatsUntilChainReset(int beats);
```
The current chain of taps will finish this many beats after the most recent tap. It accepts an integer greater than 1.
```c++
void setTotalTapValues(int total);
```
Sets the maximum number of most recent taps that will be averaged out to calculate the tempo. Accepts int from 2 to 20. Increasing this allows the tempo to be more accurate compared to your tapping, but slower to respond to gradual changes in tapping speed. Response to different tapping speeds in new chains is unaffected.

### Skipped tap detection

 ArduinoTapTempo has skipped tap detection turned on by default. If you miss tapping a beat but tap the next one it'll recognise that a tap was meant to occur and prevent the tempo from dropping abnormally. Skipped taps are detected when a tap duration is close to double the last tap duration.

```c++
void enableSkippedTapDetection();
```
Turns skipped tap detection on.
```c++
void disableSkippedTapDetection();
```
Turns skipped tap detection off.
```c++
void setSkippedTapThresholdLow(float threshold);
```
This sets the lower threshold, how early your next tap can be after a missed tap. It accepts a float from 1.0 to 2.0.
```c++
void setSkippedTapThresholdHigh(float threshold);
```
This sets the upper threshold, how late your next tap can be after a missed tap. accepts a float from 2.0 to 4.0.

### Limits

```c++
void setMaxBeatLengthMS(unsigned long ms);
```
Sets the maximum beat length permissible. If a tap attempts to set the beat length to anything greater than this value, the new tap will start a new chain and the tempo will remain unchanged.
```c++
void setMinBeatLengthMS(unsigned long ms);
```
Sets the minimum beat length permissible. If the average tap length is less than this value, then this value will be used instead.
```c++
void setMaxBPM(float bpm);
```
Sets the minimum beats per minute permissible. This is another way of setting the minimum beat length.
```c++
void setMinBPM(float bpm);
```
Sets the maximum beats per minute permissible. This is another way of setting the maximum beat length.
