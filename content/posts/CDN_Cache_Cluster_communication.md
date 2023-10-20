+++
title = 'CDN_Cache_Cluster_communication'
date = 2022-05-16T12:29:04+08:00
draft = false
categories = ['Network']
tags = ['CDN']
+++

### Cache Cluster Communication

The main goal of collaborative interaction within the Cache server cluster is to establish a good communication channel between each server node, communicate in a timely manner about the content cache on the server, and provide users with a good service experience through collaboration between servers in the cluster.

Communication between Cache server clusters can be divided into two categories: loose coupling and tight coupling. Among them, loosely coupled Cache communication protocols based on network messages include: ICP, HTCP, Cache Digest, Cache Pre-filling, etc. Tightly coupled Cache communication protocols managed by specific data structures are represented by CARP. Below is just a brief introduction to these communication protocols, focusing on their working principles, advantages and disadvantages. 

#### ICP

RFC 2186 ICP (Internet Cache Protocol) defines a lightweight message format that is used to query Web resource information between Cache servers to determine whether the currently requested resource exists on other servers. When a Cache server sends a Web object (mainly URL information) query request to its neighbor, the server that receives the query request indicates that it has been compromised by feeding back an ICP response containing "hit" or "miss" information. Whether the queried object is saved here.

ICP is generally implemented based on the UDP protocol, although this is not a requirement of the protocol itself. This is mainly because the ICP over UDP method is more suitable for applications such as Web Cache, because the request and response of an ICP query must complete the interaction within a very short time (for example, within one or two seconds), so as to ensure that the Cache server can Quickly obtain Web objects from neighbor servers. If a requesting server using the UDP protocol detects an error when receiving a response, it may mean that the network between the servers is congested or disconnected, and the corresponding responding server is no longer a suitable neighbor to be selected as the requesting server. In addition, compared with the TCP protocol that supports reliable transmission, the UDP protocol packet size is smaller and more in line with the requirements of ICP lightweight message format. Of course, in addition to the above applicability, ICP must also consider the security issues brought by the UDP protocol.

It should be noted that ICP requests and responses do not contain resource-related HTTP header information, such as access control, cache instructions, etc. Therefore, although the Cache server has checked the availability of resources before obtaining relevant resources through the HTTP protocol, Cache failure may still occur (for example, some Web objects that exist on the Cache server do not allow neighboring servers to access them) .

ICP has a long history and has now developed to ICP v2. Most existing Cache servers can also support ICP in some form.

#### HTCP

HTCP (Hypertext Caching Protocol) is a protocol used to discover HTTP cache (Cache) servers and cache data, defined in RFC 2756. It manages a set of HTTP Cache servers and monitors related cache activity.

The operating mechanism of HTCP is similar to that of ICP, which reflects the cache status of Web objects in the cluster by issuing query requests to neighbor servers and obtaining responses. However, unlike ICP v2, which can only contain the URI of a Web object, HTCP requests and responses can contain complete HTTP header information, which allows HTCP to make more accurate decisions when subsequent HTTP requests need to access the same resources. response. In addition, HTCP adopts a variable-length message format and extends Cache management functions, such as the ability to monitor the addition and deletion of remote caches, request immediate deletion, and send Web object prompts (such as the third-party storage location of cached objects and those that cannot be cached). or a third-party storage location for unavailable Web objects).

An HTCP message consists of three parts: header information, data, and authentication. Among them, the header information part is mainly used to inform the length of the message and the protocol version; the data part is mainly used to describe specific HTCP messages, which contains various HTCP operation codes and operation data, such as TST, which is used to test whether the cached response exists. SET used to notify neighbors to update cache target header information, CLR used to notify neighbors to clear targets from their cache, MON to monitor neighbor cache activities, etc.; the authentication part is optional and is mainly used to reflect the authentication information of transaction operations. .

In specific implementations, HTCP messages can be transmitted through UDP packets or TCP connections. Among them, UDP messages must be supported, while TCP protocol is mainly used for protocol debugging. HTCP generally uses HMAC-MD5 shared key authentication because the protocol is more vulnerable to attacks if password authentication is not used.

#### Cache Digest

