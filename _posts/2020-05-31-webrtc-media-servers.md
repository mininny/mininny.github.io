---
title:  "WebRTC: Media Servers"
category: knowledge_base
tag: [WebRTC]
date: 2020-05-31
---
## Media Servers

Media servers in WebRTC are services that run on the server side that can perform different tasks with the media streams flowing through the media server. 

While it is not absolutely necessary in a WebRTC Service, it can be very helpful in creating a richer and more useful WebRTC service. A WebRTC call between two users can be done via p2p or through TURN server, and does not really require a media server.

However, to provide features like group calling, recording, and broadcasting, media server becomes an essential component to WebRTC service. 

### Group Calling

You may think that group calling is simply a p2p call at a large scale. Not quite.

When there are a number of peers in a call, the call has to somehow upload and download all the different media streams from different users without disrupting the quality and efficiency of the call. 

Each video stream has to be rendered and processed to fit the designated frame at the client's device, and when there are many users, the calculations can stack up quite easily.

For a call under 5 participants, a technique called `mesh video` is able to handle the different media streams that is being communicated between the users. In a mesh architecture, each client inidividually communicates with every other client, sending and receiving media stream with other users. However, when more participants join the call, such brute-force mechanism will create too much toll on each client's device, making the call unusable. 

With a media server, you can choose to do video mixing job at the media server, effectively reducing the burden on the client's device. However, to reduce maintenance costs and to provide better quality calls, media servers need to be optimized and best algorithms should be used for mixing and relaying media between multiple users. 

### Recording

The same goes for recording. You can choose to record the media streams on the client's device or on your media server. This choice may depend on the video quality on the server and client, and your policy regarding the management of the data as well as the media streams.

Because all media streams go through the media server, you can have the control over the streams, and can do any necessaary work with the given streams. 

### Other capabilities

Because media streams are communicated through your media server, you can integrate other communication technologies and protocols to your server to enhance your system. You can also introduce other features such as broadcasting to provide unique features.

You could run computer vision models, integrate AR, use speach recognition engines, or add other features that could enhance your service. 

## WebRTC Topologies

To send/receive media streams between multiple peers is not easy. They have to be in sync, have to be resized to fit the given view frames, and needs to be done without taking too much bandwidth or power. 

### Mesh

WebRTC's basic specification introduces a mesh topology where each participant in a session directly connects to every other participants without a server. This allows the call to be processed without the use of media servers, allowing calls to be processed at a lowest cost. However, it is limited to calls involving only few participants. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200531/Mesh.png" alt="">

When more participants are introduced to the call, communicating and sending/receiving media streams between more participants becomes very CPU intensive and inefficient. Also, because the call is maintained only between the participants, it is not easy to implement recording. 

However, Mesh topology is a simple implementation for multi-user calls under 4 participants.

### Selective Forwarding (SFU)

In SFU, there is a media server, to which each paritipant uploads its encrypted media stream, The uploaded media streams are forwarded to every other participants, which creates a downlink equal to the number of participants(except for self). 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200531/SFU.png" alt="">

With the existence of media server, the media server takes on the CPU-heavy tasks from the clients, controlling the media streams between the participants. This can also reduce the latency and allow additional feature implementation, such as transcoding, recording, and other server-side integrations.

However, while each participant have only one upstream, it has multiple downstreams, which can take up the resources of the clients. To maintain a stable connection with minimal stress to the media server, SFU is most optimal with 4 to 10 participants. 

### Multipoint Control

In multipoint control topology, the media server acts as the ultimate control unit between the participants, doing all of the work related to the media within the server. The MCU received the media from each participants, and mixes them together to create a single media stream which is sent to each participant. 

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/posts/20200531/MCU.png" alt="">

With only single media upstream and downstream for each participants, MCU allows for low bandwidth usage and low CPU usage for the clients. However, this requires additional CPU power in the media server for mixing the audio and video into single streams. 

MCU can be beneficial in situations where many participants exists within a single call, or poor network condition, or other media features(i.e. recording) are required. However, because all the necessary work is done within the media server, it can become very expensive to maintain. 

---

Usually, SFU is the most popular way of managing multi-participant calls. However, you can implement hybrid tactic to switch between Mesh, SFU, and MCU during the call to improve its efficiency. 

Technology is advancing faster than ever, and with the introduction of 4K and 8K videos, we need to find the most efficient way to handle the media streams. 