# Albie's Homelab Experience, Summer 2026 (and beyond)

## About this write up.

Hello! In this journey, I will talk about my experience setting up my very own homelab. I will demonstrate my technical knowledge, use relevant terminology, and walk through exactly how I overcame obstacles. Most importantly, I will get hands-on experience.


## Pre-Homelabbing

The idea of creating my very own homelab has been on my mind for a while now. As good as my daily tryhackme habit has been for me, playing around with virutal machines in a browser and answering questions can only go so far. Creating a homelab, though? It goes beyond that. It seems like one of the most obvious ways to get hands-on experience with networking and cyber security, and I get a nice project out of it, that can always be upgraded and expanded as I go on. Besides, eventually I'll be able to tell people at university that I'm currently VPNed to my own server at home. If that isn't cool I don't know what is.  

As soon as I started to dive down this endless rabbit hole, it was only a matter of time until algorithms across social media caught onto this new interest of mine. Homelab content started to appear all over instagram, which often lead me to researching about the terms and such the created used in said videos. Looking at instagram this way, I don't feel as guilty for scrolling. I have a habit of wanting to do things, and then putting them aside for a while. It's fairly frustrating, to be honest, because by the end of it I'll end up with 10 projects on my "to-do list". This time though, I was committed, and from research I knew I didn't need fancy enterprise grade servers and cisco managed switched to start.  

I got to work searching online for options. The first thing that hit me was the fact that rather huge servers were being sold for fairly reasonable prices. 96GB of RAM? Seriously? Sounded like a steal. I mean, it made sense, right? Once a company had shiny new tech, they could flog the old stuff cheap. Luckily, I didn't go this route. There's a few good reasons for this. One that didn't even cross my mind at the start was power consumption. I'd probably be kicked out if the electricity bill happened to spike by £30 a month - there was no way that was worth it. Secondly, the noise. Yeah, not ideal. So what was the solution? Everyone says that you can start a homelab with just an old laptop, but where's the fun in that? I needed something that merely sips power, is quiet, won't break the bank, and is at least powerful enough to play around with. The answer was simple. Preowned old mini pcs. The type you'd find behind the desk at the denist or something.  

Now while one mini pc would do the job, and let me play around and tinker with various things that will help me in the future, the specs are never exactly impressive, and during this "AI boom", the prices of RAM and memory are scary. A little more research later, I discovered the terms "cluster" and "node" (in this context). A cluster is exactly what it sounds like. Multiple computers working together. These devices are referred to as nodes. There are many benefits to this, with a huge one being uptime. If one node gets overwhelmed, loses power, or something similar, the other nodes in the cluster can continue working. While I doubt my little homelab (to begin with) will rely on 99.9% uptime, it will be good practice. The entire project is good practice.  

In the end, taking strongly into consideriation the availability and prices of the systems, I went for THREE of the following computers:  
HP EliteDesk 800 G3 Mini PC (65W)
I5-7500  
16GB RAM  
256GB Nvme SSD  

I also got my hands on a 6 way surge protection strip, and a cheap, simple unmanaged switch. Why I do eventually want to implement VLANs and have access to switch configuration, I believe that should come later, once I've already established my homelab a little. Speaking of future upgrades, I'd also love to eventually replace the ISP router with my own, have a dedicated firewall, and some kind of network attached storage. All of these can come later though, and I don't need them to start learning.

As I waited for my hardware to arrive, I drafted out rough a rough plan. Proxmox sounded like the perfect hypervisor for my setup, and I also liked how the UI looked when researching it.


## Pre-Proxmox

Before I could get into the interesting stuff, I still had three machines that were running Windows 11. No thanks. The first thing I did was boot up each of the machines, and ensure the specs were what I paid for. Of course they would be, right? No. For whatever reason, two of them had 12GB of RAM instead of 16GB. I don't know if I was more annoyed about the timewaste or the fact someone had installed an 8GB stick of RAM alongside a 4GB stick in the same machine. Twice.

With a partial refund secured, and my newly purchased sticks of RAM, I was finally ready to begin. With an old TV, keyboard and mouse on hand for setup, I entered each of the machine's BIOS and enabled virtualisation. Without it, I wouldn't be able to run virtual machines. Once it was enabled, I flashed Proxmox onto a USB stick, and sucessfully completed the setup on all three machines. I decided on ".homelab" for the domain, and pve01, pve02, pve03 for the nodes. At this early stage, I might as well keep everything nice and simple. I also assigned them IP addresses, ensuring they were out of my DHCP range to avoid any potential IP address conflicts. Like a lot of people at home, my router (default gateway) is 192.168.0.1. The DHCP range by default was a bit strange, so I set in to 10 - 180. This is plenty, and leaves a nice 10 address gap between DHCP and my tinkering. With DHCP taken into consideration, I decided on the cleanest and least confusing IP addresses possible for my systems. 192.168.0.201, 192.168.0.202, and 192.168.0.203. This leaves .200 spare, and has nice corrolation. The final digit matches the node number of each machine. I then set DNS to my router's IP address as it should just forward traffick to a public DNS provider, but something didn't work, so I just set DNS to 1.1.1.1, which is Cloudflare. Eventually I may want to run my home network through a custom DNS setup implementing PiHole, but for that I'd rather wait until I have my own router, firewall, etc.


