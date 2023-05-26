# TryHackMe CTF: Pickle Rick | Writeups

## 0.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)


A Rick and Morty CTF. Help turn Rick back into a human!
https://tryhackme.com/room/picklerick

### 26-05-2023
### Level Of Difficulty : Easy

## Introduction

This Rick and Morty-themed challenge requires you to exploit a web server and find three ingredients to help Rick make his potion and transform himself back into a human from a pickle.

# Resources/Tools Used

* nmap
* gobuster/dirb

# Scanning & Enumeration

First, we have to scan the target machine(webserver)

`nmap -sC -sV -Pn <target_IP>`

## 1.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

We can see that port 22 and 80 is open. Port 22 is for ssh service to connect to the target machine by the remote client. And Port 80 is for web service let's open the target website on the browser and look for further clues.

## 2.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

the website doesn't show anything important. Now we can look for the Page source code.

## 3.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

Sweet, we have a username! Now we need to hunt around for the password. Always a good idea to enumerate the website with something like dirb or gobuster, or a tool of your choice:

`dirb http://10.10.90.88 `

## 4.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

Using drb we find some interesting pages!  Let's give them a visit.

## 5.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

That looks like it can be useful... I'll keep it in the back pocket for now and check out the other directories we found.

There is another login.php directory which is discovered by another tool during directory Enumeration. let's visit it.

## 6.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

Hmm... A login page, and we already know the username.  Let's see if what we found on robots.txt is the password:


## 7.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

It is! We have logged in.

There is a Command panel look's like we can input any command which will execute in the web server and display the output on the screen.

Let's try to gather more information.

`ls -al`

## 8.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

look's like there an interesting file name `Sup3rS3cretPickl3Ingred.txt`
Let's try to read that file:

## 10.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

Bummer looks like we're not allowed to use the 'cat' command.

I do notice, however, that the other files in the current directory are: robots.txt, login.php, etc.  We were able to reach those by navigating on the website earlier, so maybe we can do the same for the ingredient:

## 9.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

We have found our first ingredient!

-------------------------------------------------------

# Ingredients 2 & 3

In our current directory there is a clue.txt file which can tell as about other ingredients. Let's open it.

## 11.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

Ah, so we know that the other ingredient is also in the file system! We don't know what it's called, so we can't do a find, unfortunately.  I took a guess that it'd be in the user folder and happened to be right:

`ls -al /home/rick`

## 12.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

We have found the second ingredient. As we try to read the file of the second ingredient but the cat , nano, more etc command is disable.

But there is a command `less` we can use. which is not restricted to use.

`less /home/rick/"second ingredients"`

## 13.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

So. there is our second ingredient. Now we have to find the last ingredient.

We have to look around the root directory. But as a now user we can't traverse the root directory.

First we have to found Out our sudo privilege were we can use it.

`sudo -l`

## 14.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

From that we can see we can do anything!

`sudo la -al /root`

## 15.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

With the use of sudo we can read the root directory file.

`sudo less /root/3rd.txt`

## 16.png

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)


Finally we have got the third ingredient!





## Screenshots

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

