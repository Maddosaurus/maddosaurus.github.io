---
layout: post
title:  "What are Intrusion Detection Systems?"
date:   2018-02-24 10:38:20 +0100
tags: ids master
---
Interconnected computer systems have an ever increasing importance in our modern lives.  
As these networks grow in complexity, human-based monitoring of activities is unlikely to find malicious activity.  
Therefore it can be beneficent to add another layer of defense into the system after authorization and authentication to ensure that possible intrusions are detected and reported.  

<!--more-->

An intrusion can be defined as "any set of actions that attempt to compromise the integrity, confidentiality or availability of a resource" [[Heady1990]](https://www.researchgate.net/publication/242637613_The_architecture_of_a_network_level_intrusion_detection_system).  

An IDS (intrusion detection system) is a system that protects *resources*, uses *models* that characterize a normal or illegitimate state and makes use of *techniques* that compare the current state of the system under evaluation to the defined models. [[Lee1998]](https://dl.acm.org/citation.cfm?id=1267555). They can be classified as either host-based or network-based systems.  

A host-based IDS is a system that resides on a single host to protect local resources on this very host. Examples are [OSSEC](https://ossec.github.io/) and [Tripwire](https://github.com/Tripwire/tripwire-open-source).  
On the other hand, network-based systems sit in between network traffic and monitor all traffic that is flowing past them. Examples are programs like [Snort](https://www.snort.org/), [Bro](https://www.bro.org/) or [Suricata](https://suricata-ids.org/).  
The effectiveness of NIDS (Network-based IDS) differs with the point of deployment, as they can only analyze the traffic that passes the specific point of deployment.  
It is therefore advised to deploy NIDS on the borders or chokepoints of a network to ensure broad protection.  
Although it has to be noted that NIDS pose an inherent performance problem:  
If all data is processed online (that is, real time), these systems need to be equipped with sufficient network bandwidth and processing power.  

IDS can also be categorized by their detection mehtod (signature- vs anomaly-based).  
Check my [post on the differences here.]({{ site.baseurl }}{% post_url 2018-02-20-signature-vs-anomaly-detection%})