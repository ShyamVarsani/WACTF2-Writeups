# _WACTF2 2018 FORENSICS Writeups - Shyamal Varsani (**@baconandegg5**)_

## Forensics-0 I'm lovin it
### Challenge Prompt:
>here is an image in this folder. Can you tell me its SHA-1 value?

### Writeup:
 For the forensics-0 challenge in WACTF2, we were given a jpeg file in which we were required to give a SHA1 hash of

To do this, I used `sha1sum`, which is a tool that prints (to stdout) or checks SHA1 (160-bit) checksums.

This was a fairly straightfoward procedure, which can be seen below:

```
root@kali:~/Desktop/Forensics-0# sha1sum delicious.jpg 
abf03b17013e5aa440ba8c5cac11c4dec18e4084  delicious.jpg
```
This digest in conjuction with the flag format, gave us the flag:

*WACTF2{abf03b17013e5aa440ba8c5cac11c4dec18e4084}*

---
## Forensics-1 SMS-forensics-challenge
### Challenge Prompt:
>An SMS database has been extracted from an Android device under investigation. Use sqlite3 to view the SMS records and find the password.
### Writeup:
For the forensics-1 challenge in WACTF2, we were presented with a SQLite 3 database in which we had to find the user's password

Used the `file` command to try and determine what I was dealing with:
```
root@kali:~/Desktop/Forensics-1# file mmssms.db
mmssms.db: SQLite 3.x database, user version 67, last written using SQLite version 3022000 
```
I had learnt about another common service previously running on Linux system called MySQL during the completion of the PentesterLab's Unix Badge. As I expected the SQLite CLI to be similiar to I deployed similiar tactics to how I would access a MySQL database.

I initially opened the database using `sqlite3`:

```
root@kali:~/Desktop/Forensics-1# sqlite3 mmssms.db 
SQLite version 3.25.3 2018-11-05 20:37:38
Enter ".help" for usage hints.
sqlite> .tables
addr                 pdu_restricted       threads            
android_metadata     pending_msgs         words              
attachments          rate                 words_content      
canonical_addresses  raw                  words_segdir       
drm                  sms                  words_segments     
part                 sms_restricted     
pdu                  sr_pending         
sqlite> select * from sms;
1|3|0491570156||1540429912462|0||1|-1|2|||Hi||0|1|0|com.google.android.apps.messaging|1
2|3|0491570156||1540429914550|0||1|-1|2|||Hello||0|1|0|com.google.android.apps.messaging|1
3|3|0491570156||1540429918070|0||1|-1|2|||calm down||0|1|0|com.google.android.apps.messaging|1
4|3|0491570156||1540430379138|0||1|-1|2|||keep this password a secret:||0|1|0|com.google.android.apps.messaging|1
5|3|0491570156||1540430461839|0||1|-1|2|||Siliciumphantomplant||0|1|0|com.google.android.apps.messaging|1
6|3|0491570156||1540430465667|0||1|-1|2|||Thanks||0|1|0|com.google.android.apps.messaging|1

```
Within this database, I found a number of tables that existed using `sqlite> .tables`.

I then saw the _sms_ table, and thought that the password would have communicated via sms, so i `sqlite> select * from sms;` (select all data from the sms table), which displayed a series of messages.

It was possible to see that six messages had either been sent/recived, with the fifth one being the password which was: Siliciumphantomplant

*WACTF2{Siliciumphantomplant}*

---

## Forensics-2 It's GIF not JIF
### Challenge Prompt:
>I put my favorite GIF in an archive for safe keeping but it seems to be corrupt! I bet it was my coworker who thinks it's pronounced 'JIF'.
### Writeup:

I initially went on to extract the 'mybeloved.tar' archive that was supplied to us, which gave the output of 3 .txt files (Flash_Animation.txt,GIF.txt and Meme.txt). I spent a while looking at these text files trying to discern if anything fishy was going on them but I had hit a wall for a period of time. 

