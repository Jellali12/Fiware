# Getting started with LoraWAN, loraserver.io
 # Refferences:
 .LoRa: https://www.lora-alliance.org.
 .LoRa Semtech: http://www.semtech.com/wireless-rf/internet-of-things/.
 .LoRaWAN Tutorial: http://www.instructables.com/id/Use-Lora-Shield-and-RPi-to-Build-a-LoRaWAN-Gateway/.
 .Raspbian OS: https://www.raspberrypi.org/downloads/raspbian/.
 .Single Channel Package Forwarder: https://github.com/tftelkamp/single_chan_pkt_fwd.
 .Duty Cycle of LoRa: https://www.thethingsnetwork.org/wiki/LoRaWAN/Duty-Cycle .
 .RFM95 assembly with lora gateway: https://learn.adafruit.com/adafruit-rfm69hcw-and-rfm96-rfm95-rfm98-lora-packet-padio-breakouts/assembly.
 .See also http://cpham.perso.univ-pau.fr/LORA/RPIgateway.html.
 # hardware requirements(gateway and devices)
  .Raspberry Pi 3
  .2 RFM95 @868.1 MHz
  .Arduino Uno R3
  .DHT22 humidity and temperature sensor.
  # Device setup
   .Loading the existing code RedMineAtIpnet/Fiware/cayennlpp.ino into the DHT22 sensor. In order to transmit date to the fiware iot           agent the payload has to be encoded with the Cayenne Low Power Payload (Cayenne LPP). To learn more about the Cayennelpp data model        see https://mydevices.com/cayenne/docs/lora/#lora-cayenne-low-power-payload
   .To see similar project visit https://github.com/sabas1080/CayenneLPP/blob/master/examples/WeatherStation_LMIC/WeatherStation_LMIC.ino
  # Gateway setup RPI Gateway
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

  


