# Introduction

Spring of 2022. The terror of exam weeks has ended just a few days ago. Best time of the year to replace the nervous silence of classroom with tinitus caused by the roaring instruments of unknown local band.
Still unable to forget technical terms used in exams, my friend and I enter through the door and decide to soak in the atmosphere of the place in which a rehersal takes place. One of us notices the struggle a sound engineer has to endure to get the sound of the instruments just right. So many buttons, so little time. Imagine if he could wave his hands like a conductor of a grand orchestra to make things funnier and less efficient. That would actually be funny.

It's almost Spring of 2024. My post-concert tinitus is replaced with deafening silence of desk job at an office. There is still one loud sound occupying my head, which says: "I FINALLY DID THE FUNNY CONDUCTOR THING".

This is my original project. I am very proud to say that I haven't found anything similar on the web nor relied on other peoples' concepts, and as such this project allows me to present my personality and way of thinking. It took long hours of unguided thinking, imagining and waiting to set every detail right. If you find something similar, it's probably a project of the guy I was talking to back in 2022 and he ripped my idea.

# The Code

The basic premise of this project was to control something by hand gestures. Years of SF movies are responsible for this. 
The easiest way to implement this kind of interaction is by using Google's framework mediapipe created primarily for Python.
Mediapipe contains a lot of computer vision models for all kinds of purposes and one of them is Hand Tracking. Using two modules at the backend (Palm Detection and Hand Landmarks),
the Hand Tracking crops the given image so that only one hand is visible, adding landmarks afterwards.
This is useful for triggering events based on the position of hands on the screen, as well as relative position of fingers.

![Hand landmarks.](hand-landmarks.png "Hand landmarks.")

The easiest way to implement mediapipe is by using PyCharm IDE, which enables a simple installation of all available packages with a few clicks. This necessitates usage of OpenCV library as well.
After typing the basic lines of code to display the webcam, Hands module must be implemented.
It requires four parameters: static_image_mode (default: False), max_num_hands (default: 2), min_detection_confidence (default: 0.5) and min_tracking_confidence (0.5).
The first parameter determines wheter the module detects or tracks hands on the image. This choice depends on the tracking confidence level:
low level equals detection, high level equals tracking. Thresholds for detection and tracking are set up with min_detection_confidence and min_tracking_confidence.
The code snippet below displays detected landmarks:

```python

import cv2
import mediapipe as mp
import time

cap = cv2.VideoCapture(0)

mpHands = mp.solutions.hands
hands = mpHands.Hands()		# static_image_mode (default: False), max_num_hands (default: 2), min_detection_confidence (default: 0.5) and min_tracking_confidence (0.5)
mpDraw = mp.solutions.drawing_utils

while True:
	success, img = cap.read()
	imgRGB = cv2.cvtColor(img, cv2.COLOR_BG2RGB)	# mediapipe requires RGB image as input
	results = hands.proces(imgRGB)

	if results.multi_hand_landmarks:	# if landmark is detected, draw each landmark as well as connections between them 
		for handLms results.multi_hand_landmarks:
			for id, lm in enumerate (handLms.landmark):	# calculate position of each landmark using size of camera
				h, w, c = img.shape
				cx, cy = int(lm.x * w),  int(lm.y * h)

			mpDraw.draw_landmarks(img, handLms, mpHands.HAND_CONNECTIONS)
			
 
     	cv2.imshow("Screen", img)
	cv2.waitKey(1)
```
To make this code accessible, this simple snippets needs to be written in a form of a class. After proper rearrangement, the code now looks like this:

```python

import cv2
import mediapipe as mp
import time

class handDetector():
	def __init__(self, mode = False, maxHands = 2, detectionCon = 0.5, trackCon = 0.5):
		self.mode = mode
		self.maxHands = maxHands
		self.detectionCon = detectionCon
		self.trackCon = trackCon
		
		self.mpHands = mp.solutions.hands
		self.hands = self.mpHands.Hands()
		self.mpDraw = mp.solutions.drawing_utils
		
	def findHands(self, img, draw = True):													
		imgRGB = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)										# mediapipe works only with RGB images
		self.results = hands.process(imgRGB)												# algorithm implementation
	
		if results.multi_hand_landmarks:													# iterate through detected landmarks
			for handLms in self.results.multi_hand_landmarks:
				if draw:			
					self.mpDraw.draw_landmarks(img, handLms, mpHands.HANDS_CONNECTIONS)		# displays original BGR image with the handmark layer
				
		return img
		
	def findPosition(self, img, HandNo=0, draw = True):
	
		lmList = []
		
		if self.results.multi_hand_landmarks:
		
			myHand = self.results.multi_hand_landmarks[handNo]
		
			for id, lm in enumerate(handLms.landmark):
				h, w, c = img.shape()
				cx, cy = int(lm.x * w), int(lm.y * h)										# calculating x and y coordinates of each landmark using image dimensions
				lmList.append([id, cx, cy])

		return lmList
		

	
	
def main():
	cap = cv2.VideoCapture(0)
	detector = handDetector()
	
	while True:
		success, img = cap.read()
		img = detector.findHands(img)
		lmList = detector.findPosition(img)
		
		cv2.imshow("Screen", img)
		cv2.waitKey(1)
		
if __name__ == "__main__":
	main()
	
```

