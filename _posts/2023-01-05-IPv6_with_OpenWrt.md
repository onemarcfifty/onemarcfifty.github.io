---
title: "IPv6 with OpenWrt"
last_modified_at: 2023-01-05T17:00:00
categories:
  - Blog
  - Video
tags:
  - OpenWrt
videoid: LJPXz8eA3b8
videotitle: "IPv6 with OpenWrt"
toc: true
header:
  teaser: /assets/images/thumbnails/LJPXz8eA3b8.jpg
---

## IPv6 with OpenWrt

There are at least three ways to use IPv6 ith OpenWrt: Prefix Delegation, NDP Proxy and 6in4 tunnel with Hurricane Electric or the like. In this video we will walk through the configuration for each of them.

The IPv6 from Scratch Episodes are here:

[Episode 1](https://youtu.be/oItwDXraK1M)
[Episode 2](https://youtu.be/jlG_nrCOmJc)

<a href="https://www.youtube.com/watch?v={{page.videoid}}"><img src="/assets/images/thumbnails/{{page.videoid}}.jpg">Watch the video on YouTube</a>

<details>
	<summary>Click to view the entire transcript</summary>
(IPv6 with OpenWrt)
I have some good news and some bad news for

you.

First the good news: IPv6 is fully supported
by OpenWrt.

Because OpenWrt is Linux and Linux does support
it.

Now the bad news: If and to what extend you
can really use IPv6 with OpenWrt is largely

determined by your ISP – that’s the people
who provide internet access to you.

Not by OpenWrt.

Just because you have an IPv6 address on your
router does not necessarily mean that you

can do IPv6 in the way it had been intended
by its inventors.

But we’ll see into that in detail.

There is however some more good news: Even
if you only get IPv6 in a crippled way, then

OpenWrt can help you to get the most out of
it.

Either with prefix delegation or with NDP
proxy or with a 6in4 tunnel.

Let’s walk through our options.

(Option 1 – Prefix delegation)

In an ideal world, your ISP would delegate
an IPv6 prefix to you.

That means that the first 40 or 48 or 56 bits
of the IPv6 address are fix and you could

do whatever you wanted with the remaining
bits.

How can you find out if you have prefix delegation
or not?

Look at the status page of Openwrt.

If you have IPv6 enabled, then you should
see an IPv6 Upstream here.

And if your ISP does delegate a prefix, then
there would be an entry called “Prefix delegated”.

In my case it’s a 59 bit prefix.

My ISP delegates a 56 bit prefix but this
router is inside a test lab sandbox and there

are some more routers in between.

If you don’t see IPv6 at all, then either
your ISP is not giving it to you or you have

previously disabled it.

Let’s check the relevant settings quickly.

And this is on an OpenWrt router with default
settings.

First we go to network – interfaces.

There should be interfaces for LAN and for
WAN and also for WAN6.

The relevant settings on the WAN6 interface
are DHCPv6 client as a protocol, then Request-IPv6

address should not be set to disabled and
in order to actually get assigned a prefix

you should either select a fixed length setting
or chose Automatic from the drop down that

is titled “Request IPv6-prefix of length”.

I have set it to Automatic.

If we want IPv6 prefixes to be delegated further
down the chain, for example to the LAN interface,

then we would also tick this box on the advanced
tab that is titled “Delegate IPv6 prefixes”.

Watch what happens when I untick that box.

You can see that my LAN interface has an IPv6
address in the 2003:de range.

That’s Deutsche Telekom.

Now let me go to the advanced settings of
the WAN interface and remove the tick from

the IPv6 prefix delegation box.

Save and apply.

Checking back on the LAN interface, you can
see that the 2003:de address has now disappeared.

The only remaining IPv6 address on this interface
is the Unique Link (ULA) Address in the fd

something range.

If I go back to the advanced settings of the
WAN interface and tick the box back in, then

save and save and apply, then the global IPv6
address comes back on the LAN interface.

Before we move on let’s quickly talk about
that fd something address here.

You may remember the IPv6 from scratch video
where we talked about scopes.

This unique local address comes from a ULA
scope that we can define here on the network

interfaces on the Global Options tab.

There is an IPv6 ULA prefix tab here.

This is randomly generated at install time.

You could either set it to something more
simple such as fddd::/48 or you can as well

safely delete it – in my opinion there is
no point in having ULA addresses if you have

prefix delegation.

You can safely do everything with global and
link local addresses.

So let me delete it.

Save and Apply – see?

The address has disappeared from the LAN interface.

(Special case: PPPOE)
Before we dig deeper into the LAN configuration,

let’s quickly check two more possible situations.

Maybe you had deleted your WAN6 interface
in the past or your WAN interface is a PPPOE

interface.

So let me delete my WAN6 interface and recreate
it.

Add new Interface, select DHCPv6 client as
a protocol and now – rather than selecting

the same eth1 interface like the WAN interface
has, I now tell OpenWrt that this is a so

called alias interface.

It’s the same interface like the wan interface
but we do not need to do PPOE again, just

DHCPv6.

This way you can run multiple protocols over
one interface.

Just don’t forget to assign it to the right
firewall zone, for instance WAN in my case.

Save and apply and – here we go – we get
everything back.

IPv6 prefix and address on WAN6 as well as
on LAN.

(Firewall rules WAN side)
You remember that IPv6 needs the ICMPv6 protocol

to function correctly?

That means that we might need to tweak the
firewall settings a little bit – especially

if you have deleted the corresponding rules
in the past because you didn’t want to have

IPv6 in your network.

Let’s head over to Network- Firewall.

Here on the traffic rules tab there are just
three or four rules that we need to check

on really.

First of course we need to let in traffic
on UDP port 546 to the router itself for DHCPv6.

The next rule lets in any ICMPv6 traffic from
any device on the local link.

That would be the ISP’s router really.

The next two rules actually let in ICMPv6
traffic from any device in the internet and

will even forward it to any zone.

But we will limit it in two ways.

First, we only need a certain number of ICMPv6
types to be let in.

And second, we will limit the number of packets
to 1000 per second in order to be protected

against flooding attacks.

I will put a dump of the relevant firewall
configuration into the cheat sheet repository

on my github site.

The link is in the description of the video.

(LAN interface settings)
Nice – now let’s quickly walk through

the settings of the LAN interface.

After all, what we want is that OpenWrt assigns
or suggests IPv6 addresses to our clients

in the LAN.

If you remember the IPv6 from scratch videos
then you know that we have essentially three

possibilities to assign IPv6 addresses to
our nodes.

We could give them a static address, we could
use DHCPv6 and/or we could use SLAAC.

Let’s check the settings for this.

I edit the LAN interface and go to the advanced
tab.

We already know this tick box from the WAN
interface.

Just now – we don’t want to delegate IPv6
prefixes further down the stream as presumably

you don’t have any more routers behind the
LAN interface.

So let’s untick that box here.

Next – the IPv6 assignment length.

The network part of the IPv6 addresses are
the first 64 bit – you remember?

As we don’t want to delegate further down,
we therefore set the assignment length to

64 bits.

That means that the LAN network will now have
its own full /64 network but we can’t subnet

any further behind that interface.

But which subnet will the LAN interface get?

I have tuned the WAN interface to have a /60
prefix which means that I can use the last

4 bits of the network mask for various interfaces.

That’s what the IPv6 assignment hint does.

Let’s say I wanted the LAN to be subnet
5 – then I would set the IPv6 assignment

hint to 5.

The IPv6 suffix here defines which address
my router will get inside that subnet.

Usually this is set to ::1, but let me set
it to 99 so that we see what it does.

Save.

Now I save and apply.

Look what this did.

My LAN interface now has a /64 IPv6 address.

The subnet is 74d5 (it was 74d0 before) – that’s
because of the assignment hint which I had

set to 5.

And the node address is ::99.

Let’s add another interface, for example
guest.

I give that one another IPv4 address range,
then I go to the advanced tab and put in the

same values except this time I use let’s
say “a” as IPv6 assignment hint.

I would then expect the guest network to have
the subnet 74da rather than 74d0 or 74d5.

Save and apply.

I did some more things here like define a
device and a firewall zone and I had to restart

the networking service so that OpenWrt applied
the changes.

But once I did that, as you can see - the
subnet on the guest interface is now 74da

as expected.

By the way, when I stop the WAN6 interface,
then the IPv6 addresses of the LAN and guest

interface go away.

Once I restart it, then they come back.

In other words – the easiest way to shut
down IPv6 in your whole network is to just

stop the WAN6 interface.

When you bring it back up, IPv6 comes back.

Nice and clean.

Cool, now let’s have a look at the DHCP
settings.

How do we tell our clients in the LAN that
they can get IPv6 addresses from OpenWrt ?

(Configuring DHCPv6 options)

If we want to have global IPv6 addresses in
our LAN, then we need to let our clients in

the LAN know which IPv6 prefix they should
use, which IPv6 router and a couple more things

like the address of the DNS server.

We can do this either via router advertisement
or via DHCPv6 or – we can mix both mechanisms.

Here is how.

I go to Network-Interfaces, then I click on
“Edit” next to the desired interface – let

me use the LAN interface for starters.

Under “DHCP Server” I can see the IPv4
DHCP settings.

If the DHCP Server is not yet configured,
then you get a button here to enable it.

What I am interested in, are the IPv6 settings.

Here I can tell OpenWrt that I want to have
two things.

First, I want this router to be advertised
using ICMPv6 router advertisement.

For this I select “server mode” in the
RA-Service dropdown.

But I also want this router to be an IPv6
DHCP Server.

Therefore I also select “server mode”
in the DHCPv6-Service dropdown.

My router shall also serve DNS so I do check
the “Local IPv6 DNS server” tick box.

Please do not tick the designated master tick
box and also set the NDP-Proxy dropdown to

disabled.

We don’t need this here, we’ll talk about
this in the second scenario.

On the next tab (that is titled IPv6 RA settings)
I can define some more details.

First – do I want my router to be the default
router?

I set this value to automatic.

Now for the interesting part.

The tick box “enable SLAAC” actually decides
whether we allow our clients to do stateless

automatic address configuration.

If we tick this box, then the corresponding
flag in the router advertisement packet gets

set.

In the next box we can configure some more
flags for the router advertisement.

If we set the managed config flag, then this
tells our clients that they should request

an IPv6 address via DHCPv6.

If we tick the “other config” box here
then this means that we let our clients know

that they can have more info such as DNS servers
over DHCPv6, even if they don’t request

an IPv6 address via DHCPv6.

I know that this part may sound a bit confusing,
especially when you are new to IPv6.

Please refer to the second episode of my IPv6
from scratch videos and also have a look at

the comparison table on my IPv6 cheat sheet
from my github repository.

The table here helps you decide which mechanism
to use.

How am I using this at home?

You may remember from other videos that I
have a dedicated management VLAN here.

This is where all my routers, servers, switches
– generally speaking the whole infrastructure

– provides https and or ssh interfaces.

These devices need to be identifiable, therefore
I don’t use SLAAC on that interface.

On my LAN interface however, where all the
laptops are in, I do not need fixed addresses

and therefore I only have SLAAC on that interface.

Still – even if I do not set the “Managed
config flag”, I can still use DHCPv6 even

in my LAN by forcing the client to do so.

For example these Proxmox containers here.

They do get an IPv6 address from the LAN range
over DHCPv6, not SLAAC.

I can do this because I set the IPv6 address
to DHCP in Proxmox and also I told the container

to NOT use SLAAC by setting this autoconf
value to 0.

As you can see, it does have a global IPv6
address, but its a nice clean address coming

from DHCPv6.

In a nutshell – the fact that we set the
DHCPv6 Service to server mode on the first

tab means that the router will answer to DHCPv6
requests on UDP port 546 to the multicast

address ff02::1:2.

The RA-Service dropdown determines whether
we answer Router Solicitation requests on

ICMPv6 or not.

If I set this to disabled, then the second
tab disappears.

(static leases and DNS)

Perfect.

Now we have IPv6 configured on the WAN and
LAN side.

Just – we have a handful of little challenges
here.

Let’s say we don’t have a fixed IPv6 prefix,
we rather get a new one from the ISP every

24 hours or so (like I do with Deutsche Telekom).

How would we manage DNS and firewall rules
for nodes that have ever changing IPv6 addresses?

Let me show you the problem.

I go to the firewall Traffic rules and I want
to make a rule for this sandbox-adguard server

here.

If I made a rule now and then later the prefix
or the address of this adguard server changes,

then my rule would be useless.

How can I solve this?

First, I would need to make sure that this
server gets a constant or static node address.

I am not talking about the prefix.

Just the lower 64 bits that identify the node
itself.

Also – I know that the server will always
have the value 5 as the last 4 bits of the

subnet – you remember?

So – let’s start with the node address.

Going to the status page of my router, I can
see all the clients that have DHCP leases

at the moment.

For both IPv4 and IPv6.

Using this “Set static” button I can make
sure that this same device gets the same IP

address the next time it requests one.

The unique identifier for this would be the
MAC address in the IPv4 world.

In the IPv6 world, the machine has sent a
unique identifier, the so called DUID to the

DHCPv6 server.

The client is responsible for generating this
and needs to make sure that it is always the

same value.

You can actually see portions of the MAC address
in the DUID.

Unfortunately, if I now make both the IPv6
and the IPv4 address static, then OpenWrt

will create two entries here under Network-DHCP
and DNS- Static leases.

That would work but I want to make sure that
the IPv6 address also resolves correctly with

DNS.

Therefore, let me delete the IPv6 entry and
rather edit the IPv4 entry.

Here I have two parameters that I can tweak
for IPv6.

In the DUID dropdown box I can actually select
the adguard server.

I can see that’s the same MAC address.

Or – if I have many machines or if I can’t
find it here, then I could also just copy

paste it in here.

The next parameter is interesting and actually
lets me define the node address portion of

that machine.

Let me put in 245, because the IPv4 address
ends in 245.

I know it’s not the same thing.

The one is decimal and the other one is hex,
but they look the same.

Save, then Save and apply.

The next time that this machine requests an
IPv6 address, that address will end in 245.

I can accelerate that a bit if I restart the
odhcpd service on the router and down and

up the network interface on the client.

Takes a while.

Here we go.

Quick cross check with nslookup.

Yep.

That comes back with both addresses.

IPv4 and IPv6.

Beautiful.

So – how does that help me with my firewall
rule now ? Let’s have a closer look at this

server’s address.

We have the prefix which is 60 bits long.

Then we have the 4 bit subnet which I can
manage on my own, then we have the node address

which is many zeros and then 245.

If I apply a bitmask to this in the sense
that I set all bits of the relevant (fixed)

part to 1 and all bits of the changing prefix
to 0, then I can write this address in subnet

mask notation like this.

Not beautiful, but it works.

So rather than having a firewall rule for
the whole IPv6 address, I now have one for

any IPv6 address in subnet 5 that ends in
000... 245.

I could now even have one single rule for
IPv4 and IPv6.

Of course I could have used the MAC address
as well.

(NDP proxy – no prefix delegation)

So far so good.

This all works if we have prefix delegation
by our ISP.

Unfortunately, many ISPs do not offer prefix
delegation, but they rather just give you

a /64 subnet.

What can we do here?

In this case, you would not see any delegated
prefix here on the status page or on the WAN

interface.

Just a /64 address really.

And no IPv6 address on the LAN or guest interface,
because there are no more subnets to be delegated

really.

The whole /64 space is already “eaten up”
by the WAN interface.

If you still want to use IPv6 in your LAN,
then you can configure OpenWrt as an IPv6

NDP proxy between the WAN and LAN zone.

NDP stands for the Neighbor discovery protocol.

Here is how.

First, we need to edit the WAN6 interface
and add DHCP settings.

Important – we don’t want to run a DHCP
Server on the WAN zone here, so make sure

that this “Ignore Interface” Checkbox
is ticked.

On the IPv6 Settings tab, we tick the box
“Designated Master” and we set all other

drop downs to “relay mode”.

On the interfaces in our LAN or guest network
we now set all drop downs to relay as well.

Once we save and apply, we won’t really
notice any difference on this screen.

But watch what happens if I turn the network
connection on my client off and on again.

Tadaaa – I get an IPv6 address.

My OpenWrt router now proxies all my requests
through to the WAN side.

Unfortunately, our possibilities what we can
do with IPv6 LAN side are quite limited with

this setup.

In a nutshell we get a SLAAC or DHCPv6 address
for our nodes and that’s pretty much it.

(6in4 tunnel)
So what can we do if we want to enjoy full

IPv6 functionality but our provider doesn’t
give it to us?

There is a third alternative called 6in4.

This basically tunnels IPv6 through IPv4.

In order to use this you have to sign up to
a tunnel broker on one side and create a 6in4

interface on your side.

The tunnel broker will now give you ipv6 connectivity
through a tunnel inside ipv4.

Pretty similar to a VPN really.

There are IPv6 tunnel brokers who give you
a whole /48 prefix for free like Hurricane

Electric for example.

All that you have to do is sign up on their
site tunnelbroker.net.

From here, you can create a new tunnel to
many locations on various continents.

Let me add one for New York here.

All that you have to give them in order to
be able to generate a tunnel is a valid IPv4

address which they can ping in this IPv4 endpoint
field here.

So I am not sure if this works with double
Nat (Carrier grade NAT / CGN).

Leave me a comment if you have it up and running.

Also you might need to add a firewall rule
to your router so that they can actually ping

you.

Don’t forget to select /48 prefix.

Now for the configuration on the OpenWrt side.

You go to system-software and install the
6in4 software package.

You will need to reboot after this so that
LuCI shows the new protocol.

Once you’re back in, go to network-interfaces
and add a new interface with the IPv6-in-IPv4

protocol.

Enter all the information from the tunnelbroker
website.

Just down here where it says HE.net password,
enter the dynamic update key.

This way hurricane electric will automatically
update the IPv4 endpoint for you.

Don’t forget to assign the interface to
the WAN firewall zone.

And – there we go.

Beautiful /48 prefix delegated to us.

Full IPv6 functionality.

Awesome.

You might wonder – why does Marc not talk
about IPv6 NAT?

Well, there’s two reasons for that.

First – I have never used it.

I am only using IPv6 for a couple of months
now and really want to understand the whole

thing fully before I examine things like IPv6
NAT.

Second, OpenWrt have just recently moved from
iptables to nftables as their firewall and

currently it looks like IPv6 NAT relies on
Iptables.

So I guess we will just have to wait a bit.

Even though IPv6 NAT might not sound right
to the IPv6 purists among us, I do actually

have a use case for it which is reverse proxy
between network segments.

So – there’s probably more to come.

Stay tuned.

Until then – many many thanks for watching,
liking and commenting.

Stay safe, stay healthy, bye for now.
</details>
