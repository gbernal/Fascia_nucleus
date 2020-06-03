### Data Visualization Code (.py)

In the file [MainGUI.py](https://github.mit.edu/gbernal/Fascia_nucleus/blob/master/Fascia_dataViz/Fascia_sensor_data_plotter/python/MainGUI.py), you must make sure the IP address that it is connecting to is correct, and the same as the one you inputted into the firmware code. This is in line 120:

```python
self.ip = '192.168.0.101'
```

Next, take a look at [BCI_Data_Receiver.py](https://github.mit.edu/gbernal/Fascia_nucleus/blob/master/Fascia_dataViz/Fascia_sensor_data_plotter/python/BCI_Data_Receiver.py), and the lines 58-60: 

```python
num_elements = 17
num_bytes = 4*num_elements
num_packets = 22
```

Make sure that `num_elements` in the python script matches the `NUM_ELEMENTS` in the Arduino header file, and that `num_packets` in the python script matches `SEND_SIZE` in the Arduino header file. Lastly, ensure that `ELEM_SIZE` in the Arduino header file matches the multiplier in calculating `num_bytes` in the python script (currently correctly `4`).

Lastly, if you want the program to run indefinitely, collecting the data until you exist out of the window and stop the process, ensure that line 30 of the same file is set to `math.inf` as follows:

```python
self.num_data_to_halt = 5000 #TODO: set to 'math.inf' or comment-out lines 63-65 to run continuously
```

Now, you are ready to set GUI options to your preference, which are all the lines marked with a `TODO` comment in the [MainGUI.py](http://maingui.py) file, near the top (lines 37-59):

```python
# for plotting
self.start_idx = 2 #TODO: make this 0 if you want to graph all the packet data
...
# for FFT
self.graph_fft = 1 #TODO: change this to 1 if you dont want FFT graph
self.FFT_CHANNEL = 4 #TODO: make sure this is the channel you want the FFT for (0 indexed)
...
# for filters
data_rate = 1000 #TODO: make sure this matches the data rate of the ADS1299 in the firmware
```

- `start_idx` is the index of the data packet at which to begin graphing. You can start at 0 in order to graph **all** the data, or you can skip the first n data points by setting this value to n
- `graph_fft` is a boolean, if set to `1`, an FFT plot will be generated, and if it is `0`, no FFT graph will be plotted in the GUI window
- `FFT_CHANNEL` if you would like to have an FFT, this allows you to select which ADS1299 channel to run the FFT algorithm on and produce the frequency data for
- `data_rate` must match the ADS1299 data rate that is set in the firmware code— specifically this would be the settings for the ADS1299 register `config1`

Once you run the Python GUI program, you should be able to view the signals you selected, and move around and zoom in and out of each graph, as well as see the Current data rate and current measured heart rate.

### Packet break-down

[detailed packet break-down](https://www.notion.so/c39bc3815621426d9a86594971850b3b)

Please note: the valid array holds a 1 in the bit location corresponding to the data point in the packet which is invalid. For example, if the PPG data is invalid, then the valid array's 18th bit will be a 1. The index of the bit maps to the location of the data in the packet.

# IMU Data Conversion

If you decide you'd like the IMU data to be converted to metric units instead of being unitless ADC counts, you must uncomment the lines which perform this conversion in the `get_IMU_data()` function (lines 752-755 for gyroscope data, and lines 765-767 for acceleration data), and update the WiFi packet with the floating point data (lines 770-775). 

Since this modification changes the size of each IMU data point, it also affects the size of the data packets, so it is crucial to modify `num_elements` in the python script, and `NUM_ELEMENTS` in the Arduino code to **20**. This enables the firmware to send a data packet of the correct size, and the software to receive a packet of the correct size.

The last thing you need to change in order to get the IMU conversion working end-to-end is ensure that the python code can interpret the sent packet data correctly. This is done by specifying the type, in order, of the elements of the packet. In BCI_data_receiver.py, you'll find the line (line # 37):

```python
unpacked_data = struct.unpack('i'+'i'+'f'*8+'h'**6+'f'+'f'+'ii', data[i*num_bytes: (i+1)*num_bytes])
```

The data types and the order they appear in maps to the order in the packet, as shown in the table above. In order to complete this set up, you must change the `'h'*6` which means six elements of type `short` (which is 2 Bytes), to `'f'*6`, meaning six elements of type `float`, which is 4 Bytes and holds our converted data values.

# Follow-up Tasks

- [ ]  bluetooth instead of wifi
- [ ]  lead-off detection
- [ ]  record/store data
- [ ]  LSL
- [ ]  brain flow
- [ ]  security