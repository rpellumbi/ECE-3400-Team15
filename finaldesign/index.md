# Final Robot

## Radio Communication 

In order to get the Arduinos to communicate wirelessly using the RF chips, we studied and experimented with the GettingStarted.ino file given in Lab 3 to find out which parts were necessary to transmit and receive information and which parts were extraneous to our objective for the competition. The configuration settings in our code were drawn heavily from the given configuration settings, with the only changes being an increased power (max) and data rate (2MBPS) and a decreased payload size for more reliability (8 bytes). Since the robot was always sending information and the base station was always receiving it, we found the role switching capabilities of the original code to be unnecessary. Both pipes for reading and writing were set from the beginning and we found no need for the base station to send information for the robot to receive beyond an acknowledgment of a successfully received message. As a result, the final RF code used for this lab was significantly cut down in length while still remaining functional.
The protocol for communication between the base and robot was based on a two byte integer named data on the robot that is updated, sent to the base, and then cleared at every intersection. Whenever the robot hits an intersection, based on its current heading, it updates its position and scans the walls around it. The appropriate bits are set in data and then data is sent to the base. We also update and send data after seeing a robot. After it is sent, data is reset to 0x0000 to ensure information for future transmissions does not contain pieces of previous transmissions.
The encoding for data is as follows:
Nibble 1 
[0:3] - Robot x co-ordinate (Range is 0-8 since its a 9*9 matrix) 

Nibble 2 
[4:7] - Robot y co-ordinate (Range is 0-8 since its a 9*9 matrix) 

Nibble 3
[8] - West Wall ( 0 - no wall ; 1 - wall exists ) 
[9] - North Wall ( 0 - no wall ; 1 - wall exists ) 
[10] - East Wall ( 0 - no wall ; 1 - wall exists ) 
[11] - South Wall ( 0 - no wall ; 1 - wall exists )

Nibble 4
[12-14] - Treasure Type  ( Predetermined treasure type codes as below) 
No Treasure = 0;
Blue Triangle = 1; 	Blue Square = 2;	 Blue Diamond = 3;
Red Triangle = 4; 	Red Square = 5; 	 Red Diamond = 6;
[15] - Opponent Robot ( 0 - no Robot exists ; 1 - Robot obstructing path ) 



Once the robot sends the base station a message with all of the information encoded in the format described above, the base station decodes the message with the use of masking and bit shifting in order to extract information from specific bits. The base station iterates over the bits of the message and prints (without newlines) the information contained within them. For example, if our robot detects another robot, bit 15 will be set to 1 and the base station will print “,robot=true”. Once the message has been fully parsed and interpreted, it prints a new line so that the GUI will receive the information and update accordingly.
## Printed Circuit Board 

As encouraged by the course staff, we decided to make a printed circuit board (PCB) to both clean up our wiring for our final robot and to remove the problem of wires falling out that is inherent when using a breadboard. The idea for the PCB is that the PCB would sit like a 'hat' on the Arduino and have headers for all sensors and other inputs. We used EAGLE PCB Design Software to design the PCB. 

The first step in this process was creating the schematic. This involved downloading all the parts we are using from online libraries and wiring everything as we have on the breadboards. I downloaded a library for the multiplexor and for the Arduino to ensure the dimensions of the pinouts and spaces were all correct. Here is a picture of the schematic.

![schematic image](Media/schematic.png)

The next part of the process was to use the schematic to layout the board. First I placed all components on the board. When placing the components on the board, all pins on each component have straight 'air wires' that show which pins on other components the given pin needs to be wired to. So, when placing the components, it is most ideal to have as few 'air wires' crossing each other as to make wiring the components easier later.

We chose to have a 2-layer board. The bottom layer is ground. This simplifies the wiring because it allows all pins that need to be grounded to be shorted to the bottom layer instead of all being wired together. After creating the ground layer, we started connecting the wiring on the top layer. After choosing a trace width of about 0.6 mm as determined by the max voltage of 5V, with as straight wires as possible, we wired all components together. Because some wires cross, some wires need to go through a via to the bottom layer to go under the wires they intersect with when being routed. Here is a picture of the schematic. Red wires are through the top layer and blue wires are through the bottom layer.

