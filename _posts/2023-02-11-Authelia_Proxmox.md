---
title: "Authelia as OpenID Server on Proxmox"
last_modified_at: 2023-02-11T17:00:00
categories:
  - Blog
tags:
  - Proxmox
  - Linux
toc: true
header:
    teaser: /assets/images/Authelia.jpg
---

## How I use Authelia

I use Authelia as an Identity Provide in my network. That means that everyone who wants to use resources (such as Jellyfin or Gitea or Nextcloud) or who wants to "traverse" VLANs (e.g. to manage an OpenWrt router or switch or my Proxmox VE) has to login first. Authelia was my product of choice because

- It is free and Open Source Software (FOSS)
- It can act as an OpenID Connect (OIC) Identity provider
- It can be integratet into an NGINX reverse proxy
- It supports Two Factor Authentication (2FA) with Time based One Time Passwords (TOTP) or Fido2 compatible Keys such as the Yubikey.

## Installing Authelia on Proxmox

I set up a small LXC container running Debian Bullseye (Debian 11) on Proxmox. It needs to be privileged because it needs to have access to `/dev/urandom` or `/dev/random` for the generation of random numbers.<br>
Then we need to install some software, add the authelia repos and finally install authelia:<br>

```bash
# run as root
apt update
apt install -y curl gnupg apt-transport-https sudo
curl -s https://apt.authelia.com/organization/signing.asc | sudo apt-key add -
echo "deb https://apt.authelia.com/stable/debian/debian/ all main" >>/etc/apt/sources.list.d/authelia.list
apt-key export C8E4D80D | sudo gpg --dearmour -o /usr/share/keyrings/authelia.gpg
apt update
apt install -y authelia
```

## Configuration of Authelia

I mostly followed [this guide here by Florian Mueller](https://florianmuller.com/setup-authelia-bare-metal-with-openid-and-cloudflare-tunnel-on-a-hardened-proxmox-lxc-ubuntu-22-04-lts-container#configureauthelia)

for the configuration. Basically we create sub-directories for all secrets and auto-generate them, create keys and add the secrets to the environment of authelia (the below is a shortened version of Florian's scripts)

**important note** The scripts below use a single SQLITE file rather than mysql! Also, no OIDC provider is configured - just a dummy entry. Please see the implications of this [here](https://www.authelia.com/configuration/storage/sqlite/)

### create the secrets and service systemd unit file

```bash
for i in .secrets .users .assets .db ; do mkdir /etc/authelia/$i ; done
for i in jwtsecret session storage smtp oidcsecret redis ; do tr -cd '[:alnum:]' < /dev/urandom | fold -w "64" | head -n 1 | tr -d '\n' > /etc/authelia/.secrets/$i ; done
openssl genrsa -out /etc/authelia/.secrets/oicd.pem 4096
openssl rsa -in /etc/authelia/.secrets/oicd.pem -outform PEM -pubout -out /etc/authelia/.secrets/oicd.pub.pem
(cat >/etc/authelia/secrets) <<EOF
AUTHELIA_JWT_SECRET_FILE=/etc/authelia/.secrets/jwtsecret
AUTHELIA_SESSION_SECRET_FILE=/etc/authelia/.secrets/session
AUTHELIA_STORAGE_ENCRYPTION_KEY_FILE=/etc/authelia/.secrets/storage
AUTHELIA_NOTIFIER_SMTP_PASSWORD_FILE=/etc/authelia/.secrets/smtp
AUTHELIA_IDENTITY_PROVIDERS_OIDC_HMAC_SECRET_FILE=/etc/authelia/.secrets/oidcsecret
AUTHELIA_IDENTITY_PROVIDERS_OIDC_ISSUER_PRIVATE_KEY_FILE=/etc/authelia/.secrets/oicd.pem
EOF
chmod 600 -R /etc/authelia/.secrets/
chmod 600 /etc/authelia/secrets
(cat >/etc/systemd/system/authelia.service) <<EOF
[Unit]
Description=Authelia authentication and authorization server
After=multi-user.target

[Service]
Environment=AUTHELIA_SERVER_DISABLE_HEALTHCHECK=true
EnvironmentFile=/etc/authelia/secrets
ExecStart=/usr/bin/authelia --config /etc/authelia/configuration.yml
SyslogIdentifier=authelia

[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload
```

### create the user file 

Next, we create a rudimentary User database yaml file with randomly generated passwords (the users can reset them with the "forgot password" link):

```bash
echo "users:" > /etc/authelia/.users/users_database.yml
for user in bob alice dave frank ; do
  randompassword=$(tr -cd '[:alnum:]' < /dev/urandom | fold -w "64" | head -n 1 | tr -d '\n')
  encryptedpwd=$(authelia hash-password --no-confirm   -- $randompassword  | cut -d " " -f 2)
  ( 
    echo "  ${user}:" 
    echo '    displayname: "First Last"'
    echo "    password: $encryptedpwd"
    echo "    email: ${user}@yourdomain.com"
  ) >> /etc/authelia/.users/users_database.yml
done
chmod 600 -R /etc/authelia/.users/
```

### create the configuration.yml

Now we need to create a configuration file for authelia in `/etc/authelia/configuration.yml`

```bash
cd /etc/authelia
# save the old version of the file
if [ -e configuration.yml ] ; then
  mv configuration.yml configuration.yml.old
fi
# Now let's use Marc's version of Florian's Template File for the new config:
wget https://raw.githubusercontent.com/onemarcfifty/cheat-sheets/main/templates/authelia/configuration.yml
chmod 600 configuration.yml
```

## Starting Authelia for the first time

There we go - authelia should be able to run already - if you do

```bash
systemctl start authelia
systemctl status authelia
```

you should see all green and be able to browse to `http://localhost:9091` and see the authelia login prompt. Even though this is an important checkpoint (in the sense that we can see if authelia will run at all), we can't really use it yet. We need to take care of the following things:

1. All domain names still point to example.com
2. The start up checks are disabled (especially checking for an e-mail Server)

**You will need a real e-Mail account / Server for Authelia to work correctly** (Users need this to reset their password and to register 2FA devices)

3. We have no TLS enabled
4. The policies need to be adapted
5. We need to increase the security and hide Authelia behind an NGINX Server
6. We need to harden the server
7. The user accounts are not real

## Adapting and securing authelia

to get a first idea of how much you need to change do `grep example.com /etc/authelia/configuration.yml` - that shows you all the lines where we specified the example.com domain.

### Enabling the startup checks and change the domain names

The first one is the mail server. We need to edit the configuration.yml and give it the login data of a real mail server. This is done in the `notifier` section of the config:

```yml
notifier: 
  disable_startup_check: true
  smtp:
    host: smtp.domain.com
    port: 465
    timeout: 5s
    username: noreply@auth.example.com
    sender: "Authentication Service <noreply@auth.example.com>"
    subject: "{title}"
    startup_check_address: noreply@auth.example.com

```
Change all the settings to reflect a real mailbox that you control. Once you have done that, change the `disable_startup_check: true` to `disable_startup_check: false` and restart authelia:

```bash
systemctl restart authelia
systemctl status authelia
```
If you see errors, i.e. Authelia idn't start then that's because now it checks if it can log into the mailbox at startup. Reminder: The password it uses for the mailbox Server is in `/etc/authelia/.secrets/smtp`

Onc Authelia starts, move on to the next step.

### TLS

Before we can _really_ use Authelia, we need to provide it with _real_ SSL certificates. Use your own or get them from letsencrypt. [This video on my channel](https://youtu.be/Z81jegMCrfk) shows more.

Once you have copied the certificate and key to the server, adapt the configuration.yml file:

```yml
server:
  host: 0.0.0.0
  port: 9091
  asset_path: /etc/authelia/.assets/
  tls:
    key: /etc/authelia/certs/server.key
    certificate: /etc/authelia/certs/server.crt
```

Basically I've just added the tls section and point it to the certificates. Again - restart Authelia, check the status. If it starts and if you can browse to `https://...:9091` rather than `http...` then you can move to the next step.

### bind port

For the moment Authelia listens on _any_ interface, i.e. we can browse to port 9091 from the outside. We will however hide it behind an NGINX server. For this, Authelia should only listen to the localhost interface. Change the configuration.yml from 

```yml
server:
  host: 0.0.0.0
  port: 9091
```

to

```yml
server:
  host: 127.0.0.1
  port: 9091
```

Restart and check - you should not be able to browse to it from the outside.

## Hiding Authelia behind NGINX

In this step we install NGINX, let it listen to the outside world on port 443 and forward all requests to Authelia on the local host. You can use the templates from [My cheat sheet repo on Github](https://github.com/onemarcfifty/cheat-sheets/tree/main/templates/nginx/authelia)

```bash
# install nginx
apt install -y nginx
# stop NGINX
systemctl stop nginx
# remove the default site
rm /etc/nginx/sites-enabled/*
# download the templates from Marc's cheat sheets
wget https://raw.githubusercontent.com/onemarcfifty/cheat-sheets/main/templates/nginx/authelia/siteconf -O /etc/nginx/sites-available/authelia.conf
wget https://raw.githubusercontent.com/onemarcfifty/cheat-sheets/main/templates/nginx/authelia/proxy-snippet -O /etc/nginx/snippets/proxy.conf
wget https://raw.githubusercontent.com/onemarcfifty/cheat-sheets/main/templates/nginx/authelia/ssl-snippet -O /etc/nginx/snippets/ssl.conf
# link back the authelia site as enabled to NGINX 
ln -s /etc/nginx/sites-available/authelia.conf /etc/nginx/sites-enabled/authelia.conf
# restart NGINX
systemctl start nginx
```

## adapting the policies

The template file contains three sample policies for bypass, one factor and two factor. You will want to adapt these to your needs. My Server only has one policy:

```yml
access_control:
  default_policy: deny
  rules:
    - domain: '*.mydomain.com'
      policy: two_factor
```

## Last but not least

The last steps are 

- harden the server
- make the user accounts real

Server hardening is not in the scope of this article. Basically at least you should do the following:

1. Lock down the firewall to only let pass port 443 tcp incoming (optionally port 67 UDP if you use dhcp and port 22 tcp if you want to access the server via ssh)
2. disable / expire the password of the root account
3. Create a non-root user for login who has sudo capabilities with a loooong password
4. switch off password authentication and root login for sshd

Now just review the settings in the `/etc/authelia/.users/users_database.yml` and make sure that the user accounts are real accounts with real mail addresses.

That's it - you're all set and can now use Authelia in front of your servers with NGINX/Traefik/Caddy or the like and/or add OIDC providers for Proxmox, Gitea, Portainer, Nextcloud and so on....