By this point the code doesn't do much except drawing  a very crude outline of hands. To put Google's research to good use, we need utilize its function.
One of the most basic signals we humans can do with our hands is pointing fingers. Another well known signal we humans tend to produce is a closed fist, often times representing something bad is about to happen to the observer.
In light of these observations I decided to intepret raised finger as a positive and lowered (folded) finger as negative. Consulting the landmark diagram, it is obvious that the y coordinate of the tip of the finger is positioned higher than the other parts of the finger. The opposite is true for the folded finger: the tip is positioned lower than other parts. By comparing coordinates of the tip with some other part of the same finger it is possible to control the value of variables using fingers as input. In this project each finger is represented by a variable with two possible states: raised (1) or lowered (0). These variables are then stored into a list for improved readability.
This thought process is realized with the following code:

```python

import cv2
import time
import HandTrackingModule as htm

detector = htm.handDetector()
tipIDs = [4, 8, 12, 16, 20]

while 1:
	success, img = cap.read()
	img = detector.finHands(img)
	lmList = detector.findPosition(img, draw = False)
	
	if len(lmList) != 0:
		fingers = []

   		if lmList[tipIDs[0]][1] > lmList[tipIDs[0]-1][1]:
   			fingers.append(1)
   		else:
   			fingers.append(0)
   			
   		for id in range(1,5):	#iterate through rest of fingers
   			if lmList[tipIDs[id]][2] < lmList[tipIDs[id]-2][2]:
   				fingers.append(1)
   			else:
   				fingers.append(0)

```

# PRELISTAJ NA 12:20

#SN76489

Barely larger than a raisin, this chip contains a whole decade of sound design and still refuses to leave in silence. It was made by Texas Instruments in 1979 and for the next 10 years it sat comfortably in cases of many home computers and arcade machines. This chip is a Digital Complex Sound Generator (DSCG) with 3 square wave tone generators and one noise generator. The three tone generators were used for melodies, while the noise channel found its use in simulating sounds of explosions or percussion.

## Pins
![SN76489 pins.](SN76489_pinout.png "SN76489 pins")

As shown in the image, the pins can be divided into these categories:

* input pins (D0-D7)
* audio out (pin 7)
* clk (pin 14)
* /WE (active low write enable, pin 5)
* VCC (pin 15)
* /CE (active low chip enable, pin 6)
* GND (pin 8)
* RDY (unused)

## Sending bytes

The frequency of the square waves produced by the tone generators on each channel is derived from two factors:
* the speed of the crystal oscillator
* value of desired frequency

or in form of equation:

![fraction.](freqfrac.png "Formula")

where f is the desired frequency. This number must be translated to binary form.

The chip receives input from 8 digital pins, as well as from a 4 MHz crystal oscillator which powers its IC.
A microprocessor programs the IC by sending bytes of data from pins D0 - D7.
The bytes which can be sent to the chip are one of the two - either latched or data byte. 
* Latched bytes: Four significant bits carry the information about the desired channel, while the lower four bits store the first four bits of the number calculated with the formula above
* Data bytes: 2 most significant bits are 0, while the rest contain last 6 bits of calculated number.

Once a latched byte is sent to the register, all subsequent data bytes will be applied to that register until another latch byte is sent.
 
Image below shows all possible values of bytes that can be interpreted by the microprocessor.

![bytes.](bytes.png "Bytes")

The data sheet of SN76489 provides a diagram which shows how to send bytes.
The basic process with this chip is that you set up a byte on the eight data pins, then briefly pulse the /WE pin low to tell the chip that the byte is ready.

![flow.](flow.png "Flow")




## COMPOSER OR COMPILER?

In contrast to everything mentioned above, coding proves to be quite simple. All it takes is three functions.
Pins are assigned in the setup() function with pinMode() and data is sent with digitalWrite(). Sending bytes works the same way as sending one bit, only difference being that the same function must be called 8 times for every bit in the byte.
My solution to this was checking the value of each bit in a for loop. Checking is performed with *bitwise and operator* and a variable called *bit*, which is shifted at the end of every loop to isolate and verify the value of each bit.
It ends up looking like this:

(SendByte funkcija)

```c

void putByte(byte b)
{
	int i = 1;
	int pins = {D0, D1, D2, D3, D4, D5, D6, D7};
	int bit = 1; 

	for(i; i < 8; i++)
	{
		digitalWrite(pins[i], (b&bit) ? HIGH, LOW);
		bit *= 2;
	}
}
```
Sending pulse to WE pin:

```c

void sendByte(byte b)
{
	digitalWrite(WE, HIGH);
	putByte(b);
	digitalWrite(WE, LOW);
	delay(1);
	digitalWrite(WE, HIGH);
}

```

Many music teachers will tell you that melodies sound good precisely becuse of the silence between the notes. So to ascend to the higher level of musical prowess a silencing function is added:
It basically sends latched bytes with address and 4-bit data regarding volume: higher number means lower volume. Hence, total silence equals 1111 or F if you want to pay respects.

```c

void SilenceAllChannels()
{
	SendByte(0x9F); //mute channel 0
	SendByte(0xBF);	//mute channel 1
	SendByte(0xDF);	//mute channel 2
	SendByte(0xFF);	//mute noise channel
}

```

Here is a simple Arduino sketch playing notes A, B and C:

(STAVI OBRAZLOŽENJE, KAKO SE DOBIJE BAJT, KAKO I ZAŠTO IZGLEDA LATCHED I DATA BYTE)
In this example, I decided to continuously play C3 (128 Hz), E3 (165 Hz) and G3 (196 Hz).

### C3
* I increase the volume of channel 0 to maximum by sending 0x90
* using the formula for the frequency I got 919 or 1110010111 (result must be stored as an int variable)
* since I want to use channel 0 for this tone, first four bits are 1000 (0x8) while 4 least significant bits are 0111 (0x3).
* the remaining 6 are 111001. When padded with additional two zeros to form a data byte, I got 00111001 (0x39).
* this yields 0x83 as a latched and 0x39 as a data byte 

### E3
* I increase the volume of channel 1 to maximum by sending 0xB0
* Using the frequnecy formula I got 757 or 1011110101: 4 least significant bits form 0x5 while the remaining 6 padded with additional two zeros at the end form 0x2F 
* I want to play this tone at channel 1 (0xA), so my latched and data bytes are 0xA5 and 0x2F

### G3
* I increase the volume of channel 2 to maximum by sending 0xD0
* Using the frequnecy formula I got 637 or 1001111101: 4 least significant bits form 0xD while the remaining 6 padded with additional two zeros at the end form 0x27
* I want to play this tone at channel 2 (0xC), so my latched and data bytes are 0xCD and 0x27

This is how the Arduino sketch version of this thought process looks like:
(OVDJE IDE KOD)



# (Monty) Python sketch

My Chimera is almost finished. This chapter shows how I merged Computer Vision and Arduino programming with DIY electronics and soldering. This last part was made as a part of a workshop led by Tin Dužić (link neke stranice). 


## Amplifier
This part is practically identical to the sound detection project I made a few folders ago. The only differences are at the ends of the circuit: mic is replaced with an audio jack, while Arduino is replaced with a speaker.
All capacitors are primarily used to reduce the noise of the signal, while C2 is used to increase the amplification of LM386.

(SLIKA STRUJNOG KRUGA)

To connect the speaker with the rest of the project I used a 3.5mm Jack cable by cutting it in half and removing isolation around the + and ground channels. The tip of the jack is also fitted with a 6mm Jack converter.
The + and ground wires are connected to pin 7 of SN76489 and common ground respectively.

(SLIKA SVEGA SPOJENOG)

 

## (Monty) Python sketch