The emergence of Cache Digest is mainly to solve the network delay and congestion problems during the use of ICP and HTCP protocols. Cache Digest does not use an in-band query method based on the request-question mode. Instead, it establishes a peer-to-peer relationship between servers, that is, each Cache server stores cache information digests of all its neighbors. When receiving a user's Web object access request, Cache Digest directly searches the local Cache content digest and learns whether the requested Web object URI is in a neighbor cache.

Compared with ICP and HTCP, Cache Digest is actually an idea of exchanging space for time, that is, using the local storage space of the Cache server to save the cache content information of neighbor servers, thereby saving transmission delays on the network during each query. For Cache Digest, the choice of summary algorithm is particularly important. Considering the transmission delay and storage overhead of the summary file, the size of the summary file should be as small as possible, but this may also lead to a reduction in query accuracy. The choice of algorithm is a balance A question of trade-offs.

Using Cache Digest requires special security considerations. For example, server A generates a summary file and sends it to its peer server B. Server B will obtain the content from different neighbors based on this summary. In other words, the summary file of A can directly affect the network traffic flowing to B. If the summary file is maliciously tampered with, B will face serious security problems. To reduce this risk, you can consider only allowing servers to actively obtain summary files from other servers using a "pull" method, rather than passively receiving summary files "pushed" from other servers.


#### Cache Pre-filling

Cache Pre-filling implements a mechanism to push Cache content, which can be well applied to IP multicast networks. It enables pre-selected resources to be inserted into all Cache servers in the target multicast group at the same time, thereby achieving synchronization of content saved by each server in the cluster.

At present, Cache Pre-filling technology has been implemented in many places, especially in satellite communication scenarios. Its biggest advantage is that it can transmit large-capacity data to multiple distributed ground satellite receivers at high speed at the same time, so that when the network transmission speed is low, Greatly improve the data access experience under high conditions. But in general, Cache Pre-filling currently lacks a unified standard. Relevant manufacturers generally need to develop special equipment to implement this kind of push Cache technology based on actual scenarios, or add related special-purpose equipment to some general Cache equipment. module.


4.2.5 CARP

CARP (Cache Array Routing Protocol) is essentially a distributed caching protocol that divides the URL space of the Cache server cluster by establishing a hash function. The core of CARP is to define a Cache server array member table for the cluster, and a hash function used to distribute cache URL information to the Cache server. CARP provides users with a path to obtain a Web object URL, which is generated through a hash operation based on the name of the server array member and the corresponding URL content, which means that for any specific URL request, it can be accurately known. The required information is stored on which Cache server in the array, regardless of whether this is information that has just been requested and cached before, or information that needs to be delivered and cached when clicked for the first time.

CARP uses a hash algorithm to accurately route user requests for URLs to any member of the server array, eliminating duplicate cache data in the array and achieving efficient positioning of Cache resources. Because there is no need to consider more irreversibility and encryption requirements, the algorithm used by CARP is very simple and has extremely high performance.

Since CARP is used by many commercial systems, let's take a closer look at how it works. First, the name string of each member in the Cache server array is rotated left bit by bit to form its corresponding hash ID. This information will be added to the array member table and saved in each server. ; Secondly, a similar algorithm is used to operate many URL strings to obtain the corresponding URL hash value; then, each URL hash value must be XORed with the hash ID of each server and multiplied by one constant, and the resulting product is then circularly shifted to the left by the specified number of bits bit by bit to obtain the corresponding score; finally, CARP compares the score values, and finally allocates the content corresponding to the URL to the server with the highest score for the operation. When a Cache server receives a user's URL request, it will perform a hash operation on the URL hash value corresponding to the URL and the hash ID of each server in the locally saved cluster array member table, and then calculate the hash value based on the score. Determine which server the URL content should be on.

Taking into account the differences in processing capabilities of different Cache servers, CARP also introduces a load factor in the hash algorithm to clarify the workload that different servers in the array can bear and assign appropriate URL cache content to them to optimize performance.

CARP is a tightly coupled Cache communication method. Compared with loosely coupled Cache communication based on network messages, the main advantages of CARP are as follows:

-  No resource query request and response processes are required to avoid network impact and reduce transmission overhead.

-  Duplicate cached data is eliminated, and only one copy of each URL content is retained in the system, saving space.

-  It has better scalability because it does not require network interaction with many other Cache servers.

-  Server nodes can be added and deleted flexibly, and the application of hashing algorithms can minimize the impact of data distribution caused by changes in the number of nodes.

-  Able to ensure that all URL data can be effectively cached in the system