## Up and Running

Once I could access each machine through port 8006, I installed updates. The Proxmox UI is pretty nice and I found myself figuring out where everything was pretty quickly. There were a few more settings I had to check before I could begin deploying, such as ensuring I was using the correct repositories.

Randomly, I encountered a weird situation where my computers were suddenly unreachable. I ensured all of the settings were correct, and they were. Thats when I noticed there was no green light next to the router ethernet cable in my switch. Unplugging it and plugging it in again did nothing, same with disconnecting everything physcially. I still don't understand why, but after switching around some ethernet cables in the back of my ISP router, it suddenly worked. I'm guessing there's some kind of conflict? It's happened again since the first time, and the fix remains the same.

With all of my machines running, it was tempting to begin messing around, but I held back as the entire point of getting three machines was to implement a cluster.


## Clustering

Clustering was very easy thanks to Proxmox. I simply had to create the cluster on pve01, and paste a token from pve02 and pve03 into the setup. Within no time at all, my machines were clustered. It's very cool being able to see the combined stats of all three machines in one interface.


## The CyberLab Setup

With no access to VLANs and physical firewalls yet, I'm wary about deploying ethical hacking tools and vulnerable machines. Ideally, I'd want a seperate VLAN for this, and then firewall rules that block traffic certain ways. Having something escape and find its way onto personal devices of mine sounds like a nightmare. For this reason, I decided to completely isolate pve03, which I dedicated to this type of thing for now. Of course, I can still use pve03 for things I do want to access normally, but for now, this is clean. To do this, I put the virutal machines on vmbr1 instead of vmbr0. vmbr1 has no physical port for an ethernet cable, so it literally cant connect to my home network. This being said though, the devices in vmbr1 will be able to communicate with each other. For now, this is perfect for tinkering.

For the time being, I decided to keep if very simple. I'd deploy two virutal machines, and get them to communicate with each other. I decided on Kali Linux, of course, and Windows 11. The nice thing about a hypervisor like Proxmox is that I can discard these machines whenever I feel like it. The deployment itself was simple, similar to VirutalBox. I gave each machine enough hardware but nothing crazy. After getting through the setups, I confirmed that neither could access the internet or devices on my home network, and assigned them simple IP addresses: 10.10.10.10 and 10.10.10.20. Sure enough, after disabling windows firewall for a moment, I could communicate back and forth between machines.

## CyberLab Projects

Work in progress :)


## Services

To make use of my new homelab, I decided to implement various services on my home network. While I'm not one of those people who deseperately wants to replace subscriptions as I don't have many, being able to host my own picture storage sounds very nice. While my most important pictures are backed up in many ways, a central hub for my large quanity of photos and videos would be great. With this in mind, to have a meaningful setup I'd need a lot of storage, which in 2026, is expensive. A NAS device is an investment I'd like to eventually make, however. For now, I plan to setup Immich as a learning experience, not as much for practicality. That being said, I will also deploy an RSS reader as staying up to date with developing technology in cyber security and AI is important. Having a centralised area to do so removes the annoyance of having many tabs open with different articles on. FreshRSS seems like a good option as it is extremely lightweight, and has good mobile sync support. Setting these up will be a great learning experience, even more so since I plan to use Docker Compose.

On pve01, I decided on Debian 13 because that is what is familiar. I gave it basic resources, nothing crazy. I also chose to install it without a GUI to make it even more lightweight. This means it'll be CLI only. Although it's a little daunting, I do have some experience with Linux commands, and the payoff seems more than worth it because the GUI is pretty reductant for a machine using only Docker and containers. I gave this machine the private IP address of 192.168.0.210. This felt like a sensible option. It's leaves a nice gap between it and my nodes. That said though, as I'm writing this, I may change it to 192.168.0.211 because I like having the final digit corrolate to the number of device it is. For example, this vm is my first vm on vmbr0.

Before I can install containers, I need Docker for containers and Docker Compose for managing them. I ssh'ed into my .210 vm, which made my life easier as I can copy and paste commands from the Docker setup documentation. Installing was easy, and I implemented the example hello world container to ensure my installation was good.

### Immich

I then created a directory on the vm for the Immich .yml and the .env and used wget to download them. For this experience, I left the .env configuration as the default. I ran the main Immich for Docker Compose command in the new directory, and it pulled the image from the internet. Moments later, Immich was up and running. I'm already fond of Docker. I could then list my containers, and see that everything was running. The next thing was obvious - access it through its port, 2283. 

After setting up an admin account, the mobile app, and some example pictures, I'm very happy with how Immich looks and feels. I've decided that I will in fact invest in some storage and use Immich going forward.  

### FreshRSS


## Day-to-day