---
title:  "WebRTC - STUN & TURN extended"
category: knowledge_base
tag: [WebRTC]
date: 2020-07-05
---

Last time, I talked briefly about TURN and STUN in this [post](https://www.mininny.dev/knowledge_base/webrtc-turn-stun/), and today, we'll look more deeply into it.

## To recap...
There are three methods that WebRTC uses to connect peers. 
- Direct P2P
- STUN
- TURN

While direct P2P connection is most ideal as it does not require additional servers or other infrastructures, P2P is not possible under certain scenarios. 

Many network devices today are located privately inside a NAT. NAT is a Network Address Translator that holds multiple devices with their own private IPs under one public IP address. It allows the internet to save IP-space, and strengthens security by separating the public network space and private network space. When NAT is used, each private network devices are mapped to a specific public IP addresses, which may be accessible from other devices or may block out unknown network connections. When a peer uses NAT that disallows unknown network connections, for instance, peers can not connect directly. 

Also, use of firewalls can hinder with P2P. Firewalls are used for security and can block out specific connections and ports in order to maintain security. 

## STUN Server
In order to resolve issues with NATs, WebRTC uses STUN Server. 

> STUN Server: Session Traversal Utilities for NAT

What STUN does is it uses variety of methods to identify the public IP addresses mapped to their own corresponding private IP address. Because private IPs and public IPs are mapped, if we can find out the public IP address and send any data through the public address, that may be delivered to the private user.

First, peers send a `Who am I?` request to the STUN server. Then, from that request, STUN servers can identify the peer's public ip/port, and send that information to the other peers. With that public address, other peers can attempt to connect directly using the mapped public address. 

Because STUN can identify the network details of the peers, including specifics about the NAT used, etc, STUN servers are usually the minimum server that you need to use in order to deliver smooth WebRTC service. 

## Limitations to STUN Server
However, even if we figure out the public address of the peers, we may still not be able to connect to them directly. Depending on the types of NAT used, multiple public addresses may be used, or connections may be blocked. 

## NAT Types
When NATs connect with outside devices, it creates a address mapping between the private address and the public address, and relay packets to the public address into the private address. 

There are many different types of NAT, largely differentiated with their methods of mapping and filtering connections. While this grouping is deprecated, it is still good to get a grasp of the types of NAT. 

|           |     Mapping      | Filtering |
| Full cone | Source IP / Port | Allow all packets to source IP/Port |
| Restricted cone | Source IP / Port | Allow packets from known external IP |
| Port Restricted cone | Source IP / Port | Allow packets from known external IP / Port |
| Symmetric | Source IP / Port & Destination IP / Port | Allow packets from known IP / Port |

- Full cone

Full cone NATs only use the source IP and port to create a public IP and Port. So, for every connection that a user makes, only one public address is created. When a user connects to a two different peers, both connections will be created using the same public IP and port. 

When filtering incoming packets, Full cone allows all packets to the source IP and port. If another user knows the public IP and Port of the user, they can send any packets directly to that address. 

- Restricted Cone

Like full cone, Restricted cone only uses the source IP and Port to create a mapped public IP and Port. 

Unlike a Full cone NAT, Restricted Cone only allows packets from known external IP that the user has communicated before. For example, when user B sends packet to user A for the first time, the packet will be denied. However, if user A sends a packet to user B and registers user B to the filtering list, user B can now send packets to user A directly. 

- Port Restricted Cone

Like full cone, Port restricted cone only uses the source IP and Port to map a public IP and Port. 

Similar to Restricted cone, Port restricted cone only allows packets from known external IPs *and* Ports. Users with the same IP and Port can only send packets to this type of NAT.

- Symmetric

Symmetric cone is unique in that it uses Source IP/Port as well as Destination IP/Port to map the private address and public address of the devices. So, when a user connects to multiple peers, each connection uses different public address. When user A connects to user B and C, the public address that the user B and C sees will be different. 

For filtering, Symmetric NAT works like Port restricted cone, only allowing packets from known IP and Port. 


Unlike other types of NAT, Symmetric NAT creates difficulty for peers to connect because each connection uses different public IP/port mapping. Usually, when the STUN server identifies the public address of each peer from the `who am I?` request, the peers use the identified public address to connect to them directly. However, when the NAT is symmetric, the public address that the STUN server and the peer see will be different because the destination(opponent) is different. So, even if the STUN server gets the public address of the peers, the peers won't be able to connect to that address directly because it is only available to the initial device that the peer connected to.

## Hole Punching
When peers connect using STUN, they usually use technique called **Hole Punching**.

Let's say that there is user A and user B. User A uses Full cone NAT and user B uses Restricted cone NAT. 
1. Both users send `who am I?` request to the STUN server.
2. The STUN server will return the peers' public address.
    - Because user B is under a restricted cone NAT, user A can not directly send packets to user B's public address. So, we need to hole punch here.
3. User B sends a message to user A first. Because user A is under a full cone NAT, it successfully receives every packets received.
4. This registers user A to user B's `known user list` after a successful connection. 
5. Now, because user A has communicated with user B, user A can send messages to user B without getting blocked. 

This way, peers "punch" available holes first in order to create a successful connection. 

## TURN
While hole punching is a great technique to circumvent NAT restrictions, some NAT types do not allow direct connection at all. This is the case with the symmetric NAT, where public address mapping changes whenever the opponent user changes. 

Even if the peer try to connect with the given public address(from STUN server), because the peer will create different public address for STUN and the opponent peer, using the public address from the STUN server does not work. 

So, instead of STUN, WebRTC uses TURN in these kinds of scenarios. 

> TURN: Traversal Using Relays around NAT

Instead of connecting peers directly, peers can use the TURN server to relay the media. Because the TURN server is on the public network, there is no NAT restrictions, and it can communicate like a regular internet connection. 

When TURN server is used, every media packets between each peers goes through the server and become relayed. Because of this, using a TURN server may be pricey and less efficient/performing than a P2P connection or STUN.

Here is a table that shows which kind of connection that WebRTC uses for NAT types. 

|    | Full cone | Restricted Cone | Port Restricted Cone | Symmetric |
| Full cone | *STUN* | *STUN* | *STUN* | *STUN* |
| Restricted cone | *STUN* | *STUN* | *STUN* | *STUN* |
| Port restricted cone | *STUN* | *STUN* | *STUN* | **TURN** | 
| Symmetric | *STUN* | *STUN* | **TURN** | **TURN** |


## ICE
> ICE: Interactive Connection Establishment

ICE is a framework that is used by WebRTC to connect peers.  

ICE uses various available "candidates" from the peers and attempt to create a successful connections using those candidates. Here, the following candidates are used: 
- Local candidate (Local address)
- Server Reflexive Candidate (Address from STUN server)
- Relayed Candidate (TURN address)

These candidates are collected and sent to the remote peer via SDP. 

### Connectivity Checks
Using these candidates, WebRTC attempts all possible combinations of connections. Candidate pairs are created using the available candidates.

With each candidate pair, candidate A sends a request and receives a response from candidate B, then candidate B sends a request and receives a response from candidate A again. When this happens successfully, that candidate pair is now **valid candidate**. 

Candidate pairs are sorted in priority, and WebRTC chooses the best and most ideal candidate pair to create a connection. This process is called a candidate nomination.

---

This is how WebRTC uses all available techniques to create connections between each peers in different networking conditions! 

---

__mininny