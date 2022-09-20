---
name: "Or0ch1"
linktitle: "Heist"
title: "Heist"
Cover: "/heist/Heist.png"
categories: ["HacktheBox"]
toc: true
images: 
tags: ['HackTheBox', 'evil-winrm', 'impacket', 'windows', 'John The Ripper']
date: 2022-09-16T20:17:27+03:00

---

This will be my dive into writeups on boxes from Hack The Box.

Heist is a retired windows box which has been rated easy and is part of the **intro to dante** Track.

Lets jump right in!

## Recon
After starting the box we get the ip as **10.10.10.149** and as always I start with nmap to scan for open ports and services running on them.

![Nmap_results]({{<static "/heist/nmapscan.png">}})

I ran into a few problems with Nmap so I rescanned the machine with rustscan which got me all the open ports.

![rustscan]({{<static "/heist/rustscan.png">}})

## Web Enumeration

Navigating to **10.10.10.149:80** we get a login page.

![Login]({{<static "/heist/port80.png">}})

Trying to login prompts us to enter an email address,I decide to try and log in as guest and we get something interesting.

Instead of a login or registration page we're redirected to a support page containing a thread about someone named **Hazard** who has attached part of a cisco config file.

![Guest_login]({{<static "/heist/supportpage.png">}})

Clicking on the attachment opens it in the browser and we see password hashes and usernames.

![AttachmentFile]({{<static "/heist/cisco_config.png">}})

I use John the Ripper with the format set to **md5crypt** to crack the Type 5 password.

![type5]({{<static "/heist/password5crack.png">}})

For the type 7 password hashes I use a custom perl script to decrypt them.

![type7]({{<static "/heist/password7crack.png">}})


## Smb Enumeration

In the support thread **Hazard** asked the support admin to create an account for him on the server, so using the decrypted credentials I try to access the shares.

Using **Hazard: stealth1agent** we are able to authenticate however, we are only able to list the shares and we can neither mount or access them.

![smb_shares]({{<static "/heist/smbshares.png">}})

Since we have rpc on the windows box I enumerate other users using impackets' **lookupsid**.

![enum_users]({{<static "/heist/enum_users.png">}})

## Winrm enumeration

Since we also have winrm,using evil-winrm I try and see which user and credential work, after a few tries im able to authenticate as **chase : Q4)sJu\Y8qz*A3?d** .

Poking around I see two interesting files in the Desktop directory **todo.txt** and **user.txt**.

**todo.txt** contains some hints which we will use later.

![evil_winrm]({{<static "/heist/evilwinrm_access.png">}})

**user.txt** contains the user flag hash which we submit.

![user_flag]({{<static "/heist/userflag.png">}})

Moving back to the **todo.txt** file we can see some list of things to do some of which are already done and some which aren't.

We can see Chase needs to keep checking the issues list and supposed to fix a route config hence I come to the conclusion he is the support admin.

He will be monitoring the issues list website via browser so I decide to check the processes running and I see firefox.

![firefox_process]({{<static "/heist/firefox_process.png">}})

I upload both procdump64.exe and strings64.exe from the sysinternals suite to the machine.

![procdump]({{<static "/heist/upload_procdump.png">}})

![strings]({{<static "/heist/upload_strings.png">}})

Using procdump.exe I dump one of the firefox processes and extract strings from it to another file using strings64.exe.

![dump_firefox]({{<static "/heist/dump_firefox.png">}})

![dump_strings]({{<static "/heist/dump_strings.png">}})

I then searched for the string **login_password** from the txt file and found a password.

![admin_pass]({{<static "/heist/admin_pass.png">}})

From the user enumeration we saw an administrator account,tying the credentials we've gotten with it we are successful and on the desktop we find **root.txt** which contains the admin flag hash.

![]({{<static "/heist/flaghash.png">}})

I Submitted the hash and completed the box.


You can get Sysinternals Suite [HERE](https://live.sysinternals.com/)

I had a lot of fun doing this box and writeup, I really hope you enjoyed the read.

Happy hacking everyone :sunglasses: and as we say at f334aks-mini #IWDWD :skull: