---
layout: post
title: CTF Pocket Guide
permalink: /CTF-Pocket-Guide/
---

# CTF Pocket Guide
> Keep these tools handy and check their man pages for help if you need ideas to get you through obtaining flags throughout the CTF!

### Port Scans
> Use Nmap or Nessus to scan for open ports/services

### Directory Searches
> Dirsearch, GoBuster, DirBuster are all tools to help you enumerate directories and pages visible on a given website. For example, using lists from the `/usr/share/dirbuster/wordlists` directory is usually good enough for finding the majority of useful directories on a page.

### Hydra/xHydra
> Hydra is a great tool for brute-forcing passwords. xHydra is the easier-to-use tool (installed in most security-focused Linux distributions), since it provides an interface with a lot of fill-in-the-blank sections to help guide you in building the Hydra command.

### John The Ripper
> John is an excellent resource for cracking passwords on local files. There are many modules for John, such as:
> `7z2john` - provides a John-crackable hash from a 7-zip archive
> `pdf2john`, `bitcoin2john`, and `ssh2john` all do the same for their respective file types.
> Once you have a file hash, you can run `john --wordlist=<location of wordlist file> <hash file name>` to crack the hash


### Burp Suite
> Burp Suite provides an excellent internal proxy where you can grab input/output before it gets to the destination and modify data. You can also use this to attempt a brute-force password attack against a web interface, through the use of Burp Suite's "Intruder" module.

### SearchSploit/Exploit-DB
> SearchSploit is a command-line tool that checks your query against the `exploitdb` folder (`/usr/share/exploitdb/exploits`) and returns all results, usually with the specific version numbers that are affected by the exploit, along with the file path to the exploit and further details about its use.

### Python's `pty.spawn`
> Some CTFs provide you a method of gaining a shell with severe limits on available commands. Often, the ability to do traditional bash commands, like `ls`, `cat`, and even `cd` are disabled. The `pty` module in Python is a great way of forcing the bash shell to run.
> `$ python3 -c "import pty;pty.spawn('/bin/bash')"`

### Important bash Commands
> There are many useful bash commands that can be used/abused to gain better access within an environment.

`find / -user root -perm -4000 -print 2>/dev/null` - Displays all files that are owned by root but can be executed by any user. These aren't all exploitable, but some tools, like Vim, Nmap, bash, and nano, can be used to allow privilege escalation. For useful examples of how these and other executables can be abused to escalate privileges, [check here](https://pentestlab.blog/category/privilege-escalation/).
`sudo -l` - Lists commands that the current Linux user can execute with sudo privileges
`bash -p` - With access to the bash command as sudo (for example, if you are able to copy `/bin/bash` to /tmp), `bash -p` will run bash without a user id set, meaning you can execute commands at root level[^1].

[^1]: This is probably the wrong way to explain it. The `man` page says `If the -p option is supplied at invocation, the startup behavior is the same, but the effective user id is not reset.`. I interpreted this to mean that it effectively runs with root privileges.
