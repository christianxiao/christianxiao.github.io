---
layout: post
title: Simplest way to set up your own VPN server based on AWS
category: programming
---
Your own VPN server = AWS EC2 + docker + L2TP image
1. Register your AWS account
2. Use the following EC2 userdata to launch a free tier Ubuntu EC2, you need to replace YOUR_SEC,YOUR_USERNAME,YOUR_PASSWORD:

```
#!/bin/bash
curl -s https://get.docker.com | sh
/etc/init.d/docker start
##
docker pull fcojean/l2tp-ipsec-vpn-server
modprobe af_key
docker run \
    --name l2tp-ipsec-vpn-server \
    -e "VPN_IPSEC_PSK=YOUR_SEC" \
    -e 'VPN_USER_CREDENTIAL_LIST=[{"login":"YOUR_USERNAME","password":"YOUR_PASSWORD"}]' \
    -p 500:500/udp \
    -p 4500:4500/udp \
    -v /lib/modules:/lib/modules:ro \
    -d --privileged \
    fcojean/l2tp-ipsec-vpn-server
```
3. Security group: you need to allow UDP port 500, 4500
4. Done
