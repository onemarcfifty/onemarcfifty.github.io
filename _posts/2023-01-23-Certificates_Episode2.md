---
title: "Self Signed and LetsEncrypt Server Certificates for the LAN"
last_modified_at: 2023-01-23T17:00:00
categories:
  - Blog
  - Video
tags:
  - Proxmox
  - Linux
videoid: Z81jegMCrfk
videotitle: "Server Certificates - Self Signed and LetsEncrypt Certificates for the LAN"
toc: true
header:
  teaser: /assets/images/thumbnails/Z81jegMCrfk.jpg
---

## Self Signed and LetsEncrypt Server Certificates for the LAN

How to use Certificates in the LAN? What are our options? We can use self-signed certificates, but we can also use public Let's Encrypt Certificates LOCALLY - in the LAN. In this video we will look at the options such as self-signed CA and Certificates, Let's Encrypt Server Certificates and Wildcard Certificates

The XCA Tool can be obtained [here](https://hohnstaedt.de/xca/)

More Info on my [Cheat Sheet Repo on Github](https://github.com/onemarcfifty/cheat-sheets/tree/main/Certificates)



<a href="https://www.youtube.com/watch?v={{page.videoid}}"><img src="/assets/images/thumbnails/{{page.videoid}}.jpg">Watch the video on YouTube</a>

<details>
	<summary>Click to view the entire transcript</summary>
  (about certificates)
We all know this nasty warning here when we

open web pages in our own intranet.

We don’t have a certificate.

Today I want to show you three alternative
ways to get rid of that.

How can we do that?

First option – self signed certificates.

We will use this great free and open source
software called XCA to generate our own Certification

Authority or CA and our own certificates.

No fiddling around with command line OpenSSL
or the like.

Nice software.

I love it.

Second option – We can use Let’s Encrypt
Server certificates in our LAN – and the

third option that I want to show you are Wildcard
certificates with a dedicated sub-domain.

Let’s go step by step

(self signed certificates)
In the first episode I have already shown

you this very cool software called XCA by
Christian Hohnstädt.

It’s awesome and it’s free open source.

As a reminder – if you browse to https://hohnstaedt.de/xca,
then you can download it for various platforms

such as Linux, MacOS or Windows plus there
is a portable version.

After you start the software, you first need
to generate a database where we will store

all the certificates.

So you click on File – New Database, then
you give the database file a name and after

you have created the file, XCA asks you to
assign a password to that database.

Make sure you use a password you can remember
– there is no way to recover it if ever

you forget it.

(create CA)
So now, we can create our own CA – just

like we did in the first episode.

Click on “New Certificate”, select and
apply the default CA template down here at

the bottom, then move over to the “Subject”
tab and fill out at least the fields Internal

Name, countryName, organizationName and commonName.

Last but not least, generate a new key for
the CA and confirm everything.

(create Server certificate)
Here we go, we have our own CA.

Now let’s generate a Server certificate
that we sign with this CA.

Again, I click on “New Certificate”, this
time I select the existing CA for signing

and I apply the default TLS_Server template.

As an internal and commonName, I now type
in the name of the server that I want to create

the certificate for.

Again – fill out the country and organizationName
fields and generate or use a key.

If you want this certificate to be valid for
multiple names – for example short names

and fully qualified domain names or IP addresses,
then you can add these here under Extensions

as Subject Alternative names.

You could as well generate a wild card certificate,
if you wanted to use one single certificate

for a whole domain for example *.onemarcfifty.com.

(export components)
Cool – now we just have to export the various

components in the right formats and then distribute
them to the right places.

The web server which we create the certificate
for needs the server certificate and the server

key.

Our browser just needs the CA certificate
(again without the private key).

So let’s export everything.

There are various export formats.

Some of them contain the certificate and the
private key, others do only contain the certificate

or the key in separate files.

Let’s start with the CA.

Click on export – the most common format
without the key here is PEM – that’s basically

a base64 encoded or packed version of the
text certificate or chain.

The file extension .crt just makes it easier
for some software to recognize it as a certificate.

So let’s call this one ca.crt.

Next we need the server certificate.

So this is the same game really, click on
export, select PEM as the format, let’s

call this one server.crt.

Last but not least, we need the key on the
server.

So move over to private keys, make sure you
select the right key, Export in pem format.

Just this time let’s call it server.key.

The file extensions and names are not really
standardized anywhere.

The PEM format itself is a standard but the
naming of the files is entirely up to you

or to the software that you are using.

I just like .crt and .key because it clearly
shows what it is.

(distribute components)
I want to use this certificate with an OpenWrt

router.

If I browse to the router’s address then
you can see that I get that ugly security

warning.

Here’s the detail of the current certificate.

I will now replace the original certificate
with my own.

I just need to copy the two server files to
/etc on the router and name them uhttpd.key

and uhttpd.crt.

Before I can use them, I need to quickly restart
the uhttpd daemon on the router.

If I now open the page, then I get the same
warning.

Just this time the certificate is my own.

In order to trust it, I need to import the
CA to my web browser.

I go to settings then security and down here
I can manage the certificates and also import

the CA.

Let me do that.

If I now refresh the page, then it shows up
secure and without a warning.

Mission accomplished.

Let me quickly view and check the certificate
– here you can see all the details like

the issuer, the subject and also all the alternate
names that it is valid for.

Where exactly you have to put these certificates
on your server and how you have to name them

really depends on the server software.

Check my cheat sheet repository on Github.

I will try and make a short overview of the
most common products and the place where they

expect the certificates to be.

So far for self-signed certificates.

The advantage of this mechanism is, that you
can generate any certificate really.

The disadvantage is that you really need to
import the CA certificate to any client and

to all clients that want to browse to that
server.

Now for another method – Let’s encrypt
certificates.

(Let’s encrypt server certificates)
Here is how you would typically use Let’s

Encrypt certificates.

You have a web server running in the internet
and on this server you run the certbot software.

That software periodically connects to Let’s
Encrypt and requests a new certificate using

the ACME protocol.

Let’s Encrypt would then do a DNS lookup
of your server and check if this is the server

that made the request.

If yes, then they would either say – “no
need to renew, your certificate is still valid”

or they would give you a new one.

So how could we use this in our LAN then – in
our local network?

Let’s first have a look how we can use server
certificates and then look at wildcard certificates.

(How to use Let’s Encrypt in the LAN)
If you own a domain – like I do with onemarcfifty.com

– then you could do the following.

You define your internal domain name, so the
DNS domain that you use inside your lan, to

be the same domain like the one that you use
in the internet.

I could therefore use the domain onemarcfifty.com
inside my LAN.

Why not?

I have control over the DNS inside my LAN
because that’s running on my router.

So my proxmox server for example would then
be called pve.onemarcfifty.com.

It’s not reachable from the outside, it’s
just a name I give it internally.

So how would I give this server a public certificate
then?

Well, I would just make an entry for the server
in my public DNS as well and let that entry

point to the same server that has the certbot
agent for let’s say www.mydomain.com.

That entry could be an A record pointing to
the server’s IP address or it could be an

Alias or CNAME pointing to the name of that
server.

The certbot on that server can now request
certificates for that name.

If I browse to that server name from inside
my LAN, then I would not be redirected to

that www server, but rather the machine that
I have running inside my LAN.

So all I’d have to do is copy those certificates
along with the server’s private key from

the WAN machine to the server in my LAN and
that server would then have a valid public

certificate.

Those certificates are located under /etc/letsencrypt
on the public server.

This solution has a couple of challenges though.

Of course, the certificate would only work
if I called the server page by it’s fully

qualified domain name.

Because the certificate had been issued to
pve.onemarcfifty.com but not just to pve.

So I would still get a certificate warning
if I used the short name.

Also – I would potentially need to make
a lot of entries in my public DNS if I wanted

to use this for multiple servers.

Plus – I either need a cloud server or I
have to open ports on my firewall so that

the certbot can be checked by Let’s encrypt.

Here’s another solution: Wildcard certificates.

(Let’s Encrypt wild card certificates)
Wild card certificates from Let’s Encrypt

work like this: your certbot requests a wildcard
certificate let’s say for *.onemarcfifty.com

– now, rather than connecting back to a
specific host, Let’s Encrypt now give you

a challenge: They tell you to put a txt record
named something like _acme_challenge into

your dns.

They give you an arbitrary value to put into
that record.

If they can successfully query that txt record
and if it does contain the value that they

have requested you to put in there, then they
know that you are the owner of that domain.

They issue the wild card certificate.

Yo do NOT need to open any ports.

You do NOT need a server in the cloud for
this.

This method of course does have challenges
as well.

If you want to automate it, then you would
need an API for your DNS provider.

But there’s a lot of howtos on the net for
digitalocean, cloudflare and many others.

I am using a German Web space provider who
do not offer an api.

In this case I can use a manual authentication
hook script that basically logs into their

web ui and makes the entry as requested.

So now I have a certificate that is valid
for any host inside onemarcfifty.com and also

for any domain under onemarcfifty.com – I
could therefore use a subdomain inside my

lan – such as local.onemarcfifty.com.

And I do not have to create any public DNS
entries for my hosts inside my lan.

Any host in that sub domain would show as
valid in the browser if I provided a copy

of the wildcard certificate to it.

I am using this in my network on a reverse
proxy server running nginx.

Whenever I cross network or rather VLAN boundaries,
then the traffic is redirected over this proxy

server in order to ask for a second factor
of authentication.

That reverse proxy has a wildcard certificate
for my domain.

That means that all SSL or TLS traffic that
I have running via that nginx would automatically

be shown as secured and OK.

Just to round this subject up: If you are
using the Let’s Encrypt Certificate method,

you should be aware that due to Certificate
Transparency (CT) or rather Audit requirements

the CA Authorities keep publicly available
logs of every single certificate that they

issued.

That means that everyone on this planet may
know that I have a server called pve.onemarcfifty.com

and also everyone may get a copy of that certificate.

Of course without the private key.

But still I am exposing the naming scheme
of my servers.

May or may not be an issue for you.

Perfect – I hope I could give you a couple
of ideas on how to use Server certificates

in your LAN.

Let me know if you want to have a follow up
on anything – leave me a comment here on

YouTube or start a discussion on my discord
server.

In the next episode I want to show you how
you can secure access to web servers by using

x.509 client certificates.

This does not seem to be used so frequently.

However it can shrink the attack surface of
your servers exposed to the internet considerably.

This is going to be a very interesting episode.

Don’t miss out on it.

Until then – thank you so much for watching,
liking and leaving comments.

Stay safe.

Stay healthy, bye for now.
</details>
