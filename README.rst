WIZnet WizFi 250
================

In this tutorial we'll explain how to send a value to Ubidots with the WizFi shield.

.. end_read_more_include

Introduction
^^^^^^^^^^^^
WizFi Shield (Fi250）is an economic internet solution for the Arduino. The Wi-Fi module supports IEEE 802.11b/g/n mode, with speeds up to 65Mbit/s.

Components
^^^^^^^^^^
*	`WIZnet WizFi 250 shield <http://www.seeedstudio.com/depot/Wifi-Shield-Fi250-p-1719.html>`_:

.. figure:: https://github.com/ubidots/ubidots-wiznet/blob/master/pictures/wifishield-Fi250_03.jpg
    :name: wizfi250
    :align: center
    :alt: wizfi250

* `Arduino uno <http://arduino.cc/en/Main/ArduinoBoardUno>`_

.. figure:: https://github.com/ubidots/ubidots-wiznet/blob/master/pictures/arduino-uno.png
    :name: arduino-uno
    :align: center
    :alt: arduino-uno

* Seedstudio's `Wizfi library <http://www.seeedstudio.com/wiki/File:Wizfi250.zip>`_

Install the Library
-------------------

1. Download the library as a ZIP file 

2. Open the Arduino IDE and go to "Sketch" --> "Import Library" --> "Add Library".

3. Select the downloaded ZIP file


Coding
^^^^^^

Note the function used to send the data from analog pin A0 and post it to Ubidots:


This is the code to send the value to Ubidots:

.. code-block:: cpp
      
       /*
        Simple example using WizFi device

        This example was made using an example of the original library and was modified to work with tje Ubidots platform.

        This sketch connects to Ubidots (http://www.ubidots.com) using an WizNet Shield

        Components:
        * WizFi Hardware
        * Arduino UNO
        * WizFi250 library

        Modified 15 Sep 2014
        by Mateo Vélez - Metavix - for Ubidots Inc.

        This code is in the public domain.

        */
      

      #include <Arduino.h>
      #include <SPI.h>
      #include "WizFi250.h"
      #include "WizFi250_tcp_client.h"


      #define SSID  "Atom House Medellin"
      #define KEY "atommed2014"
      #define AUTH  "WPA2"

      #define  REMOTE_PORT    80
      #define  LOCAL_PORT     5004

      IPAddress destIP  (50,23,124,68);                  //ip of the server


      WizFi250    wizfi250;
      WizFi250_TCP_Client myClient;

      boolean Wifi_setup = false;


      //The setup function is called once at startup of the sketch
      void setup()
      {
      // Add your initialization code here
        Serial.begin(9600);
        Serial.println("\r\nSerial Init");

        wizfi250.begin();
        wizfi250.setDebugPrint(4);
        wizfi250.hw_reset();

        wizfi250.sync();
        wizfi250.setDhcp();

        for(int i=0; i<10; i++)
        {
          if( wizfi250.join(SSID,KEY,AUTH) == RET_OK )
          {
            Wifi_setup = true;
            break;
          }
        }
      }

      
      void loop()
      {
      //Add your repeated code here
        uint8_t retval=0, len=0;
        uint8_t ch;
        uint8_t Txbuf[150];
              int data = analogRead(A0);
              char post_data_buf[32];
              snprintf(post_data_buf, sizeof(post_data_buf), "{\"value\": %d}\r\n", data);
        if( Wifi_setup )
        {
          wizfi250.RcvPacket();

          if( myClient.available() )
          {
            ch = myClient.recv();
            if(ch != NULL)
              Serial.print((char)ch);
          }
          else
          {
            myClient = WizFi250_TCP_Client(destIP, REMOTE_PORT);
            retval = myClient.connect();

            if(retval == RET_OK)
            {
            //change the "xxxxx..."" with your variable id and "yyyy..." with your token
              Serial.println("Connected! ");
              sprintf((char*)Txbuf,"POST /api/v1.6/variables/xxxxxxxxxxxxxxxxx/values HTTP/1.1\r\nContent-Type: application/json\r\nContent-Length: %d\r\nX-Auth-Token: yyyyyyyyyyyyyyyyyyy\r\nHost: things.ubidots.com\r\n\r\n",strlen(post_data_buf));
              myClient.send((uint8_t*)Txbuf,sizeof(Txbuf));
            }
          }
        }
      }


Wrapping it up
^^^^^^^^^^^^^^
In this guide we learned how to read an analog input from the WizFi WizNet shield and send this value to Ubidots. After getting familiar with it, you can modify your hardware setup to send readings from any other type of sensors attached to it.


More projects...
^^^^^^^^^^^^^^^^^^

Check out other cool projects using Ubidots:
 
* `Arduino WiFi <http://ubidots.com/docs/devices/arduino-wifi.html>`_
* `Arduino Wifly Shield <http://ubidots.com/docs/devices/wifly.html>`_
* `Dragino IoT Gateway <http://ubidots.com/docs/devices/Dragino.html>`_
    