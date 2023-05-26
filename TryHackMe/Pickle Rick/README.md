# TryHackMe CTF: Pickle Rick | Writeups

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/0.PNG)


A Rick and Morty CTF. Help turn Rick back into a human!
https://tryhackme.com/room/picklerick

### 26-05-2023
### Level Of Difficulty: Easy

## Introduction

This Rick and Morty-themed challenge requires you to exploit a web server and find three ingredients to help Rick make his potion and transform himself back into a human from a pickle.

# Resources/Tools Used

* nmap
* gobuster/dirb

# Scanning & Enumeration

First, we have to scan the target machine(webserver)

`nmap -sC -sV -Pn <target_IP>`

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/1.PNG)

We can see that port 22 and 80 is open. Port 22 is for ssh service to connect to the target machine by the remote client. And Port 80 is for web service let's open the target website on the browser and look for further clues.

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/2.PNG)

the website doesn't show anything important. Now we can look for the Page source code.

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/3.PNG)

Sweet, we have a username! Now we need to hunt around for the password. Always a good idea to enumerate the website with something like dirb or gobuster, or a tool of your choice:

`dirb http://10.10.90.88 `

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/4.PNG)

Using drb we find some interesting pages!  Let's give them a visit.

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/5.PNG)

That looks like it can be useful... I'll keep it in the back pocket for now and check out the other directories we found.

There is another login.php directory that is discovered by another tool during directory Enumeration. let's visit it.

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/6.PNG)

Hmm... A login page, and we already know the username.  Let's see if what we found on robots.txt is the password:


![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/7.PNG)

It is! We have logged in.

There is a Command panel look's like we can input any command which will execute in the web server and display the output on the screen.

Let's try to gather more information.

`ls -al`

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/8.PNG)

look's like there is an interesting file name `Sup3rS3cretPickl3Ingred.txt`
Let's try to read that file:

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/10.PNG)

Bummer looks like we're not allowed to use the 'cat' command.

I do notice, however, that the other files in the current directory are: robots.txt, login.php, etc.  We were able to reach those by navigating on the website earlier, so maybe we can do the same for the ingredient:

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/9.PNG)

We have found our first ingredient!

-------------------------------------------------------

# Ingredients 2 & 3

In our current directory, there is a clue.txt file that can tell us about other ingredients. Let's open it.

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/11.PNG)

Ah, so we know that the other ingredient is also in the file system! We don't know what it's called, so we can't do a find, unfortunately.  I guessed that it'd be in the user folder and happened to be right:

`ls -al /home/rick`

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/12.PNG)

We have found the second ingredient. As we try to read the file of the second ingredient but the cat, nano, more, etc command is disabled.

But there is a command `less` we can use. which is not restricted to use.

`less /home/rick/"second ingredients"`

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/13.PNG)

So. there is our second ingredient. Now we have to find the last ingredient.

We have to look around the root directory. But as a current user, we can't traverse the root directory.

First, we have to find out our sudo privilege and where we can use it.

`sudo -l`

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/14.PNG)

From that, we can see we can do anything!

`sudo la -al /root`

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/15.PNG)

With the use of sudo, we can read the root directory file.

`sudo less /root/3rd.txt`

![App Screenshot](https://github.com/rohit00712/CTF-Writeups/blob/main/TryHackMe/Pickle%20Rick/images/16.PNG)


Finally, we have got the third ingredient!