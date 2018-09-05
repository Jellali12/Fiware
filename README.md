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



