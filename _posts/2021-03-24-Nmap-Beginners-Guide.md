---
layout: post
title: Beginner's Guide | Nmap
categories: [Instructional, Metasploit]
tags: [Guide]
---

# Nmap
{:.no_toc}


## Table of Contents
{:.no_toc}

* TOC
{:toc}


# Nmap - "Network Mapper"
{:.no_toc}

Nmap is a free and open source utility for network discovery and security auditing. 

` nmap [Options] {Target}`

## Choose Target
 Target can be in almost any format:

`sub.domain.com; domain.com/24; 192.168.1.1; 192.168.1.0-255; 192.168.1.0/24; `

`-iL <filename>` - reference a list of targets

`--exclude <targets>` or `--excludefile <filename>` to ignore host(s)

## Select Port

To specify one or more ports for scanning, simply use `-p` and then the port(s)

```bash
nmap -p 3389 192.168.1.1
nmap -p 22-100 192.168.1.1
nmap -p 22,135,445,3389 192.168.1.1
```

The "fast" port scan option selects the 100 most common ports

```bash
nmap -F 192.168.1.1
```

For all ports, use `-p-` to scan all 65535 ports

```bash
nmap -p- 192.168.1.1
```



## Scan Types

Nmap (when run with `sudo`) can also scan using multiple methodologies.  For example, Nmap can conduct a full TCP handshake using `-sT` or just scan UDP ports with `-sU`.  By default, Nmap will conduct a TCP SYN scan `-sS` if nothing is specified.

```bash
nmap -sT 192.168.1.1	# TCP Connect
nmap -sS 192.168.1.1	# TCP SYN
nmap -sU 192.168.1.1	# UDP scan
```



## Service and OS Detection Flags

Nmap keeps definitions of services and operating systems and uses these definitions to identify which services and systems are running on target ports.  This includes the ability to identify which version of some services are running.

```bash
nmap -A 192.168.1.1       # Detects Operating System and all Services
nmap -sV 192.168.1.1      # Detects all software and version numbers 
```



## Nmap Scripting Engine (NSE)

Nmap also has the ability to run scripts using it's very simple NSE.  By default, Nmap is updated will a great selection of scripts that can be run directly via Nmap by simply calling their name.  

Nmap's script flag can accept individual scripts from it's library in `/usr/share/nmap/scripts/` or providing just the name of a category will run all scripts that fall within that category.

```bash
nmap -sV --script smb-vuln-ms17-010 192.168.1.1     # Scans for all service versions and the checks for the EternalBlue vulnerability
nmap -sV --script vuln 192.168.1.1                  # Runs all of the vulnerability nmap scripts
nmap --script ~/custom_script.nse 192.168.1.1       # Runs custom NSE script file
nmap --script asn-query,whois 192.168.1.1           # Runs lookups against the IP address
```



## Output

Nmap understands that you may want to run scans and review the results for further information later.  As such, there are multiple options for outputting the results.

```bash
nmap -oX <file_name>.xml target.com     # Outputs results to an XML
nmap -oN <file_name>.txt target.com     # Outputs results to a txt file
nmap -oG <file_name>.txt target.com     # Outputs as txt file that's easy to grep
nmap -oA <file_name> target.com         # Outputs in all formats
```

## Conclusion

Nmap is an amazing tool with all sorts of uses. While this guide delves into some of the different ways you can utilize Nmap, understand that there is so much more it is capable of. Once you feel confident enough to start branching out and learning more, I highly suggest that you try out the TryHackMe's [In-Depth Nmap room](https://www.tryhackme.com/room/furthernmap) or [Easy Peasy Nmap/GoBuster CTF](https://www.tryhackme.com/room/easypeasyctf), as well an Nmap's own [Reference Guide](https://nmap.org/book/man.html)!