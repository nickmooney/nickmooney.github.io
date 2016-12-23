---
layout: post
title: Using your own hardware with CenturyLink Fiber
comments: true
---

CenturyLink's FTTH (fiber to the home) service recently became available in our neighborhood in Seattle. After having issues with Comcast for quite a while, it was a welcome change.

When the CenturyLink technician came to install service at our house, he installed two pieces of hardware. The first piece of hardware is the ONT -- "optical network terminal" -- and the second is a device that CenturyLink refers to as a modem. In our case, the "modem" was a ZyXEL C1100Z. What CenturyLink refers to as a modem is really a router, although it seems that they have intentionally tried to obscure this to make it difficult for consumers to use their own hardware. This is likely valuable to CenturyLink since they charge $9.99/mo for the rental of a "modem."

Some research had shown that it is possible to connect your own hardware directly to the ONT -- [others have been successful](http://kmwoley.com/blog/bypassing-needless-centurylink-wireless-router-on-gigabit-fiber/) doing this, so I decided to take a shot at it. My housemates and I are all college students and we don't like getting ripped off, so I wanted to see if we could use nothing but our existing hardware.

Necessary features
------------------
To connect your own hardware to the CenturyLink ONT, your router needs to support two things:

1. Logging into the ONT via PPPoE
2. VLAN tagging over the WAN port, since the ONT expects all packets between it and the router to be tagged with VLAN ID 201

PPPoE
-----
Getting our PPPoE credentials was the easy part. I found [this post](https://n8henrie.com/2015/01/how-to-find-your-centurylink-ppp-password-on-a-zyxel-c1000z-modem/) detailing a couple ways to get that information. In short:

1. Enable telnet on the ZyXEL device
2. Open a standard shell with `sh`, then `/usr/bin/pidstat -l -C pppd` to see the username and base64-encoded password provided to the pppd process.
3. Decode the password: `echo "encoded_password" | base64 --decode`

I saved these credentials for later.

Our hardware
-----------
We have a TP-Link TL-WR841N router (note that this is the same as the WR-841ND -- "D" means "detachable antennas"). This router has a built-in switch that supports VLAN functionality, and I figured getting it to play nice with the ONT should be fairly easy. I flashed the router with [OpenWRT](https://wiki.openwrt.org/toh/tp-link/tl-wr841nd) and started setting things up. First I edited the WAN interface's settings to use the PPPoE credentials I harvested from the ZyXEL device, and then I went to configure the VLAN tagging. I won't go into too much detail here since there are many other blog posts that do a fantastic job of explaining this process.

I was a little confused when I got to configuring the switch, since my default configuration didn't seem to match what most other people were seeing in OpenWRT's web interface. What I later found out is that the TL-WR841N *does* support VLAN tagging, but only on the 4 LAN ports; the WAN port is connected directly to `eth1` -- no switch. I figured I was out of luck and almost went to purchase a switch to place between the router and the ONT, but I was curious if there was any reason the TL-WR841N's dedicated WAN port *had* to be the one connected to the WAN. Some digging revealed that although it's not particularly well-documented, you are free to use any port as the WAN port with OpenWRT. It just requires that you set up different VLANs. If I could use one of the LAN ports as my WAN port, I could use the switching functionality to tag all the packets between the router and the ONT with VLAN ID 201.

The setup
---------

**Note:** In my router, port 0 is the CPU port, and ports 1-4 correspond to the labeled ports 4-1 respectively (i.e. all the LAN ports are in reverse order). Make sure you read the OpenWRT documentation for your specific hardware to figure out which ports are which.

It's likely possible to do this in the GUI, but I'm more comfortable editing config files, so I edited `/etc/config/network` to look like this:

{% gist 89b3130efed48286587dde4054f07da6 %}

There are a couple important bits here. `eth0` is all the LAN ports (which are attached to the switch that enables the VLAN functionality. `eth1` is the WAN port, connected directly to the CPU, which we do not use in this configuration. You can ignore the lines containing `orig` and any other cruft. The stuff you need to change is this:

  * Line 11: `eth0.1` means that the LAN should be VLAN 1 on eth0 
  * Line 23: `eth0.201` means that the WAN interface should be VLAN 201 on eth0
  * Lines 31-34: ensure switching and VLANs are enabled on your device
  * Lines 36-39: create a VLAN with ID 1, containing the CPU port (tagged) and LAN ports 1, 2, and 3 (untagged)
  * Lines 41-44: create a VLAN with ID 201, containing the CPU port (tagged) and the LAN port 4 (tagged) -- as mentioned above, on my router, LAN port 4 in the software corresponds to LAN port 1 on the hardware

We want to always tag port 0, since the CPU should see traffic from separate VLANs as separate pseudo-interfaces (`eth0.1` and `eth0.201`). We also tag port 4 because we want traffic sent to/from the ONT to be tagged. We leave ports 1-3 untagged, meaning "treat traffic on these ports as implicitly part of VLAN 1, but do not tag the ethernet frames." I don't have much networking background so it took me a while to get my head around this. I found [OpenWRT's switch documentation](https://wiki.openwrt.org/doc/uci/network/switch) helpful.

Still not working?
------------------
At this point, I reloaded my network configuration on the TL-WR841N with a `/etc/init.d/network reload`, but I still wasn't able to access the internet. I believe this is due to a bug in OpenWRT's `swconfig`, as my second VLAN was showing up with an incorrect VLAN ID and without tagging on port 4. I fixed this by adding a few lines to `/etc/rc.local`, but this should not be necessary. I'm currently trying to figure out if `swconfig` is actually broken (so I can submit a pull request), or if it's just user error on my part. Either way, adding the following lines to `/etc/rc.local` finally got the switch to match its intended configuration:

    swconfig dev eth0 vlan 0 set vid 201
    swconfig dev eth0 vlan 0 set ports '0t 4t'
    ifconfig eth0 down
    sleep 1
    ifconfig eth0 up

Once I figure out why OpenWRT isn't correctly respecting `/etc/config/network`, I'll update this post.

Success!
--------
With all of the above done, and the ethernet cable from the ONT plugged into LAN port 1 on my router (referred to as port 4 in the software), the router was successfully able to talk to the ONT and log in via PPPoE. Now we have the TP-Link TL-WR841N connected directly to the ONT, and we no longer have to rent the ZyXEL C1100Z router from CenturyLink. We are only on a 40 megabit connection, so our existing hardware was more than sufficient, but if you have a gigabit connection it's possible that your hardware will have trouble keeping up, so YMMV.

All in all I'm glad we switched away from Comcast, and having fiber directly connected to our house is pretty cool. That said, it feels that CenturyLink has been pretty deceptive. Calling the device they provide a "modem" is misleading, since it doesn't actually directly interface with the fiber -- it really does feel like a cash grab targeted at people who don't know better. Additionally, we are on a 40/5 connection, but were advertised upload speeds up to 20 megabits/s for a little extra per month. After getting rid of the rented modem we decided to upgrade our upload speed. On further investigation, 5 megabits/s is apparently the maximum possible upload in our area, which is really disappointing for fiber. I'm not sure if CenturyLink plans to upgrade the infrastructure in our neighborhood any time soon, but the reps I talked to weren't able to give me any information, despite being shown the option to get 20 megabits/s upload during the signup process. Hopefully these are just some growing pains for CenturyLink's GPON fiber offerings, but I know others in Seattle have experienced similar issues with under-delivery by CenturyLink.

Lastly, I have read reports of people getting CenturyLink to just disable VLAN tagging on the ONT, which would make it a lot easier to connect your own hardware. From what I can tell, this is something they used to do, but no longer offer. I called in four or five times to try to figure out if this was possible, but couldn't get them to make the change. I wouldn't be surprised if someone still managed to convince them to do it though -- the information I received seemed to vary a lot between representatives.
