---
title: "IPv6 from scratch"
last_modified_at: 2022-09-05T17:00:00
categories:
  - Blog
  - Video
tags:
  - Networking
  - IPv6
videoid: oItwDXraK1M
videotitle: "IPv6 from scratch - the very basics of IPv6"
toc: true
header:
  teaser: /assets/images/thumbnails/oItwDXraK1M.jpg
---

## IPv6 basics 

The basics of IPv6, IPv6 addresses, IPv6 scopes - kind of IPv6 for dummies ;-) I took a looong IPv6 course on Udemy in order to learn the very basics of IPv6 - but - I was struggling with it. Until I freed up my mind and forgot everything I knew about IPv4 - from then on I was able to learn IPv6 from scratch ;-)

<a href="https://www.youtube.com/watch?v={{page.videoid}}"><img src="/assets/images/thumbnails/{{page.videoid}}.jpg">Watch the video on YouTube</a>

<details>
	<summary>Click to view the entire transcript</summary>
IPv6 from scratch

(Please forget everything you know about IPv4)

I recently took a lengthy course about IPv6 on Udemy. And I was struggling with it in the beginning. I know IPv4 quite well, but just couldn’t make up my mind to those new ugly IPv6 addresses.  I then stopped and asked myself the question what is it? Why am I struggling? Until I realized – I was constantly “translating” from IPv6 into IPv4 in my head. I was trying to find an analogy to the IPv6 stuff in the IPv4 world.  And that just didn’t work for me. From the moment that I freed up my mind and kind of forgot everything I knew about IPv4, things went totally smoothly. Great learning experience. As I wanted to make a video on IPv6 for a long time already I thought – how would I transmit that learning experience to a similar audience? So here’s the idea of this video: You forget everything you know about IPv4. And we will design a new protocol – IPv6 -  from scratch. Together. We will do as if there was no protocol at all and as if we would need to design it from nothing. Shall we do that? Let’s go.

(Creating the addresses)

We have two nodes here. PC, phone, tablet, anything. How can we connect them? How can we build a network and have them talk to each other? I’d say each of these needs an IPv6 address. Like homes in the real world have addresses. As those devices are computers and as computers know zero’s and one’s (aka bits), let’s just think quickly how many bits we would want to give away for an IPv6 address. If we used 10 then we would have two to the power of ten different addresses available. That’s 1024 IPv6 addresses. Not enough for this world. If we used 128 bits or in other words 16 Bytes – that’s really not much memory these days – then our IPv6 could use two to the power of 128 which equals – erm – gazillions of addresses. That should do. Let’s rather go large. Our IPv6 addresses shall hence be 128 bits large, that’s 16 bytes. This way we can nicely group them in let’s say eight groups of two bytes. If we write the groups in hex numbers, then we would have eight blocks of 4 hex digits. This way people could even build short words with the blocks such as f00d or cafe or fade or whatever. In order to separate the blocks, let’s arbitrarily chose the colon character. So a valid address would be f00d:cafe:0000:0000:0000:0000:0000:0001. Just one thing is annoying here – this IPv6 address is long. Let’s shorten it. Let’s remove leading zeros in each block. In this case that would be f00d:cafe:0:0:0:0:0:1. You know what? Let’s not write the zeros at all. Let’s just write a double colon instead. So f00d:cafe::1. Looks much better. Just – we can only do this once in order to avoid ambiguity. FF:0:0:0:1:0:0:1 could hence be written like this (FF::1:0:0:1) or like this (FF:0:0:0:1::1) but not like this (FF::1::1), because we would not know how many zeros to put into which block.

Let’s stop here and write down our findings on the OneMarcFifty IPv6 cheat sheet. We can use this later to look things up. IPv6 Address length, how to shorten IPv6 addresses, how to abbreviate zeros in IPv6. Here we go.

(Creating the protocols)

So now we have these two or three IPv6 nodes that can talk to each other. Just – we might need one more thing. We need to specify how they talk to each other. If we were sending voice or video data then we should go as fast as possible. In turn, it doesn’ really matter if we lose a packet or two. Let’s therefore create an IPv6 protocol and call it the User Datagram Protocol or short UDPv6. This IPv6 protocol will be stateless, that means we just fire and forget packets and do not check if they have arrived. We just throw the IPv6 packets over the fence. On the other hand, if we need to make sure that all IPv6 packets have arrived – kind of like with a registered letter at the mail where the recipient signs that it has arrived – we use another IPv6 protocol. Lets call it Transmission Control Protocol or TCPv6. But – we have a third use case. Actually a bunch of use cases. How are we going to tell those nodes which IPv6 address they will be having for example? How are we going to announce to them that there is an IPv6 router which they can use? Let’s define a third IPv6 protocol which we will basically use to manage or rather control everything in our IPv6 world. Let’s call this protocol Internet Control Message Protocol or short ICMPv6.

Let’s assign numbers to those IPv6 protocols and quickly write them down on the IPv6 cheat sheet.

(Bootstrapping and scopes)

We have one last problem to solve. How are we kicking everything off – that means – how do we start ICMPv6 communication with the nodes if they don’t have an address yet? You know what? Let’s actually allow them to generate IPv6 addresses for themselves independently. And we call that mechanism Stateless Address Autoconfiguration (SLAAC). The Ipv6 address is stateless because it comes from nowhere. We would just need to make sure that they are unique. Actually, let’s make our IPv6 network a bit larger in order to see what we need. Let’s say we had a bunch of IPv6 routers here. They form IPv6 subnets. That means that they divide the network into smaller IPv6 networks. And actually – we could define three or actually four different ranges or areas or let’s call them IPv6 scope. One would be the internet. Let’s call this IPv6 scope global. Now inside our network – behind the internet gateway - we could basically do whatever we want. Let’s call this IPv6 scope unique local. And if all hosts are on the same collision domain or in other words in the same subnet or on the same switch,  they share the same local link. Let’s therefore call that IPv6 scope Link local. If we go further to the inside then we might add a fourth IPv6 scope and call that one localhost. That would be the machine itself, for example a user connecting with a webbrowser to a software that is running on the very same machine.

Awesome. We are making great progress with our IPv6 protocol. I am positive that at the end of this video we have the perfect IPv6 protocol stack. Quickly writing everything into the IPv6 cheat sheet – here we go. Scope and meaning.

(Scope Prefixes)

Now we thought that we had one challenge, but actually that leads us to a couple more things. First – how would a machine know which scope it is in? Let’s actually solve this by defining address ranges or blocks. This way a node would know the scope just by looking at the address. We arbitrarily chose the following address blocks. Let’s start with let’s say 2001 and then go on … 2002 and so on. Let’s assign blocks inside that IPv6 range to different ISPs or Internet service providers. If we need additional IPv6 blocks for special purposes, then the Internet Assigned Numbers Authority – the IANA – shall manage that. The unique local IPv6 scope shall get the range fc something to fd something. Or – in Classless Inter-Domain Routing (CIDR) notation – fc00::/7 – because the first seven bits are significant here. The link local scope shall get the fe80 colon something range. For the localhost let’s use the most simple thinkable address. ::1

Before I forget it – we want to write this into our IPv6 cheat sheet – address blocks – here we go.

So now we can do the first thing – we can bootstrap or kick off IPv6 communication with addresses that the IPv6 node creates for itself. Inside the link local scope – the fe80 range. To make sure that the IPv6 address is unique, the node can use for example its Mac address or a UID or a timestamp or similar. From now on, the IPv6 node can speak to the first IPv6 router which in turn could then give out another IPv6 address in other IPv6 scopes. But then – if a router goes down, would we then need to change the IPv6 addresses from the global range to the let’s say link local range? You know what, let’s solve this in another way. Let’s allow nodes to have multiple IPv6 addresses. So a node will always have the local host address and a link local address starting with fe80, potentially a unique local IPv6 address starting with fc or fd plus it could have a public or rather global scope IPv6 address starting with 2 or 3 or maybe 4 at some point in time. In other words – if you see multiple IPv6 addresses on your host if you type ip addr or ipconfig /all in Windows – don’t panic – that’s by design. Here’s what that looks like on my PC.

Let’s quickly write all this down in the cheat sheet. Address ranges per scope. Multiple addresses. Plus – let’s make a definition here in order to optimize traffic. Let’s say if two nodes are in the same subnet then they should use the fe80 IPv6 address to communicate. That will avoid that routers need to route local traffic that could have gone over the switch. So I add to the IPv6 cheat sheet – Always use the smallest possible scope for communications.

(Prefixes and Subnets)

What’s missing to make the perfect protocol? Maybe we should add a couple of mechanisms for subnetting here. As we have so many addresses, we can be utterly generous in giving them away. Let’s say we cut the IPv6 address in half. The first half of the IPv6 address would identify the network and the second half the node. So all nodes in one network would share the same four identical first blocks and could have four unique blocks for their IPv6 address inside the network or for their interface I should say. 

Actually – let’s even go further. We don’t have that many networks or - providers I should say - in the internet. 6 bytes or 48 bits will be enough to determine the site or the isp or the backbone where a packet needs to go to in the internet. That will drastically speed up routing in the internet. We can then use the remaining 16 bits of the first block for networks or rather subnets. We could even give whole IPv6 networks to the subscriber – the consumer – you and I. Here in Germany, Deutsche Telekom - who have the 2003 something network - actually allow me as a simple consumer to use the last 8 bits for subnetting. That means they are delegating a 56 bit prefix to me which I can use for subnets as I please. I can therefore create 256 subnets with zillions of public IPv6 addresses inside. Amazing how big the thing is.

Cool – let’s stop here and summarize our findings on the cheat sheet. Subnets, Prefixes, Interface addresses. 

Before we close, I quickly want to show you a couple of very basic IPv6 tools that you of course know already. They are the same like in IPv4 – you may now remember it again. Just – on linux they very often have a 6 at the end in order to indicate IPv6 usage, such as ping6 or traceroute6. Or – with many of those – you can just add -6 in order to tell the tool to use IPv6 rather than IPv4. If you want to open a web page’s IPv6 address, then you can just write the address into the browser’s address bar with square brackets. 

Guys – that’s it for today – We have successfully designed the IPv6 protocol from scratch. I hope you liked the episode. If so – a like on Youtube is much appreciated. In one of the next episodes I will show you how to configure IPv6 with OpenWrt. Plus we might want to see how we can transform our IPv4 arpspoof device into an IPv6 network analyzer. The link to the IPv6 cheat sheet is in the description. Many thanks for watching! Stay safe, stay healthy, bye for now.
</details>
