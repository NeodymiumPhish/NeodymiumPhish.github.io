---
layout: post
title: CTF Pocket Guide
redirect_from:
    - CTF/
---

CTF Pocket Guide
===

> Keep these tools handy and check their man pages for help if you need ideas to get you through obtaining flags throughout the CTF!

---
# Setup Virtual Environment

__Dragons__, the Target VM is available to download [here](https://transfer.sh/T54cl/3FIS_CTF.zip), and the password to decrypt the file will become available the morning of the event. Please download the file in advance, so we can expedite the setup process before the event begins.

## VirtualBox & Internal Network
In order to prepare your environment for the competition, ensure that you download and install the following:

* [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
* [VirtualBox Extension Pack](https://download.virtualbox.org/virtualbox/6.1.18/Oracle_VM_VirtualBox_Extension_Pack-6.1.18.vbox-extpack)

Secondly, once you have installed VirtualBox, you need to open Command Prompt (or Terminal on MacOS) and run the following command:
__Windows:__
```cmd
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" dhcpserver add --netname ctf --ip 10.10.10.1 --netmask 255.255.255.0 --lowerip 10.10.10.100 --upperip 10.10.10.200 --enable
```

__Mac:__
```bash
vboxmanage dhcpserver add --netname ctf --ip 10.10.10.1 --netmask 255.255.255.0 --lowerip 10.10.10.100 --upperip 10.10.10.200 --enable
```

This will create a network that prevents you from accidentally scanning/interacting with your home network/devices.

---
## Attack Box
Next, download your attack system of choice. This can be whatever you like, although the recommended system (in case you want to make it easy to Google for instructions) in Kali. Here are a few links:

[Kali VirtualBox Image](https://images.kali.org/virtual-images/kali-linux-2021.1-vbox-amd64.ova) - Default login: kali/kali

[ParrotOS VirtualBox Image](https://download.parrot.sh/parrot/iso/4.10/Parrot-home-4.10_virtual.ova) - Default login: user/toor

[SANS Slingshot OS (Requires SANS Account)](https://www.sans.org/slingshot-vmware-linux/download) - Default login: slingshot/slingshot

Once the VM is installed, turn off the VM and open the VM's settings in VirtualBox. Under `Network`, make sure one of your Adapters is set according to the image below[^1]:

![](/assets/img/Pasted%20image%2020210325145006.png)

[^1]: If you want to be able to get to the internet from your attack box, for things like Googling issues, etc, you'll want to set one adapter to `Attached to: NAT`. For example, I keep my Parrot OS VM set with `Adapter 1: NAT` and `Adapter 2: Internal Network - ctf`

---
## Port Scans
Use Nmap or Nessus to scan for open ports/services. Check out my [Beginner's Guide to Nmap](/Nmap-Beginners/Guide/) for some common commands and tips!

---
## Directory Searches
Dirsearch, GoBuster, DirBuster are all tools to help you enumerate directories and pages visible on a given website. For example, using lists from the `/usr/share/dirbuster/wordlists` directory is usually good enough for finding the majority of useful directories on a page.

---
## Hydra/xHydra
Hydra is a great tool for brute-forcing passwords. xHydra is the easier-to-use tool (installed in most security-focused Linux distributions), since it provides an interface with a lot of fill-in-the-blank sections to help guide you in building the Hydra command.

---
## John The Ripper
John is an excellent resource for cracking passwords on local files. There are many modules for John, such as:
`7z2john`: which provides a John-crackable hash from a 7-zip archive. `pdf2john`, `bitcoin2john`, and `ssh2john` all do the same for their respective file types.
Once you have a file hash, you can run `john --wordlist=<location of wordlist file> <hash file name>` to crack the hash

---
## Burp Suite
Burp Suite provides an excellent internal proxy where you can grab input/output before it gets to the destination and modify data. You can also use this to attempt a brute-force password attack against a web interface, through the use of Burp Suite's "Intruder" module.

---
## SearchSploit/Exploit-DB
SearchSploit is a command-line tool that checks your query against the `exploitdb` folder (`/usr/share/exploitdb/exploits`) and returns all results, usually with the specific version numbers that are affected by the exploit, along with the file path to the exploit and further details about its use.

---
## Python's `pty.spawn`
Some CTFs provide you a method of gaining a shell with severe limits on available commands. Often, the ability to do traditional bash commands, like `ls`, `cat`, and even `cd` are disabled. The `pty` module in Python is a great way of forcing the bash shell to run.
`$ python3 -c "import pty;pty.spawn('/bin/bash')"`

---
## Important bash Commands
There are many useful bash commands that can be used/abused to gain better access within an environment.

`find / -user root -perm -4000 -print 2>/dev/null` - Displays all files that are owned by root but can be executed by any user. These aren't all exploitable, but some tools, like Vim, Nmap, bash, and nano, can be used to allow privilege escalation. For useful examples of how these and other executables can be abused to escalate privileges, [check here](https://pentestlab.blog/category/privilege-escalation/).

`sudo -l` - Lists commands that the current Linux user can execute with sudo privileges

`bash -p` - With access to the bash command as sudo (for example, if you are able to copy `/bin/bash` to /tmp), `bash -p` will run bash without a user id set, meaning you can execute commands at root level[^2].

[^2]: This is probably the wrong way to explain it. The `man` page says "If the -p option is supplied at invocation, the startup behavior is the same, but the effective user id is not reset.". I interpreted this to mean that it effectively runs with root privileges.
