<p align="center"> <img src="https://imgur.com/N4r9nQ7.png" alt="osTicket logo"/> </p>

<h1>Verizon FIOS router cutover to FortiGate 60e </h1> In this tutorial, I will walk through cutting over a Verizon FIOS router to a FortiGate 60e. If you plan to do this, I recommend also using a switch (managed or unmanaged) and a wireless access point.

<h2>Video Demonstration</h2>

<h2>Environments and Technologies Used</h2>

- Fortigate 60e

- FIOS router

- Netgear Access Point

- Adtran NetVanta 1534P

- Computer

- Console cable

- Ethernet cables

- Internet Information Services (IIS)

<h2>List of Prerequisites</h2>

Have a basic understanding of networking principles. These instructions explain most steps, so you mainly need to know which cables are which and what IP addresses are.Know how to use PuTTY (or any other terminal emulator).

<h2>Setting up IP-passthrough/ Bridge mode on Fios router</h2>

<p> <img src="https://imgur.com/aW17Pql.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> First, we need to enable IP passthrough mode on the FIOS router. To do this, access the router’s web GUI. The default address is usually 192.168.1.1, but you can confirm by running <code>ipconfig</code> in Command Prompt. The default gateway listed there is your router’s IP. Enter that address into your browser and press Enter.You will be taken to the FIOS router GUI, where you must enter the network settings password found on the back of the router. </p>

<p> <img src="https://imgur.com/G2XDk2m.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> After logging in, click the Advanced tab at the top. In the left-hand menu, select Network Settings, and a few options will drop down. Choose Network Connections. Under Network Names, you should see something called Network (office/home). Click Edit on that entry. </p> <br />

<br />

<p> <img src="https://imgur.com/l6orI76.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> Click the Settings tab at the top. Scroll down until you see the checkbox for IP Passthrough under Bridge. Check the box and click Save Changes. At this point, you will lose network connectivity, and the router’s front LEDs will begin blinking. Let it finish. After it stops blinking, unplug the router for 30 seconds and plug it back in. </p> <br />

<h2>Connecting the Fortigate and configuring it</h2> <p> <img src="https://imgur.com/Q4MUebh.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> We will perform most of the configuration through the FortiGate CLI. You can also use the Web GUI if your FortiGate already has an IP assigned to its LAN interface (the hardware switch) and HTTPS access is enabled. Mine did not, so I will show how to configure it manually.Start by connecting the WAN1 port on the FortiGate to the LAN2 port on the FIOS router. (Using WAN1 instead of WAN2 saves an extra step.)

<p> <img src="https://imgur.com/ZYQbcvQ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> Connect your console cable to the FortiGate’s console port and plug the USB end into your computer. Open PuTTY and start a session.First, we need to assign an IP address to the interface your switch will connect to. The FortiGate 60e has a hardware switch consisting of seven ports. We must assign an IP to the hardware switch interface, which will act as the default gateway for devices connected to those ports.To identify the interface name, run: <br> <code>show system interface ?</code> </p> <br />

<p> <img src="https://imgur.com/6v3UM4n.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> Look for the interface that ends with <code>hard-switch</code>. That is the configurable interface for the built‑in switch. In the screenshot, mine already has 192.168.1.99 assigned. If yours does not, enter the following commands. This ensures devices connected to the switch can communicate with each other and reach the internet.Here are the commands: </p>

- config system interface

- edit internal

- set mode static

- set ip 192.168.1.99 255.255.255.0

- set role lan

- set allowaccess https ssh ping

- next

- end

<br />

<p> <img src="https://imgur.com/mKFqFL4.png"/> </p> <p> Now your devices should be able to access the internet, as long as they receive an IP address. To handle that, we need to set up DHCP. Use the following commands: </p>

- config system dhcp server

- edit 1

- set interface internal

- set lease-time 86400

- config ip-range

- edit 1

- set start-ip 192.168.1.10

- set end-ip 192.168.1.200

- next

- end

- set netmask 255.255.255.0

- set default-gateway 192.168.1.99

- set dns-service default

- next

- end

<p> You can adjust the IP scope and lease time if you prefer.If you run <code>show system dhcp server</code>, your configuration should match the screenshot above.Now any device you connect to the network will receive an IP address and be able to communicate locally and access the internet. </p> <br />

<h2> Connecting our switch and access point.</h2>

<p> <img src="https://imgur.com/ncndAUX.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> You could stop here, but to have a complete home network, you should configure Wi‑Fi. We will do this using a wireless access point. There are two ways to deploy an AP, but I will use a PoE switch to power mine through Ethernet.Plug your PoE‑capable switch into a wall outlet, then run an Ethernet cable from the switch to any port (1–7) on the FortiGate. </p> <br />

<p> <img src="https://i.imgur.com/okiiOZv.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> After connecting and powering on the switch, choose a good location for your AP—ideally somewhere with minimal interference. Avoid placing it near the kitchen, since microwaves operate at 2.4 GHz.Mount the AP, then run an Ethernet cable from the switch to the AP. Make sure to use a PoE‑capable cable (mine is Cat 6). Check your AP’s manual to understand the indicator lights. </p> <br />

<p> <img src="https://imgur.com/RhRP9Ns.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> Now we need to configure the wireless access point. The easiest way is to use a computer connected to the network via Ethernet.On your AP, find the label with the model number, serial number, and MAC address. Note the MAC address.Return to your console session and run: </p>

- get system arp

<p> This command displays the FortiGate’s ARP table, which maps IP addresses to MAC addresses. Find the entry matching your AP’s MAC address and write down the IP address assigned to it.(You should only see two or three entries unless more devices are connected.) </p> <br />

<p> <img src="https://imgur.com/kKwzotR.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> If your computer is not already connected via Ethernet, plug it in now. Enter the AP’s IP address into your browser. This should open the AP’s web GUI.My Netgear AP asked me to set an SSID, an admin password, and a Wi‑Fi PSK. Your AP may differ slightly, but the process is usually similar. </p> <br />

<p> <img src="https://imgur.com/WxpjWvY.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> After configuring your AP, you should be able to connect to the internet through Wi‑Fi or Ethernet. It’s always a good idea to test your network after a cutover, so here I ran a quick online speed test. </p> <br />

<img src="https://imgur.com/DIuO3Mx.png" height="80%" width="80%" alt="Disk Sanitization Steps"/> </p> <p> You should now be able to access the FortiGate Web GUI by entering the IP address you assigned to the internal interface. If you ever forget it, run <code>ipconfig</code> in Command Prompt and check your default gateway. </p> <br />

Thanks for following along. Be safe, and have fun!