I tried using `binwalk` on the tar archive, which resulted in the following:
```
root@kali:~/Desktop/Forensics-2# binwalk mybeloved.tar 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             POSIX tar archive, owner user name: "imation.txt"

```

All binwalk had shown here was that this file was a POSIX tar archive, but what had caught my eye was the "imation.txt". I spent the next half hour trying to see if i could find the *imation.txt* in the .tar file. 

I then pestered the don @Snide during Pizza time on Day 1 (props for creating this challenge). I  attempted to ask for any hints, and he told me to _Try Harder_ && _Look at it differently dude_. I then tried a bunch of things to avail, until I realized that I hadn't done the most basic of stuff with the .tar archive.

I then went to `strings` the .tar file, where I found the following:

```
<SNIPPED>
heritage.jif
0100777
0000000
0000000
00004263244
13367576604
010333
ustar
JIF89a@
( #1"
%$%#.%
,,+.-
&)y>.
'5%06
342<6
<SNIPPED>
```
This was the .jif file the challenge prompt was talking about!

The attempt that was successful in pulling out this file was to run `bless` (hex editor) on the .tar file. 

In bless I deleted everything that came before (upto the 4A 49 46 38 39 61) header and everything that was after the heritage.gif portion starting from the next file's header.

Before saving this block of bytes/file, the header had to be changed from 4A 49 46 38 39 61 (JIF89a) >> 47 49 46 38 39 61 (GIF89a).

I then saved this file (as blessed.gif), and it opened as a valid .gif when using `eog` (eye-of-gnome/image viewer). When viewing the gif, it was possible to some quick flashes/frames that looked out of norm. 

Using `ffmpeg` (image and audio manipulator), to pull out every single individual frame out of the GIF for inspection, by doing the following:
`ffmpeg -i blessed.gif thumb%04d.jpg -hide_banner`

This resulted in a total of 152 frames being extracted, and when using `eog` to open all of them, I found the flag in classic Snide fashion with a Haunter Pokemon background on frame 121.

*WACTF2{H3y_Ya!!!}*

---
## Forensics-3 Hiding in plain sight 
### Challenge Prompt:
>An attacker has stolen a sensitive document. We have compromised her proxy and recovered the network activity from the time of the attack. Can you find the file she stole? (The flag is in the file.)

### Writeup:
>In this challenge, we were given a packet capture, which we were told some secret communication of somesort was taking place? Strange mane?

I opened up the provided *forensics3.pcapng* in `wireshark` (packet capture multitool) to analyze and sift through the network traffic that had been captured.

The initial look in wireshark didn't immediately show anything that stood out, so i binwalked the .pcapng to pull out everything that has a valid header from it, using:
`Binwalk --dd='.*' forensics3.pcapng`

I was looking through the extracted files, and found that there was this png diagram talking about TCP over ICMP.

I researched more about IP/TCP over ICMP, and learnt that you could use ping/ICMP packets to dissassamble/send/recieve IP/TCP stuff over pings with requests and replies.

When Inspecting the ICMP packets and applying a filter 'icmp', so that you view ICMP packets, I was able to determine that not all of the ICMP were alike:

The data portions of various ICMP packets revealed the following:

