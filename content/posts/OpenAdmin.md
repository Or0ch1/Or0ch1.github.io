---
name: "Or0ch1"
linktitle: "OpenAdmin"
title: "OpenAdmin"
Cover: "/OpenAdmin/OpenAdmin.png"
categories: ["HackTheBox"]
draft: false
toc: true
images:
tags: ['HackTheBox', 'linux']
date: 2022-09-19T08:48:53+03:00
---

Welcome to another Hack the Box writeup today we will be doing OpenAdmin.

OpenAdmin is a retired Linux box which has been rated easy and is part of the **intro to dante** Track.

Lets jump right in!

## Recon

After starting the box we get the ip as **10.10.10.171** and as always I start with nmap to scan for open ports and services running on them.

![Nmap_results]({{<static "/OpenAdmin/nmapscan.png">}})

As we can see from the results we have two ports open, port **80** for HTTP and port **22** for ssh.

## Web Enumeration

Navigating to **10.10.10.171:80** displays the default Apache2 page, nothing of interest is present so I need further enumerate.

![Apache2]({{<static "/OpenAdmin/ApacheUbuntu.png">}})

The next step is to enumerate by bruteforcing directories, I prefer using gobuster however you can user any directory bruteforcing tools available like dirb,dirsearch or feroxbuster.

![Gobuster]({{<static "/OpenAdmin/gobuster.png">}})

The music endpoint looks interesting, I navigate to it and it has the banner **Music | NOT LIVE/NOT FOR PRODUCTION USE**, clicking on login redirects us to the endpoint **10.10.10.171/ona/** which has the banner **OpenNetAdmin :: 0wn Your Network**.

![Music]({{<static "/OpenAdmin/music.png">}})

![ONA]({{<static "/OpenAdmin/ONA.png">}})

I am logged in as guest and notice an alert message stating that the version of OpenNetAdmin running is v18.1.1 which isn't the latest version.

Searching on searchsploit we can see this version is vulnerable and has a few exploits available.

![Searchsploit]({{<static "/OpenAdmin/searchsploit.png">}})

Google also provided me with a few POCs, I cloned one of them and proceeded to execute it.

![Google]({{<static "/OpenAdmin/google_results.png">}})

We get a shell as **www-data** on the machine however its not pretty so i spawn a reverse shell and connect with netcat.

![Shell]({{<static "/OpenAdmin/shell.png">}})

Running **ls** shows us the **ona** site directory and a few others,I changed into the **ona** directory and started checking the config files in it which eventually led me to something juicy, a database settings php file containing credentials **/var/www/html/ona/local/config/database_settings.inc.php**.

![ls]({{<static "/OpenAdmin/index_php.png">}})

![Creds]({{<static "/OpenAdmin/configcreds.png">}})

Next I enumerated users on the machine and found two **joanna** and **jimmy**

![Userenum]({{<static "/OpenAdmin/userenum.png">}})

## SSH as jimmy

Now that we have credentials and users, I try to ssh first as joanna which fails and then as jimmy which is succeeds.

![joanna]({{<static "/OpenAdmin/joannassh.png">}})

![jimmy]({{<static "/OpenAdmin/jimmyssh.png">}})

Now that we have a shell as jimmy, navigating in the previous section while running **ls** I saw a directory named **internal** which I couldn't access as the user **www-data** however I can now access as the user **jimmy**.

Listing the directories in **internal** reveals the following files and on checking **main.php** I get a hint on the user joanna.

There was also an index.php file which contained a hash belonging to jimmy for the main.php page, which can be used to get the SSH key however I didn't use this method.

![ls]({{<static "/OpenAdmin/index_php.png">}})

![main]({{<static "/OpenAdmin/main_php.png">}})

In order to get the SSH key, I needed to run curl on the main.php page and In order for this to work I needed to know the port that the page was using. 

To list the listening ports on this box I ran netstat -tulpn.

![ports]({{<static "/OpenAdmin/portsrunning.png">}})

I ran through each port until I got a hit which was port **52846**, I copied the SSH key to a file and used john to crack it.

![RSA]({{<static "/OpenAdmin/sshkey.png">}})

![John]({{<static "/OpenAdmin/john.png">}})

## SSH as joanna

I then proceeded to ssh as joanna using the SSH key and the passphrase cracked by john, however I ran into a problem ssh was throwing an error stating that the permissions on the key file were too open.

This stumped me for a bit but looking at the permissions on the file **0644** I decided to remove the group and global permissions on the file as this would probably be the obvious cause for the degraded security. 

![Changer_permissions]({{<static "/OpenAdmin/joanna_change_permissions.png">}})

After this I was able to ssh successfully as joanna.

![Joannassh]({{<static "/OpenAdmin/joannassh_success.png">}})

## User Flag

The user flag was easy to find, running **ls** showed me a **user.txt** file which contained the flag.

![Userflag]({{<static "/OpenAdmin/userflag.png">}})

## Root Flag

This was a bit trickier to get, first I decided to check what root access joanna had and found that the nano tool could be ran with elevated privileges.

Running linpeas would also have shown this.

![Sudocheck]({{<static "/OpenAdmin/check_sudo.png">}})

From here I had two methods I could use to read the **root.txt** file

### Method 1

Checking GTFObins we get a method to get a shell using nano.

![GTFO]({{<static "/OpenAdmin/gtfobins.png">}})

So I open nano as sudo without specifying a file name then do a **ctrl+r** and **ctrl+x** allowing us to run a command then I type in **reset; sh 1>&0 2>&0**, after getting a shell we just need to cat the **root.txt** file and we get the root flag.

![method1_flag]({{<static "/OpenAdmin/flag_method1.png">}})

### Method 2

For this method we open the /opt/priv file using nano as sudo then try changing directories and it succeeds.

So I do a **ctrl+r** which will allow me to insert a different file into the /opt/priv one and read the flag.

![method2_1]({{<static "/OpenAdmin/method2_1.png">}})

![method2_2]({{<static "/OpenAdmin/method2_2.png">}})

![method2_flag]({{<static "/OpenAdmin/method2_flag.png">}})


I had a lot of fun doing this box and writeup, I really hope you enjoyed the read.

Happy hacking everyone :sunglasses: and as we say at f334aks-mini #IWDWD :skull: