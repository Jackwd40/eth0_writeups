# TAMUCTF Stop and Listen (Network/Pentest) Write-Up

This is a pretty simple challenge. Great to get started with.

1. Very first thing. Lets connect to the VPN
		openvpn --config listen.ovpn
2. Next we need to scan the entire subnet to see what we need to do
		nmap 172.30.0.0/28

3. The output of the command is:

		Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-02 21:56 CST
		Nmap scan report for 172.30.0.2
		Host is up (0.086s latency).
		All 1000 scanned ports on 172.30.0.2 are closed
		MAC Address: 02:42:8D:86:EF:97 (Unknown)
		
		Nmap scan report for 172.30.0.14
		Host is up (0.0000060s latency).
		All 1000 scanned ports on 172.30.0.14 are closed
		
		Nmap done: 16 IP addresses (2 hosts up) scanned in 4.77 seconds
		
	No ports open. Lets move to wireshark

	**Information not relevant to challenge**
	When I worked this challenge nothing came up on the nmap. Coming from Hack The Box I thought it just wasn't a common port. If this happens on another CTF the next steps are to follow.

	nmap -p 1-65535 172.30.0.0/28 to try every port in that range. 

4. The next logical move is to see what traffic is on this network already. Funny enough opening wireshark solves the challenge. 

	Once you open wireshark you will slowly see UDP packets start to populate the area. Important note is that you do not need to scan the subnet to initiate those packets

	If you right click any of those UDP packets and select follow \> UDP stream you can start to see text appear. I recommend closing that window and waiting a few minutes for the entire cycle to complete.

	**Leave wireshark open for capturing but close the window that shows you the text**

5. Why read all that?? Well lets look back at the first challenge that taught you the flag format. It’s `gigem{FLAG}` right? Well taking that into account lets use wireshark find feature to just search for that. Completing that produces the flag.

Flag: gigem{f0rty_tw0_c9d950b61ea83}