```
1. (packet 5146)
RT¦ÁwEç*þ®ÙÀ¨zÀ¨z9U?ÜJê¹EËu@@1
²fÛ»W¢ÙOöÇ¦à÷
;LªdGET /347895z HTTP/1.1
User-Agent: Wget/1.19.4 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 178.128.102.4:443
Connection: Keep-Alive

2. (packet 5149)
RTrêEÜòAÿMÍÀ¨z9À¨zJpã©ûEÀjÖ@3³Û²f
»ÛOöÇ·W¢Ù¦ëd·
ªr;LServer: SimpleHTTP/0.6 Python/2.7.13
Date: Fri, 05 Oct 2018 01:21:54 GMT
Content-type: application/octet-stream
Content-Length: 2197
Last-Modified: Fri, 05 Oct 2018 01:10:51 GMT

cm9vdDokNiRJYzVGQmZMRiRXQUNURjJ7QkRFQThDQzU4MkEwNjRGOTUyOTI2RDI1NDgzRkExQTF9
OjE3ODAyOjA6OTk5OTk6Nzo6OgpkYWVtb246KjoxNzc4NTowOjk5OTk5Ojc6OjoKYmluOio6MTc3
ODU6MDo5OTk5OTo3Ojo6CnN5czoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpzeW5jOio6MTc3ODU6MDo5
OTk5OTo3Ojo6CmdhbWVzOio6MTc3ODU6MDo5OTk5OTo3Ojo6Cm1hbjoqOjE3Nzg1OjA6OTk5OTk6
Nzo6OgpscDoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgptYWlsOio6MTc3ODU6MDo5OTk5OTo3Ojo6Cm5l
d3M6KjoxNzc4NTowOjk5OTk5Ojc6OjoKdXVjcDoqOjE3Nzg1OjA6OTk5OTk6Nzo6Ogpwcm94eToq
OjE3Nzg1OjA6OTk5OTk6Nzo6Ogp3d3ctZGF0YToqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpiYWNrdXA6
KjoxNzc4NTowOjk5OTk5Ojc6OjoKbGlzdDoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgppcmM6KjoxNzc4
NTowOjk5OTk5Ojc6OjoKZ25hdHM6KjoxNzc4NTowOjk5OTk5Ojc6OjoKbm9ib2R5Oio6MTc3ODU6
MDo5OTk5OTo3Ojo6Cl9hcHQ6KjoxNzc4NTowOjk5OTk5Ojc6OjoKc3lzdGVtZC10aW1lc3luYzoq
OjE3Nzg1OjA6OTk5OTk6Nzo6OgpzeXN0ZW1kLW5ldHdvcms6KjoxNzc4NTowOjk5OTk5Ojc6OjoK
c3lzdGVtZC1yZXNvbHZlOio6MTc3ODU6MDo5OTk5OTo3Ojo6Cm15c3FsOiE6MTc3ODU6MDo5OTk5
OTo3Ojo6CkRlYmlhbi1leGltOiE6MTc3ODU6MDo5OTk5OTo3Ojo6CnV1aWRkOio6MTc3ODU6MDo5
OTk5OTo3Ojo6CnJ3aG9kOio6MTc3ODU6MDo5OTk5OTo3Ojo6CnJlZHNvY2tzOiE6MTc3ODU6MDo5
OTk5OTo3Ojo6CmVwbWQ6KjoxNzc4NTowOjk5OTk5Ojc6OjoKdXNibXV4Oio6MTc3ODU6MDo5OTk5
OTo3Ojo6Cm1pcmVkbzoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpudHA6KjoxNzc4NTowOjk5OTk5Ojc6
Ojo

3. (Packet 5152)
RTrêEFáÿú÷À¨z9À¨zéÂb|Eöj×@3µ¤²f
»ÛOöÍCW¢Ù¦ë0=
ªr;LKcG9zdGdyZXM6KjoxNzc4NTowOjk5OTk5Ojc6OjoKZG5zbWFzcToqOjE3Nzg1OjA6OTk5OTk6
Nzo6OgptZXNzYWdlYnVzOio6MTc3ODU6MDo5OTk5OTo3Ojo6CmlvZGluZToqOjE3Nzg1OjA6OTk5
OTk6Nzo6OgphcnB3YXRjaDohOjE3Nzg1OjA6OTk5OTk6Nzo6OgpEZWJpYW4tc25tcDohOjE3Nzg1
OjA6OTk5OTk6Nzo6OgpzdHVubmVsNDohOjE3Nzg1OjA6OTk5OTk6Nzo6OgpydGtpdDoqOjE3Nzg1
OjA6OTk5OTk6Nzo6Ogpzc2xoOiE6MTc3ODU6MDo5OTk5OTo3Ojo6CmluZXRzaW06KjoxNzc4NTow
Ojk5OTk5Ojc6OjoKc3NoZDoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpzcGVlY2gtZGlzcGF0Y2hlcjoh
OjE3Nzg1OjA6OTk5OTk6Nzo6Ogpjb3VjaGRiOio6MTc3ODU6MDo5OTk5OTo3Ojo6CmdsdXN0ZXI6
KjoxNzc4NTowOjk5OTk5Ojc6OjoKZ2VvY2x1ZToqOjE3Nzg1OjA6OTk5OTk6Nzo6Ogpjb2xvcmQ6
KjoxNzc4NTowOjk5OTk5Ojc6OjoKc2FuZWQ6KjoxNzc4NTowOjk5OTk5Ojc6OjoKYXZhaGk6Kjox
Nzc4NTowOjk5OTk5Ojc6OjoKcHVsc2U6KjoxNzc4NTowOjk5OTk5Ojc6OjoKZHJhZGlzOio6MTc3
ODU6MDo5OTk5OTo3Ojo6CmtpbmctcGhpc2hlcjoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpiZWVmLXhz
czoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpEZWJpYW4tZ2RtOio6MTc3ODU6MDo5OTk5OTo3Ojo6CnN5
c3RlbWQtY29yZWR1bXA6ISE6MTc4MDI6Ojo6OjoK
```
It was obvious from looking at the data that there is base64 encoded data hidden in these packets. As the data had been split up into 3 part, it was neccesary to reconstitute the parts back together, before decoding it:

```
m9vdDokNiRJYzVGQmZMRiRXQUNURjJ7QkRFQThDQzU4MkEwNjRGOTUyOTI2RDI1NDgzRkExQTF9
OjE3ODAyOjA6OTk5OTk6Nzo6OgpkYWVtb246KjoxNzc4NTowOjk5OTk5Ojc6OjoKYmluOio6MTc3
ODU6MDo5OTk5OTo3Ojo6CnN5czoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpzeW5jOio6MTc3ODU6MDo5
OTk5OTo3Ojo6CmdhbWVzOio6MTc3ODU6MDo5OTk5OTo3Ojo6Cm1hbjoqOjE3Nzg1OjA6OTk5OTk6
Nzo6OgpscDoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgptYWlsOio6MTc3ODU6MDo5OTk5OTo3Ojo6Cm5l
d3M6KjoxNzc4NTowOjk5OTk5Ojc6OjoKdXVjcDoqOjE3Nzg1OjA6OTk5OTk6Nzo6Ogpwcm94eToq
OjE3Nzg1OjA6OTk5OTk6Nzo6Ogp3d3ctZGF0YToqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpiYWNrdXA6
KjoxNzc4NTowOjk5OTk5Ojc6OjoKbGlzdDoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgppcmM6KjoxNzc4
NTowOjk5OTk5Ojc6OjoKZ25hdHM6KjoxNzc4NTowOjk5OTk5Ojc6OjoKbm9ib2R5Oio6MTc3ODU6
MDo5OTk5OTo3Ojo6Cl9hcHQ6KjoxNzc4NTowOjk5OTk5Ojc6OjoKc3lzdGVtZC10aW1lc3luYzoq
OjE3Nzg1OjA6OTk5OTk6Nzo6OgpzeXN0ZW1kLW5ldHdvcms6KjoxNzc4NTowOjk5OTk5Ojc6OjoK
c3lzdGVtZC1yZXNvbHZlOio6MTc3ODU6MDo5OTk5OTo3Ojo6Cm15c3FsOiE6MTc3ODU6MDo5OTk5
OTo3Ojo6CkRlYmlhbi1leGltOiE6MTc3ODU6MDo5OTk5OTo3Ojo6CnV1aWRkOio6MTc3ODU6MDo5
OTk5OTo3Ojo6CnJ3aG9kOio6MTc3ODU6MDo5OTk5OTo3Ojo6CnJlZHNvY2tzOiE6MTc3ODU6MDo5
OTk5OTo3Ojo6CmVwbWQ6KjoxNzc4NTowOjk5OTk5Ojc6OjoKdXNibXV4Oio6MTc3ODU6MDo5OTk5
OTo3Ojo6Cm1pcmVkbzoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpudHA6KjoxNzc4NTowOjk5OTk5Ojc6
OjoLKcG9zdGdyZXM6KjoxNzc4NTowOjk5OTk5Ojc6OjoKZG5zbWFzcToqOjE3Nzg1OjA6OTk5OTk6
Nzo6OgptZXNzYWdlYnVzOio6MTc3ODU6MDo5OTk5OTo3Ojo6CmlvZGluZToqOjE3Nzg1OjA6OTk5
OTk6Nzo6OgphcnB3YXRjaDohOjE3Nzg1OjA6OTk5OTk6Nzo6OgpEZWJpYW4tc25tcDohOjE3Nzg1
OjA6OTk5OTk6Nzo6OgpzdHVubmVsNDohOjE3Nzg1OjA6OTk5OTk6Nzo6OgpydGtpdDoqOjE3Nzg1
OjA6OTk5OTk6Nzo6Ogpzc2xoOiE6MTc3ODU6MDo5OTk5OTo3Ojo6CmluZXRzaW06KjoxNzc4NTow
Ojk5OTk5Ojc6OjoKc3NoZDoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpzcGVlY2gtZGlzcGF0Y2hlcjoh
OjE3Nzg1OjA6OTk5OTk6Nzo6Ogpjb3VjaGRiOio6MTc3ODU6MDo5OTk5OTo3Ojo6CmdsdXN0ZXI6
KjoxNzc4NTowOjk5OTk5Ojc6OjoKZ2VvY2x1ZToqOjE3Nzg1OjA6OTk5OTk6Nzo6Ogpjb2xvcmQ6
KjoxNzc4NTowOjk5OTk5Ojc6OjoKc2FuZWQ6KjoxNzc4NTowOjk5OTk5Ojc6OjoKYXZhaGk6Kjox
Nzc4NTowOjk5OTk5Ojc6OjoKcHVsc2U6KjoxNzc4NTowOjk5OTk5Ojc6OjoKZHJhZGlzOio6MTc3
ODU6MDo5OTk5OTo3Ojo6CmtpbmctcGhpc2hlcjoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpiZWVmLXhz
czoqOjE3Nzg1OjA6OTk5OTk6Nzo6OgpEZWJpYW4tZ2RtOio6MTc3ODU6MDo5OTk5OTo3Ojo6CnN5
c3RlbWQtY29yZWR1bXA6ISE6MTc4MDI6Ojo6OjoK
```

