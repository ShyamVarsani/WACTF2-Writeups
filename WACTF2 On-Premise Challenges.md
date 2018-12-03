# _WACTF2 2018 On-Premise Challenges Writeups - Shyamal Varsani (**@baconandegg5**)_

These challenges required us to physically plug into a network/switch, which had 2 Raspberry Pi's on it doing some strange things :) 

On-Premise Challenge Description:

>You can connect to the on-premise switch that is labelled "RedTiger Communications" to get onto the development network. The RedTiger devices are within the following subnet: 10.1.1.0/24

>The management software can be found attached to this challenge. (the .asar file I mention below)
---
>With these challenges, we were provided with a redtiger.asar file - "Asar is a simple extensive archive format, it works like tar that concatenates all files together without compression, while having random access support"

> This file had a bunch of configuration information that came in handy to complete the following challenges 
---
## 0 RedTiger - Find the flag
### Challenge Prompt:
>You overheard this conversation a few days ago… Person 1: Hey, did you see that RedTiger Communications is going to be announcing their latest Internet of Things (IoT) device next week? Person 2: Yeah! I heard it was an IoT connected pacemaker! Person 3: Nah, I was told it's a 30-seat bus that you drive with your smartphone! Person 4: I heard it was a lightbu- wait… a bus?

### Writeup:
>To find this flag we were told that we had to be on the subnet 192.168.4.0/24.

I initially tried the approach of pulling out my shiny new Thunderbolt to Ethernet adapter out my bag, plugging in one of a bazillion Cat5e cables into said dongle and just doing a dirty `netdiscover`.
After many failed attempts I didnt get anywhere with that..

I did a little google-fu and found out that I could connect to a subnet by manually assigning myself an IP Address - I did this on my Macbook by going into System Preferences > Network > Thunderbolt Ethernet > Configure IPv4 (set it to manual) > and assigning myself the IP address of 192.168.4.224 

Now that I had put myself onto the subnet, I had learnt earlier that i could `nmap` a whole subnet to discover hosts on the network as well as what ports are open on the hosts that are found on the network. 

I had to use `sudo nmap -sV -p- 192.168.4.0/24` to scan all of the potential hosts on the subnet, and scan all the ports (1-65535) on the machines that are up.

The following is the result from the nmap scan:

