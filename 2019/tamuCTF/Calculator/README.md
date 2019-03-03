# TAMUCTF Calculator (Network/Pentest) Write-Up

### Given
	Using a teletype network protocol from the 70s to access a calculator from the 70s? Far out!
	
	Note to new players: You won't see anything in Wireshark / tcpdump when you initially connect. (i.e. packets are sent unicast on a bridged network)
	
	Wireshark
	tcpdump
	ettercap
	Difficulty: easy
	
	2/23 8:56 am: Added suggested tools

### Setup
1. Always gotta get setup first right? Of course we need to start somewhere.
		openvpn --config calculator.ovpn
2. Lets see who’s online
		nmap 172.30.0.0/28

	Once that finishes we can see some interesting open port. Yes port singular. We only have telnet

		Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-02 22:13 CST
		Nmap scan report for 172.30.0.2
		Host is up (0.092s latency).
		Not shown: 999 closed ports
		PORT   STATE SERVICE
		23/tcp open  telnet
		MAC Address: 02:42:69:0E:CA:56 (Unknown)
		
		Nmap scan report for 172.30.0.3
		Host is up (0.092s latency).
		All 1000 scanned ports on 172.30.0.3 are closed
		MAC Address: 02:42:B2:5B:2E:CA (Unknown)
		
		Nmap scan report for 172.30.0.14
		Host is up (0.0000060s latency).
		All 1000 scanned ports on 172.30.0.14 are closed
		
		Nmap done: 16 IP addresses (3 hosts up) scanned in 5.54 seconds

	What we find interesting here is that there are two hosts online. One with port 23 open for telnet and one who is just closed completely. Now we are at a fork. We can do super enumeration and see if the second host has any open ports or we can connect some dots and go the better way. Notice it’s the better way.

3. From the given information we can see that “packets are sent unicast on a bridged network” and to be honest with you I don’t know what a lot of that means. What I took from this is that packets are being sent. So why can’t I see them? Well I decided that now I want to see everyones packets so I became the router.

	**Arp Spoofing**

	Lets crack open Ettercap
		ettercap -G 				Opens etter in graphical mode

	First we gotta tell ettercap what it’s doing open.
		Sniff > Unified Sniffing > tap0 > OK
		
		This is telling ettercap to sniff the unified network tap0 which is going to set out interface to the VPN we connected to for the challenges

	Next we need to tell ettercap where the hosts are. Or actually let it find them itself. 

		Hosts > Scan for hosts

	Done. It now knows who’s online

	Now we need to actually specify the mode of the ettercap man in the middle attack. 

		Mitm > Arp Poisioning > Sniff remote connections (leave Only poision one way OFF) > OK
		
		This is setting the Mitm mode to ARP spoofing.  


4. Wireshark time!
	Crack open that great program

	Select tap0 as the interface and start to see all those packets roll in. 

	Huh notice that cool telnet traffic? OoOAaa. Yeah so cool.

	Okay enough flirting with telnet. Click on that traffic and follow the TCP stream. You might need to change streams to see a full connection but hey look what’s that?? 

	Wow that’s a telnet login. Would you look at that. 
	**If you do not see a full login change streams and/or close the viewing window and wait a few seconds to have more traffic and get a full connection**

5. Getting the flag is not super hard but requires a VERY small amount of linux command knowhow. 

	Let’s telnet in using the credentials we just grabbed.
		telnet 172.30.0.2

	Now that we are in we can see the flag.

		ls

	Oh wait this directory is empty?? Nope but the flag is hidden

		ls -al

	Look I see something shiny! It’s `.ctf_flag `

		cat .ctf_flag

Flag: gigem{f5ae5f528ed5a9ad312f75bd1d3406a2}
