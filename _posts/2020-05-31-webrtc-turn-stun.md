---
title:  "WebRTC: STUN and TURN"
category: knowledge_base
tag: [WebRTC]
date: 2020-05-31
---

WebRTC is an open-source framework that enables real-time communications between multiple devices/browsers. 

With WebRTC, there are multiple ways that the users can connect to each other. To connect with other users, WebRTC uses multiple methods like p2p and relay to provide seameless service in different situations. 

The preferred way would be to transfr data between the users directly, by using their IPs directly. 

However, this is not possible sometimes because there are possibilities where some users have firewalls and NATs that prevents direct access to the public IPs. This prevents regular RTC communications, so WebRTC uses ICE(Interactive Connectivity Establishment) protocol to find feasible and appropriate means of communication between the peers. 

## NAT
All internet devices require individual IP addresses to connect to the internet and communicate with each other. However, there are only a certain number of IP addresses available and fastly increasing number of internet devices. 

A tactic used to solve possible problem with the lack of available IP addresses is to group multiple devices under one public address. This public address will usually be the router that routes incoming packets to its connected devices. 

The devices under the public router may have a private IP address that is not accessible from the public. Because of this, devices may not be able to communicate directly with other devices that may be hiding behind a NAT.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200531/nat.png" alt="">

To identify such devices and grab the public IP address and accessible port if possible, WebRTC uses a protocol called STUN.

## STUN
STUN is short for `Session Traversal Utilities for NAT`. 

A STUN server will reach out and send various requests to the connected peers to determine if they are behind a NAT and to get a public IP address if possible. 

This will return the presence and types of NATs used, as well as the firewalls that may be present between the peers. By requesting to the STUN server, the peers can get the public network addresses which may allow them to directly connect to other's public addresses.

While many calls are able to connect by fetching public network addresses of the peers, some peers may be entirely blocked from the public internet, making the direct communication between peers impossible. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200531/stun.png" alt="">

To handle such scenario, WebRTC uses another protocol called TURN, which is an extension of STUN servers. 

## TURN
If peers are not able to communicate with each other publicly, they can resort to TURN server, which stands for `Traversal Using Relays aaround NAT`.

As the name suggests, the connection will be relayed through the TURN server to communicate around the NAT.

If the peers do not have available public IP addresses, they can instead send the whole communication stream to the TURN server which will hand that stream to the other peer again. 
<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200531/turn.png" alt="">

Because the TURN server has an accessible Public IP Address, the peers' private IP address won't be an issue, and can process communications even if the peers are behind firewalls or proxies. TURN can be used to relay audio, video, and data streams between peers. 

Basically, TURN server works as the middle man between blocked-out peers. 

Because TURN relays the whole media stream between the peers, it consumes a lot of bandwidth and requires much more maintenance costs. 

Now, there are three different types of TURN servers.
- TURN via UDP
- TURN via TCP
- TURN with TLS

These types use different communication protocol to connect with each other, and certain browsers and device only allow communication through certain types of TURN servers.