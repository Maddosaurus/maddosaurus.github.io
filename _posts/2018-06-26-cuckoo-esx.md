---
layout: post
title:  "Building a Cuckoo Sandbox on ESXi"
date:   2018-06-26 22:10 +0100
tags: malware
---
As there are some honeypots and miscellaneous mail accounts that collect spam and malware, I am in need of a secured environment that is capable of running and dynamically analyzing the collected payloads.  
There are many commercial services available, i.e. [vmray](https://www.vmray.com) or [Hybrid Analysis](https://www.hybrid-analysis.com/), but there is also an Open Source contestor: [Cuckoo Sandbox](https://cuckoosandbox.org/)

<!--more-->

## What is cuckoo?
Cuckoo is an open source software for automated dynamic analyis of suspicious files.  
To do so it makes use of a Windows VM (Linux is currently only partially supported) to run and analyze the behaviour of files.  

## Installation
Please have a look at the Cuckoo [ESXi FAQ](https://docs.cuckoosandbox.org/en/latest/faq/#what-do-i-need-to-use-cuckoo-with-vmware-esxi) first: You need a licensed version of VMWare ESX;  
**the free version won't be sufficient**.  
Apart from this, I can wholeheartly recommend the official [Cuckoo installation guide](https://docs.cuckoosandbox.org/en/latest/installation/) that shows in great depth the requirements and steps to reproduce.  
Additionally, there are some concrete details of how I adapted various details to my lab environment that I introduced [here]({{ site.baseurl }}{% post_url 2018-06-12-lab-overview%}).  

## Master Network Configuration
As I wanted to be able to make use of the PCAP feature, I made sure to route all traffic of Client VMs through the Cuckoo master.  
For this purpose I used a network bridge. Technically, my Cuckoo Master VM is seated in the Management Net (172.16.1.0/24).  
Unfortunately, using it as Passthrough-Proxy for Client VMs, this places Client VMs also in my Management Net. (This is a definitive TODO for later!)  

<div class="mermaid">
graph LR
    Z["fa:fa-globe Outside Net"] --- A["fa:fa-fire pfSense"]
    A --- B["fa:fa-wrench Management Net"]
    B --- C["Cuckoo Master (172.16.1.5)"]
    C --- D["Windows x64 (172.16.1.6)"]
    C --- E["Linux x64 (172.16.1.7)"]
    C --- F["Linux x86 (172.16.1.8)"]
</div>

As one can observe, my Cuckoo instance is currently featuring one Windows VM (stable) and to Linux VMs (unstable testing).  
To create this setup, I went ahead and gave the Cuckoo Master two network cards: One is connected to the vSwitch *Management*, the other one to a vSwitch called *CuckooMonitor* that is used by the one Master NIC and all Cuckoo Client VM NICs.  
This forces all traffic between the Cuckoo Client VMs and the Internet through the Cuckoo Master.  
The aforementioned setup only works because I set up a [NetworkBridge](https://help.ubuntu.com/community/NetworkConnectionBridge) between the two NICs:  
`vim /etc/network/interfaces`
```bash
# don't forget to adapt this to your network config <3`
# also: comment out any config for your bridge ports!
auto br0
iface br0 inet static
        address 172.16.1.5
        netmask 255.255.255.0
        network 172.16.1.0
        gateway 172.16.1.1
        dns-nameservers 172.16.1.1
        bridge_ports ens192 ens160 # tweak this to represent your NICs
        bridge_stp off
```

## Master Cuckoo Conf
The main config logic for cuckoo and the used hypervisor are defined in `cuckoo.conf` and `vsphere.conf`, which both can be found at `$cuckoo_user_home/.cuckoo/config`.
Let's have a look at `cuckoo.conf` first. I will show the most important stuff only; feel free to tweak the rest to your liking:
```bash
# Specify the name of the machinery module to use.
# As we are using a licensed ESXi, we provide ESXi
machinery = vsphere

[resultserver]
# The Result Server is used to receive in real time the behavioral logs
# produced by the analyzer.
# Specify the IP address of the host. The analysis machines should be able
# to contact the host through such address, so make sure it's valid.
# NOTE: if you set resultserver IP to 0.0.0.0 you have to set the option
# `resultserver_ip` for all your virtual machines in machinery configuration.

# REMINDER: Don't forget to set this to your Cuckoo Master IP
ip = 172.16.1.5
```

Now that this is out of the way, lets look at the vsphere.conf:  
```bash
[vsphere]

# Host connection parameters. This host can be a standalone ESXi hypervisor,
# or a vCenter host. It must be licensed for vSphere Web API access (the free
# edition of ESXi is insufficient).
#
# NOTE: In order for the full memory dump feature to work, the credentials must
# have permission to access the datastore files for the relevant machine via HTTP,
# otherwise you will see HTTP status errors (Unauthorized) in the Cuckoo log while
# attempting to download the .vmsn or .vmem memory dump file. Consult the VMware
# documentation for more details:
#
# http://pubs.vmware.com/vsphere-60/topic/com.vmware.wssdk.pg.doc/PG_Appx_Http_Access.21.3.html
host = ESXi-host IP
port = 443
user = Username
pwd = Password

# Specify a comma-separated list of available machines to be used. For each
# specified ID you have to define a dedicated section containing the details
# on the respective machine. (E.g. cuckoo1,cuckoo2,cuckoo3)
machines = Analysis,AnalysisLinux,AnalysisLinux32

# Specify the name of the default network interface that should be used
# when dumping network traffic with tcpdump.
# REMINDER: Change this to your Cuckoo Master VM NIC that is connected to the CuckooMonitor vSwitch
interface = ens192

# Turn this on if you have a self-signed certificate on your vSphere host
# and need to work around the stricter PEP-0476 validation in recent
# Python versions
unverified_ssl = yes


[Analysis]
# Specify the label name of the current machine as specified on your vSphere host.
label = Analysis

# Specify the operating system platform used by current machine [windows/darwin/linux].
platform = windows

# Please specify the name of the snapshot. 
# This snapshot should be taken while the machine is running and the agent started.
snapshot = starthere

# Specify the IP address of the current virtual machine. Make sure that the
# IP address is valid and that the host machine is able to reach it. If not,
# the analysis will fail.
ip = 172.16.1.6

[AnalysisLinux]
<adapted config here>

[AnalysisLinux32]
<adapted config here>

```

I did only show relevant config for one of three VMs in this example to keep the code listing somewhat manageable.  
Please keep in mind that you have to provide the given VM entry for every VM that you listed in the `machines` list.  
If everything went well, you will be greeted with an option in your Cuckoo instance that asks you on which machine you want to run your payload.  
As always: If you've got questions or feedback, ping me via mail or twitter!