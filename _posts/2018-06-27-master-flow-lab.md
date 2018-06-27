---
layout: post
title:  "Master Netflow Lab"
date:   2018-06-27 10:00
tags: master ml ids splunk
---
For the matter of my masters' thesis I am in need of a well defined lab environment that is capabale of simulating traffic as well as running on test data.  
This is how I built it.

<!--more-->

## Requirements
Let's have a look at the requirements first:
The lab should be able to
- replay / work with common test datasets (i.e. CICIDS17, NSL-KDD)
- collect current traffic as netflow
- qualitatively benchmark different training-/test-runs of the IDS 

So in short, the lab environment should be able to present the IDS "real" traffic and standard test datasets in the same way while simultaneously collecting benchmark data.  
For this purpose, I repurposed the existing ESX setup I introduced [here]({{ site.baseurl }}{% post_url 2018-06-12-lab-overview%}).  

## Lab Architecture
<div class="mermaid">
graph LR
    Z["fa:fa-globe Outside Net"] --- A["fa:fa-fire pfSense"]
    A ---|Management Net| B["Splunk"]
    A --- |IPS Net| C["Attack VM"]
    C -.- D["Victim VM"]
    M["ML Workstation"] --- Z
</div>

The pfSense VM is acting as the Core Router as well as a central firewall between the outer net and the lab environment.  
It exports Netflows of all internal interfaces (that is, without the Outside Net connection) into a single purpose Splunk index named *labflow*.  

## Dataset Import
As I want my findings to be comparable, I need a way to not only work with live data but also archived Netflows or miscellanous datasets of other scientists (i.e. CSV, JSON).  
This is where Splunk really shows its flexible architecture. Any kind of (semi-)structured data can be imported and treated as events.  
For example I imported the [CICIDS17](http://www.unb.ca/cic/datasets/ids-2017.html) dataset into its own Splunk index named *cicids*.

## Dataflow
The abovementioned setup enables me to literally ignore differences between datasets, archived netflows and live network data, as all of them are queried through the same [Splunk API](https://github.com/splunk/splunk-sdk-python).  
Normally, a GPU accelerated VM that is sitting in the Management Net would take the role of my soon-to-be Machine Learning powered IDS; unfortunately, as I am running this on private hardware, I only have my main PC available as a Machine Learning Workstation, which is sitting outside the Lab (in network terms).  
This is why the ML Workstation is connected to the outer net.  

## Conclusion
With this setup I am able to fullfill the given requirements, as it is able to handle both, historical/test data as well as live network data.  
Furthermore, through the usage of Python timing packages it is possible to measure runtimes of different parts of the architecture.  
Finally, as I am using [Keras](https://keras.io/) as Deep Learning Framework, I can leverage the integrated test and validation mechanics, as well as [tensorboard](https://www.tensorflow.org/programmers_guide/summaries_and_tensorboard) for advanced detailled visualization of the ML architecture.  
This setup will be introduced in a later post in detail.