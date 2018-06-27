---
layout: post
title:  "Lab Overview"
date:   2018-06-12 14:15 +0100
tags: master malware ml ids splunk
---

As I am building a different approach to IDS from the bottom, I am in need of a proper lab setup.  
This post outlines a high level overview to nuture a basic understanding of future architecture decisions.  

<!--more-->

The setup I am using is based on the excellent book "Building Virtual Machine Labs"[^1] by [@da_667](https://twitter.com/da_667).  
If you are interested in building a lab for yourself, consider buying the aforementioned book - I can more than recommend it.

The whole setup is based around a frankenstein-ished ESXi-Server I've buit with spare parts lying around.  
Currently, the ESXi-Server is powered by an Intel Core i7 930 running on a Gigabyte X58A-UD3R with 12GB RAM and running ESXi 6.5.0 Update 1.  
It is using a 120GB SSD for IO-heavy VMs and a 1.5TB HDD for mass storage and slower VMs as well as 2 NICs, one for ESX management (vmnet1) and one for VM traffic (vmnet0).

This server runs all VMs neccessary for a running lab environment and is self-sufficient on its own.  

## General network layout
The whole networking is based around a pfSense VM that functions as a router and firewall. 
This central pfSense is connected to vmnet0, which is the VM traffic NIC.  
It also has two separate networks connected in direct fashion, __Lab Management Net__ and __IPS Network 1__.  
There is a third network, __IPS Network 2__ which lies behind __IPS Network 1__, that is connected through an *AF_Packet Monitoring bridge*, but more on that lateron.

<div class="mermaid">
graph LR
    Z["fa:fa-globe Outside Net"] --- |192.168.99.0/24| A["fa:fa-fire pfSense"]
    A ---|172.16.1.0/24| B["fa:fa-wrench Management Net"]
    A --- |172.16.2.0/24| C["fa:fa-exclamation IPS Net 1"]
    C -.- D["fa:fa-exclamation-circle IPS Net 2"]
</div>

These network zones are reflected though seperated IP networks living on seperate __Virtual Switches__ on the ESX server.  
So basically, the pfSense VM has three dedicated network adapters. They are connected as follows:
- __NIC1__: Briged (Outside Net) - *192.168.99.3/32*
- __NIC2__: Management Net - *172.16.1.1/32*
- __NIC3__: IPS Net 1 - *172.16.2.1/32*

__IPS Network 2__ is only reachable if the IDS VM is running, as this VM bridges IDS Net 1 and 2 together (whilst enabling packet inspection on all traffic to/from IDS Net 2).  
This is why the Suricata VM has three dedicated network adapters as well:  
- __NIC1__: Management Net - *172.16.1.4/32*
- __NIC2__: IPS Net 1 - *No IP assigned*
- __NIC3__: IPS Net 2 - *No IP assigned*

### Network Contents
The management net is engineered to be kind of a safe zone for data collection and visualization.  
This is why the main __SIEM__ (Splunk) as well as a common __IDS__ Suricata are living here.  
IDS net 1 contains various support VMs that are designed to aid in daily use, i.e. a staging VM for exchanging malware with a victim VM.  
This is also the network where my [cuckoo sandbox installation]({{ site.baseurl }}{% post_url 2018-06-26-cuckoo-esx%}) lives right now.
IDS net 2 only contains victims that will be infected with malware for dynamic anaylsis.

<div class="mermaid">
graph TB
    subgraph IDS Net 2
        E["Sandbox VM"]
    end
    subgraph IDS Net 1 - 172.16.2.0/24
        C["Staging VM"]
        D["Attack VM"]
    end
    subgraph Management Net - 172.16.1.0/24
        A["Splunk"]
        B["Suricata"]
    end
</div>

This environment can be adapted to basically any need there could be.  
In my case, I chose to set up [Netflow monitoring with Splunk]({{ site.baseurl }}{% post_url 2018-05-27-splunk-and-netflows%}) to enable data collection and processing for a Deep Learning-based IDS. 

----------

[^1]: Robinson, Tony V. (2017). Building Virtual Machine Labs: A Hands-On Guide. 1st Ed. CreateSpaceIndependentPublishingPlatform. ISBN:978-1546932635.