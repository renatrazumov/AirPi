AirPi
========

A Raspberry Pi weather station and air quality monitor alongside IPFS.

The original code can be located at http://airpi.es, https://github.com/tomhartley/AirPi for the original version or
https://git.cccmz.de/julric/airpi.git for the version with textfile-support, mysql-support, rrd-support and lcd-support.


This branch will use the https://git.cccmz.de/julric/airpi.git version of the Airpi code because we want enable the Airpi to print output on a txt files. 

We will use IPFS to hash the txt files and add it to the InterPlanetary File System (IPFS) a content-addressable, peer-to-peer hypermedia distribution protocol https://github.com/ipfs/ipfs.

The objectif here is to remove the need for external data collected by the Airpi to be stored on 3rd party server.

Currently it is split into airpi.py, as well as multiple input and multiple output plugins. airpi.py collects data from each of the input plugins specified in sensors.cfg, and then passes the data provided by them to each output defined in outputs.cfg. The code for each sensor plugin is contained in the 'sensors' folder and the code for each output plugin in the 'outputs' folder.

Some of the files are based off code for the Raspberry Pi written by Adafruit: https://github.com/adafruit/Adafruit-Raspberry-Pi-Python-Code

For installation instructions, see http://airpi.es/kit.php

## Installation

### Prerequisites

You will need to install the following dependencies:

`sudo apt-get install git-core python-dev python-pip python-smbus libxml2-dev libxslt1-dev python-lxml i2c-tools`

and

`sudo pip install rpi.gpio requests`

AirPi requires python-eeml.  To install:

```
cd ~/git
git clone https://github.com/petervizi/python-eeml.git
cd python-eeml
sudo python setup.py install
```

### i2c

To set up i2c, first add your user to the i2c group.  For example, if your username is "pi":

`sudo adduser pi i2c`

Now, add the modules needed.

`sudo nano /etc/modules`

Add the following two lines to the end of the file:

```
i2c-bcm2708
i2c-dev
```

Exit by pressing CTRL+X, followed by y to confirm you want to save, and ⏎ (enter) to confirm the filename.

Finally, unblacklist i2c by running the following command:

`sudo nano /etc/modprobe.d/raspi-blacklist.conf`

Add a `#` character  at the beginning of the line `blacklist i2c-bcm2708`. Then exit in the same way as last time.

Now, reboot your Raspberry Pi:

`sudo reboot`

### Board Version

Let's check to see which board version you have.  Run:

`sudo i2cdetect -y 0`

You should see this as the output:

```
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- 77
```

If not, run:

`sudo i2cdetect -y 1`

and you should see the above.  This tells you if your board is version 0 or 1.  This is important for the next step.

### Get The AirPi Code

Clone this repo into your git directory (or wherever you want):

```
cd ~/git
git clone https://git.cccmz.de/julric/airpi.git
cd AirPi
```

### Configuring

Edit the settings file by running:

`nano sensors.cfg`

The start of the file should look like this:

```
[BMP085-temp]
filename=bmp085
enabled=on
measurement=temp
i2cbus = 1

[BMP085-pres]
filename=bmp085
enabled=on
measurement=pres
mslp=on
i2cbus = 1
altitude=40
```

If your board version is "0" change both instances of `i2cbus = 1` to `i2cbus = 1`

Press CTRL+X to exit the file, when prompted, press "y" to save the file.

If you want to push the data to an external 3rd party server, edit the `outputs.cfg` file:

`nano outputs.cfg`

The start of the file should look like this:

```
[Print]
filename=print
enabled=on

[Name of your website]
filename=XXXXXX
enabled=on
APIKey=xxxxxxxxxx
FeedID=xxxxxxxxxx
```



## Running

AirPi **must** be run as root.

`sudo python airpi.py`

If everything is working, you should see output similar to this:

