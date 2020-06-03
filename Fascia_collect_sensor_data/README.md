

# About

This document explains the usage and functionality specification of the [Fascia_collect_sensor_data](https://github.com/gbernal/Fascia_nucleus/tree/master/Fascia_collect_sensor_data) Arduino firmware in the [Fascia_nucleus](https://github.com/gbernal/Fascia_nucleus) repo 

# Set Up

### Software

In order to begin running this code, you must have all the required libraries installed. This includes `WiFiNINA`, and `MPU6050` by Electronic Cats, both of which you can get from the Arduino library manager, and `SparkFun_MAX3010x_Pulse_and_Proximity_Sensor_Library` which you need the modified version of in the repo. This is located in [Fascia_nucleus/Firmware/development/Libraries/SparkFun_MAX3010x_Pulse_and_Proximity_Sensor_Library](https://github.mit.edu/gbernal/Fascia_nucleus/tree/master/Firmware/development/libraries/SparkFun_MAX3010x_Pulse_and_Proximity_Sensor_Library). I have made changes to speed up the data acquisition in the library, and reduce wasted wait time. Namely, I made changes to two different functions:

- `readTemperature()`

     I split this function into two functions: `requestTemperature()` , and `readTemperature()`. Previous to this, the library would request a temperature reading from the MAX3010x and wait for a while for the data to be ready. With my change, I request the temperature using the request function, then perform other things in my code, and the read the temperature by calling the read function, without waiting idly for the sensor to be ready.

- `getIR()`

    This function did a similar thing where once you called it, it would wait 250 ms for the new data to be ready. Instead of doing this, I made the function take as input the number of milli seconds you'd like it to wait for the data to be ready, and if the data is still not ready by that time, the function returns `0`, which indicates an invalid/unavailable reading of the PPG sensor.

In order to run the code, you must copy this modified library and paste it into your Arduino libraries folder (usually in your documents folder).

### Hardware

Once you're able to start running the code, you might notice that the code hangs in certain areas: this will happen if any of the sensors the code expects you to have proper connection to are unreachable. Make sure your ADS1299, MAX3010, and MPU5060 are properly connected all the way to the Fascia Main Board, and that the Arduino can communicate with them.

If you are missing one or more of the sensors, you can get the code to run without expecting said sensor(s) to be connected by commenting out the lines where the code is setting the sensor(s) up, and where it is retrieving data from the sensor(s). You can find these lines in the two functions `setup()` and `loop()`, where sensor setup and data retrieval occur, respectively.

# Configurations

In order to properly get the code to run and do what you'd like to do with it, you'll want to make sure you have correctly configured the top of the .ino file with your current setup and desired output. These settings can be found on lines 19-28 of Fascia_collect_sensor_data.ino:

```arduino
// settings
#define CONNECT_WIFI 1

#define BOARD_V FASCIA_V0_0
#define DATA_MODE RDATA_SS_MODE
#define RUN_MODE NORMAL_ELECTRODES
// v for verbose: lots of prints
#define v 0
// debug: serial reads and writes
#define debug 1
```

- `CONNECT_WIFI` would enable(`1`)/disable(`0`) WiFi on the chip. Fascia will not send any data packets over WiFi when this is disabled (`0`).
- `BOARD_V` allows you to specify the board you are working with. This code is specifically written for Fascia, so I cannot guarantee that any data, other than the ADS1299 data, would be correct or using the proper pin to which that sensor is connected, if you use any other `BOARD_V` other than `FASCIA_V0_0` or `FASCIA_V0_1` with this version of this code.
- `DATA_MODE` this version of the code only supports `RDATA_SS_MODE` currently; please do not change this line. This is referring to the ADS1299 Single Shot mode, versus the Continuous Conversion mode
- `RUN_MODE` enables you to generate internal-test signals in the ADS1299 instead of reading external data and signals from the electrodes on the 8 channels of the ADS1299 if you set it to `GEN_TEST_SIGNAL`. Keep it as `NORMAL_ELECTRODES` for any purpose other than generating the mentioned internal test signals.
- `v` enables/disables verbose print statements in the Serial port. If `1` (enabled) you'll see all the collected sensor data values printed in the Arduino Serial Monitor. `0` disables this.
- `debug` enables/disables the ability to change some specific ADS1299 channel settings via serial messages through the serial monitor. The detailed description of the things you can do will be printed once setup is complete, if you set `debug 1` and run the code. These things include enabling/disabling Bias, SRB2, and changing the gain of each channel, as well as turning each channel on/off, and even checking the current status of all the ADS1299 registers.

### Using the Serial Debug Interface

Here is a summary of the commands you can run in the debug serial interface, and what they do:

- type the channel number to print that ADS1299 channel's data [1-8] (and plot, if you switch to Serial Plotter)
- or type '0' to stop printing the data.
- type BN#0 to deactivate biasN for channel # and BN#1 to activate it
- type BP#0 to deactivate biasP for channel # and BP#1 to activate it
- type S#0 to deactivate SRB2 for channel # and B#1 to activate it
- type G#N to set the gain for channel # to N=0:1, N=1:2, N=2:4, N=3:6, N=4:8, N=5:12, N=6:24
- type T#0 to toggle channel # off, and T#1 to toggle channel # on
- type 'R' or 'r' to print the current register settings of the ADS1299
- type 'P' or 'p' to print these instructions again

### Configuring Data Rate

You can adjust the data rate of the ADS1299 (and since all the other signals' data rates are programmed to be a tenth of it) in the firmware, in line 183 of Fascia_collect_sensor_data.ino  by changing the last argument of this line `ADS_WREG(ADS1299_REGADDR_CONFIG1,________);` 

The available sampling rates are listed starting on line 152 in the header file ADS1299.h:

```arduino
#define ADS1299_REG_CONFIG1_16kSPS      0     // Data is output at FMOD/64, or 16 kHz at 2.048 MHz.
#define ADS1299_REG_CONFIG1_8kSPS       1     // Data is output at FMOD/128, or 8 kHz at 2.048 MHz.
#define ADS1299_REG_CONFIG1_4kSPS       2     // Data is output at FMOD/256, or 4 kHz at 2.048 MHz.
#define ADS1299_REG_CONFIG1_2kSPS       3     // Data is output at FMOD/512, or 2 kHz at 2.048 MHz.
#define ADS1299_REG_CONFIG1_1kSPS       4     // Data is output at FMOD/1024, or 1 kHz at 2.048 MHz.
#define ADS1299_REG_CONFIG1_500SPS      5     // Data is output at FMOD/2048, or 500 Hz at 2.048 MHz.
#define ADS1299_REG_CONFIG1_250SPS      6     // Data is output at FMOD/4096, or 250 Hz at 2.048 MHz.
```

# Running/visualizing the code

We have Python code in place to help visualize the sensor data received over WiFi. In order for this to work, you must set `CONNECT_WIFI` to `1` in the .ino file in order for the two scripts to be able to connect and share the data. In addition to this, you have to make sure that both the Arduino and the python code are connecting to the correct WiFi network.

### Firmware (.ino)

In the file [WiFi_Settings.h](https://github.com/gbernal/Fascia_nucleus/blob/master/Fascia_collect_sensor_data/WiFi_Settings.h)

```arduino
#define SECRET_SSID "raspi_wifi"
#define SECRET_PASS "fluidfluid"
#define HOST_ID     "192.168.0.101"
```

Ensure that `SECRET_SSID` has the name of your private WiFi network, and `SECRET_PASS` has the password to that network. `HOST_ID` should be the IP address of your computer if you type `ifconfig` in your terminal, and find `en0`, include the IP address listed under that.

In the same file, take a note of these lines:

```arduino
#define SEND_SIZE 22
#define NUM_ELEMENTS 17
#define ELEM_SIZE 4
```

You'll need these values to ensure that the connection to the python script is correct.


