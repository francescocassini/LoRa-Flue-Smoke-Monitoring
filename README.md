# LoRa Flue Smoke Gas Monitoring System
### IOT Exam Project - La Sapienza  Rome - Spring 2019 - prof. Ioannis Chatzigiannakis 

| Contributors: | email |
| --- | --- |
| Francesco Cassini | cassini.785771@studenti.uniroma1.it |
| Lorenzo Leschiera | ginesi.1832198@studenti.uniroma1.it |
| Matteo Ginesi     | leschiera.1325697@studenti.uniroma1.it|

## The Story
LoRa Flue Smoke Gas Monitoring Repository contains all file to config a LoRa device, The Things Network and create an Android application. We have developed and studied a system to track the smoke level in a flue and send data by a LoRa transmitter. This could be useful for very aspects:

private telemetry og boiler smart city certification of flue (every year in Italy we spend 70 euro to do this check) scientific scope because the pollution from flue are the 60% of the total pollution.

## Resources
- [Medium](https://medium.com/@francesco.cassini/lora-flue-gas-monitoring-system-27dc85aeaf66): with all descriptions of the story
- [Hackster](https://www.hackster.io/169782/lora-flue-gas-monitoring-system-e51259): with other resource files
- [Youtube playlist](https://www.youtube.com/watch?v=AftJ-V92vwQ&list=PLAr1WaZbNX6d71oqCRcAHnkAAhue8hiqc): a list of walkthrough config videos
- [ArmBed](cassini.785771@studenti.uniroma1.it): repository codes for ArmMbed cpu
- [Slideshare](cassini.785771@studenti.uniroma1.it): slides of the presentation
- [AMI 2019](cassini.785771@studenti.uniroma1.it): we have showed a paper at this conference in October 2019


## README explanation
This README contains a little guide to process of setup of the several implied devices.
Anyway we invite you to read the complete story on Medium or Hackster and get here only the codes.
 
The workflow of LoRa Flue network is the following:
#### 1) Dragino LoRaWAN Gateway configuration
At beginning of this you find instructions to connect Dragino with Internet and setup it to be an IOT gateway for every near sensors
#### 2) LoRaWan on Mbed 
Put the code on the LoRa transmitter device to use the gas sensor and execute a sequence of measures and transmissions.
#### 3) The Things Network Decoder code 
 On The Things Network you will need a decoder code for the entrance data
#### 4) AWS IOT  exaplanation
AWS IOT chapter
#### 5) Android code exaplanation
Android code application

## 1) Dragino LoRaWAN Gateway configuration

If pre-existent data are present, please reset dragino:

- Press toggle button while dragino is running for about 30 seconds.

Connect to dragino access point with any device (pc suggested).
Open on your browser at link: [10.130.1.1](10.130.1.1) or via url: *dragino-`<same code of WiFi Access Point>`*

| Login | |
| --- | --- |
| user: | root |
| pass: | dragino |

### dragino network settings

- Network > *Internet Access*
  - Select WiFi connection
  - Set WiFi password and SSID
  - Set a page for ping test (like www.google.it)
  - **Save**
- Network > *Access Point*
  - Disable WiFi AP
  - **Save**
- Unsaved changes
  - **Save & Apply**
  - Wait for about 2 minutes and reboot.

### Setting up WLAN connection

- Network > *Internet Access*
  - Select WLAN connection
- Unsaved changes
  - **Save & Apply**
  - Wait for about 2 minutes and reboot.

### Setting up TTN data

- Open [TheThingsNetwork](thethingsnetwork.org) and go to Gateways settings.
- Register a new Gateway with **legacy packed forwarder**
- Copy the address on the back of the dragino and add a number like between (00 00) and (FF FF).
- Add a description.
- Set *Frequency Plan*
- Register your gateway.

Open on your browser dragino IP (based on your network).

- Sensor > LoraWAN
  - Server address: (example) router.eu.thethings.network
  - **Save & Apply**
- Sensor > IOT Server
  - IoT Server: LoRaWAN
  - **Save & Apply**

## Firmware

### 2) LoRaWan on Mbed

All the comunication system is based on LoRaWAN stack protocol managed by STM32 microcontroller board `DISCO-L072CZ`, programmed by `Mbed` (www.mbed.com). Firmware uses `Mbed OS 5` to provide drivers and pre-built code for:

- LoRaWAN radio module: SX1276
- Analog Input/Output
- Digital GPIOs
- Debug and tracing
- RS232 over USB stack

Code is divided in two main *blocks*:

 1. System initialization: `main.cpp`
 2. mbed-os configuration (LoRaWAN login): `mbed_app.json`

## System initialization

In the `main.cpp` file, there are all the functions/declarations needed to deploy the **send data function**.
The user has to change the `ID` constant in the file, as a hexadecimal value of 16 bit (`uint16_t`):

``` c++
...
    #define ID              0xAAFF
...
```

Sensor, for the board used in the project, is addressed on pin analog 1 (`A1`). Change it in case of need. Pin `A0` can't be used.

```c++
...
    AnalogIn sensor(A1);
...
```

In the `send_data()` function the user can set any kind of encoding of transmitted data, writing the correct message into the `tx_buffer` and settings the correct dimension value of it, into `packet_len` variable.

```cpp
    static void send_message()
    {
        uint16_t packet_len;
        int16_t retcode;

        /*
            Reading sensor data (unsigned int 16 bit)
            Writing ID and sensor data into transmission buffer
            Sending data to Gateway by LoRaWAN stack
        */
        uint16_t data = sensor.read_u16();
        packet_len = sprintf(
            (char *) tx_buffer, "%4X%u", ID, data);

        printf("\r\ndata sent: %s\r\n", tx_buffer);
    }
```

## LoRaWAN login configuration

In the `mbed_app.json` file, each user **NEED TO CONFIGURE** their own *TheTingsNetwork* credential to their proper application.

Note:
> This code works only for `APB` protocol. The `OOTA` way isn't verificated yet.
> For all information about *TheTingsNetwork* procedures, please refer to www.thethingsnetwork.com.

In order to change radio module, set the proper value in:

```json
...
    "config": {
        "lora-radio": {
            "help": "Which radio to use (options: SX1272,SX1276)",
            "value": "SX1276"
        },
```

For *TheThingsNetwork* credentials, change:

```json
...
    "target_overrides": {
        "*": {

            "lora.appskey": "{C-STYLE HEX App Session Key}",
            "lora.nwkskey": "{C-STYLE HEX Network Session Key}",
            "lora.device-address": "0x<HEX DATA OF device address>"
```