```
Nmap scan report for 192.168.4.134
Host is up (0.00044s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
1789/tcp open  hello?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1789-TCP:V=7.70%I=7%D=12/2%Time=5C034512%P=x86_64-apple-darwin17.3.
SF:0%r(GenericLines,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720
SF:436f6e6e656374696f6e2e0a")%r(GetRequest,4A,"496e76616c696420436f6d6d616
SF:e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(HTTPOptions,4A,"496
SF:e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a
SF:")%r(RTSPRequest,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720
SF:436f6e6e656374696f6e2e0a")%r(RPCCheck,4A,"496e76616c696420436f6d6d616e6
SF:42e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(DNSVersionBindReqTCP,
SF:4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696
SF:f6e2e0a")%r(DNSStatusRequestTCP,4A,"496e76616c696420436f6d6d616e642e0a4
SF:36c6f73696e6720436f6e6e656374696f6e2e0a")%r(Help,4A,"496e76616c69642043
SF:6f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(SSLSession
SF:Req,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e65637
SF:4696f6e2e0a")%r(TLSSessionReq,4A,"496e76616c696420436f6d6d616e642e0a436
SF:c6f73696e6720436f6e6e656374696f6e2e0a")%r(Kerberos,4A,"496e76616c696420
SF:436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(SMBProgN
SF:eg,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374
SF:696f6e2e0a")%r(X11Probe,4A,"496e76616c696420436f6d6d616e642e0a436c6f736
SF:96e6720436f6e6e656374696f6e2e0a")%r(FourOhFourRequest,4A,"496e76616c696
SF:420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(LPDSt
SF:ring,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e6563
SF:74696f6e2e0a")%r(LDAPSearchReq,4A,"496e76616c696420436f6d6d616e642e0a43
SF:6c6f73696e6720436f6e6e656374696f6e2e0a")%r(LDAPBindReq,4A,"496e76616c69
SF:6420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(SIPO
SF:ptions,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e65
SF:6374696f6e2e0a")%r(LANDesk-RC,4A,"496e76616c696420436f6d6d616e642e0a436
SF:c6f73696e6720436f6e6e656374696f6e2e0a")%r(TerminalServer,4A,"496e76616c
SF:696420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(NC
SF:P,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e6563746
SF:96f6e2e0a")%r(NotesRPC,4A,"496e76616c696420436f6d6d616e642e0a436c6f7369
SF:6e6720436f6e6e656374696f6e2e0a");
MAC Address: B8:27:EB:2E:8B:46 (Raspberry Pi Foundation)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap scan report for 192.168.4.140
Host is up (0.00010s latency).
All 65535 scanned ports on 192.168.4.140 are closed
MAC Address: 80:FA:5B:2E:F5:10 (Clevo)

Nmap scan report for 192.168.4.166
Host is up (0.00012s latency).
All 65535 scanned ports on 192.168.4.166 are closed
MAC Address: 3C:97:0E:96:B6:B3 (Wistron InfoComm(Kunshan)Co.)

Nmap scan report for 192.168.4.242
Host is up (0.00098s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
1789/tcp open  hello?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1789-TCP:V=7.70%I=7%D=12/2%Time=5C0345CA%P=x86_64-apple-darwin17.3.
SF:0%r(GenericLines,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720
SF:436f6e6e656374696f6e2e0a")%r(GetRequest,4A,"496e76616c696420436f6d6d616
SF:e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(HTTPOptions,4A,"496
SF:e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a
SF:")%r(RTSPRequest,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720
SF:436f6e6e656374696f6e2e0a")%r(RPCCheck,4A,"496e76616c696420436f6d6d616e6
SF:42e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(DNSVersionBindReqTCP,
SF:4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696
SF:f6e2e0a")%r(DNSStatusRequestTCP,4A,"496e76616c696420436f6d6d616e642e0a4
SF:36c6f73696e6720436f6e6e656374696f6e2e0a")%r(Help,4A,"496e76616c69642043
SF:6f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(SSLSession
SF:Req,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e65637
SF:4696f6e2e0a")%r(TLSSessionReq,4A,"496e76616c696420436f6d6d616e642e0a436
SF:c6f73696e6720436f6e6e656374696f6e2e0a")%r(Kerberos,4A,"496e76616c696420
SF:436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(SMBProgN
SF:eg,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374
SF:696f6e2e0a")%r(X11Probe,4A,"496e76616c696420436f6d6d616e642e0a436c6f736
SF:96e6720436f6e6e656374696f6e2e0a")%r(FourOhFourRequest,4A,"496e76616c696
SF:420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(LPDSt
SF:ring,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e6563
SF:74696f6e2e0a")%r(LDAPSearchReq,4A,"496e76616c696420436f6d6d616e642e0a43
SF:6c6f73696e6720436f6e6e656374696f6e2e0a")%r(LDAPBindReq,4A,"496e76616c69
SF:6420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(SIPO
SF:ptions,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e65
SF:6374696f6e2e0a")%r(LANDesk-RC,4A,"496e76616c696420436f6d6d616e642e0a436
SF:c6f73696e6720436f6e6e656374696f6e2e0a")%r(TerminalServer,4A,"496e76616c
SF:696420436f6d6d616e642e0a436c6f73696e6720436f6e6e656374696f6e2e0a")%r(NC
SF:P,4A,"496e76616c696420436f6d6d616e642e0a436c6f73696e6720436f6e6e6563746
SF:96f6e2e0a")%r(NotesRPC,4A,"496e76616c696420436f6d6d616e642e0a436c6f7369
SF:6e6720436f6e6e656374696f6e2e0a");
MAC Address: B8:27:EB:CD:22:DE (Raspberry Pi Foundation)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Stats: 0:11:56 elapsed; 255 hosts completed (7 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 28.93% done; ETC: 11:00 (0:15:11 remaining)
Stats: 0:12:03 elapsed; 255 hosts completed (7 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 29.09% done; ETC: 11:00 (0:15:22 remaining)
Stats: 0:12:04 elapsed; 255 hosts completed (7 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 29.11% done; ETC: 11:00 (0:15:23 remaining)
Stats: 0:13:54 elapsed; 255 hosts completed (7 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 33.02% done; ETC: 11:03 (0:16:32 remaining)
Nmap scan report for 192.168.4.224
Host is up (0.00024s latency).
Not shown: 65533 closed ports
PORT      STATE SERVICE     VERSION
17500/tcp open  ssl/db-lsp?
60666/tcp open  unknown < Dont know wtf this?!!!?! must ask the boiz! 

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 256 IP addresses (7 hosts up) scanned in 3791.66 seconds
```
We can see that the two Raspberry Pi's on the subnet/network have the following information about them:

