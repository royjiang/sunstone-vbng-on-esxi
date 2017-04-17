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
