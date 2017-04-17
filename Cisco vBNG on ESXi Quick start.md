#Cisco vBNG on ESXi Quick start
##-Cisco IOS XRv 9000 Router based vBNG lab guide

Version 1
Jun-2017





# Introduction

In this document, I will guide you, step by step, to build a simple vBNG testbed on ESXi hypervisor running on a single physical X86 server.
We will have following VMs running on top of the ESXi host
- A virtual BNG, which is essentially a Cisco IOS XRv 9000 Router
- A Virtual Server  - linux based VM acting as a RADIUS server and/or DHCP server
- Couple of virtual CPEs - Windows VM or linux VM to simulate CPEs to initiate PPPoE and/or IPoE sessions
- Optional, a Virtual traffic generator
The EXSi host has networks reach out to the outside, so you can also use external physical RADIUS server and external client to cooperate with the vBNG.

The purpose of this simple testbed is to show off the functionality of Cisco vBNG. Performance and scale are not our target, so no optimisation is made for this purpose.

People have interest in Cisco IOS XRv 9000 Router based vBNG can follow this guild to build his/her own testbed quickly with limited hardware resources and conduct some simple prove of concept test, such as
- IPoEv4 and v6 session bringing up.
- DHCP addressing 
- RADIUS based AAA
- Getting familiar with Cisco IOS XR based BNG command line (share the same CLI with ASR900 physical BNG)


#Topology

##Physical topology
~<img src="https://docs.google.com/drawings/d/17lqMl8q18DxynflSm5A-zUcHKJ8EonfUGIHre9vJFyI/pub?w=960&amp;h=720">~

There are three physical hosts in this testbed.	
- An ESXi host running ESXi 5.50, detail release information shown as following display in ESXi host CLI

'' ~ # uname
'' VMkernel localhost.Roy_F1 5.5.0 #1 SMP Release build-1331820 Sep 18 2013 23:08:31 x86_64 GNU/Linux
Hardware configuration of the ESXi host show as following
(img) 
- A Linux host named “ubuntu-demo” used as SSH client.
- A Win\_PC
	- Running ESXi vSphere client
	- Running Spirent Testcerter application to control a virtual Testcerter VM running as guest in ESXi host
	
	
	'' ZHJIANG-M-L02P:~ royjiang$ ssh zhjiang@sjc-xdm-141
