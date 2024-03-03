# Real-life Assembler

This chapter shows how I merged Computer Vision and Arduino programming with DIY electronics and soldering. The last part was made as a part of a workshop led by Tin Dužić (link neke stranice). 


## Amplifier
This part is practically identical to the sound detection project I made a few folders ago. The only differences are at the ends of the circuit: mic is replaced with an audio jack, while Arduino is replaced with a speaker.
All capacitors are primarily used to reduce the noise of the signal, while C2 is used to increase the amplification of LM386.

(SLIKA STRUJNOG KRUGA)

To connect the speaker with the rest of the project I used a 3.5mm Jack cable by cutting it in half and removing isolation around the + and ground channels. The tip of the jack is also fitted with a 6mm Jack converter.
The + and ground wires are connected to pin 7 of SN76489 and common ground respectively.

(SLIKA SVEGA SPOJENOG)

 

## (Monty) Python sketch

To finally break silence of this project, it is necessary to break the ice between Arduino and Python. This is made possible by bridging the gap from both sides.
* [pymata4]((https://mryslab.github.io/pymata4/)): a Python 3 compatible Firmata protocol which enables the user to control Arduino with the power of the most accessible programming language out there.
	* a high-performance multi-threaded Python application. Its "command thread" translates user API calls into Firmata protocol messages and forwards these messages to the Arduino microcontroller.
	* the "reporter thread" receives, interprets and acts upon the Firmata messages received from the Arduino microcontroller.
* [FirmataPlus](https://github.com/acen2009/FirmataPlus): a generic protocol for communicating with microcontrollers from software on a host computer
	* it uses serial interface to transport both command and report information between an Arduino microcontroller and a PC, typically with a serial/USB link set to a rate of 57600 bps.
	* essentially an upgraded version of Firmata enabling tone commands among others.

In this project, the PC using PyMata library serves as a client to the server which is uploaded to Arduino in form of FirmataPlus.
Firmata uses a serial communications interface to transport data to and from the Arduino. The Firmata communications protocol is derived from the MIDI protocol which uses one or more 7 bit bytes to represent data.
Because 7 bits can hold a maximum value of 128, a data item that has a value greater than 128, must be disassembled into multiple 7 bit byte chunks before being marshaled across the data link.
By convention, the least significant byte (LSB) of a data item is sent first, followed by increasingly significant components of the data item.
The most significant byte (MSB) of the data item is the last data item sent.

## The Code

This project requires opencv and pymata4 libraries. The HandTrackingModule discussed in the Computer Vision Chapter is also included.
Some of the functions replicate behavior of functions written in Arduino sketch. Back there, delay() function was used to synchronize signals.
The same effect can be achieved using time module's sleep() method.

```python

import cv2
import time
import HandTrackingModule as htm
from pymata4 import pymata4

``` 

Next step is setting up connetion with the server on the Arduino and declaring output pins. Variables for certain frequencies are also defined:

```python

D0 = 2
D1 = 3
D2 = 4
D3 = 5
D4 = 6
D5 = 7
D6 = 8
D7 = 9
WE = 10

pins = [D0, D1, D2, D3, D4, D5, D6, D7, WE]

for i in range(len(pins)):
	board.set_pin_mode_digital_output(pins[i])

board.digital_write(WE, 1)
	
c3 = 128
d3 = 147
e3 = 165
f3 = 175
g3 = 196
a3 = 220
b3 = 247
c4 = 2 * c3
d4 = 2 * d3
e4 = 2 * e3
f4 = 2 * f3
g4 = 2 * g3 

freqs = [c3, d3, e3, f3, g3, a3, b3, c4, d4, e4, f4, g4]

lnh = [[0]*2 for i in range(len(freqs))] 

for i in range(len(freqs)):
	data = int(4*10**6/(32*freqs[i]))
	lnh [i][0] = data & 0xF
	lnh [i][1] = (data & 0x3F0) >> 4

note = [[0]*2 for i in range(3)]

```

As described in the SN76489 segment, the frequency is sent with the number calculated with the formula. This number is then masked with 0xF to isolate 4 least significant bytes, while the remaining 6 bits are determined using 0x3F0 mask and arithetic right shift.
The lower and higher bits will be sent to the SN76489 with latched and data bits later in the code. A list which will contain certain frequencies based on position of the hand is also initialized.

This segment defines resolution of the camera, hand detector module and list containing fingertip IDs:    

```python

wCam, hCam = 640, 480

cap = cv2.VideoCapture(0)
cap.set(3, wCam)
cap.set(4, hCam)

detector = htm.handDetector(detectionCon=0.75)
tipIds = [4, 8, 12, 16, 20]

```

The three functions shown in the segment below perform identical task as the functions from the sketch, along with added functionalities which compensate for the different data interpretation between Python and C++.
Aside from ternar operator used in the Arduino sketch, the main difference is the if statment
C++ automatically sends data in the form of a byte, which means all of the pins automatically receive the necessary data on the pins. Python does not work in this way and excludes significant bits if their value is zero.
To compensate for this, the putByte functions checks if data needs additional padding: if the value sent is less then 128, the original value is padded with two additional zeros. Otherwise, original bits are sent. This condition is necessary to send data bytes properly.
Since Data bytes contain values written with only 6 bytes, their value does not exceed 128 and that is the key difference between them and the latched bytes: having 1 as their most significant bit makes their value at least equal to 128.

Sendbyte works exactly like its Arduino counterpart, while SilenceChannel is a slightly modified version and enables muting of as many channels as the user wants, as long as this number is 3.

```python

def putByte(a):

    board.digital_write(D0, 1 if (a & 1) else 0)
    board.digital_write(D1, 1 if (a & 2) else 0)
    board.digital_write(D2, 1 if (a & 4) else 0)
    board.digital_write(D3, 1 if (a & 8) else 0)
    board.digital_write(D4, 1 if (a & 16) else 0)
    board.digital_write(D5, 1 if (a & 32) else 0)

    if(a < 128):
        board.digital_write(D6, 0)
        board.digital_write(D7, 0)
    else:
        board.digital_write(D6, 1 if (a & 64) else 0)
        board.digital_write(D7, 1 if (a & 128) else 0)


def sendByte(b):
    board.digital_write(WE, 1)
    putByte(b)
    board.digital_write(WE, 0)
    time.sleep(0.001)
    board.digital_write(WE, 1)

def silenceChannel(n):
    if n == 0:
        sendByte(0x9F)
    elif n == 1:
        sendByte(0xBF)
    elif n == 2:
        sendByte(0xDF)

def silenceAllChannels():
    sendByte(0x9F)
    sendByte(0xBF)
    sendByte(0xDF)
    sendByte(0xFF)
    
```

The meat of this project is the endless while loop shown below. It can be summarized like this:
* read data from camera and initialize hand tracking algorithm to recognize one hand. Store results of detection in a list containing hand landmark indexes
* if a hand is detected, for each individual finger check if its raised up or folded down. Append 1 to the fingers list if raised, otherwise append 0.
* If condition for thumb compares x coordinates, while the rest of the fingers are determined with y coordinate
* depending on the position of the hand, choose the apropriate frequency. Lower notes are played for lower position
* if the fingers list is not empty, send bytes of chosen frequency to the SN76489 soundchip. If the finger is closed, mute the corresponding channel
* display the result on the camera using imshow

```python

while 1:
    success, img = cap.read()
    img = detector.findHands(img)										# start detection
    lmList1 = detector.findPosition(img, 0, draw=False)					# store landmark position in lmList1 

    fingers = []
    if lmList1:															# if a hand is detected, determine if each finger is raised or folded 

        if lmList1[tipIds[0]][1] > lmList1[tipIds[0] - 1][1]:
            fingers.append(1)
        else:
            fingers.append(0)

        for id in range(1, 5):
            if lmList1[tipIds[id]][2] < lmList1[tipIds[id] - 2][2]:
                fingers.append(1)
            else:
                fingers.append(0)

        if lmList1[13][2] < 60:											# change contents of list called note depending on the position of detected hand
            note = [[c4l, c4h], [e4l, e4h], [g4l, g4h]]
        elif lmList1[13][2] < 120 and lmList1[13][2] > 60:
            note = [[b3l, b3h], [d4l, d4h], [f4l, f4h]]
        elif lmList1[13][2] < 180 and lmList1[13][2] > 120:
                note = [[a3l, a3h], [c4l, c4h], [e4l, e4h]]
        elif lmList1[13][2] < 240 and lmList1[13][2] > 180:
                note = [[g3l, g3h], [b3l, b3h], [d4l, d4h]]
        elif lmList1[13][2] < 300 and lmList1[13][2] > 240:
                note = [[f3l, f3h], [a3l, a3h], [c4l, c4h]]
        elif lmList1[13][2] < 360 and lmList1[13][2] > 300:
                note = [[e3l, e3h], [g3l, g3h], [b3l, b3h]]
        elif lmList1[13][2] < 420 and lmList1[13][2] > 360:
                note = [[d3l, d3h], [f3l, f3h], [a3l, a3h]]
        elif lmList1[13][2] < 480 and lmList1[13][2] > 420:
                note = [[c3l, c3h], [e3l, e3h], [g3l, g3h]]
				
        #print(fingers)
        print(lmList1[13][2])

    if fingers:															# send data to SN76489 based on position of fingers

        if fingers[0]:
            sendByte(0x90)
            sendByte(8 * 16 + note[0][0])
            sendByte(note[0][1])
        else:
            silenceChannel(0)

        if fingers[1]:
            sendByte(0xB0)
            sendByte(10 * 16 + note[1][0])
            sendByte(note[1][1])
        else:
            silenceChannel(1)

        if fingers[4]:
            sendByte(0xD0)
            sendByte(12 * 16 + note[2][0])
            sendByte(note[2][1])
        else:
            silenceChannel(2)

    cv2.imshow("img", img)												# display results
    cv2.waitKey(1)

```
