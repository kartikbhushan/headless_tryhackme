# Headless Hacking Machine Rough Walkthrough

Author : Kartik Bhushan
Created on : 18-12-2021 

This repository contains the files and the notes for the the tryhackme Room - [Headless](https://tryhackme.com/room/headless) created by [Cyberearth](https://tryhackme.com/p/cyberbreath).

## Machine IP - 10.10.110.189 
Yours might be a bit different

## Ennumeration - Using Nmap 
This is just the main Snippet of the Scan the full detailed scan file is named headless_nmap.txt
```
Nmap command in use - nmap -sS -A -T4 -p- 10.10.110.189 -oN headless_nmap.txt  
Nmap scan report for 10.10.110.189
Host is up (0.14s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.17.35.42
|      Logged in as ftp
|      TYPE: ASCII
|      Session bandwidth limit in byte/s is 2048000
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.2 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxr-xr-x    2 ftp      ftp          4096 Dec 12 00:41 11:11pm
| drwxr-xr-x    2 ftp      ftp          4096 Dec 11 12:24 cisco
| drwxr-xr-x    2 ftp      ftp          4096 Dec 11 12:25 juniper
|_drwxr-xr-x    2 ftp      ftp          4096 Dec 12 01:11 n33l3sh
80/tcp   open  http        Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
5522/tcp open  ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
```
## SMB Share Enumeration
Additional nmap to enumerate deeper and find more about the SMB shares.
```
sudo nmap -p 445 --script smb-enum-shares 10.10.110.189

Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-18 12:34 EST
Nmap scan report for 10.10.110.189
Host is up (0.15s latency).
PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.110.189\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (Centos server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.110.189\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>

Nmap done: 1 IP address (1 host up) scanned in 16.83 seconds

Finding - Anon Login is allowed 
```

## Web Enumeration - Enumerating the Apache webpage on PORT 80
```
Command - 
sudo gobuster dir -u http://10.10.110.189 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt

```
You can find different word list preinstalled (Kali Linux in my case) under - /usr/share/wordlist/dirbuster/

Did not find anyting intresting in this case.

### The Box time ended so I had to restart Hence the New IP address - 10.10.231.191

## Anonymous login into the FTP server 
Since we can see from the Nmap scan that the Ftp server allows anonymous login we can try to scavenge for some files.
```
ftp 10.10.231.191

Connected to 10.10.231.191.
220 Welcome to blah FTP service.
Name (10.10.231.191:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Dec 12 00:41 11:11pm
drwxr-xr-x    2 ftp      ftp          4096 Dec 11 12:24 cisco
drwxr-xr-x    2 ftp      ftp          4096 Dec 11 12:25 juniper
drwxr-xr-x    2 ftp      ftp          4096 Dec 12 01:11 n33l3sh
226 Directory send OK.

Note - We can fetch files from ftp by using the "get" command.

Going through all the directories - Findings
11:11pm - goodiya.txt.gpg
		- happiness.jpeg

Hidden .readme.md file "ls -la" revealed it out 
n33l3sh - .README.md
```

## Hidden .README.md  File - Intresting Content -
```
                                             // h3atwav3s//
???:I just wonder what you're dreaming of
When you sleep and smile so comfortable
I just wish that i could give you that
That look that's perfectly un-sad.

goodiya: wait.. i think I knew this song!, what you want to describe...

???:I have a some hidden message in this.. for you, 

```

### Findings - 
- Song name - Glass Animals by Heat Waves

## Exploring - Happiness.jpeg
```
Tried online steagnography tool - https://futureboy.us/stegano/decinput.html

Passphrase tried - glassanimals
		 - heatwaves
		 - h3atwav3s

Passphrase Worked - h3atwav3s
Extrated Text - {SECRET STRING}
```

Intially looked Base64 so check it on cyberchef - Error - Data is not a valid byteArray: [48,70,83,27,162,83,49...

Finally landed upon a function called "Magic" - "The Magic operation attempts to detect various properties of the input data and suggests which operations could help to make more sense of it."

```
Found the Base32 decoding - From_Base32('A-Z2-7=',false)
Decoded String - a33y3fu:{DECODED PASSWORD}
```

## Further Decoding the Base32 String - 

The Base32 string looked like a rot algorithm(Rotation cipher) because I had come across a directory called : "n33l3sh" - "a33y3fu"

Rot13 Decoding - n33l3sh:{THEPASSWORD} .

This looked like a user login with password :) lets gooo.


## Getting the flag - 

Looking at the previous nmap scan there was a port running 
- OpenSSH on Port : 5522

Trying to Login in via SSH using the port 5522
```
ssh n33l3sh@10.10.231.191 -p 5522          

The authenticity of host '[10.10.231.191]:5522 ([10.10.231.191]:5522)' can't be established.
ED25519 key fingerprint is SHA256:zTCmi/I/i2JiR6esjibnIgM3tHHB76gAeDEZ5tCMYAw.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[10.10.231.191]:5522' (ED25519) to the list of known hosts.
The Quieter you are, the More you hear!!
n33l3sh@10.10.231.191's password: 
Welcome to Ubuntu 14.04 LTS (GNU/Linux 3.13.0-24-generic x86_64)

 * Documentation:  https://help.ubuntu.com/
The Quieter you are, the More  you can hear
Last login: Sun Dec 12 02:46:03 2021
n33l3sh@Centos:~$ ls
```
Lets Go we are in time to get that Flag once and for all 

## Flag Hunt - 

```
n33l3sh@Centos:~$ ls -la
total 36
drwxr-xr-x 4 n33l3sh n33l3sh 4096 Dec 12 02:56 .
drwxr-xr-x 4 root    root    4096 Dec 11 23:57 ..
lrwxrwxrwx 1 n33l3sh n33l3sh    9 Dec 12 02:56 .bash_history -> /dev/null
-rw-r--r-- 1 n33l3sh n33l3sh  220 Dec  8 05:07 .bash_logout
-rw-r--r-- 1 n33l3sh n33l3sh 3637 Dec  8 05:07 .bashrc
drwx------ 2 n33l3sh n33l3sh 4096 Dec  8 06:49 .cache
-rw------- 1 root    root      46 Dec 11 23:56 .flag.txt
drwx------ 2 n33l3sh n33l3sh 4096 Dec 12 00:06 .gnupg
-rwxrwxrwx 1 root    root      64 Dec 12 02:51 .home.sh
-rw-r--r-- 1 n33l3sh n33l3sh  675 Dec  8 05:07 .profile

```
I finally see the Flag file - '.flag.txt'

After trying to "cat" the flag file we needed root privilegs to read the .flag.txt file.

A simple way of checking if you can sudo is just to try:
- sudo -l
- sudo -V
- sudo ls
- sudo whoami

In my case I tried "sudo whoami" and the terminal replied with **root** which meant I could run "sudo" commands.

```
n33l3sh@Centos:~$ sudo whoami
sudo: unable to resolve host Centos
root
n33l3sh@Centos:~$ sudo cat .flag.txt
sudo: unable to resolve host Centos
FLAG{THERE IS A FLAG HERE}
n33l3sh@Centos:~$ LETS GO
```

Submit your flag into the Task and you will be throught with the BOX .

Thanks to [Cyberearth](https://tryhackme.com/p/cyberbreath) for this amazing room.
