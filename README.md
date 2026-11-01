# TryHackMe - RootMe Write-up

Hey everyone! Here’s my detailed walkthrough for the "RootMe" room on TryHackMe. This room is a fantastic beginner-friendly CTF that focuses on reconnaissance, bypassing file upload filters, and exploiting SUID misconfigurations for privilege escalation.

## 1. Reconnaissance (Nmap Scan)

I kicked things off with a standard Nmap service scan to identify open ports and the services running on the target machine:


nmap -sV 10.112.179.155

<img width="755" height="206" alt="hklhf" src="https://github.com/user-attachments/assets/5b5fc971-3fb5-48a7-a13f-34016bd9d938" />

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
As we can see, we have SSH (port 22) and an Apache HTTP web server (port 80) open. Since I didn't have any SSH credentials, I targeted the web server.

2. Web Enumeration & Directory Fuzzing
To find hidden directories on the web server quickly, I used Gobuster with a standard wordlist and increased the threads for speed:

gobuster dir -u http://10.112.179.155 -w /usr/share/wordlists/dirb/common.txt -t 50

<img width="702" height="444" alt="قنبوستر" src="https://github.com/user-attachments/assets/5a890ef3-c3dd-4973-a001-f6f1b8864e7e" />

The scan revealed two highly interesting directories:

/panel/ -> A hidden upload form page.

/uploads/ -> A directory where uploaded files are stored.

3. Bypassing File Upload Filter & Getting a Reverse Shell
I navigated to [http://10.112.179.155/panel/](http://10.112.179.155/panel/) and found an upload form. The goal here was to upload a PHP Reverse Shell to gain remote access.

I grabbed the standard php-reverse-shell.php from Kali Linux, but the application had a filter blocking .php extensions. I bypassed this restriction by renaming my payload to shell.php5.

After modifying the $ip inside the script to my TryHackMe VPN IP, I set up a Netcat listener on my machine:

-lvnp 1234

I uploaded shell.php5 successfully, then navigated to

<img width="805" height="151" alt="38 اتصال" src="https://github.com/user-attachments/assets/febc3ad1-db95-465d-9a06-1268c27e4741" />

Boom! I caught the reverse shell as the www-data user.
After exploring the system, I found and read the first flag:

find / -type f -name user.txt 2>/dev/null
cat /var/www/user.txt

4. Privilege Escalation (Exploiting SUID)
To get the final root flag, I needed to escalate my privileges. I started by searching for files with SUID permissions enabled:

find / -type f -perm -04000 2>/dev/null

Scanning through the list, one binary stood out immediately as highly unusual: /usr/bin/python2.7

Since Python had SUID privileges, I could use it to spawn a new shell that would inherit the root permissions. I executed the following one-liner:

/usr/bin/python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'

<img width="529" height="90" alt="روت" src="https://github.com/user-attachments/assets/e9e6ddbe-9d4d-4632-947b-1a73fc264c4b" />

Running whoami confirmed I was now root. Finally, I navigated to the root directory and grabbed the final flag:

cat /root/root.txt

And that’s the box fully compromised!