1. 
``` Nmap scan report for 192.168.4.134
Host is up (0.00044s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
1789/tcp open  hello?
1 service unrecognized despite returning data. 
<SNIPPED>
MAC Address: B8:27:EB:2E:8B:46 (Raspberry Pi Foundation)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

2. Identical information as the above Raspberry Pi (Apart from the IP Address)

```
Nmap scan report for 192.168.4.242
Host is up (0.00098s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
1789/tcp open  hello?
1 service unrecognized despite returning data.
<SNIPPED>
MAC Address: B8:27:EB:CD:22:DE (Raspberry Pi Foundation)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
As the Raspberry Pi were identical, I decided to focus my plan of attack of one of them, which in this case was the one on 192.168.4.242. 

I initially tried to `ssh` into 192.168.4.242, but i could not establish a connection, as i did not have the password that the service required.

I then had a look into the _redtiger.asar_ file that was provided and found out that you could simply open in a text editor such as `gedit`.

For this challenge this particular portion of the .asar file came in handy when aiming to get the first flag:

```
var Telnet = require('telnet-client')

async function connectRT() {
  let connection = new Telnet()

  let params = {
    host: $('#ipaddr').val(),
    port: 1789,
    negotiationMandatory: false,
  }

  await connection.connect(params)
  await connection.exec('7274766572\n') // rtver
}
```
What this function was doing was that establishing a telnet connection to the ip address (192.168.4.242) on port 1789 > and executing/sending the string of "7274766572" to that address and port.

In order to emulate this, I did the following:
1. `echo 7274766572 | nc 192.168.4.242 1789`


The response that was received from that was a Hex string (5741435446327b6f6e655f6f757474615f74687265655f676f5f796f75217d0a436c6f73696e6720436f6e6e656374696f6e2e0a), which if decoded from hex gave us the flag in plain text.

*WACTF2{one_outta_three_go_you!}*



---

## 1 RedTiger - Get the config file
### Challenge Prompt:
>We have infiltrated a RedTiger Communications development network that contains two of the IoT devices on it (running the same system). As a competitor company to RedTiger, it is imperative we break into the device, steal its configuration file, and understand how the proprietary communication protocol works before the product is announced to the world!
### Writeup:

>The objective of the second challenge was to get the config file off of the Raspberry Pi. 

Taking a look at the redtiger.asar file, it could be seen that "7274676574666c616731" could be send to 192.168.4.242:1789 in return for a 'config file':
```
async function flag1RT() {
  let connection = new Telnet()

  let params = {
    host: $('#ipaddr').val(),
    port: 1789,
    negotiationMandatory: false,
  }

  await connection.connect(params)
  await connection.exec('7274676574666c616731\n') // rtgetflag1
}
```
This was accomplished by doing the following:
`echo 7274676574666c616731 | nc -vv 192.168.4.242 1789`

This returned a response of:
"504b0304140001000800287d7d4d40a269344a000000410000000c00000072656474696765722e636667afe12b9392e73c57d2564bcaf9ec2be9133b05bdc40a0aa56e821103e7bb503db74444caa2c2fa3f7df1e639ab588ba41f3ddd030da2d3e47877ea75b0b57d85cbe6a5779fc256b6c3b7504b01023f00140001000800287d7d4d40a269344a000000410000000c002400000000000000200000000000000072656474696765722e6366670a0020000000000001001800db745ce8b687d40157bdc145a287d40157bdc145a287d401504b050600000000010001005e000000740000000000436c6f73696e6720436f6e6e656374696f6e2e"

When decoding this from hex using cyberchef, we got the following output:
```
PK........(}}M@¢i4J...A.......redtiger.cfg¯á+..ç<WÒVKÊùì+é.;.½Ä

¥n...ç»P=·DDÊ¢Âú?}ñæ9«X.¤.=Ý.
¢Óäxwêu°µ}.Ëæ¥w.ÂV¶Ã·PK..?.......(}}M@¢i4J...A.....$....... .......redtiger.cfg
. .........Ût\è¶.Ô.W½ÁE¢.Ô.W½ÁE¢.Ô.PK..........^...t.....
```

Looking at the output we can determine that by the "PK" header that this is infact a zip file that has been sent back.

Exporting this out of cyberchef as "download.zip", it is clear that unzipping it would be the next logical step. When attempting to unzip the download.zip, it is soon realized that this is a password protected archive.

```
root@kali:~/Desktop/Redtiger# unzip download.zip 
Archive:  download.zip
[download.zip] redtiger.cfg password:
```
After some trial and error, trying to binwalk and crack the password for this zip, I realized that the following existed in the .asar file:

```
function dlcfg() {
  // TODOTODOTODOTODOTODO
  // SEND                 rtgetcfg
  // RESPONSE             encrypted config archive (ZipCrypto)
  // PASSWORD             YSs5Y8J6aS3Um4KxfnGzCD4w
}
```
Using the aforementioned password, i unzipped the file using the password and this inflated to a redtiger.cfg! (config file!!)

```
Archive:  download.zip
[download.zip] redtiger.cfg password: <copy pasted in YSs5Y8J6aS3Um4KxfnGzCD4w>
  inflating: redtiger.cfg
```
The redtiger.cfg file contained the following:

```
second_flag=WACTF2{ooo_you_did_a_reversn_cool!}
third_flag=false
```
*WACTF2{ooo_you_did_a_reversn_cool!}*

---

## 2 RedTiger - Upload a modified config file
### Challenge Prompt:
>We were also able to secure a BETA version of the management software used to control the device. We don't know anything about this software or how to get it running, but we did come across the following note possibly written by one of the developers:
### Writeup:
"I have never wanted to gouge my eyes out with a fork more so than when I was building this Electron application."
Now that we had a config file from the previous challenge, we had to follow the .asar file 'steps' in the uploading of a modified config file:

```
function upcfg() {
  // TODOTODOTODOTODOTODO
  // We send the `rtuploadcfg` command preceding the hex encoded base64'ed encrypted (ZipCrypto) config archive
  // Use the same key as above.
  // Response will be an encrypted (ZipCrypto) flag archive ;)
}
```
My interpretation of a modified redtiger.cfg, was to leave the file intact but the change the "third_flag" parameter to 'true'. 

Once that was done and i had saved that, I began to attempt to zip the file using the key we saw previously (YSs5Y8J6aS3Um4KxfnGzCD4w), using `7z` as the archiving tool as follows:

`7z a redtiger.zip redtiger.cfg -p YSs5Y8J6aS3Um4KxfnGzCD4w`

Once we have a redtiger.zip, we can then proceed to base64 encode it:
`base64 redtiger.zip > dongs` which contains == `UEsDBBQACQAIAHq4gU1eUrWDSQAAAEAAAAAMABwAcmVkdGlnZXIuY2ZnVVQJAAOoWQNc31kDXHV4
CwABBAAAAAAEAAAAAH7x+VBJf2NS2kgPoKcqYWIHODtBDRET0OgdMf01l4Abg6qJAFMmLdEIXgR5
+sHPzwlVCho2nuXmGRKSfbrBf7d05gbIVPo6AQNQSwcIXlK1g0kAAABAAAAAUEsBAh4DFAAJAAgA
eriBTV5StYNJAAAAQAAAAAwAGAAAAAAAAQAAAKSBAAAAAHJlZHRpZ2VyLmNmZ1VUBQADqFkDXHV4
CwABBAAAAAAEAAAAAFBLBQYAAAAAAQABAFIAAACfAAAAAAA=`

I then wrote all that base64 to a file and then hexdumped it as follows:
```
554573444242514143514149414871346755316555725744535141414145
41414141414d41427741636d566b64476c6e5a58497559325a6e5656514a
41414f6f57514e6333316b44584856340a43774142424141414141414541
414141414837782b56424a66324e53326b67506f4b6371595749484f4474
4244524554304f67644d6630316c3441626736714a41464d6d4c64454958
6752350a2b7348507a776c5643686f326e75586d47524b53666272426637
64303567624956506f3641514e5153776349586c4b3167306b4141414241
4141414155457342416834444641414a414167410a657269425456355374
594e4a414141415141414141417741474141414141414141514141414b53
424141414141484a6c5a4852705a3256794c6d4e6d5a3156554251414471
466b44584856340a437741424241414141414145414141414146424c4251
594141414141415141424146494141414366414141414141413d
```
As the function upcfg() stated that 'rtuploadcfg' must preceed the hex above, the output from the following must be prepended to the hex:
`echo rtuploadcfg | xxd -p` > 727475706c6f6164636667

Now that i had a properly constructed piece of data to send (look at the message variable in the python script below), I had tried to echo that output via a netcat connection to the Raspberry Pi (192.168.4.242) on port 1789. 

I had tried to carry this out via zsh on my native host machine, although I wasnt having any luck, with responses of either:
```
Invalid Command.
Closing Connection.

Format problems etc. etc. (*something along these lines*)
Invalid Command.
```
I was told by @sudosammy that sometimes when you send/echo a large amount of data through nc, that it can sometimes break depending on which shell you execute the command off? I thought it my zsh on my mac that was causing all of issues (not going to mention the problems i ran into trying to comply with the ZipCrypto/Keka/Wateverthefark).

To mitigate this, I decided that scripting this would probably be the most reliable route, as i could use telnet (the connection platform defined in the .asar file) through python to send the formulated "message".

Python Script to Telnet (send) the data to the RPI, and listen for a response:

```
import sys
import telnetlib

HOST = "192.168.4.242"
PORT = "1789"

telnetObj=telnetlib.Telnet(HOST,PORT)
message = ("727475706c6f616463666755457344424251444151414141487067676b31655572574454414141414541414141414d41414141636d566b64476c6e5a58497559325a6e48506842456f714769796f36632f5a3657467a44655763547050705a4e6f79535176716e57535255576b577730313968376e3372394e6e6f564c6f72526c314433555651714e72457a626d59726c7869735970534b48484e6f434955526738574675392f7831424c4151492f417851444151414141487067676b31655572574454414141414541414141414d4143514141414141414141414949436b67514141414142795a5752306157646c6369356a5a6d634b4143414141414141414145414741414178444d4a39496e554151425161316b4e69745142414e4b787467714b3141465153775547414141414141454141514265414141416467414141414141")
telnetObj.write(message)
output=telnetObj.read_all()
print(output)
telnetObj.close()
```
When this script was executed, I was returned with a response of:
`504b0304140001000000e37b7d4df97650a2320000002600000008000000
666c61672e74787446d8d5d336624d88ab50c43c02135783ee3b84ba898b
18c258b7164122a8bf8f6bb850ce55eaf49238543bde2ba8441501dd504b
01023f00140001000000e37b7d4df97650a2320000002600000008002400
0000000000002000000000000000666c61672e7478740a00200000000000
01001800b3acbf7cb587d401db0f5077b587d401db0f5077b587d401504b
050600000000010001005a00000058000000000004`


This converted from hex was once again seen to be a zip file due to the telltell PK headers being seen:
```
PK........ã{}MùvP¢2...&.......flag.txtFØÕÓ6bM.«PÄ<..W.î;.º...ÂX·.A"¨¿.k¸PÎUêô.8T;Þ+¨D..ÝPK..?.......ã{}MùvP¢2...&.....$....... .......flag.txt
. .........³¬¿|µ.Ô.Û.Pwµ.Ô.Û.Pwµ.Ô.PK..........Z...X......
```
This time we could that there was a flag.txt in it!!

When that file is saved as a zip (Flag.zip), and subsequently extracted (required the same key as above - YSs5Y8J6aS3Um4KxfnGzCD4w), the output of the file is flag.txt:

```
shyamalvarsani@Shyamals-MacBook-Pro:Desktop$ unzip Flag.zip
Archive:  Flag.zip
[Flag.zip] flag.txt password:
replace flag.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
 extracting: flag.txt
shyamalvarsani@Shyamals-MacBook-Pro:Desktop$ cat flag.txt
WACTF2{this_thing_exists_in_real_lyfe}%
```
*WACTF2{this_thing_exists_in_real_lyfe}*

___

Reflection:
I learnt alot about IOT/On-site challenges and Networking in terms of the way i should approach things when it comes to physical hardware, joining of subnets, assigning myself an IP address, setting up scripts to send large chunks of data. 
On top of that i learnt of other little nifty tricks such as the -p option with xxd and how to actually create a password protected zip lol.

Big shout out to the guys @capture_tf for setting all the On-Premise challenges up! Looking foward to get my On-site hacks on again next year :) #hacktheplanet