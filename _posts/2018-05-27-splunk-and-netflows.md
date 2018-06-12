---
layout: post
title:  "Splunk and Netflows"
date:   2018-05-27 11:01 +0100
tags: ml master splunk
---
So, you want to do your large scale intrusion detection on netflows - but how do you get them in a datasink?  
Let me tell you about Splunk Stream, the TA that saved my prolonged back in the setup phase.  

<!--more-->

While Splunk Stream is an impressive piece of software, it took me a while to set it up correctly.  
Maybe I can provide you with a shortcut if you are looking to do similar stuff.  
The setup I describe has been tested with Splunk 7.1 and works for my scenario. However, your situation might differ - keep that in mind :)  

### The Dataflow
Before getting started with the setup, let me tell you about my environment.  
Basically, I am generating netflows on my core router, a Mirkotik CRS326-24G.  
These Flows are sent to a Stream Forwarder instance that is running on the same host as Splunk Enterprise Core.
From there, I can visualize Flows in the WebUI or use them through the REST API.  

__Core Router__ *--Sends Flows to-->* __Stream Forwarder__ *--Sends Data to-->* __Splunk Enterprise__

### Router Setup
The core router is set through `IP/Traffic Flow Settings` to collect on all interfaces with default timeouts.  
It then transmits these flows to my Stream Forwarder instance, running on `$SPLUNK_HOST:9995` as netflow v5.

### Splunk Stream
Whilst you can install additional apps from the Splunk WebUI, at least in my case this led to some trouble, so I went ahead and downloaded the Stream installer package directly from [Splunkbase](https://splunkbase.splunk.com/app/1809/).  
After installing the package (check the [Splunkbase details](https://splunkbase.splunk.com/app/1809/#/details)), it is time to configure the Stream Forwarder.  

**Fair Warning: At this point, go ahead and activate netflow ingestion in your WebUI!**  
If this is not active, the configured receiver port of your Stream Forwarder instance will never come online!  
To do that, navigate to *Splunk Stream App -- Configuration -- Configure Streams*[^1] and set `netflow` to *Enabled*


### Splunk Stream Forwarder
Did that? Great! Now we need to tell the Stream Forwarder on which port it should listen and what to expect.  
The forwarder is the daemon that is responsible for collecting netflows and sending them to the Splunk Indexer for further processing.  
Have a look at the [documentation for best practices, where and how to install it](http://docs.splunk.com/Documentation/StreamApp/7.1.2/DeployStreamApp/InstallSplunkAppforStream#Manually_install_Splunk_TA_stream_on_remote_universal_forwarders).  
To archive this, connect to your Fowarder host (in my case this is the Splunk Enterprise server) via SSH and set up your Forwarder.  

Don't have a forwarder yet? No problem. You need a host (i.e. Linux) on which you [install the Splunk Universal Forwarder](https://docs.splunk.com/Documentation/Forwarder/7.1.0/Forwarder/Abouttheuniversalforwarder).  
Technically, the Stream Forwarder collects the netflows and submits them to the Universal Forwarder, which in turn submits them to the Indexer. That's why you won't find any target config in the Stream Forwarder itself.  

You need to set your inputs:  
`vim $SPLUNK_HOME/etc/apps/Splunk_TA_stream/local/inputs.conf`
``` ini
[streamfwd://streamfwd]
splunk_stream_app_location = https://$SPLUNK_HOST:8000/en-us/custom/splunk_app_stream/
stream_forwarder_id =
disabled = 0
```

as well as your streamfwd itself:  
`vim $SPLUNK_HOME/etc/apps/Splunk_TA_stream/local/streamfwd.conf`
``` ini
[streamfwd]
ipAddr = 0.0.0.0

netflowReceiver.0.ip = $FORWARDER_IP # this shouldn't be localhost
netflowReceiver.0.port = 9995
netflowReceiver.0.protocol = udp
netflowReceiver.0.decoder = netflow

netflowReceiver.1.port = 6343
netflowReceiver.1.decoder = sflow
```

If you're done changing your config, restart the Universal Forwarder and double check that the ports are open.  
My running setup looks like this on the Universal Forwarder:

``` bash
root@splunk:~# netstat -tulpen | grep streamfwd
tcp        0      0 0.0.0.0:8889            0.0.0.0:*               LISTEN      0          18764       1571/streamfwd  
udp        0      0 172.16.1.3:9995         0.0.0.0:*                           0          21657       1571/streamfwd  
udp        0      0 0.0.0.0:6343            0.0.0.0:*                           0          21658       1571/streamfwd  
```

If all went well, netflows should start appearing in your Splunk Instance.  
I went ahead and built myself a quick viz with a Sankey Diagram with SRC and DST IPs to be able to quickly judge if everything's working.

-------

[^1]: If you struggle to navigate there, you might be happy about an URL: hxxps://*$SPLUNK_HOST:8000*/en-US/app/splunk_app_stream/streams#metadata