Once the above was decoded by echoing the base64 text/string into `base64 -d`, the following was output:

```
root:$6$Ic5FBfLF$WACTF2{BDEA8CC582A064F952926D25483FA1A1}:17802:0:99999:7:::
daemon:*:17785:0:99999:7:::
bin:*:17785:0:99999:7:::
sys:*:17785:0:99999:7:::
sync:*:17785:0:99999:7:::
games:*:17785:0:99999:7:::
man:*:17785:0:99999:7:::
lp:*:17785:0:99999:7:::
mail:*:17785:0:99999:7:::
news:*:17785:0:99999:7:::
uucp:*:17785:0:99999:7:::
proxy:*:17785:0:99999:7:::
www-data:*:17785:0:99999:7:::
backup:*:17785:0:99999:7:::
list:*:17785:0:99999:7:::
irc:*:17785:0:99999:7:::
gnats:*:17785:0:99999:7:::
nobody:*:17785:0:99999:7:::
_apt:*:17785:0:99999:7:::
systemd-timesync:*:17785:0:99999:7:::
systemd-network:*:17785:0:99999:7:::
systemd-resolve:*:17785:0:99999:7:::
mysql:!:17785:0:99999:7:::
Debian-exim:!:17785:0:99999:7:::
uuidd:*:17785:0:99999:7:::
rwhod:*:17785:0:99999:7:::
redsocks:!:17785:0:99999:7:::
epmd:*:17785:0:99999:7:::
usbmux:*:17785:0:99999:7:::
miredo:*:17785:0:99999:7:::
ntp:*:17785:0:99999:7:7:::
messagebus:*:17785:0:99999:7:::
iodine:*:17785:0:99999:7:::
arpwatch:!:17785:0:99999:7:::
Debian-snmp:!:17785:0:99999:7:::
stunnel4:!:17785:0:99999:7:::
rtkit:*:17785:0:99999:7:::
sslh:!:17785:0:99999:7:::
inetsim:*:17785:0:99999:7:::
sshd:*:17785:0:99999:7:::
speech-dispatcher:!:17785:0:99999:7:::
couchdb:*:17785:0:99999:7:::
gluster:*:17785:0:99999:7:::
geoclue:*:17785:0:99999:7:::
colord:*:17785:0:99999:7:::
saned:*:17785:0:99999:7:::
avahi:*:17785:0:99999:7:::
pulse:*:17785:0:99999:7:::
dradis:*:17785:0:99999:7:::
king-phisher:*:17785:0:99999:7:::
beef-xss:*:17785:0:99999:7:::
Debian-gdm:*:17785:0:99999:7:::
systemd-coredump:!!:17802::::::
```

