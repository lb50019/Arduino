# Clap Switch

The easiest way to come up with something exciting and affordable is to think of technology people regarded as advanced 40 years ago. The first actual clap switch available to the public was invented exactly 40 years ago!
In comparison to the sound detection circuit, this one only requires an additional resistor and an LED diode, but also features a significant step-up in terms of coding complexity.
What I mean is the real meat of this project lies in signal processing. With this project Arduino finally doesn't work as just an overhyped battery, but as a microcomputer as was intended.

## Headers

# This is a Heading h1
## This is a Heading h2


![This is an alt text.](op_amp_srkt.png "Circuit diagram.")
![This is an alt text.](op_amp_srkt_tagged.png "Circuit mess.")

Capacitor C2 is added to further increase the amplification of the signal. This is more-or-less neccessary for any practical use.

Below is a simple code used to display the signal, along with a plotted output. Despite all the capacitors the noise is still present! The only likely reasons for this are imperfections of cheap components and the voices in my walls.

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

![This is an alt text.](ledo.png "LED diode circuit.")

## Summary

> **What is this?**
>
>> Sound Detection circuit.