'' '' zhjiang@sjc-xdm-141's password:
'' Last login: Sun Apr  9 18:52:46 2017 from shn-zhjiang-8812.cisco.com
'' Terminal is set to: xterm-256color
'' Display is not set.
'' sjc-xdm-141:97> cd /auto/prod_weekly_archive1/bin/6.3.1.13I.DT_IMAGE
'' sjc-xdm-141:98> ls -l
'' total 52
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 22:01 asr9k-px
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 17:54 asr9k-x64
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 14:57 enxr
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 16:51 hfr-px
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 16:07 iosxrv-x64
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 15:27 ncs1001
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 15:41 ncs1k
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 15:42 ncs4k
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 16:36 ncs5500
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 21:40 ncs5k
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 18:10 ncs6k
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 19:41 xrv9k
'' drwxr-xr-x 2 nkhai xrops 4096 Apr  5 16:16 xrvr
'' 
'' sjc-xdm-141:99> cd xrv9k/
'' sjc-xdm-141:100> ls -l
'' total 27617600
'' -rw-r--r-- 1 nkhai xrops      599869 Apr  5 14:38 xrv9k-eigrp-1.0.0.0-r63113I.x86_64.rpm
'' -rw-r--r-- 1 nkhai xrops   939212800 Apr  5 14:44 xrv9k-full-x-6.3.1.13I.iso
'' -rw-r--r-- 1 nkhai xrops  1156792320 Apr  5 15:21 xrv9k-full-x-6.3.1.13I.ova
'' -rw-r--r-- 1 nkhai xrops  1962373120 Apr  5 15:22 xrv9k-full-x-6.3.1.13I.qcow2.tar
'' -rw-r--r-- 1 nkhai xrops   939214848 Apr  5 14:32 xrv9k-full-x.dev-6.3.1.13I.iso
'' -rw-r--r-- 1 nkhai xrops   939214848 Apr  5 14:32 xrv9k-full-x.vga-6.3.1.13I.iso
'' -rw-r--r-- 1 nkhai xrops        6983 Apr  5 15:22 xrv9k-full-x.virsh-6.3.1.13I.xml
'' -rw-r--r-- 1 nkhai xrops   939214848 Apr  5 15:50 xrv9k-full-x.vrr-6.3.1.13I.iso
'' -rw-r--r-- 1 nkhai xrops  1156403200 Apr  5 15:49 xrv9k-full-x.vrr-6.3.1.13I.ova
'' -rw-r--r-- 1 nkhai xrops  1960089600 Apr  5 15:50 xrv9k-full-x.vrr-6.3.1.13I.qcow2.tar
'' -rw-r--r-- 1 nkhai xrops        6983 Apr  5 15:50 xrv9k-full-x.vrr.virsh-6.3.1.13I.xml
'' -rwxr-x--- 1 nkhai crypto  944568320 Apr  5 15:50 xrv9k-fullk9-x-6.3.1.13I.iso
'' -rwxr-x--- 1 nkhai crypto 1161850880 Apr  5 16:18 xrv9k-fullk9-x-6.3.1.13I.ova
'' -rwxr-x--- 1 nkhai crypto 1965690880 Apr  5 16:20 xrv9k-fullk9-x-6.3.1.13I.qcow2.tar
'' -rwxr-x--- 1 nkhai crypto  944570368 Apr  5 14:44 xrv9k-fullk9-x.dev-6.3.1.13I.iso
'' -rwxr-x--- 1 nkhai crypto  944570368 Apr  5 17:26 xrv9k-fullk9-x.vga-6.3.1.13I.iso
'' -rwxr-x--- 1 nkhai crypto 1162158080 Apr  5 17:24 xrv9k-fullk9-x.vga-6.3.1.13I.ova
'' -rwxr-x--- 1 nkhai crypto 1967616000 Apr  5 17:26 xrv9k-fullk9-x.vga-6.3.1.13I.qcow2.tar
'' -rwxr-x--- 1 nkhai crypto       6985 Apr  5 16:20 xrv9k-fullk9-x.virsh-6.3.1.13I.xml
'' -rwxr-x--- 1 nkhai crypto  944570368 Apr  5 16:49 xrv9k-fullk9-x.vrr-6.3.1.13I.iso
'' -rwxr-x--- 1 nkhai crypto 1161912320 Apr  5 16:48 xrv9k-fullk9-x.vrr-6.3.1.13I.ova
'' -rwxr-x--- 1 nkhai crypto 1966387200 Apr  5 16:49 xrv9k-fullk9-x.vrr-6.3.1.13I.qcow2.tar
'' -rwxr-x--- 1 nkhai crypto  944570368 Apr  5 17:55 xrv9k-fullk9-x.vrr.vga-6.3.1.13I.iso
'' -rwxr-x--- 1 nkhai crypto 1161912320 Apr  5 17:54 xrv9k-fullk9-x.vrr.vga-6.3.1.13I.ova
'' -rwxr-x--- 1 nkhai crypto 1966325760 Apr  5 17:55 xrv9k-fullk9-x.vrr.vga-6.3.1.13I.qcow2.tar
'' -rwxr-x--- 1 nkhai crypto       6985 Apr  5 16:49 xrv9k-fullk9-x.vrr.virsh-6.3.1.13I.xml
'' -rw-r--r-- 1 nkhai xrops     2438696 Apr  5 14:38 xrv9k-isis-2.1.0.0-r63113I.x86_64.rpm
'' -rwxr-x--- 1 nkhai crypto    5396509 Apr  5 14:38 xrv9k-k9sec-3.1.0.0-r63113I.x86_64.rpm
'' -rw-r--r-- 1 nkhai xrops      336948 Apr  5 14:32 xrv9k-li-x-1.1.0.0-r63113I.x86_64.rpm
'' -rw-r--r-- 1 nkhai xrops        4953 Apr  5 14:38 xrv9k-m2m-2.0.0.0-r63113I.x86_64.rpm
'' -rw-r--r-- 1 nkhai xrops    24993524 Apr  5 14:38 xrv9k-mgbl-3.0.0.0-r63113I.x86_64.rpm
'' -rw-r--r-- 1 nkhai xrops   889292800 Apr  5 14:26 xrv9k-mini-x-6.3.1.13I.iso
'' -rw-r--r-- 1 nkhai xrops     2558898 Apr  5 14:38 xrv9k-mpls-2.1.0.0-r63113I.x86_64.rpm
'' -rw-r--r-- 1 nkhai xrops     9741896 Apr  5 14:38 xrv9k-mpls-te-rsvp-2.2.0.0-r63113I.x86_64.rpm
'' -rw-r--r-- 1 nkhai xrops     3796729 Apr  5 14:32 xrv9k-ospf-2.0.0.0-r63113I.x86_64.rpm
'' -rw-r--r-- 1 nkhai xrops      921785 Apr  5 14:38 xrv9k-parser-2.0.0.0-r63113I.x86_64.rpm
'' sjc-xdm-141:101>

2. From a local MAC or linux host, SCP the image to my MAC. Please be noted, in my case I have no permission to download the K9 version. 



'' royjiang@ubuntu-demo:~$ scp zhjiang@sjc-xdm-141:/auto/prod_weekly_archive1/bin/6.3.1.13I.DT_IMAGE/xrv9k/xrv9k-full-x-6.3.1.13I.ova xrv9k-full-x-6.3.1.13I.ova
'' 
''   --snip-- 
'' zhjiang@sjc-xdm-141's password:
'' xrv9k-full-x-6.3.1.13I.ova                                                                                                                                                                                                                   21%  234MB 169.7KB/s 1:27:26 ETA
