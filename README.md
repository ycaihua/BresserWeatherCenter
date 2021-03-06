# Bresser Weather Center

The [BRESSER weather center 5-in-1](http://www.bresser.de/en/Weather-Time/BRESSER-Weather-Center-5-in-1.html) (model 7002510) outdoor sensor transfers all measured values for wind speed, wind direction, humidity, temperature and precipitation rate to the base station using radio signals and a proprietary protocol.

This library decodes the readings sent by the Bresser sensors. I tested it only with a RTL2838 dongle, using the rtl-sdr software ([http://www.rtl-sdr.com/](http://www.rtl-sdr.com/)).

Note that I've not fully reversed yet the data packed sent by the sensor, the work is still ongoing and the library still need to be tested a lot.

## Reversing and packet structure
The sensor transmit the packet on the 868.300M frequency with AM modulation.

Following a capture of the wave already demodulated:

![Radio Signal](https://www.andreafabrizi.it/img/bresser_radio_signal.png "Radio wave")

The packet is 264 bits long and the bits are ecoded with 1 for high and 0 for low. The data should be read as nybble (half byte) in BCD format.

With the sampling rate set to 48Khz we have an average of 6 samples per bit.

Following the packet structure I've reversed so far, not highlighted the parts who still need to be identified.

![Packet structure](https://www.andreafabrizi.it/img/bresser_packet.png)

The checksum is just a XOR of the following data.

## Get the code
Visit the project page on [GitHub](https://github.com/andreafabrizi/BresserWeatherCenter) or get the code with the command:
```
git clone https://github.com/andreafabrizi/BresserWeatherCenter.git
```

## Simple usage
```
from bresser import *

#The noise value should be manually adjusted for the moment
b = Bresser(printdata=True, noise = 700)
b.process_radio_data()
```

```
rtl_fm -M am -f 868.300M -s 48k -g 49.6 | ./example.py

2016-09-09 19:59:07:  Humidity: 50%  Temperature: 20.7°  Wind: 2.2 Km/h NNE  Rain: 4.0 mm
2016-09-09 19:59:17:  Humidity: 50%  Temperature: 20.7°  Wind: 2.2 Km/h NNE  Rain: 4.0 mm
2016-09-09 19:59:30:  Humidity: 49%  Temperature: 20.6°  Wind: 2.2 Km/h NNE  Rain: 4.0 mm
```

## Advanced usage
```
from bresser import *

def process_packet(p):
                        
    print "Humidity: %d%% " % p.getHumidity(),
    print "Temperature: %.1f" % p.getTemperature() + u"\u00b0 ",
    print "Wind: %.1f m/s %s" % (p.getWindSpeed(), p.getWindDirection()),
    print "Rain: %.1f mm" % p.getRain(),
    print ""
 
if __name__ == "__main__":

    #Noise neede to be adjusted manually
    b = Bresser(noise = 700)
    b.set_callback(process_packet)
    b.process_radio_data()
```

```
rtl_fm -M am -f 868.300M -s 48k -g 49.6 | ./example.py

Humidity: 91%  Temperature: 6.4°  Wind: 0.0 m/s E Rain: 78.8 mm
Humidity: 91%  Temperature: 6.4°  Wind: 0.0 m/s E Rain: 78.8 mm
Humidity: 91%  Temperature: 6.5°  Wind: 0.0 m/s E Rain: 78.8 mm
```
Note that most probably the gain and the frequency needs to be adjusted, depending on your device and antenna.

## Antenna
As antenna I used a self made metal wire 8.64 cm long (300000/868000/4) and it works quite well.

## To do
* Remove dependency from rtl_fm using pyrtlsdr
* Implement an automatic noise detection
