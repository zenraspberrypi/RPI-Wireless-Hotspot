RPI-Wireless-Hotspot With TOR Network
====================

Configures your Raspberry Pi with an attatched WiFi dongle or a Raspberry Pi3 with built in WiFi as a hotspot,
broadcasting your ethernet connection to other devices. Could be useful in hotel rooms, college dorms
or if you just don't feel like buying a router!


Features:
---------

* Configured hotspot starts automatically on boot, no extra configuration necessary

* Configured WiFi network is WPA encrypted.

* Default SSID of "RaspberryPiFi" and WPA key of "0123456789A" can be modified during install

* Once set up, the local network facilites of the Pi will still operate as normal

* Easy setup of either a custom or preconfigured DNS server (including unblock-us for removing netflix geoblocks)

* Router enumeration for WiFi network

* Allows chromecast compatibility with unblock-us by intercepting google's DNS requests on the pi

The Raspberry Pi provides a very cheap and power efficient way of setting up a TOR access point, and it also has the bonus of being incredibly easy to move around, meaning you can take your TOR access point with you anywhere you go.

 Equipment List
You can find all the recommended pieces of equipment for this Raspberry Pi TOR access point tutorial right below.

Recommended:
 Raspberry Pi

 Micro SD Card or a SD card if you’re using an old version of the Pi.

 Power Supply

 Ethernet Connection

 Wifi dongle (The Pi 3 has WiFi inbuilt)

Optional:
 Raspberry Pi Case
 
 What You Need
You don’t need anything special to make a Tor-powered Pi proxy, but you will need to round up a few materials before you get started:

Once you’ve gathered everything together, make sure your Raspberry Pi is connected directly to your router with an ethernet cable, then go ahead and plug it in.

Step One: Install the Necessary Software

The first thing we’ll need to do is make the Raspberry Pi 3’s Wi-Fi capable of acting like an access point. This turns it into a hotspot so you’ll be able to connect to it from your main computer just like you would any wireless network. We’ll be doing all this from the Raspberry Pi’s command line:

Type in sudo apt-get update and press Enter.
Type in sudo apt-get install iptables-persistent git
Select Yes and press Enter the two times you’re prompted to.
Now that everything’s been downloaded and installed, it’s time to set it up.

Step Two: Turn Your Raspberry Pi Into An Access Point

The process for turning a Raspberry Pi into an access point is a bit complicated, but thankfully GitHub user Richard Nelson | https://github.com/unixabg | http://fyeox.com/ created a script that automates the whole process.

Type in git clone https://github.com/zenraspberrypi/RPI-Wireless-Hotspot and press Enter.
Type in cd RPI-Wireless-Hotspot and press Enter.
Type in sudo ./install and press Enter. This starts the installation process.
Press Y to agree to the terms, Y to use preconfigured DNS server, N to using Unblock-Us servers, Y to using OpenDNS, then N for the Wi-Fi defaults.
When prompted after the default question, enter a new password. This is the password to connect to your Pi-powered network.
When prompted, enter a new SSID, this is the name of your network.
Enter a channel number. 11is fine, unless you know you need something else.
Enter N for the rest of the questions.
Once it’s complete, your Raspberry Pi will reboot and should now work as an access point. You can test this out by heading to another computer or phone, selecting your Raspberry Pi from the Wi-Fi network list, and seeing if the internet works. If for some reason it does not, Adafruit has a guide for doing this all manually. Otherwise, continue on and install the Tor proxy software.


Step Three: Install TOR

Setting up the TOR Access Point
To set up our TOR Access Point you will first have to of followed our wireless access point tutorial, as this will set up your Raspberry Pi correctly for this tutorial.

1. We need first to make sure we are running up to date software before we set up our TOR Access Point. To do this, we can run the following two lines in the terminal.

sudo apt-get update
sudo apt-get upgrade

2. With the Raspberry Pi now freshly updated we can get along with installing TOR itself, this is easily done by running the following command into the terminal.

sudo apt-get install tor -y

3. Now that we have installed TOR itself let’s begin by modifying its configuration, let’s open up the file for this command:

sudo nano /etc/tor/torrc

4. To this file, add the following configurations just under the FAQ notice. These lines will configure TOR to run on port 9050 and port 53.

Log notice file /var/log/tor/notices.log
VirtualAddrNetwork 10.192.0.0/10
AutomapHostsSuffixes .onion,.exit
AutomapHostsOnResolve 1
TransPort 9040
TransListenAddress 192.168.42.1
DNSPort 53
DNSListenAddress 192.168.42.1

Now we can save and quit out of the file by pressing Ctrl +X then Y and then Enter.

5. With TOR now set up, we need to flush the iptables, and we can do this by running the following two commands:

sudo iptables -F
sudo iptables -t nat -F

6. With the IPTables now flushed we can now install our new IPTables. This setup will route all the traffic incoming from the wlan0 connection through to our TOR connection that is running through port 53. The first line will add an exception for port 22 since we need that to be able to SSH to the Raspberry Pi.

 If you have upgraded to Raspbian Stretch, then wlan0 may need to be changed. Use the ifconfig command to see what the new names are, they’re likely quite long and will contain the MAC address.

sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --dport 22 -j REDIRECT --to-ports 22
sudo iptables -t nat -A PREROUTING -i wlan0 -p udp --dport 53 -j REDIRECT --to-ports 53
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --syn -j REDIRECT --to-ports 9040

If you need to check that the IPtables have been correctly entered you can use the following command.

sudo iptables -t nat -L

 
7. With our new iptables rules in place we will want to store this into the file we set up in our wireless access point, this will ensure the new IP Tables are loaded instead.

sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"

8. Now let’s create our log file, and this will be handy for tracking problems. To do so, run the following commands.

sudo touch /var/log/tor/notices.log
sudo chown debian-tor /var/log/tor/notices.log
sudo chmod 644 /var/log/tor/notices.log

9. We can check to see if the log file has now been created and permissions set correctly by utilizing the following command.

ls -l /var/log/tor

10. Now we can finally fire up the TOR service.

sudo service tor start

11. With the TOR service started we can check that the service is running by using the following command, if anything has gone wrong you will see a big FAIL notice appear.

sudo service tor status

12. Now finally, let’s make the TOR service start on boot, this will ensure that the traffic will always be routed through it. Do this with the following command.

sudo update-rc.d tor enable

If TOR isn’t really taking your fancy, then there are plenty of alternatives. The one I use almost daily is a simple Raspberry Pi VPN router that spawns a WiFi access point that you’re able to connect to. Once connected you’re on the VPN, and your origin is hidden.
When that’s finished, go ahead and reboot one more time. Type in sudo reboot and press Enter. Your Raspberry Pi should now launch everything automatically on startup.

Step Four: Connect and Browse with Your New TOR Proxy

Now, all you need to do is connect any device you want to browse anonymously on to your new Raspberry Pi Wi-Fi network. Both your regular home Wi-Fi and this one will exist, so select this as you would any Wi-Fi network. When you’re connected, head to https://check.torproject.org/ to verify that you’re on the Tor network. Enjoy your slow, but anonymous internet!

Thank you!
  ZeN
