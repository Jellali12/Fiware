# Getting started with LoraWAN, loraserver.io
 ## Refferences:
 .LoRa: https://www.lora-alliance.org.
 .LoRa Semtech: http://www.semtech.com/wireless-rf/internet-of-things/.
 .LoRaWAN Tutorial: http://www.instructables.com/id/Use-Lora-Shield-and-RPi-to-Build-a-LoRaWAN-Gateway/.
 .Raspbian OS: https://www.raspberrypi.org/downloads/raspbian/.
 .Single Channel Package Forwarder: https://github.com/tftelkamp/single_chan_pkt_fwd.
 .Duty Cycle of LoRa: https://www.thethingsnetwork.org/wiki/LoRaWAN/Duty-Cycle .
 .RFM95 assembly with lora gateway: https://learn.adafruit.com/adafruit-rfm69hcw-and-rfm96-rfm95-rfm98-lora-packet-padio-breakouts/assembly.
 .See also http://cpham.perso.univ-pau.fr/LORA/RPIgateway.html.
 ## hardware requirements(gateway and devices)
  .Raspberry Pi 3
  .2 RFM95 @868.1 MHz
  .Arduino Uno R3
  .DHT22 humidity and temperature sensor.
  ## Device setup
   .Loading the existing code RedMineAtIpnet/Fiware/cayennlpp.ino into the DHT22 sensor. In order to transmit date to the fiware iot           agent the payload has to be encoded with the Cayenne Low Power Payload (Cayenne LPP). To learn more about the Cayennelpp data model        see https://mydevices.com/cayenne/docs/lora/#lora-cayenne-low-power-payload
   .To see similar project visit https://github.com/sabas1080/CayenneLPP/blob/master/examples/WeatherStation_LMIC/WeatherStation_LMIC.ino
  ## Gateway setup RPI Gateway
  .activation of SPI see reference
  .Ligne de commande : $ sudo raspi-config Navigate to Interface Options -> SPI -> enable it.
  .Next comes downloading the WiringPi library: $ sudo apt-get install wiringpi
  .We need git, too: $ sudo apt-get install git
  .On the LoRa software We use the single-channel packet redirector, tftelkamp / single_chan_pkt_fwd. It's a little c / cpp program to        listen to in order to communicate with a transceiver Semtech SX1276, listing  LoRa paquets and transmitting it. 
  $ git clone https://github.com/tftelkamp/single_chan_pkt_fwd.git 
  $ cd single_chan_pkt_fwd .
  $ nano main.cpp (//changing the server address from the TTN address to the lora gateway bridge host address).
  $ make .
  $ ./single_chan_pkt_fwd .

  # Loraserver's implementation
  ## LoRa gateway bridge
### Requirements
•	Mosquitto MQTT broker: In order to install it, I executed the following command: sudo apt-get install mosquitto\
•	PostgreSQL database: LoRa Server persists the gateway data into a PostgreSQL database. To install the latest PostgreSQL:\
wget –quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - sudo echo "deb http://apt.postgresql.org/pub/repos/apt/ jessie, trusty or xenial-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list sudo apt-get update sudo apt-get install postgresql-9.6\\
•	Redis database: LoRa Server stores all non-persistent data into a Redis datastore. Installation on Debian / Ubuntu: sudo apt-get\ install redis-server
### Installation of LoRa Gateway Bridge:
 It’s done simply by executing the following command that creates a configuration file is located at /etc/lora-gateway-bridge/lora-gatewaybridge.toml.\
sudo apt-get install lora-gateway-bridge
### Configuration
•	Gateway: Modify the packet-forwarder of the gateway so that it will send its data to the LoRa Gateway Bridge. serveraddress to the IP address / hostname of the LoRa Gateway Bridge servportup to 1700 (the default port that LoRa Gateway Bridge is using) servportdown to 1700 (same)\
•	The configuration file: entering the packet forwarder’s address which is the same as gateway’s.
## LoRa network server
Having the same requirements as the gateway bridge, LoRa Server needs its own database.
### Postgres database creation
1.	Starting the prompt: sudo -u postgres psql
2.	Creating a user: create role loraserverNs with login password ’dbpassword’;
3.	Creating a database create database loraserverNs with owner loraserverNs;
### Installing the Network Server
sudo apt-get install loraserver 

## LoRa application server
Creating a user, database and a PostgreSQL pqtrgm extension After starting the prompt we entered the following commands:
1.	Creating a user: create role loraserverAs with login password ’dbpassword’;
2.	Creating a database create database loraserverAs with owner loraserverAs;
3.	Changing to the LoRa App Server database: l,oraserverAs
4.	Enabling the extension: create extension pgtrgm;
### Installing the Application Server
sudo apt-get install lora-app-server\
After installation, modify the configuration file which is located at /etc/lora-app-server/loraapp-server.toml

Given that the password that I used when creating the PostgreSQL database is ’dbpassword’, the config variable postgresql.dsn had to be changed into:\
 
Another thing that has to be changed in the name of the node in the uplink topic subsciption. In pour case the node is called 'node' so the code in the configuration file has to look like the screenshot below\
 


### The application server’s web interface
LoRa application server comes with a user-web-interface that enables the management of the different components of the network and that allows the decoding and the visualization of the received data. The following steps lead to the configuration of this interface.
1. Configuring the bind address TLS certificates and the authentication key
 

2. Launching the gateway bridge
 
3.	Launching the loRa network server
 
4.	Launching LoRa application server 

5.	Access the configured bind address, in our case it’s: https://52.211.159.35:8080
 
6.	In there, the used gateway’s as well as the device’s information have to be provided, respectively in the ’gateway profiles’ and ’device profiles’ sections showed in the figure above. The used network server with which the application server will deal must be entered in the ’network servers’ section. And in the ’Applications’ section, an application using the mentioned gateway and devices can be made. In the application we can choose to decode using a self developed javascript code or using CayenneLPP schema. In our case the latter one was chosen.
## Device provisioning and data’s circulation visualization
1.	Launching the packet forwarder

And here are the values that are being sent by the node
 
2.	Encoded data will be then forwarded to the gateway bridge
3.	In the application server’s interface, we can visualize frames that are being received by the gateway
4.	Device provisioning: for the data to be forwarded to fiware’s IoT agent through an MQTT broker, the device has to be provisioned and that is by: entering the attributes which, in our case, are ’temperature’ and ’humidity’ as well as providing loraserver’s information, the device’s eui, application eui, id , key and the data model which is Cayennelpp.\
 
In the next figure, is the IoT agent processing the provisioning, connecting to the MQTT broker, starting the application and then subscribing to the application’s topic\
 
In the figure below, the received and decoded data in the application server is shown\
 
In the following figure, data that has been forwarded to the IoT agent is shown\



And below, is data stored in to context broker’s database after being translated to
NGSI by the IoT agent
 





