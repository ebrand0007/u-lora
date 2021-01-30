# u-lora

This is a port of raspi-lora (https://pypi.org/project/raspi-lora/) for micropython.  I have currently only tested on raspberry pi pico.  It allows your microcontroller to use an RFM95 to communicate.

## Wiring

The pinout for the RFM95 module can be found on page 10 of teh documentation (https://cdn.sparkfun.com/assets/learn_tutorials/8/0/4/RFM95_96_97_98W.pdf).

The RFM95 module requires 3.3V and GND from your microcontroller:  
+ connect 3.3V to pin 13  
+ connect GND to pin 1, 8, or 10 on the RFM95 module

For SPI communication (pin numbers are for the RFM95 - look at your microcontroller for the pins to connect):  
+ MISO to pin 2 (MISO)  
+ MOSI to pin 3 (MOSI)  
+ SCK to pin 4 (SCK)  
+ CS to pin 5 (NSS)  
    
Other pins:  
+ Use a GPIO output to pin 6 (RESET) for resetting the RFM95  
+ Use a GPIO input to pin 14 (D) to trigger that a message has been received  

## Configuration
**INITIALIZATION**
```
LoRa(channel, interrupt, this_address, cs_pin, reset_pin=None, freq=868, tx_power=14,
    modem_config=ModemConfig.Bw125Cr45Sf128, acks=False, crypto=None)
```

**channel** SPI channel to use (either 0 or 1, if your LoRa radio is connected to CE0 or CE1, respectively)  
**interrupt** GPIO pin to use for the interrupt  
**this_address** The address number (0-254) your device will use when sending and receiving packets.  
**cs_pin** chip select pin from microcontroller  
**reset_pin** : the GPIO used to reset the RFM9x if connected  
**freq** Frequency used by your LoRa radio. Defaults to 868Mhz  
**tx_power** Transmission power level from 5 to 23. Keep this as low as possible. Defaults to 14  
**model_config** Modem configuration. See RadioHead docs. Default to Bw125Cr45Sf128.  
**receive_all** Receive messages regardless of the destination address  
**acks** If True, send an acknowledgment packet when a message is received and wait for an acknowledgment when transmitting a message. This is equivalent to using RadioHead's RHReliableDatagram  
**crypto** An instance of PyCryptodome Cipher.AES (not tested) - should be able to use ucrypto  

## Examples
There are two examples to test sending and receiving data in the examples folder

### Server mode:

Copy the file server.py to your main.py and copy it across together with the library ulora.py to your microcontroller

```
from time import sleep
from ulora import LoRa, ModemConfig

# This is our callback function that runs when a message is received
def on_recv(payload):
    print("From:", payload.header_from)
    print("Received:", payload.message)
    print("RSSI: {}; SNR: {}".format(payload.rssi, payload.snr))

# Lora Parameters (this is on the raspberry pi pico)
RFM95_RST = 27 # wire gpio 27 to RST
RFM95_CS = 0 # use the spi0 (pins 4 (MOSI), 5 (CS), 6 (SCK), 7 (MISO))
RFM95_INT = 28 # wire gpio 27 to D0
RF95_FREQ = 868.0 # RF for Europe
RF95_POW = 20
SERVER_ADDRESS = 2

# initialise radio
lora = LoRa(RFM95_CS, RFM95_INT, SERVER_ADDRESS, reset_pin=RFM95_RST, freq=RF95_FREQ, tx_power=RF95_POW, acks=True)

# set callback
lora.on_recv = on_recv

# set to listen continuously
lora.set_mode_rx()

# loop and wait for data
while True:
    sleep(0.1)
```

### Client mode:
Copy the file server.py to your main.py and copy it across together with the library ulora.py to your microcontroller

```
from time import sleep
from ulora import LoRa, ModemConfig

# Lora Parameters
RFM95_RST = 27
RFM95_CS = 0
RFM95_INT = 28
RF95_FREQ = 868.0
RF95_POW = 20
CLIENT_ADDRESS = 1
SERVER_ADDRESS = 2

# initialise radio
lora = LoRa(RFM95_CS, RFM95_INT, CLIENT_ADDRESS, reset_pin=RFM95_RST, freq=RF95_FREQ, tx_power=RF95_POW, acks=True)

# loop and send data
while True:
    lora.send_to_wait("This is a test message", SERVER_ADDRESS)
    print("sent")
    sleep(10)
```
