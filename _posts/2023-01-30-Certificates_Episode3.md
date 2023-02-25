---
title: "TLS X.509 Client Certificates"
last_modified_at: 2023-01-30T17:00:00
categories:
  - Blog
  - Video
tags:
  - Proxmox
  - Linux
videoid: 5lYQRuzdZr0
videotitle: "Secure your Cloud Services with TLS X.509 Client Certificates"
toc: true
header:
  teaser: /assets/images/thumbnails/5lYQRuzdZr0.jpg
---

## TLS X.509 Client Certificates

How to secure Internet Servers with X.509 Client Certificates? How to deploy X.509 Client Certificates ? How does a Certificate Signing Request (CSR) work ? In this hands-on video we will run a little nodejs Server that requests Authentication with an X.509 Client Certificate, we will Sandbox a CSR with XCA and we will have a look at OpenXPKI which is a great Software to automate processes around TLS and Certificate Generation, Key Management and the like. Last but not least I show a Blueprint on how to securely link a hosted MQTT into your home automation Software.


The nodejs Server Example is [on my github](https://github.com/onemarcfifty/client-cert-test)

The XCA Tool can be obtained [here](https://hohnstaedt.de/xca/)

More Info on my [Cheat Sheet Repo on Github](https://github.com/onemarcfifty/cheat-sheets/tree/main/Certificates)


<a href="https://www.youtube.com/watch?v={{page.videoid}}"><img src="/assets/images/thumbnails/{{page.videoid}}.jpg">Watch the video on YouTube</a>

<details>
	<summary>Click to view the entire transcript</summary>
  (the use case)
Let me show you how I open the door to my

house or to my garage at home with my mobile
phone.

I open this app where I can type in a pin,
click on this button and then the door opens.

That app is in fact just a web page running
on a web server in the cloud.

Now I hear you say – “Marc, are you mad
– do you really allow everyone to open the

door to your house at night?” - of course
not – the site is protected with a client

certificate.

That means that my phone is the only device
on this planet that can browse to that web

site.

Here is how I did it.

We had talked about certificates and about
this nice GUI here called XCA in the last

episode.

I showed you how to generate a Certificate
Authority, a CA, and how to generate server

certificates.

Now the great thing with X.509 certificates
is, that we can also use them for client authentication.

That means that a web server would only answer
requests coming from a device that has the

right certificate.

All others will be rejected.

Imagine how you could secure for example your
Nextcloud instance, your personal photo collection

or – like I do – simple apps to interface
with your IOT at home in a way that only you

can use them from your tablet or phone.

(Demo server from my Github)
If you browse to my github repository, then

you can find an example for nodejs that does
exactly what I showed here – it shows a

web page with the two input lines for a pin
and two buttons.

In order to use it, you will need a host with
nodejs installed.

Windows, MacOS or Linux – it doesn’t matter.

Just copy the files to a subdirectory and
start node testserver.js.

The server now answers requests on port 8443.

So if you now browse to let’s say https://testserver:8443/form
then you will most probably get a certificate

warning.

In the ca subdirectory you can find a self-signed
certificate authority (CA) that you can import

into your browser.

In Chrome and Firefox this is done under the
security settings.

Alternatively - if you want to run this on
a server with valid Let's encrypt certificates

or the like, just replace the files server.crt
and server.key with the certificate and private

key of your "real" server.

Don't replace the ca.crt file as it's used
for client authentication!

The files server.crt and server.key in the
ca subdirectory have been signed with the

ca.crt for testing purposes.

If you want to test without certificate warning,
then you could just make an entry into your

host file and point testserver to any host,
for example your localhost by adding the following

into your /etc/hosts file (on Linux) or your
C:\Windows\System32\drivers\etc\hosts file

(on Windows).

If you now browse to the URL again, then your
browser should tell you that you have been

rejected as you don't have the right client
certificate installed.

That's the purpose of this exercise ;-) - In
order to gain access, you need to import the

file testuser.p12 from the xca.db sub directory
into your browser.

Let me do this.

I’ll go again into the security settings
of my browser, then import the certificate.

The password for this is hello.

Let me open the page again – and now the
magic happens.

My browser tells me that the server is requesting
authentication.

It asks me which certificate I should provide.

If I now select the testuser certificate,
then the page opens.

I can use it as desired.

Without having to type any username or password,
without using a VPN.

Just plain TLS certificates.

Very transparent.

On my phone, it would just use the certificate
without asking.

(How to generate client certificates)
So – how can we generate our own client

certificates then?

Just like in the last episode, we will use
XCA for this (you could do this on the command

line as well with OpenSSL of course).

Launch XCA and open the xca.db file from my
github – this already contains some sample

data.

As you can see, there is a CA, a server certificate
and also a client certificate.

How did I generate the client certificate?

Well, pretty much like the server certificate
in the last episode.

I clicked on “New Certificate”, just this
time I chose and applied the TLS_client template.

As a common name this time I just entered
the name “testuser” And of course this

one also needs a key.

So I generated a new one.

In order to be able to use this with my web
browser or phone, I exported it as PKCS#12

file, because that file format contains both
the key and the certificate and also I know

that my iPhone can read that type of certificate
files.

As there is a private key inside, I now needed
to protect it with a password.

The password for all the sample data on my
github is just “hello”.

(how secure is this)
You might have doubts on the security of this.

And you are right to challenge that.

We should not take security lightly.

In a nutshell, the certificate is just another
authentication factor.

An authentication factor can be something
you know – such as a user name and password

– it can be something that you are – so
biometric data, or it can be something you

have – like a key, a token or a certificate.

So if you wanted to let’s say make the admin
frontend of your nextcloud or photo sharing

app or anything else super secure, then I
would not ask you to remove or replace the

password authentication, but rather add the
client certificate requirement on top of it.

That would reduce the surface that you potentially
expose to hackers by a huge percentage – I’d

say above 99% really.

Why?

If you have services running in the web – be
it a password protected web site or an ssh

server or whatever – then you will see hundreds
if not thousands of log entries from people

or bots attempting to connect to it.

Every single day.

They get to the login prompt and now they
can start guessing passwords or using cross

site scripts or injection attacks to your
frontend.

If you had that site locked down with a client
certificate, then all these attempts would

not even get to that page on the application
layer because they would be rejected by TLS

a layer below.

On the presentation layer, between the TCP
protocol and the application.

Still, there are remaining attack vectors.

If someone gained control over the server,
then they could of course do whatever they

wanted there.

You still need to secure the server.

If someone had gained control over your client,
then they could potentially steal the client

certificate and use it.

If someone is able to get into the middle
between you and the server, then they can

potentially intercept data as well and exploit
weaknesses of the protocol.

Last but not least, if someone had a day 0
exploit to TLS, or if you used an old version

of TLS that has known vulnerabilities, then
they could use that as well.

But still – the security level is not worse
with certificates than it was without certificates.

All these attack vectors can be exploited
with or without certificates.

(how to deploy client certificates)
The main challenge with this concept however

is the distribution of the client certificates.

Not a big thing if you need to put it into
one single web browser.

But how would you distribute these let’s
say to 10 or 100 clients?

And how would you distribute them to let’s
say an iPhone?

In an enterprise environment those devices
would be managed by a central Mobile Device

Management platform such as intune or altiris
or the like.

In the home environment this is a challenge
though.

Let me first quickly show you how it should
NOT be done.

I’ll attach the PKCS#12 file (which includes
the certificate _and_ the private key) to

an email and send that email to my mobile
phone.

Now this is great in the sense that I can
now easily open and install the certificate.

But even though the PKCS#12 format is password
protected, this is a weak protection.

Especially if you chose a super weak password
like 12345 or hello.

This would actually open an additional attack
vector – the interception of the certificate.

(Certificate Signing Requests, CSR)
So how to deal with this?

In a real life secure scenario, the private
keys should never ever leave the system where

they have been generated on.

Now – I do not have an app that allows me
to generate keys on my iphone, so let me show

you this with two instances of XCA.

One instance would be the client who wants
to have a client certificate signed by the

CA, let’s call him Marc – that’s me,
and the other instance would be the site that

has the private key for the CA, let’s call
it onemarcfifty.com.

So first, I would download the ca certificate
from onemarcfifty.com.

The public part.

Without the private key.

Then I would generate a private key which
I would want to use with that certificate.

Now – rather than creating a certificate
(which I could not sign anyhow because I don’t

have onemarcfifty.com’s private key), I
would now generate a Certificate Signing request,

a CSR.

That CSR contains the public key that complements
my private key.

And I would then submit that CSR to onemarcfifty.com.

So let me export this on Marc’s side and
then import it on onemarcfifty.com’s side.

On the Onemarcfifty.com side, this CSR can
now be signed with their CA.

That generates a certificate below the CA
here.

Now Onemarcfifty would export that ready made
certificate which now contains the public

keys of Marc and of the onemarcfifty.com CA.

But still no private key has been transferred.

Onemarcfifty.com send me that certificate
and I’ll import it.

Because I do have my own private key plus
the public key of the CA, the certificate

will now be properly linked to the CA and
I can use it.

You could in theory do that workflow over
e-mail – I would really not recommend it,

but you could.

No secret information was transmitted at any
point in time.

Just - a thoroughly forged man-in-the-middle
attack could still be possible.

A malicious attacker who has access to your
e-Mail could replace your request with their

own and hence get granted access to onemarcfifty.com.

Also – some information that you transfer
with the CSR could be used to lower the effort

for brute forcing the encryption.

Please remember that most successful attacks
do not attack the encryption.

They rather try to circumvent it.

And many popular attacks in the past aimed
at the authentication process.

That means the part of the process where you
prove that you are authorized to obtain a

certificate.

If you send the CSR via e-mail then the only
authentication is your e-mail address.

The CA can’t verify your key.

There is actually free and open source software
available for that type of workflow.

Have a look at openxpki for example.

You can self host this software.

For testing purposes, there is a demo site
where we can actually log in with demo accounts

and submit CSRs and they’ll sign it for
us.

So I create a CSR

- export it
- Generate a Request on the openxpki platform

- 

Now – here I am at the same time the officer
who approves this request – normally that

would be a 3rd person.

- And now I can download the signed certificate
from here.

So – if I made the access to that site quite
secure with let’s say an additional 2nd

factor such as a one time password or a temporary
code sent via text message, or the site would

only be available in the intranet for example,
then I could make that quite secure – by

home or small business standards.

If I go back to the initial application that
I have shown in the beginning, then we could

even take this blue print further.

Many home automation products can use MQTT
as a protocol.

Let’s say you have your MQTT Server running
here at home in a container or on your router.

And here on the other end we are in the internet,
so outside of the LAN.

In order to open the door at home we need
to publish a command to MQTT.

Now here we could have a VPS, a cloud server
for a dollar or a free oracle instance or

the like that has Letsencrypt Server certificates.

If we add a self-signed CA here then we can
do the following.

Our MQTT at home could communicate over TLS
with an MQTT instance here on the VPS.

Our phone could securely connect to a little
nodejs server here using client certificates

and issue the command to the local MQTT which
would in turn relay the command to the MQTT

at home which would then instruct the home
automation to open the door.

No passwords, no VPN, no hole in the firewall,
no port forwarding.

Nothing.

Just the right certificates at the right places.

Guys, we’re running out of time.

Let me know if you liked this episode.

Leave me a comment if you want follow-ups.

All links and the example code are in the
description and on my github server.

In any case – thank you so much for watching.

Stay safe, stay healthy, bye for now.
</details>