```
Success: Loaded sensor plugin BMP085-temp
Success: Loaded sensor plugin BMP085-pres
Success: Loaded sensor plugin MCP3008
Success: Loaded sensor plugin DHT22
Success: Loaded sensor plugin LDR
Success: Loaded sensor plugin MiCS-2710
Success: Loaded sensor plugin MiCS-5525
Success: Loaded sensor plugin Mic
Success: Loaded output plugin Print


Time: 2014-06-04 09:10:18.942625
Temperature: 30.2 C
Pressure: 992.55 hPa
Relative_Humidity: 35.9000015259 %
Light_Level: 3149.10025707 Ohms
Nitrogen_Dioxide: 9085.82089552 Ohms
Carbon_Monoxide: 389473.684211 Ohms
Volume: 338.709677419 mV
Uploaded successfully

```
To add Textfile support you need to add the following lines to your outputs.cfg:

```
[Log]
filename=log
enabled=on
text_file=sensors.log

```
In the line text_file=... You set the location of Your text file, where to write the data to.

The log contains all sensors+values of one measurement in one line, separated by semicolons, the sensor-name and values are separated by comma:

```
time,1414045654;TemperatureBMP,22.5;Pressure,1006.56;Light_Level,24100.0;Smoke_Level,9301.80806676;Nitrogen_Dioxide,28897.338403;Carbon_Monoxide,527607.361963;Volume,3077.41935484

```

## Running IPFS

https://ipfs.io/docs/install/
http://www.siliconian.com/blog/16-bitcoin-blockchain/23-beginner-s-guide-to-installing-ipfs-on-a-raspberry-pi-2
https://www.youtube.com/watch?v=pap18o5Ntxw

### Prerequisites

IPFS works with Go 1.7.0 or later. To check what go version you have installed, type go version. Here's what I get:

```
> go version
go version go1.7

```
### Uploading data to IPFS

Once IPFS is installed on your Raspberry pi To check what go version you have installed, open a new terminal type go version. Here's what you should get:

```
> ipfs version
ipfs version 0.4.4

```

Then start the ipfs daemon in order to connect youre Raspberry pi to the IPFS network.
Here's what you should get:

```
> ipfs daemon
Initializing daemon...
Adjusting current ulimit to 1024...
Successfully raised file descriptor limit to 1024.
Swarm listening on /ip4/xxx.xx.xx/tcp/xxxx
Swarm listening on /ip6/xxxx:xxx:xxxx:xxxx:xxxx/tcp/xxxx
API server listening on /ip4/xxx.xx.xx/tcp/xxxx
Gateway (readonly) server listening on /ip4/xxx.xx.xx/tcp/xxxx
Daemon is ready

```

Open an other terminal and go into youre Airpi repository 

```
cd Airpi

``` 

make sure that your sensor output log 'sensors.log' is located inside the folder

```
>ls
```

Then add the files to IPFS with the following command

```
ipfs add sensors.log
added QmPKcCtDgvV4y3fezPwFGjQV4SVFbiVXCD6H37WTp9FLgT sensors.log
```
IPFS will create a multi hash of the files and distribute the hash to all the nodes connected to the decentralised network.
The address of the 'sensors.log' files with all the input recorded by the Airpi is:

'QmPKcCtDgvV4y3fezPwFGjQV4SVFbiVXCD6H37WTp9FLgT'

It's address is based on his content which mean if more input are added to the 'sensors.log' files then a new hash need to be publish by the Pi.

This method solve the  reliance problem of :

- Centralised Host and website Administrators: server hosting a site goes down because of censorship, lack of interest or insolvancy.

- Reliance on Users: Content often must have a critical mass of users or visitors to even merit hosting





As long as nodes are connected to the IPFS then your input will be stored forever and anyone who is aware of the last hash published can retrieve the files.

You can retrieve the files from your command line by directly connect your node to ipfs and type

```
ipfs cat QmPKcCtDgvV4y3fezPwFGjQV4SVFbiVXCD6H37WTp9FLgT

```

Or connect to a more friendly user web console but "partially decentralised" (IPFS is still in alpha version) :)

http://localhost:5001/ipfs/QmU3o9bvfenhTKhxUakbYrLDnZU7HezAVxPM6Ehjw9Xjqy/#/objects/\ipfs\QmPKcCtDgvV4y3fezPwFGjQV4SVFbiVXCD6H37WTp9FLgT 

or

https://ipfs.io/ipfs/QmPKcCtDgvV4y3fezPwFGjQV4SVFbiVXCD6H37WTp9FLgT
