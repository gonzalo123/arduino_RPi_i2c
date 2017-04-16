Arduino and Raspberry Pi working together (with i2c)
======

The most easy way to connect our Arduino board to our Raspberry Py is using the USB cable, but sometimes this communication is a nightmare, especially because there isn't any clock signal to synchronize our devices and we must rely on bitrate. There're different ways to connect our Arduino and our Raspberry Py such as I2C, SPI and serial over GPIO. Today we're going to speak about I2C, especially because it's pretty straightforward if we take care with a couple of things. Let's start.

I2C uses two lines SDA(data) and SCL(clock), in addition to GND (ground). SDA is bidirectional so we need to ensure, in one way or another, who is sending data (master or slave). With I2C only master can start communications and also master controls the clock signal. Each device has a 7bit direction so we can connect 128 devices to the same bus.

If we want to connect Arduino board and Raspberry py we must ensure that Raspberry pi is the master. That's because Arduino works with 5V and Raspberry py with 3.3V. That means that whe need to use resistors if we don't want destroy our Raspberry pi. But Raspberry pi has 1k8 ohms resistors to the 3.3 votl power rail, so we can connect both devices (if we connect other i2c devices to the bus they must have their pull-up resistors removed)

Thats all we need to connect our Raspberry pi to our Arduino board.
* RPi SDA to Arduino analog 4
* RPi SCL to Arduino analog 5 
* RPi GND to Arduino GND

Now we are going to build a simple prototype. Raspberry pi will blink one led (GPIO17) each second and also will send a message (via I2C) to Arduino to blink another led (controlled by Arduino). That's the Python part

```python
import RPi.GPIO as gpio
import smbus
import time
import sys

bus = smbus.SMBus(1)
address = 0x04

def main():
    gpio.setmode(gpio.BCM)
    gpio.setup(17, gpio.OUT)
    status = False
    while 1:
        gpio.output(17, status)
        status = not status
        bus.write_byte(address, 1 if status else 0)
        print "Arduino answer to RPI:", bus.read_byte(address)
        time.sleep(1)


if __name__ == '__main__':
    try:
        main()
    except KeyboardInterrupt:
        print 'Interrupted'
        gpio.cleanup()
        sys.exit(0)

```

And finally the Arduino program. Arduino also answers to Raspberry pi with the value that it's been sent, and Raspberry Pi will log the answer within console

```c
#include <Wire.h>

#define SLAVE_ADDRESS 0x04
#define LED  13

int number = 0;

void setup() {
  pinMode(LED, OUTPUT);
  Serial.begin(9600);
  Wire.begin(SLAVE_ADDRESS);
  Wire.onReceive(receiveData);
  Wire.onRequest(sendData);

  Serial.println("Ready!");
}

void loop() {
  delay(100);
}

void receiveData(int byteCount) {
  Serial.print("receiveData");
  while (Wire.available()) {
    number = Wire.read();
    Serial.print("data received: ");
    Serial.println(number);

    if (number == 1) {
      Serial.println(" LED ON");
      digitalWrite(LED, HIGH);
    } else {
      Serial.println(" LED OFF");
      digitalWrite(LED, LOW);
    }
  }
}

void sendData() {
  Wire.write(number);
}
```

# Hardware:
* Arduino UNO (https://www.arduino.cc/en/Main/ArduinoBoardUno)
* Raspberry Pi
* Two LEDs and two resistors

# Demo
[![Arduino and Raspberry Pi via i2c](http://img.youtube.com/vi/EMunlSg77DA/0.jpg)](https://www.youtube.com/watch?v=EMunlSg77DA)
# References:
https://oscarliang.com/raspberry-pi-arduino-connected-i2c
http://www.electroensaimada.com/i2c.html
