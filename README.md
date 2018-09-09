# Getting started with LoraWAN, loraserver.io
 ## References:
 .LoRa: https://www.lora-alliance.org \
 .LoRa Semtech: http://www.semtech.com/wireless-rf/internet-of-things/ \
 .LoRaWAN Tutorial: http://www.instructables.com/id/Use-Lora-Shield-and-RPi-to-Build-a-LoRaWAN-Gateway/ \
 .Raspbian OS: https://www.raspberrypi.org/downloads/raspbian/ \
 .Single Channel Package Forwarder: https://github.com/tftelkamp/single_chan_pkt_fwd/ \
 .Duty Cycle of LoRa: https://www.thethingsnetwork.org/wiki/LoRaWAN/Duty-Cycle \
 .RFM95 assembly with lora gateway: https://learn.adafruit.com/adafruit-rfm69hcw-and-rfm96-rfm95-rfm98-lora-packet-padio-breakouts/assembly \
 .See also http://cpham.perso.univ-pau.fr/LORA/RPIgateway.html
 ## hardware requirements(gateway and devices)
  .Raspberry Pi 3\
  .2 RFM95 @868.1 MHz\
  .Arduino Uno R3\
  .DHT22 humidity and temperature sensor 
  ## Device setup
  .Loading the existing code RedMineAtIpnet/Fiware/cayennlpp.ino into the DHT22 sensor. In order to transmit date to the fiware iot        agent the payload has to be encoded with the Cayenne Low Power Payload (Cayenne LPP) \ 
  .To learn more about the Cayennelpp data model see https://mydevices.com/cayenne/docs/lora/#lora-cayenne-low-power-payload\ \
  .To see similar project visit https://github.com/sabas1080/CayenneLPP/blob/master/examples/WeatherStation_LMIC/WeatherStation_LMIC.ino 
  ## Gateway setup RPI Gateway
  .activation of SPI see reference\
  .command line : $ sudo raspi-config Navigate to Interface Options -> SPI -> enable it\
  .Next comes downloading the WiringPi library: $ sudo apt-get install wiringpi\
  .We need git, too: $ sudo apt-get install git\
  .On the LoRa software We use the single-channel packet redirector, tftelkamp / single_chan_pkt_fwd. It's a little c / cpp program to      listen to in order to communicate with a transceiver Semtech SX1276, listing  LoRa paquets and transmitting it\
  $ git clone https://github.com/tftelkamp/single_chan_pkt_fwd.git \
  $ cd single_chan_pkt_fwd \
  $ nano main.cpp (//changing the server address from the TTN address to the lora gateway bridge host address)\
  $ make \
  $ ./single_chan_pkt_fwd 

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
![image](https://user-images.githubusercontent.com/40475940/45075541-58465500-b0df-11e8-8ea1-e66ab24ee17c.png)\
Given that the password that I used when creating the PostgreSQL database is ’dbpassword’, the config variable postgresql.dsn had to be changed into:\
 ![image](https://user-images.githubusercontent.com/40475940/45075556-62685380-b0df-11e8-89ac-cdb7233a2713.png)\
Another thing that has to be changed in the name of the node in the uplink topic subsciption. In pour case the node is called 'node' so the code in the configuration file has to look like the screenshot below\
![image](https://user-images.githubusercontent.com/40475940/45075613-9a6f9680-b0df-11e8-922a-f68276c667a7.png)\

## Security in a LoRaWAN network

In terms of security, the LoRANWAN standard specifies three AES-128 keys:\
NwkSKey, network session key, which uses exchanges between the terminal and the core network. It ensures the authenticity of devices in calculation and by checking the integrity code of the message, MIC, from the Header cloth and the Payload.\
AppSKey, Application Session Key, specific to end-device is used to encrypt and decrypt the payload\
AppKey, application key known only by the application and the final device and which allows to deduce the two previous keys.
![1](https://user-images.githubusercontent.com/42407959/45169205-61c5df00-b1fd-11e8-94fd-2f88fa602e37.png)
The application session key is only known by the application provider. It is impossible for a third party, including the operator, to consult the data.
A frame counter is also used to protect against replay attacks, which would consist of repeating a maliciously intercepted transmission. The counter is incremented with each transmission. The gateway and the end devices reject transmissions whose counter value is less than that present.
 =>   To be part of a LoRaWAN network, each device must obtain both session keys. This activation step can be done in two ways: Over-The-Air Activation (OTAA) or Activation By Personalization (APB).
 
### Activation By Personalization

In ABP, the NwkSKey session keys, AppSKey and DevAddr device address are directly stored on the end-device. It is then possible to bypass the procedure of join request, join accept.
NwkSKey and AppSKey must be unique!
![2](https://user-images.githubusercontent.com/42407959/45242982-78019700-b2f2-11e8-9330-20374b55dd5a.png)

|                  +                                  |                         -                           |
| ----------------------------------------------------|---------------------------------------------------- |
|  The device does not need the ability or resources to perform a join procedure.| The generation scheme of NwkSKey and AppSKey must ensure that they are unique, to avoid a widespread violation if only one device is compromise .And the system must be secure to prevent the keys from being obtained or derived by dishonest parties. If the device is compromised at any time, even before activation, the keys can be discovered.Network settings can not be specified at join time.Events that require a key change (for example, moving to a new network, the device being compromised, or expired keys) require reprogramming the device. 
|The device does not need to decide if a join is needed at any time, as this is never necessary.
No schema is needed to specify a unique DevEUI or AppKey.|                                     
 
### Over-The-Air Activation
Radio link activation consists of a "join request" and a "join acceptance" between a terminal and a server.
<img width="725" alt="3" src="https://user-images.githubusercontent.com/42407959/45243838-8b623180-b2f5-11e8-8301-719d8a33ecd7.png">

#### JOIN REQUEST
When the activation process starts, an AppKey must be assigned to both devices
and the network server. The end device should know AppEUI andDevEUI, and should be able to generate its DevNonce.
AppKey is an AES-128 root key specified for an end device.
AppEUI is an identifier of an application, while DevEUI is a unique identifier of a terminal. DevNonce is a sequence of random numbers and it is generated
by emitting a signal strength indicator metric sequence (RSSI)
and it is supposed to be ideally random.
When the join procedure begins, it first sends a join request over the air
The "Join Request" message is not encrypted, but it uses AppKey to generate the message
Integrity code (MIC) to ensure the integrity of the message.

#### JOIN ACCEPT
The server network sends a seal acceptance message to each device containing AppNonce (server-generated random value), NetID (network identifier), DevAddr (device identifier), DLSettings (device configuration), RXDelay (The delay between transmission and reception) and CFList which is optional (for channel frequencies). The Join Accept message is encrypted using AppKey instead of the join request message, which can lead to serious security issues. Anyone can know the device ID and the ID of the application to which the device is dedicated. You can also know the DevNonce included in the key generation.
Then the server transfers the AppSkey to the application server for use in decrypting messages.
After receiving the acceptance confirmation message, the device decrypts it and generates the session keys using the parameters included in the join acceptance message.

![4](https://user-images.githubusercontent.com/42407959/45243916-d7ad7180-b2f5-11e8-83ea-e4a38f6013cc.png)

#### CHARACTERISTICS OF OTAA
OTAA provides security mechanisms.
First, it uses unique parameters. In OTAA, AppKey, DevEUI, AppEUI, AppNonce and
DevNonce should all be unique between terminals. In this case, compromise a
Final device does not mean compromising the entire network.
Secondly, there is a buffer for DevNonce to prevent replay attacks. Each time a new
join request is received, the server should check the buffer to see if the nuncio has been
used before If it has been used, the end device is not allowed to join the network.
In this case, copying a join request and replaying it is not possible.

![5](https://user-images.githubusercontent.com/42407959/45243962-ff9cd500-b2f5-11e8-8ff9-35844b22851e.jpg)

|                  +                                  |                         -                           |
| ----------------------------------------------------|---------------------------------------------------- |
Session keys are generated only when necessary, so they can not be compromised before activation.|A schematic is required to pre-program each device with a unique DevEUI and AppKey, as well as the correct AppEUI. The device must support the join function and be able to store the dynamically generated keys.
If the device goes to a new network, it can reconnect to generate the new keys - rather than having to be reprogrammed.
Network settings such as RxDelay and CFList can be specified at join.
# Fiware installation 
## Prerequisites :
### DOCKER AND DOCKER COMPOSE

To keep things simple all components will be run using Docker. Docker is a container technology which allows to different components isolated into their respective environments

We used it on “Linux” environment

•	First of all, we have to install Docker on Linux. follow the instructions here :

Before you install Docker CE for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

#### SET UP THE REPOSITORY
1.	Update the apt package index:

$ sudo apt-get update

2.	Install packages to allow apt to use a repository over HTTPS:

$ sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
software-properties-common 
3.	Add Docker’s official GPG key:

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

Verify that you now have the key with the fingerprint9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88, by searching for the last 8 characters of the fingerprint.

$ sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22

      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>\
sub   4096R/F273FCD8 2017-02-22

4.	Use the following command to set up the stable repository. You always need the stable repository, even if you want to install builds from the edge or test repositories as well. To add the edge or test repository, add the word edge or test (or both) after the word stable in the commands below.

Note: The lsb_release -cs sub-command below returns the name of your Ubuntu distribution, such as xenial. Sometimes, in a distribution like Linux Mint, you might need to change $(lsb_release -cs) to your parent Ubuntu distribution. For example, if you are using Linux Mint Rafaela, you could use trusty.

o	x86_64 / amd64\
o	armhf\
o	IBM Power (ppc64le)\
o	IBM Z (s390x)\
$ sudo add-apt-repository

"deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"\
Note: Starting with Docker 17.06, stable releases are also pushed to the edge and testrepositories.\

INSTALL DOCKER CE

1.	Update the apt package index.

$ sudo apt-get update

2.	Install the latest version of Docker CE, or go to the next step to install a specific version:

$ sudo apt-get install docker-ce

Got multiple Docker repositories?

If you have multiple Docker repositories enabled, installing or updating without specifying a version in the apt-get install or apt-get update command always installs the highest possible version, which may not be appropriate for your stability needs.

3.	To install a specific version of Docker CE, list the available versions in the repo, then select and install:

a. List the versions available in your repo:

$ apt-cache madison docker-ce

docker-ce | 18.03.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages\
b. Install a specific version by its fully qualified package name, which is package name (docker-ce) “=” version string (2nd column), for example, docker-ce=18.03.0~ce-0~ubuntu.\
$ sudo apt-get install docker-ce=<VERSION>\
The Docker daemon starts automatically.\
4.	Verify that Docker CE is installed correctly by running the hello-world image.
$ sudo docker run hello-world
 
This command downloads a test image and runs it in a container. When the container runs, it prints an informational message and exits.
Docker CE is installed and running. The docker group is created but no users are added to it. You need to use sudo to run Docker commands. \

## Fiware installation :

It is to note that Fiware is an existant platform and we are not the ones that developed it. Fiware is known by a unique and complex architecture that is based on its own components. We have tried different approached in Fiware’s installation and decided to use a project that has been worked on by Fiware’s partner ATOS. The installation and configuration steps are presented below:\
1.	Clone the repository with the following command:

git clone https://github.com/Atos-Research-and-Innovation/IoTagent-LoRaWAN.git

2.	Once the repository is cloned, you have to download the dependencies for the project. To do so, from the root folder of the project execute:

npm install

3.	If you ever want to test the IoT Agent or make sure that it runs correctly, you can run it with the default configuration by executing the following command

node bin/iotagent-lora

The bootstrap process should finish with:

info: Loading devices from registry\
info: LoRaWAN IoT Agent started

4.	Check that the IoTA is running correctly:

curl -v http://localhost:4061/iot/about

The result must be similar to:

{"libVersion":"2.6.0-next","port":4061,"baseRoot":"/"}

5.	To run orion manually for test purposes, you can run the following commands

o	Manually, run MongoDB on another container for the data to be stored in it. Then, run orion while linking it to the already running MongoDB docker:

sudo docker run --name mongodb -d mongo:3.2\
sudo docker build -t orion .\
sudo docker run -d --name orion1 --link mongodb:mongodb -p 1026:1026 orion -dbhost mongodb.

o	Specify where to find your MongoDB host:

sudo docker build -t orion .\
sudo docker run -d --name orion1 -p 1026:1026 orion -dbhost <MongoDB Host>.\
Check that everything is correctly working with the command below
 
            curl localhost:1026/version
            
PS: The parameter -t orion in the docker build command gives the image a name.

### Docker-compose
For simplicity reasons, you can also run all the dockers in one go by using docker-compose. \
You can configure as many containers as you want, how they should be built and connected, and where data should be stored. \
A docker-compose.yml file\
A docker-compose.yml file is a YAML file that defines how Docker containers should behave in production.

#### docker-compose.yml

You can run a single command to build, run, and configure all of the containers.This will be done by running the following command:\
docker-compose -f docker/docker-compose.yml up


The following figure presents the docker-compose.yml file with which we can run all the dockers containing fiware’s components with a single command:

![doker-compose1](https://user-images.githubusercontent.com/42373973/45123462-43ad9f80-b167-11e8-8a6a-5540c3d2a7a4.png)
![doker-compose2](https://user-images.githubusercontent.com/42373973/45123640-da7a5c00-b167-11e8-944c-edba7d381227.png)

This docker-compose.yml file tells Docker to do the following:

•	Pull the image mongodb from mongo 3.2, the image orion from fiware/orion:latest and the image iotagent-lora from ioeari/iotagent-lora.\
•	Immediately restart containers if one fails.\
•	Map port 1026 on the host to web’s port 1026.\
•	Map port 4061 on the host to web’s port 4061.



### The application server’s web interface
LoRa application server comes with a user-web-interface that enables the management of the different components of the network and that allows the decoding and the visualization of the received data. The following steps lead to the configuration of this interface.
1. Configuring the bind address TLS certificates and the authentication key
 ![image](https://user-images.githubusercontent.com/40475940/45075625-a5c2c200-b0df-11e8-805f-cc11b345d4bc.png)

2. Launching the gateway bridge\
![image](https://user-images.githubusercontent.com/40475940/45075637-af4c2a00-b0df-11e8-82fd-da18d0967bed.png)
 
3.	Launching the loRa network server
 ![image](https://user-images.githubusercontent.com/40475940/45075651-b96e2880-b0df-11e8-84d7-f0e7f515852b.png)

4.	Launching LoRa application server \
![image](https://user-images.githubusercontent.com/40475940/45075660-c428bd80-b0df-11e8-8d71-d42141ab525f.png)

5.	Access the configured bind address, in our case it’s: https://52.211.159.35:8080
 ![image](https://user-images.githubusercontent.com/40475940/45075667-cc80f880-b0df-11e8-8422-65cfbbba8e78.png)

6.	In there, the used gateway’s as well as the device’s information have to be provided, respectively in the ’gateway profiles’ and ’device profiles’ sections showed in the figure above. The used network server with which the application server will deal must be entered in the ’network servers’ section. And in the ’Applications’ section, an application using the mentioned gateway and devices can be made. In the application we can choose to decode using a self developed javascript code or using CayenneLPP schema. In our case the latter one was chosen.
## Device provisioning and data’s circulation visualization
1.	Launching the packet forwarder\
![image](https://user-images.githubusercontent.com/40475940/45075779-2681be00-b0e0-11e8-957e-5156ca38856f.png) \

And here are the values that are being sent by the node\
 ![image](https://user-images.githubusercontent.com/40475940/45075797-36010700-b0e0-11e8-8ca3-761139ec500c.png)

2.	Encoded data will be then forwarded to the gateway bridge\
![image](https://user-images.githubusercontent.com/40475940/45075813-40230580-b0e0-11e8-9ddb-e83fe50a05ae.png)

3.	In the application server’s interface, we can visualize frames that are being received by the gateway\
![image](https://user-images.githubusercontent.com/40475940/45075856-58932000-b0e0-11e8-91ba-69f091aaabc6.png)

4.	Device provisioning: for the data to be forwarded to fiware’s IoT agent through an MQTT broker, the device has to be provisioned and that is by: entering the attributes which, in our case, are ’temperature’ and ’humidity’ as well as providing loraserver’s information, the device’s eui, application eui, id , key and the data model which is Cayennelpp.\
 ![image](https://user-images.githubusercontent.com/40475940/45075865-621c8800-b0e0-11e8-98b7-011fbf402e1c.png)\

In the next figure, is the IoT agent processing the provisioning, connecting to the MQTT broker, starting the application and then subscribing to the application’s topic\
 ![image](https://user-images.githubusercontent.com/40475940/45075891-6fd20d80-b0e0-11e8-84b9-5b523ae94c08.png)\

In the figure below, the received and decoded data in the application server is shown\
 ![image](https://user-images.githubusercontent.com/40475940/45075907-7ceefc80-b0e0-11e8-82ab-66aead4702f9.png)\
 
In the following figure, data that has been forwarded to the IoT agent is shown\
![image](https://user-images.githubusercontent.com/40475940/45075928-89735500-b0e0-11e8-983d-e452b3452483.png)\



And below, is data stored in to context broker’s database after being translated to NGSI by the IoT agent\
![image](https://user-images.githubusercontent.com/40475940/45075935-942dea00-b0e0-11e8-81a8-e1613486f5a7.png)

 

# Security 
## Replay attack 
#### ATTACK GOAL
This attack is designed to achieve spoofing and DoS.
For the server, the attack goal is to achieve spoofing. After the attack, it will accept
a malicious replayed message from the attacker’s end device, and the server will believe
the message is from an accepted working end device.
For the victim end device, the attack goal is to achieve DoS. After the attack, the message
that the victim end device sends will not be accepted in the server. The period of
DoS depends on the selection of replayed message.
#### ATTACKER CAPABILITIES
In order to achieve this attack, the attacker should be capable of:
• having knowledge of the physical payload format of LoRaWAN messages.
• knowing the wireless communication frequency band of the victim end device.
• having a device to capture LoRaWAN wireless messages.
• having a device to send LoRaWAN messages in a certain frequency.
• storing and reading plaintext of LoRaWAN messages
If the attacker does not have a specific victimtarget, in a large LoRaWAN network, it will
not take a long time for an attacker to wait for an overflow. However, if the attacker is
performing attacks in a relatively small network, it is better if the attack is able to reset
the victim end device to reduce the waiting time.
#### ATTACK DESCRIPTION
![6](https://user-images.githubusercontent.com/42407959/45245178-94560180-b2fb-11e8-97ab-92a9969894d6.png)

##### Attack steps:
Capture messages. Use a device to capture uplink messages of an ABP activated
node, and save them into the attacker’s database
• Get FCnt value. Read the uplink counter value from these messages since counter
values are not encrypted.
• Wait till the end device resets or counter overflows.
• Find a suitable message. Select a captured message with suitable counter value
from attacker’s database.
• Replay. Resend the message to the gateway.
![7](https://user-images.githubusercontent.com/42407959/45245481-e8adb100-b2fc-11e8-8f06-1fccd295b7b3.png)

This attack can be extremely harmful for ABP activated end devices in a large Lo-
RaWAN network. In a small LoRaWAN network with only a few end devices, the attacker
may need to wait a long time for a counter overflow. However, in a large LoRaWAN network
with multiple end devices, the waiting time for any one of the end devices to be
overflowed is highly decreased. Once the attacker gets the largest possible counter value
for one end device, it can periodically replay this message, to make the end device be
rejected permanently. Unless the session keys of the end device are changed, the end
device cannot be functioning again.
In addition, if the attacker can find a way to reset the end device (e.g. power outage),
then there is no need for the attacker to wait for counter overflowing. By resetting the
end device, and replaying the message with the largest counter value, messages from the
victim end device will be rejected.

#### Attack-Defense Tree
![image](https://user-images.githubusercontent.com/42407959/45246058-830ef400-b2ff-11e8-8763-7e9531ed9275.png)

==> Possible countermeasures are the use of frame counters on a network
level (when re-transmitting frames exactly as they are captured),
and/or by implementing correct encryption levels.

#### Possible applicability to LoRaWAN
A replay attack can be performed on any component of a LoRaWAN
implementation where an adversary can get access to network communication.
The easiest component for this attack is the wireless network
communication. All communication between end-devices and
the Network Server will always pass the wireless network.

If traffic between the Network Server and the gateways, or between
the Network Server and Application Servers, needs to be replayed,
then in most cases access to wired networks is required.

Most of the traffic that can be sniffed in a LoRaWAN implementation
is encrypted. Additional activities are required to get access to the
real transmitted data (if possible at all). A replay attack is therefore
not easy. However, there is unencrypted traffic, and maybe there are
options to replay encrypted data packets.

### Man-in-the-Middle
In a Man-in-the-Middle (MitM) attack, an adversary places himself
into a conversation between two parties and impersonates both parties
to gain access to the data that the two parties are sending to
each other. It involves an active form of eavesdropping. A man-in-themiddle
attack allows an adversary to intercept, send and receive data
meant for someone else, without both parties knowing. This attack is
a form of session hijacking and is often used in wireless communications.

#### Attack-Defense Tree

In a MitM attack an adversary
needs to impersonate both the sender and receiver. On behalf of
both parties, it needs to perform send- and receive activities so that it
can proxy, or relay, traffic between both parties.
The main countermeasures for this kind of attack is the use of encryption
for the data that is being send, and signing of messages so that
the receiver is sure of the sender’s identity.

![image](https://user-images.githubusercontent.com/42407959/45246277-a9815f00-b300-11e8-8b45-3e92f4fcb4bc.png)


#### Possible applicability to LoRaWAN
A MitM attack can be performed on any component of a LoRaWAN
implementation where an adversary can get access to network communication.
The easiest component for this attack, is the wireless network
communication. All communication between end-devices and
the Network Server will always pass the wireless network.
If a MitM attack between the Network Server and the gateways, or between
the Network Server and Application Servers, is required, then
in most cases access to wired networks is required.
Most of the traffic that can be sniffed in a LoRaWAN implementation
is encrypted. Additional activities are required to get access to the
real transmitted data (if possible at all). In the LoRaWAN specification,
network frame counters are defined. Encryption and frame counters
are successful counter-measures against MitM attacks.

### Denial of Service
A (Distributed) Denial of Service ((D)DOS) attack is one of the more
popular attacks nowadays because it can cause great harm with reasonably
simple efforts. The goal of a (D)DOS attack is to make sure
that the service under attack becomes unavailable (due to overload)
for a certain period of time. Usually this is accomplished by exhausting
resources in the environment that is providing the service.
During a DOS attack, only one source host is used in the attack. This
type of attack is not used very much anymore. These days, infrastructure
providing services can handle a lot of traffic, more traffic than
can be initiated by a single source host. Because of this, (D)DOS attacks
have increased in popularity. In this type of attack, the attack is being
initiated by many different hosts instead of a single one. With the
rise of botnets, which are virtual networks of hijacked hosts that are
under control of malicious actors, such (D)DOS attacks have become
easier and more powerful.
(D)DOS attacks are usually all about exhausting resources. There are
two major categories for resource exhaustion during (D)DOS attacks:
• Exhaustion of network resources: by sending excessive number
of requests (network packets). Sending excessive amounts of
network requests can be accomplished by using many source
hosts, but also by making use of amplification (amplification is
a way for an adversary to magnify the amount of bandwidth
they can target at a potential victim; examples are the DNS and
ICMP protocols).
• Exhaustion of cpu/memory/disk used by the online application
processes.
#### Attack-Defense Tree
A (D)DOS attack can
either by a Denial of Service (DOS) or a Distributed Denial of Service
(DDOS). In case of a DOS only a single host is initiating the attack.
Because it is only a single host, blocking all traffic from this host can
be very effective countermeasure. For a (D)DOS attack, an adversary
needs to have multiple (many) of hosts under control. From all these
hosts traffic should be initiated towards the target service so that resources
on the target service become exhausted. Countermeasures
against a (D)DOS attack are very hard because requests are initiated
from an enormous number of hosts. It is impossible to distinguish
between valid and invalid traffic. Usually it turns out that resources
need to be added so that the target service has more resources than
the (D)DOS attack can use. However, this is very difficult and costly.

![image](https://user-images.githubusercontent.com/42407959/45246805-a9369300-b303-11e8-9698-b55f6af15915.png)

#### Possible applicability to LoRaWAN
A (D)DOS attack can be performed on any component of a LoRaWAN
implementation where an adversary can get access to network communication.
The most obvious target components within the LoRaWAN
communication chain would be the end-devices and the Network
Server.
For now, it is difficult to determine which type of (D)DOS would be
the most efficient for the individual LoRaWAN components.
As mentioned in chapter 3.2.4, Join-Accept messages might be a possible
type of communication that can be used for (D)DOS attacks.

### EAVESDROPPING
The attack is designed to compromise the encryption method of LoRaWAN. By sniffing
the wireless traffic between the gateway and the end device, the attacker can use the
corresponding relationship between 2 messages with the same counter value to decrypt
the ciphertext.
After the attack, the attacker can compromise the confidentiality of the system, and
obtain sensor data transmitted in the system. If LoRaWAN is used to transmit secret data,
this attack can cause serious privacy issues.
#### ATTACKER CAPABILITIES
In order to performthe attack, the attacker should have the capabilities of:
• having a LoRaWAN wireless sniffer device to sniff wireless packets.
• having basic knowledge of end devices such as message type and message format.
• having a database to store and compare LoRaWAN traffic.
In order to increase the accuracy of the decryption results, it is better if the attacker also
has the ability to reset the end devices.

#### ATTACK DESCRIPTION

![image](https://user-images.githubusercontent.com/42407959/45264703-24519380-b441-11e8-95ce-ea996c9c2a58.png)

#### ATTACK STEPS
The attack can be operated in following steps:
• The attacker captures and stores LoRaWAN wireless packets, and logs basic information.
• After resetting, continue to collect packets. Compare packets before and after resetting.
Pair packets with same counter value.
• Coding with method crib dragging, see the result.
the figure shows an example of conducting an eavesdropping attack in a LoRaWAN network.
A malicious gateway with appropriate frequency can receive messages from end
device. Pairing the messages before and after resetting with same counter value, we can
use crib ragging to derive the plaintext. In different cases, the implementation of crib
dragging can be different. For example, if the plaintexts are sentences in English, it is
easy to guess. If the plaintexts are numbers, we need to find the regular patterns behind
the numbers first.

![image](https://user-images.githubusercontent.com/42407959/45264708-49de9d00-b441-11e8-9678-26a6353c8760.png)

# DATA VISUALISATION
In order to control Data integrity we should visualise them in different steps using MongoDB to know if the totality of the data are sent 

=> we enter the IP address 34.254.184.237


![image](https://user-images.githubusercontent.com/42407959/45264951-b0b18580-b444-11e8-8961-08c93862b3ad.png)

and the file ppk
![image](https://user-images.githubusercontent.com/42407959/45264952-c030ce80-b444-11e8-8229-a97cc536af13.png)

![image](https://user-images.githubusercontent.com/42407959/45264954-cc1c9080-b444-11e8-908c-6c8d54afb380.png)


![image](https://user-images.githubusercontent.com/42407959/45264959-d8085280-b444-11e8-8e91-3eba68ade9c3.png)

![image](https://user-images.githubusercontent.com/42407959/45264960-e2c2e780-b444-11e8-84cb-93458574ef66.png)
![image](https://user-images.githubusercontent.com/42407959/45265318-39322500-b449-11e8-85af-0d83f8b4cbcd.png)

Open another terminal
=> Monitor Iotagentlora database
![image](https://user-images.githubusercontent.com/42407959/45265326-5ebf2e80-b449-11e8-9658-9d2d99dde3f4.png)

monitor Orion database (orion-test)
![image](https://user-images.githubusercontent.com/42407959/45265334-93cb8100-b449-11e8-9520-d5a3a74c6dfc.png)
![image](https://user-images.githubusercontent.com/42407959/45265336-a5148d80-b449-11e8-8587-a26f84c2d38b.png)