The file that can be seen above is the /etc/~shadow file, this can be distinguished by the asterisks after the user's names, indicating the hashes have been shadowed/hidden.
Contrary to this, we can see by observation that in the place where the users hashes are shadowed, we can find the roots 'hash' contains the flag for this challenge.

*WACTF2{BDEA8CC582A064F952926D25483FA1A1}*

---

## Forensics-4 Opr8t3r
### Challenge Prompt:
>We have acquired a mobile device we suspect is being used to stores a passphrase via encrypted services. Can you find it?

### Writeup:
> This is a challenge where we were given 2 disk images (dm-0.img & dm-1.img) and a android backup to analyze, in the hopes of finding out about some sort of nefarious activity using a encrypted sms system of some sort? 

This challenge was done fairly simply using Autopsy 4.9.1 after multiple efforts with Autopsy 2.24 were undertaken.

I had initally loaded up both disk images in Autopsy 2.24 on my Kali VM and traversing through the file system trying to locate anything that looked suspicious in nature, that would imply that anything to do with a encrypted sms service. A keyword search in Autopsy 2.24 of 'WACTF' lead me to a little png WACTF logo which was located /0/media/download. I found an .apk (android application)file called thoughtcrime.securesms.421.apk on the dm-1.img. Note - I never found anything of too much value on dm-0.img

I ran `apktool` on the thoughtcrime.securesms.421.apk to attempt to disassemble it and traverse through the backend of the application in the hopes of a key/flag being hidden internally, within the application, but that just proved to be a massive red herring in the end?

After many,many hours, I had loaded up the images into v4.9.1 on a Windows VM and found that when I did a keyword search of 'wactf', a database file by the name of *msgstore.db*, which resided in 
**/img_dm-1.img/user/0/com.whatsapp/databases/msgstore.db** was found to contain a flag of:

*WACTF2{0P$3c_Ab0ve_A77}*

>>I will keep in mind for next time I undertake any sort of forensics on disk images that Autopsy 4.9.1 is much more powerful than v2.24 ;) and i should never use the version that comes with kali natively ;) 

---

All in all, I learnt a bunch of lessons completing these challenges! and I hope to compete in WACTF once again in the years to come :)