To finally break the silence of this project, it is necessary to break the ice between Arduino and Python. This is made possible by bridging the gap from both sides.
* pymata4: a Python 3 compatible Firmata protocol which enables the user to control Arduino with the power of the most accessible programming language out there. (https://mryslab.github.io/pymata4/)
** a high-performance multi-threaded Python application. Its "command thread" translates user API calls into Firmata protocol messages and forwards these messages to the Arduino microcontroller.
** the "reporter thread" receives, interprets and acts upon the Firmata messages received from the Arduino microcontroller.
* FirmataPlus: a generic protocol for communicating with microcontrollers from software on a host computer
** it uses serial interface to transport both command and report information between an Arduino microcontroller and a PC, typically with a serial/USB link set to a rate of 57600 bps.
** essentially an upgraded version of Firmata enabling tone commands among others.

In this project, the PC using PyMata library serves as a client to the server which is uploaded to Arduino in form of FirmataPlus.
Firmata uses a serial communications interface to transport data to and from the Arduino. The Firmata communications protocol is derived from the MIDI protocol which uses one or more 7 bit bytes to represent data.
Because 7 bits can hold a maximum value of 128, a data item that has a value greater than 128, must be "disassembled" into multiple 7 bit byte chunks before being marshaled across the data link.
By convention, the least significant byte (LSB) of a data item is sent first, followed by increasingly significant components of the data item.
The most significant byte (MSB) of the data item is the last data item sent.

## The Code

This project requires opencv and pymata4 libraries. The HandTrackingModule discussed in the Computer Vision Chapter is also included.
Some of the functions replicate behavior of functions written in Arduino sketch. Back there, delay() function was used to synchronize signals. The same effect can be achieved using time module's sleep() method.

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

board.set_pin_mode_digital_output(D0)
board.set_pin_mode_digital_output(D1)
board.set_pin_mode_digital_output(D2)
board.set_pin_mode_digital_output(D3)
board.set_pin_mode_digital_output(D4)
board.set_pin_mode_digital_output(D5)
board.set_pin_mode_digital_output(D6)
board.set_pin_mode_digital_output(D7)
board.set_pin_mode_digital_output(WE)
board.digital_write(WE, 1)

c3 = int(4*10**6/(32*128))
c3l = c3 & 0xF							
c3h = (c3 & 0x3F0) >> 4

d3 = int(4*10**6/(32*147))
d3l = d3 & 0xF
d3h = (d3 & 0x3F0) >> 4

e3 = int(4*10**6/(32*165))
e3l = e3 & 0xF
e3h = (e3 & 0x3F0) >> 4

f3 = int(4*10**6/(32*175))
f3l = f3 & 0xF
f3h = (f3 & 0x3F0) >> 4

g3 = int(4*10**6/(32*196))
g3l = g3 & 0xF
g3h = (g3 & 0x3F0) >> 4

a3 = int(4*10**6/(32*220))
a3l = a3 & 0xF
a3h = (a3 & 0x3F0) >> 4

note = []

```

The lower set of variables contain bits of data ready to be sent to the SN76489 chip. As described in the previous chapter, the frequency is sent with the number calculated with the formula. This number is then masked with 0xF to isolate 4 least significant bytes, while the remaining 6 bits are determined using 0x3F0 mask and arithetic right shift.
The lower and higher bits will be sent to the SN76489 with latched and data bits later in the code. A list which will contain certain frequencies based on position of the hand is also introduced

This segment defines resolution of the camera and introduces, hand detector module and list containing fingertip IDs:    

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

def silenceAllChannels(n):
    if n == 0:
        sendByte(0x9F)
    elif n == 1:
        sendByte(0xBF)
    elif n == 2:
        sendByte(0xDF)

```
(IZBRIŠI ZADNJI RED ILI SKRATI OPĆENITO)

## Headers

# This is a Heading h1
## This is a Heading h2


```c
void setup() {

  Serial.begin(9600);
}

void loop() {
  //int a0 = analogRead(A0);
  int a1 = analogRead(A1);
  Serial.print(0);
  //Serial.print(",");
  //Serial.println(a0);
  Serial.print(",");
  Serial.println(a1);
  delay(1);
}
```
###### This is a Heading h6

## Emphasis

*This text will be italic*  
_This will also be italic_

**This text will be bold**  
__This will also be bold__

_You **can** combine them_

## Lists

### Unordered

* Item 1
* Item 2
* Item 2a
* Item 2b

### Ordered

1. Item 1
2. Item 2
3. Item 3
    1. Item 3a
    2. Item 3b

## Images

![This is an alt text.](val12.png "Live circuit reaction.")

## Summary

> **What is this?**
>
>> Sound Detection circuit.
	
> **Is it useful?**
>
>> No, not by itself.
	
> **Why would I want do build one?**
>
>> You can make many other projects on top of this one to impress people coming from LinkedIn. Go and check my other projects on this repo to become inspired!    

## Links

You may be using [Markdown Live Preview](https://markdownlivepreview.com/).

## Blockquotes

> Markdown is a lightweight markup language with plain-text-formatting syntax, created in 2004 by John Gruber with Aaron Swartz.
>
>> Markdown is often used to format readme files, for writing messages in online discussion forums, and to create rich text using a plain text editor.

## Tables

| Left columns  | Right columns |
| ------------- |:-------------:|
| left foo      | right foo     |
| left bar      | right bar     |
| left baz      | right baz     |

## Blocks of code

```
let message = 'Hello world';
alert(message);
```

## Inline code

This web site is using `markedjs/marked`.

