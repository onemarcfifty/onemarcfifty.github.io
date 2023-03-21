---
title: "Portainer with self-signed certificates"
last_modified_at: 2023-02-25T17:00:00
categories:
  - Blog
tags:
  - Docker
  - Linux
toc: true
header:
    teaser: /assets/images/Channelart11b.png
---

## Portainer and self-signed certificates

I am currently preparing videos in a test environment with self-signed certificates. One of the tests was to use Authelia with Portainer for SSO. I just couldn't get it to work until I realized that Portainer (as of Version 2.17) does not seem to honor the `--tlscacert` flag.

We can give Portainer self-signed _server_ certificates with the `--sslcert` and `--sslkey` flags. With this, we can connect to Portainer without Certificate warnings if we have the Root CA imported into our browser. However, Portainer would just not trust any other site (such as Authelia).

## the Solution

the solution is more kind of a workaround really. What I ended up doing was to instal the Root CA ont he Docker host and provide a copy of the complete RootCA crt file to Portainer. Assuming that your Server key and certificate as well as the rootCA.crt are in /etc/certificates/example.com, run the following on your Docker host:

```bash
# copy the Root CA file to the system certificate subfolder
# (on Debian in /etc/ssl/certs)
cp /etc/certificates/example.com/rootCA.crt /etc/ssl/certs
# create a Symlink from the hash to the cert
ln -s /etc/ssl/certs/rootCA.crt /etc/ssl/certs/`openssl x509 -hash -noout -in rootCA.crt`.0
# this will update the /etc/ssl/certs/ca-certificates.crt
update-ca-certificates
```

We could as well just have copied the original `/etc/ssl/certs/ca-certificates.crt` file and could have done `cat rootCA.crt >> ca-certificates.crt` , thus appending our Root CA to the file.

Now we can run Portainer with the following command:

```bash
docker run -d -p 9000:9000 -p 9443:9443 \
       --name=portainer \
       --restart=always \
       -v /var/run/docker.sock:/var/run/docker.sock \
       -v portainer_data:/data \
       -v /etc/certificates/example.com:/certs \
       -v /etc/ssl/certs/ca-certificates.crt:/etc/ssl/certs/ca-certificates.crt \
       portainer/portainer-ce:latest \
       --sslcert /certs/wildcard_fullchain.crt \
       --sslkey /certs/wildcard.key \
       --tlscacert /cert/rootCA.crt
```

(basically we are mounting the modified Root CA File to the container)