<p align="center">
<img src="https://i.imgur.com/Clzj7Xs.png" alt="osTicket logo"/>
</p>

<h1>osTicket - Prerequisites and Installation</h1>
In this tutorial, I will go over cutting over a FIOS Verizon router to a Fortigate 60e. I would suggest if you are going to do this, to also get a switch (managed or unmanaged), and a wireless access point.

<h2>Video Demonstration</h2>

<h2>Environments and Technologies Used</h2>

- Fortigate 60e
- FIOS router
- Netgear Access Point
- Adtran netvant 1534P
- Computer
- Console cable
- ethernet cables
- Internet Information Services (IIS)

<h2>List of Prerequisites</h2>

  Have a basic understanding of networking principles (Since these intructions are going to explain mostly everything, just need to know which cables are which and what IP's are)
  Know how to use PuTTY (or any other Terminal emulator


<h2>Setting up IP-passthrough/ Bridge mode on Fios router</h2>

<p>
<img src="https://imgur.com/aW17Pql.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
First we need to setup IP passthrough mode on our FIOS router. To do this we need to get into the web gui. By default it should 192.168.1.1, but you can always check by going into command prompt and running ipconfig. The default gateway will be your router. Put that IP in your web browser and click enter.
You will be taken to your FIOS router web GUI. Here you will need to enter the network settings password found on the back of your router
</p>

<p>
<img src="https://imgur.com/G2XDk2m.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
After you get get logged into your router's web GUI, we are going to click on the advanced tab at the top. Once we are in the advanced section, we are going to click on network settings in the left and it should
drop down a couple options. We are choosing network connections. In network connections under network names you should see something called Network(office/home). We are going to want to edit this. Click edit on it.
</p>
<br />

<br />

<p>
<img src="https://imgur.com/l6orI76.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now this shouldn't be too difficult. Click on settings at the top. We should be able to scroll down and see a check box for IP pass through under bridge. We are going to check that box and click save changes at the top. At this point
you will lose connection to your network and the router should be blinking on the front. Let it run its course. After it is done blinking, I would unplug it for 30 seconds and plug it back in.
</p>
<br />


<h2>Connecting the Fortigate and configuring it</h2>
<p>
<img src="https://imgur.com/a/jCPNT1Y#HgsVFzb.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
We are going to be doing a lot of this configuration through the CLI of the FortiGate. It is 100% possible to do configurations through WebGUI depending if your FortiGate has an IP set to its LAN interface (aka the hardware switch)
and HTTPS traffic is allowed to it. Mine were not set like this already, so I will show you how to do it. Lets start off by connecting the WAN1 port on your Fortigate to the LAN2 port on your FIOS router. (It will be one less step for you if you connect wan1
instead of wan2)

<p>
<img src="https://imgur.com/ZYQbcvQ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
We are going to connect our console cable to the console port on the FortiGate and connect the usb portion to our computer. You should then open PuTTY and start the session. To start first, we are going to give an IP to the interface our switch will be conncting to.
The FortiGate 60e we have has a hardware switch that that consist of 7 ports. We will need to assign an IP to the hardware switch interface. This IP we add will act as the default gateway for whatever connects to those 7 ports.
Lets try to figure out what your interface is called. Run the following command
show system interface ?
</p>
<br />

<p>
<img src="https://imgur.com/6v3UM4n.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Look for the one that shows hard-switch at the end. That is configurable interface for the builtin switch on our FortiGate. In my previous image you can see that I already have 192.168.1.99. If you do too, that is great. If not, you will
need to enter the following commands I give you. This makes it so that devices connected to this switch will actually be able to communicate out to the internet and with each other.
Here are the commands
</p>
1. config system interface
2. edit internal
3. set mode static
4. set ip 192.168.1.99 255.255.255.0  
5. set role lan 
6. set allowaccess https ssh ping 
7. next 
8. end
<br />

<p>
<img src="https://imgur.com/mKFqFL4.png"/>
</p>
<p>
now your devices should be able to access the internet. That is, as long as they get an IP. We will need to set up DHCP
this can be done with the following commands
  <p/>
    
1. config system dhcp server
2. edit 1
3. set interface internal
4. set lease-time 86400
5. config ip-range
6. edit 1
7. set start-ip 192.168.1.10
8. set end-ip 192.168.1.200
9. next
10. end
11. set netmask 255.255.255.0
12. set default-gateway 192.168.1.99
13. set dns-service default
14. next
15. end
16. 
<p>
Please not that you can set your own IP scope and lease-time if you are comfortable going off on you own.
(if you run show system dhcp server, your results of that command should appear like the image above)
Now anything you connect to the network will recieve an IP and be able to communicate locally and out to the internet. 
</p>
<br />


<h2> Connecting our switch and access point.</h2>

<p>
<img src="https://imgur.com/a/jCPNT1Y#KIwkfZN.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
You could stop right here, but to truely have an at home network you should configure wifi. We will be doing 
this with the help of a wireless access point. There is two ways to go about deploying an access point. I will be using a POE switch to supply the power to my 
AP through ethernet. You will need to plug your POE capable switch into a wall outlet and then run
an ethernet from it to any port 1-7 on your FortiGate.
</p>
<br />

<p>
<img src="https://i.imgur.com/okiiOZv.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
After your switch has been connected via ethernet to your FortiGate and powered on, we need to find a location for your switch. Typically you want it in an area with the least amount of interference. Typicall away from the kitchen since 
microwaves operate on 2.4GHZ frequency. After mounting your AP we will run an ethernet cable from a port on your switch to your AP. Make sure to use a POE capable ethernet cable (mine is Cat 6). Consult the manual for your AP to diagnose what the indicator light
on it means.
</p>
<br />

<p>
<img src="https://imgur.com/RhRP9Ns.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
Now we need to configure the wireless access point we just deployed. I find it easiest to do this by getting on a computer that is connected via ethernet to your network (connected to the switch). This is how I did
it so I will explain how to do it this method. On your AP there should be a label on it with the model number, S/N, and MAC address of the AP. Lets take note of the MAC address. Lets get back on your computer that should 
still be connected to the console and run this command.
</p>
- get system arp
<p>
This command will display the current entries in the FortiGates ARP table. If you don't know what ARP is, it is what maps an IP to a MAC address on your router. Find the entry for your AP's MAC address and write that IP down.
( you should only really have 2 or 3 depending on if you have your computer already plugged in via ethernet.)

</p>
<br />

<p>
<img src="https://imgur.com/kKwzotR.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
If your computer isn't plugged in via ethernet, now is the time to do so. We will take that IP address assigned to the AP and put it in our browser. It should take us to the web GUI for our AP. Here my netgear one will ask
me to assign an SSID, an admin password for the AP, and a PSK for getting connected to the WIFI. Yours may be a slightly bit different if you have a different AP, but usually are pretty similar.
</p>
<br />

<p>
<img src="https://imgur.com/WxpjWvY.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
After getting your AP setup you should be able to connect to the internet either through wifi or ethernet. Its always nice at the end of cutting over network equipment to test system capabilities. Here I will just run a quick speed test 
online.
</p>
<br />

<img src="https://imgur.com/DIuO3Mx.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
You should be able to connect to the web GUI of your FortiGate already just by putting the IP address you assigned to the internal interface in your web browser. If you ever forget it, remember you can run ipconfig in command
prompt and check what your default gateway is. 
</p>
<br />

Thanks for following along. Be safe, and have fun!
