---

title: The Homelab - Part 1
categories: homelab

---

I was recently inspired by the [/r/homelab](https://www.reddit.com/r/homelab/) subreddit to revamp my home network and server setup. It has gone through a couple of iterations before this point, but it hasn't been deserving of the 'homelab' title until now.

![The Network Closet]({{ "/assets/img/home_net_closet-13032018.jpg" | absolute_url }})

## TL;DR

### Networking

- Edge firewall: PFSense running on an old Dell Intel i5 box (doesn't have a silly name yet)
    - DNS resolver
    - Still, need to setup OpenVPN server for remote network access.
- Router: Ubiquiti EdgeRouter Lite (aka blueberry)
    - eth0 (10.0.0.0/24) - The gateway interface, direct connection to the firewall
    - eth1 (10.0.10.0/24) - The LAN interface, connected to a smart TP-Link switch. Currently, no VLANs are set up.
    - eth2 (10.0.20.0/24) - The WLAN, connected to my wireless access point which is set up in bridge mode.
- Wireless AC: Ubiquiti Unifi AC Lite (aka space-frisbee)
- Switch: TP-Link TL-SG108E 8-port managed switch

### Servers

- HP Z210 Workstation w/ 16 GB RAM (aka: chilaquiles)
    - Proxmox hypervisor
    - Media server running Debian, services run in Docker, provisioned via Ansible. (aka muffin)
- Raspberry Pi B+ (currently unnamed)
    - Will be configuring it as a monitoring node, most likely using Prometheus and Grafana to pull information from Netdata
- Rasberry Pi 3 B+ (currently unnamed)
    - I'm thinking I will set this up as a vulnerability scanner and NIDS in the future.

### Links

[Ansible Files](https://github.com/camja014/ansible-playbooks)

### Network Diagram

![Network Diagram]({{ "/assets/img/home_net_diagram-13032018.png" | absolute_url }})

## The Long Version

### The Network

A week before I moved I went out and bought a little Asus RT-N16 wireless router in preparation. My very first wireless router, and the only time I accessed the stock firmware and web interface was to immediately wipe it and replace it with DD-WRT. That lasted for a year or two, but there were a couple things it didn't do all that well. I wanted a VPN server and a couple of other features, and I couldn't find a stable DD-WRT firmware that did everything. So eventually I moved to EasyTomato. I found it to be much more stable than the previous versions DD-WRT I had used. It had the ability to act as a VPN server and client. Unfortunately, EasyTomato didn't last too long either. It still had stability issues. Every once in a while applying a configuration change, it would lock up and reset to default settings. The wireless transmitter was starting to go out too. Coverage was spotty, and speeds were horribly slow.

With my little Asus on its last legs, I decided it was time to upgrade. But I didn't want to deal with having to switch device firmware every couple of months in order to find a stable one. That and consumer grade AC wireless routers seemed very overpriced for what you get. So I got myself a Ubiquiti EdgeRouter X and a Ubiquiti Unifi AC Lite access point. Making the switch to the EdgeRouter was a bit of a wake-up. I got everything out of the box and plugged in and logged into the web interface and immediately realized I wasn't quite sure what I was doing. I kinda knew how routing worked, and I new generally knew how most major network services worked. But I had never actually configured any of those things. So I opted for the simple route and bridged all of the router interfaces as one virtual switch interface, set up a couple port forwards, plugged my access point in and everything worked beautifully. The QOS worked 100 times better than what I had available to me on EasyTomato.

Now I could have left everything as it was until the end of time, and it probably would have worked great. Or, I could assign each interface its own subnet... Then I'd have more network segregation. And maybe I could add a managed switch... And you know, I've always wanted a PFSense firewall, just because I can. So naturally, I went eventually opted for the more complicated route for the sake of learning. I got a cheap TP-Link managed switch, and I got an old i5 Dell box from a friend that I threw another NIC into. And after a day of cursing at routing tables and firewall rules, and wondering how the heck you turn of NAT in EdgeOS, and I had a working network again! I had a little more downtime than I would have liked because I thought I had locked myself out of my router and access point so I performed factory resets on both of them. But I came out without too much trauma.

### The Server

I also have my media and storage server. Initially, I was running Plex on my desktop. But I really wanted to dual boot my desktop, and I didn't want to cut off access to Plex everytime I had to reboot. It was a great excuse for me to finally get a dedicated server. CSU has a great surplus equipment store that I picked a used i7 HP Z210 with 16 GB of RAM. It won't win any awards, and it's not the sexiest thing out there, but it works great for my purposes.

I chose Debian as my operating system of choice and decided to go with Docker for running and managing services. I wrote a docker-compose file to allow me to deploy and manage services more easily. But I shouldn't have to SSH into my server and re-run docker-compose manually every time I make a change, I can do better! So I picked up and moved to Ansible. Now, as long as I have SSH access to my server I can provision the entire thing from the ground up and make sure my favorite shell (zsh) and text editor (vim) are installed.

Now Docker provides containerization of applications, but not the operating system. In order to get this same kind of modularity and flexibility with the OS itself, I decided to go with Proxmox, an open source hypervisor, and virtual environment management tool. Once I got it installed it was easy enough to spin up a new VM, install Debian, install a couple of packages, enable passwordless SSH access, and turn it into a template for future VM's to be created from. This also ensures that if I ever need to migrate my services and data to another machine I can move the entire VM instead of having to copy individual files.

Sitting right on top of my media server are my two raspberry pi's, which are currently reserved for future projects. And that about concludes the general setup of my home network.

