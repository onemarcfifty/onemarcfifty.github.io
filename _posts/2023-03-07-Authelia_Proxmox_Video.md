---
title: "Authelia on Proxmox - 2FA SSO with Nextcloud, Proxmox, Portainer Gitea OpenID Connect Single Sign On"
last_modified_at: 2023-01-30T17:00:00
categories:
  - Blog
  - Video
tags:
  - Proxmox
  - Linux
videoid: FMMCLt9TM2U
videotitle: "Authelia on Proxmox - 2FA SSO with Nextcloud, Proxmox, Portainer Gitea OpenID Connect Single Sign On"
toc: true
header:
  teaser: /assets/images/thumbnails/FMMCLt9TM2U.jpg
---

# Running Authelia on Proxmox

This video is really following up to [this blog article](https://www.onemarcfifty.com/blog/Authelia_Proxmox/) which I published earlier here on my blog. 

## Video content

I will show how to Self-host Authelia in a Proxmox Container and use it as an OpenID Connect (OIDC) Identity Provider for 2FA Single sign On (SSO) with Nextcloud, Proxmox, Portainer and Gitea.

#nextcloud  #proxmox  #sso #portainer #gitea #authelia #openid #oidc #selfhosted 

The Github Repo is [here](https://github.com/onemarcfifty/authelia-proxmox-SSO)


<details>
	<summary>Click to view the entire transcript</summary>
(Intro – VLAN / NGINX / SSO)

I am self hosting a bunch of applications. Nextcloud, Proxmox, Portainer, Gitea and so on. The more applications you have, the more user names and passwords you need to manage. Quite a challenge. Or – you use SSO – Single Sign on. Let me show you this. I browse to my NextCloud instance. But rather than logging in with username and password, I click on this link and am redirected to Authelia for login. Authelia asks me for a user name and password as well as a second factor. Here we go, I am logged in. How does that help or make anything better than before? Let me browse to my Proxmox Server UI. This is where the beauty happens. As I am already logged in to Authelia, I can now log into Proxmox without having to type anything again. Just need to consent and I am in. Also – Up here I am not root or admin, but I am Marc. Same with Portainer. Click on SSO, consent. Logged in.  One single account for all my applications. Very convenient.

But first - here’s the breakdown of this episode. Feel free to browse to the corresponding time marker. I’ll explain how everything works, why I chose Authelia over anything else, explain the necessary setup steps in detail, then we’ll configure Nextcloud, Proxmox, Portainer and Gitea and I will then outline a couple more use cases. If you already have Authelia and you’re just interested in installing the SSO bit then go to that time marker here.

(How does that work?)
Here’s the blue print. The core is Authelia running in a container on Proxmox. You can run it in Docker as well. Authelia has a Database or a simple file with all the user accounts in it. That could be LDAP – in my case it’s just a simple text file. I only have a bunch of users, like ten or so. If I had hundreds or thousands, then I’d use LDAP for that. If I wanted to add Authentication to a web site that is not protected, then I’d just add a reverse Proxy in front of it and require Authentication prior to Access. But after that I would still have to log in to that Application. So here’s the second feature of Authelia. It can act as an OpenID Connect (OIDC) Identity provider. So if my Web App can use that, like Nextcloud, Proxmox, Gitea or Portainer can do – Rather than asking for a user name and password, they would just check with Authelia if I am already logged in. If I can present a valid cookie from Authelia, then I am logged in – if not, then I would be redirected to the Authelia Login page. Once I am logged in there, I receive a new Cookie and am then redirected to the Application again. The login is therefore done by Authelia and the Application just trusts it. That brings a couple of advantages. I can log in with the same account everywhere – I have Single Sign on, SSO. Once I am logged in to one application, the login is valid for other applications. Last but not least, I can require a second factor for additional security.  That can be a Time based One Time password (TOTP) with Microsoft or Google Authenticator or Duo or the like. Or it could be a push Authentication with Duo as a service provider. They offer ten free accounts. Or – it could be a Webauthn FIDO2 key like those Yubi Keys.

(Why Authelia?)
Why did I chose Authelia over other solutions? Well, first off – it’s free and Open Source under the Apache License. I am a big fan of Open Source. Second – I can easily self host it in a Container. I don’t have to rely on 3rd party service providers. And Last - but by no means least – They do have a clear and concise Road Map! They do have a clear view where they want to go and when they will do things – that’s really essential and unfortunately not true for many other open source projects. If we look at the OpenID Connect Roadmap, we can see that they plan 8 (!) beta phases before GA. That’s quite thorough planning I’d say. I like that. Also – there’s a bunch of other functionalities in the pipe – like multi domain, multi device WebAuthn registration and passwordless login. I personally feel quite confident about this.

(Setup Steps – Overview)
Here is an outline of the necessary setup and configuration steps. We will first do what I call a rudimentary installation – without start up checks. Just to see if we can get the software up and running. In a second step we will then adapt some settings - Domain names, e-mail server parameters, the SSL certificates. Then we enable the startup checks and get everything running with a fully blown config. Next, we will register a 2FA device and then hide Authelia behind an NGINX reverse proxy. Last but not least, we will add the OpenID connectors to our applications and then use SSO happily ever after ;-) By the way – there is also an article on my blog site www.onemarcfifty.com describing the installation. I have just packed everything into an automated script on github for you.

(1. rudimentary install)
OK, let’s start with a rudimentary install of Authelia. After this step, we will have authelia up and running, but not configured yet. First we need an LXC container in Proxmox. It needs to be a privileged container because we need the /dev/random and the /dev/urandom devices really. I have given it 2 GB of RAM and 2 Cores plus 8 GB of Disk. This is probably even a bit over-sized. As a template I used the latest Debian 11 template. Right – once you have the container up and running, just pull the installation sources from my github using this one-liner here. That should take care of all installation steps and give you an up and running authelia server. In order to test this, just browse to http port 9091 of that server and you should be presented with the authelia login screen. Use http here, not https. We don’t have certificates yet. Just we can’t really use Authelia yet because we need to tweak some more settings.

(2. Adapt settings)
Now comes the hardest part. We need to adapt the config file. Authelia heavily relies on e-mail. This is how users reset their passwords and also how they register 2FA devices. In other words – we need a real e-mail account. The second barrier are certificates. You will need TLS/SSL Server certificates. If you don’t know how to do that then please watch the second episode of my X.509 Certificate series. Plus we will need to adapt the domain and server names. Now – all these settings can be changed in the config file /etc/authelia/configuration.yaml. However – editing a yaml file with nano or vi is – well – painful at least. That’s because YAML is very picky with regards to indentation. So here are some alternatives. Either you install vs code server in the authelia container – like we did in the rundeck video. Or you use winscp or Filezilla and access the server over ssh. In order to do this, let’s temporarily enable ssh root access with password by changing the /etc/ssh/sshd_config file. Restart the sshd service and now we are able to edit the file using a GUI editor like vs code or notepad++ or the like. The settings we need to change are outlined in my blog article as well. I’ll change the default redirection URL and the TOTP issuer. I review the policies and adapt them to my needs. I’ll just request 2FA for everything in my domain and deny everything else. Then I put in the SMTP Server details. The passsword for the SMTP Server has been put into the file smtp in the .secrets subfolder of the /etc/authelia directory. It has been randomly generated at install time. You can either copy paste that and set the mail password of your provider to that secret or you overwrite the secret with your mail password. Once we have that in, then we can enable the startup checks by setting the disable_startup_check value to false. Let’s save everything and see if authelia still starts. It now tries to connect to the mail server at startup. If it can’t do that then it will not start. You will need to review the settings. Perfect. Once we have this running, then we can activate TLS. For this we need to add this TLS block here. Just point it to the key and the full certificate chain of the server certificate. I am using a self signed certificate in this test sandbox. Again, save and restart Authelia. If everything went well until here, then we should now be able to browse to port 9091 using https rather than http. Perfect. Just one more step and we’re done with the basic config here. The installation script has created a file called users_database.yml in the .users subfolder and added four example users – Bob, Alice, Dave and Frank. Of course we want to edit this and point it to real user accounts with real mail addresses. Don’t worry about the password. We will reset it in a second. You can also add additional parameters such as groups. Right – again, save and restart authelia. You should now be able to use the “reset password” link, receive an email that is valid for five minutes btw, and set a new password. Some helpful troubleshooting tips if anything goes wrong in this phase: Check the authelia log and the service status using these commands.

(3. Test 2FA)
Cool – now let’s test 2FA. If I click on register device here, I do again receive an email with instructions. The link in the mail again is valid for 5 minutes. Now I can either register an Authenticator app such as Microsoft Authenticator, Google Authenticator, Okta or Duo or the like – it doesn’t matter, they should all be able to scan that QR code and add the Authelia Secret to their config. Alternatively – if you have a Yubikey or any other Fido2 Webauthn key then you can as well register that. Depending on the policy that you set in the config file, you will or will not be prompted for a second factor once you log in. If you want to use DUO push notification then you need to register on their site and get the details like hostname, integration key and secret_key. The first ten users are free of charge at the time of making this video.

(4. Hide behind NGINX)
Awesome – now we have a working Authelia instance using https and we can now hide everything behind NGINX. The installation script should already have installed NGINX in your container. All we need to change now is the listen address in the authelia config file and point it to the local host 127.0.0.1 and also adapt the server name in the NGINX config file which you can find in this path here. Again, use Filezilla or Winscp. Once we restart both services, we should then be able to access Authelia using https on the standard port 443.

(5. Add OpenID Connect to the apps)
Perfect – now let’s add Authelia as an OpenID Connect provider to our apps. Let’s start with Nextcloud, then we’ll do Proxmox, Portainer and Gitea.

The steps for the different platforms are clearly outlined in the authelia documentation. There is a whole section with examples for many platforms. In general, we need to tell the App that it should use OpenID and we also need to tell Authelia about it. On the Authelia side this is done in the configuration YAML File. We first specify some generic settings under the identity_providers section, such as the validity duration of a login and who we accept requests from. Then we create an entry for each app under the clients subsection. Each client app needs to have a unique ID, a Secret Token, an Authorization policy and we need to tell Authelia which URLs are valid to be redirected to. This can be multiple destinations. Also, depending on the app, different scopes are possible. Usually you should always have openid and profile in here, plus maybe email and groups.  The hardest part here is really double-checking on all the information for typos etc. plus of course the indentation of the YAML syntax. Cool, now let’s add the client apps. Quick remark here – please do never use the Client secrets provided in the examples, but always create your own and make sure they are long – like at least 30 characters long. You will never have to type them by hand anyhow. This simple one liner from my cheat sheet repo can help you create them on the fly.

For Nextcloud, we first need to install the OpenID Connect Login App. Let’s do that here in the Nextcloud GUI under the active apps
The configuration for the OIDC provider is done in the config.php file which by default is located in /var/www/html/nextcloud/config.

The only settings you need to change from the template on this page really are the login provider URL which should point to the FQDN of your authelia server and the client secret. Please use a long random character chain for the secret. This one liner here from the setup script or from my cheat sheet repo on Github can help you create one. 

On the Authelia side we can also just copy paste the config section into the config file. The secret needs to be the same like the one that we just created on the Nextcloud side. Just – it needs to be preceeded by the term $plaintext$ - I will need to check if we can specify this encrypted as well.

Once you restart Nextcloud and Authelia, then you should be able to use this link in Nextcloud and be prompted for login by Authelia. On successful login you would be redirected to your Nextcloud instance. Please keep in mind that Authelia will only forward you to secure web sites. That means Web sites that use https and have valid certificates. If you don’t use https then this will not work. Authelia will categorically refuse to forward you.

When I did all this, I encountered a couple of problems. First I had to remove this line for the alternate login page from the authelia template – that page does not exist on my system. Error symptom was just a white page instead of the login screen. Second – As I am using self signed certificates, I got this error page here complaining about the certificates. Solution was to give php on the Nextcloud Server a copy of the root CA and add this line to /etc/php/7.4/apache2/php.ini. Third – After clicking the login button I received this error – turned out that the redirect URL contained the index.php parameter where it shouldn’t. I first just added that as a valid redirect. But the real “proper” solution is to add this line into the nextcloud config.php and then launch the nextcloud occ php with the maintenance:update:htaccess argument. Once I restarted Apache and reloaded the page, then the index.php was gone from the URL.

(Proxmox)
Right, now let’s do ProxMox.

The process is quite similar. Just this time we can use the Proxmox GUI in order to tweak the settings on the Proxmox side.  Under Datacenter – Permission - then Realms we can add an OpenID Connect Server. The issuer URL is the URL of our Authelia Server. The realm is a name that you can freely chose, I chose authelia. Same for the Client ID and the client Key. You just need to specify the same ID and Key on the Authelia side in the next step.  Username Claim and Scopes need to be entered exactly as shown here. If you tick this box, then users will be automatically created once they log in. They will not have admin capabilities by default. Proxmox does not seem to honor groups. You will need to manually log into Proxmox with a PAM account, then add the user to the admin group manually. If you want to create users before thy log in, just create them here under “Users” and specify the new realm.

So far for the Proxmox side, now let’s tell Authelia about it. You can again copy-paste the snippet from the Authelia documentation and just adapt the values. The id and secret correspond to the Client ID and Client Key on the Proxmox side. The redirect URL needs to be the URL of your Proxmox Server.

Once this is working, we can again log into Proxmox, just this time we select the authelia realm and – tadaa – we can log in using SSO. There are just a handful of limitations with such an account, namely you can’t open a shell on the Proxmox VE Server itself. Also, you can’t change Privilege information on Containers. You need to be root for this. Everything else should work without a problem.
I did hit a couple of road blocks here as well. First I forgot to specify the port number 8006 in the proxmox redirect URL. Second – again because I use self-signed certificates, I had to copy the rootCA over to the /etc/ssl/certs folder on the proxmox server and then install it to OpenSSL with this command.

(Portainer)
Awesome – we’re on a roll here – let’s do Portainer next. Here, we can do all the settings in the GUI again. We go to Settings-Authentication, then chose OAUTH as the method and enter all information like in the other two examples. Again – guys, use a different secret for each client app. Use a long key. Do not use the example keys. Portainer wants a lot more URLS from us so make sure you double check the Authorization, Token and Resource URL. All other values are similar to the other clients. 

Just a quick tip with regards to those three URLs here – how would I know what to put here? Actually, we can query them from the Authelia Server. If we open the “.well-known/openid-configuration” URL on the Authelia Server, then Authelia will provide us with all of them. Let me copy this into a text editor and beautify it for better readability. Here we have all the settings we need. Authorization Endpoint, Token Endpoint, Supported Claims, Userinfo Endpoint. If any of these still show example.com then you will have to revisit your Authelia Config File.

Before I can log into portainer I do however have to create the user there.

Nothing exiting to configure on the Authelia side, just this time we can add groups to the scopes.

Once this is all done, then Portainer will come up with this Login screen, basically asking you if you want to use OAUTH or use the internal authentication. If I am already logged in, then I can just consent and will be logged into Portainer.

When I did this in my lab I found again some issues with regards to self-signed certificates. Please check my blog www.onemarcfifty.com for Details. Basically I had to install the Root CA to the Docker host and give Portainer a copy of the Root CA.

Let’s wrap this up with configuration for Gitea. Here, the config is done under Site Administration, then Authentication Sources. Click on “Add Authentication Sources”, chose “OAUTH2” as a type and OpenID Connect as a Provider. Fill in Client ID and Secret. Just this time – gitea can actually use the Auto Discover URL directly. So let’s put in the “.well-known” URL here.

On the Authelia side, nothing new. Same Procedure like for the other three. Just the callback URI looks a bit different.

Just – here in Gitea we have the possibility to enable automatic user creation. I’ll just copy paste the snippet here from the Authelia Documentation into the app.ini file on my gitea server and then restart gitea.

When I now log into gitea the next time, then there is this button here “Sign in with OpenID”. And – beautiful. SSO everywhere.

On gitea I had the same issue like on Portainer with the self-signed certs. Again – check my blog article on that one. Also – I had forgotten to specify port 3000 in the config which lead to an error message. Adding the port correctly in the authelia config fixed the issue.

Chaps, we could do many more – I think you get the idea. It’s always the same procedure. Many other Apps can work with Authelia such as SeaFile, Apache Guacamole, CloudFlare ZeroTrust and so on.

(What if App does not support OpenID)
What can we do if the App we want to secure does not support OpenID? Can we still use Authelia? Basically we have three possibilities here. Either the app supports header based authentication. Rather than having the app listen on the generic IP 0.0.0.0, we let it listen on the local host address only. That means, it will not be reachable from the outside. We then install a reverse proxy on the same machine that listens to the outside world and forwards to the local app once Authentication has occurred. Authelia and NGINX can add a couple of X-Headers to the forward request which the app can then read out. For eaxample like I did here with my Shinobi Video surveillance. I actually had to change the code so that it reads out the “remote-user” header and if that header exists, it will trust that Authelia has already done the authentication. There are some examples in the Authelia docs on how to do this with Jira, Organizr and SeaFile.

Some Apps don’t support OAUTH2 or OpenID Connect. But they do support SAML. One of my next projects is to actually use a gateway application that would act as a SAML provider and use Authelia as a backend in order to authenticate. Kind of a SAML to OpenID Proxy really. Watch my blog and github for this. I will publish as soon as I get to grips with it.

Now what if the app can’t do Header based, can’t do SAML, can’t do OAUTH? Well, then you can still put NGINX or Traefik or Caddy in front of the app, integrate the reverse proxy with Authelia and then redirect – just – you would have to login to the app after you logged into Authelia. Not very nice but maybe still a viable solution if you are exposing stuff to the internet such as the web interface of a webcam or the like.

(Some last thoughts)
Here are some last thoughts and potential enhancements. The scripts I provide do not use MySQL but rather a simple SQLite file. This is not dimensioned for large scale but rather for small deployments. I’ve put some examples on the github on how to extend this to MySQL or MariaDB rather. Also, you might want to enable REDIS for larger deployments. Redis is an in-memory Database that can drastically speed up things.

Also, an important reminder especially if you are using OIDC to log into Proxmox. If you are running Authelia in a container on Proxmox and your Proxmox has a problem bringing the container up, then you will not be able to log into the GUI, hence you can’t fix the problem. So please make sure you still have the possibility to log into the Proxmox UI using PAM authentication. You still need this for some admin tasks on Proxmox anyhow.

For full disclosure – I am not the inventor of all this, but I have used mainly three resources here. First and foremost – during my research for this episode, I came across a blog by Florian Müller from Hamburg. On his blog site he describes in great detail how he set up Authelia for a CloudFlare Tunnel with OIDC on Proxmox. Florian, shout out to Hamburg and many thanks for this – your blog really helped me a lot in order to get to grips with Authelia. I have adapted and modified his code snippets in order to give you guys an automated installation experience. The code is on my github repository. Link in the description. Second – fellow YouTube Creator Techno Tim has a nice video about Authelia on Docker. You may want to watch this as well if you intend to use Authelia with Docker. Last but not least – the Authelia Documentation contains a lot of ready-to-use samples on how to configure OIDC with Authelia. 

That’s it guys – I hope you liked the episode. If so, please give it a like on YouTube and leave me a comment! Many thanks for watching. Happy SSO – stay safe, stay healthy, bye for now!
</details>