![board image](Media/board.png)

The PCB ultimately failed because of the PCB mill we used to print it. We were not able to level the board properly so each time we milled the board, some parts of the board were cut to deep so there was no path for signal, and in some place its was not cut deep enough and the copper was not broken through and thus the signal was shorted with the whole level. We tried for a long time to level the board but we eventually gave up. We realized that even if we finally printed the first side correctly by luck, we would also have to print the second side perfectly. Since we would not be so lucky twice in a row, we decided to give up on the PCB and put our circuitry on protoboards.

Below is a picture of a failed protoboard. On the top left of the board, you can see that the mill did not cut through the copper so that trace would be shorted with the rest of the layer. At the same time, on the right side, the mill cut so deep  that there is no copper left to transport the signal. This issue was caused by the uneven base of the mill. This caused us to put our circuit on a protoboard instead of using the PCB.

![board image](Media/printedboard.jpg)


## Proto Boarding

Once we realized that the printed circuit board would not work, we decided to move all our circuitry to protoboards. We moved the amplifiers, multiplexer, headers, LEDs, and microphone to small protoboards cut to the needed size as to be as small as possible. We used protoboards that had rows of holes connected by small copper pieces that imitate a breadboard to clean up the soldering work. Most boards connected to a head on the arduino. Three boards were on top of the robot, one with the push button, one with the microphone and its amplifier, and one for the IR sensor. 

Here is an example of a protoboard we used for digital pins 0 through 7. 

![D0-D7 board](Media/D07board.jpg)

On the top right you can see the 8 pins that connect right into the arduino. 3 of the pins are used as select bits for the mux (the long central chip). Two of the top pins are used as PWM ports to control the servos, and the servos plug into the headers on the top left. The bottom blue wire is the audio input to the amplifier and the grey wires are both the output of the amplifier (one is unsoldered so it can be used for testing on an oscope). On the left, you can see three headers that are the wall sensors. All three wall sensors are inputs to the mux. Finally, on the right, there are two single pin headers, one is the output of the mux, and one is where the IR signal is an input to the mux.

The rest of the protoboards were very similar but with the required pins for the given arduino headers and inputs. There were additional protoboards made for the push button, microphone, IR sensor and amplifier, arduino analog pins 0 through 5, and radio module.

## FFT

On our final robot, we had a microphone to detect 660 Hz audio signal to tell the robot to start, and a phototransistor IR sensor to detect other robots emitting IR at 6.08kHz. We processed the signals using the Arduino FFT Library. Also, we made 2 amplifiers to clearly detect the signals from a farther distance. 

To process our signals, we used the arduino FFT library. We used the free running mode and the example code from fft_adc_serial. We first had to determine which bins we expect our signal to be in. We chose a sampling frequency of 38kHz and sampled 256 times. Thus, 38kHz / 256 samples = 148.4 Hz per bin in our FFT. From here, we were able to determine that the 660 Hz audio signal would be in the 5th bin and the 6.08 kHz IR signal would be in the 43 bin. After experimenting to see where the signals did fall, we confirmed our calculations were correct.

Since during the competition we would need to identify both audio and IR coming from about a foot away, we added two amplifiers to amplify both the audio and IR signals. Here is the schematic of the amplifier with the microphone:


The same amplifier was used for the phototransistor. Below is the phototransistor circuit with Vout being the input to the amplifier.


Since we wanted to analyze signals from two sources, we had to switch which bin we were reading data from on the arduino each time we ran the FFT. Both the audio and IR came from the mux output, so we needed to select the bin to read from. If we were in the initial spinlock, a global variable was set to read from the audio bin. Once we left this spinlock, the global variable indicated to read from the IR bin since after the initial tone, the robot never needs to listen to the audio signal. To determine if we are receiving an audio or IR signal, we checked which pin we were reading from, then the corresponding bin for that signal, and if the value was above the threshold, then we were receiving a signal. Similarly, if the value was below the the threshold then we can conclude we were not receiving a signal. 

Here is a picture of the phototransistor and amplifier circuit. This circuit was placed on the top of the robot. The orange wire is the amplified output signal sent to the mux.


