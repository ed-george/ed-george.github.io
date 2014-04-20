Pi-Racey: An Initial Build
----------
My pi-Racey project has been keeping me busy the last week. It has progressed significantly in this time, as described below. 

###Initial Research

The initial research for the project, [as mentioned previously](http://ed-george.github.io/2014/04/16/piracey.html), pointed me in the direction of an [example project](http://blog.kaazing.com/2013/04/01/remote-controlling-a-car-over-the-web-ingredients-smartphone-websocket-and-raspberry-pi/) by the tech company Kaazing. 

This project used the popular Raspberry Pi Java library [Pi4J](http://pi4j.com/) to provide an interface to the Pi's GPIO pins. It also used web sockets to communicate between the Pi and a *remote* in the form of a website.
 
As mentioned in my last post, web sockets could potentially be problematic due to there being no guarantee of reliable WiFi in an area where the car should be useable. 

This led me to research other possibilities of wirelessly controlling the car.

+ Infrared: Commonly used in television remotes and other devices but requires device and sender to be in each other's [line of sight](http://en.wikipedia.org/wiki/Line-of-sight_propagation). 

+ Original Remote Control: The original remote control of the car could be modified so it's signals are picked up and handled by the Raspberry Pi instead of the original receiver. However, this could be difficult and require a detailed electronics knowledge.

+ Bluetooth: Arguably, the most sensible option available. Bluetooth's max range is [around 100m](http://en.wikipedia.org/wiki/Bluetooth_low_energy#Technical_details) and is present in many devices, such as mobile phones, that could allow easy connectivity to provide control to the vehicle. It was for this reason I decided that Bluetooth would be used to control pi-Racey.

###Bluetooth Options

Through the use of a [Bluetooth USB adapter](http://www.amazon.co.uk/gp/product/B00F0CG0N4), the Pi is able to use the Bluetooth wireless exchanging standard.

Following the Kaazing project, I searched for a Bluetooth library that could be used with Java and Pi4J. This led to [BlueCove](http://bluecove.org/), an open source JSR-82 implementation of Bluetooth written in Java. 

The library provides a bridge between the application layer of the Java VM and the Bluetooth stack/controller and can be used to pair devices, as well as send messages between each device. 

However it was clear from the beginning that the library, despite providing a potential solution, was complex and would make a relatively simple task (i.e. [pair a device](http://bluecove.org/bluecove/apidocs/overview-summary.html#DeviceDiscovery)) a lot of spaghetti code that is difficult to understand for anyone unfamiliar with the library.

For this reason, I considered other options.  

###Wiimote

Through some further research, I discovered the [CWiid project](http://abstrakraft.org/cwiid), a collection of Linux tools written in C for interfacing to the Nintendo Wiimote on Linux platforms.


[![A Wiimote](http://hacknmod.com/wp-content/uploads/2009/04/wiimote-762302.jpg)](http://en.wikipedia.org/wiki/Wii_Remote)

The Wiimote is a wireless peripheral for the Nintendo Wii games console that uses Bluetooth to allow the device to be used as the console's remote. It has a number of buttons and a directional pad that could be utilised to control the car.

The main advantage of using a Wiimote is the CWiid library makes the connection between the Wiimote and any Bluetooth adapter used with the Pi and can easily interface with the buttons on the remote.

To continue the project, I decided to scrap the use of Java and Pi4J and move to Python as the `python-cwiid` and `python-dpi.gpio` libraries allow easy programming with both the Pi's GPIO and an external Bluetooth Wiimote. 


###Setup

To start testing on the Pi, the system will require multiple libraries and drivers. These can be easily installed through the following commands:

```
sudo apt-get update
sudo apt-get upgrade
# Update packages and packages list
sudo apt-get install --no-install-recommends bluetooth
# Install bluetooth drivers to allow communication to 
# bluetooth devices, such as the Wiimote
# The parameter ensures only the bare essentials of the
# bluetooth driver is installed
sudo apt-get install python-cwiid
# Install the CWiid Python library
sudo apt-get install python-rpi.gpio
# Install the Python Raspberry Pi GPIO library  
```
Once a [compatible Bluetooth adapter](http://elinux.org/RPi_USB_Bluetooth_adapters) has been plugged into the Pi, the status of the Bluetooth service can be tested through the use of 
`sudo service bluetooth status` - This should return a message of `[ ok ] bluetooth is running.` which indicates Bluetooth is running correctly. Should any other message appear, the Pi should be restarted.

Once Bluetooth is running correctly on the device, the Wiimote can be visible to the Pi. This is easily completed using `hcitool`.

Whilst pressing the 1 and 2 buttons simultaneously on a WiiMote, run `hcitool scan` on the command line to view all visible bluetooth devices. A device found that uses the name of "Nintendo RVL-CNT-01" indicates a Wiimote is visible.

Finally, a test script can be used to test out the Wiimote and the GPIO's working in harmony.

###Test Results

[Using this Python script (wii_connect.py)](https://gist.github.com/ed-george/11084155), I was able to create a simple test along with a simple bicolour LED circuit. Pressing the A button creates a red light and pressing the B trigger shows a green light.

The video below shows this test in action

[![See the Wiimote in Action](http://img.youtube.com/vi/vq5hOptXNeg/0.jpg)](https://www.youtube.com/watch?v=vq5hOptXNeg)
  
###Finally...

I shall detail the next few stages soon. There is still plenty of coding to complete and many electrical component to add.

For more updates please visit the [pi-Racey website](http://ed-george.github.io/piRacey/), or tweet me [@EdGeorge92](https://twitter.com/edgeorge92).