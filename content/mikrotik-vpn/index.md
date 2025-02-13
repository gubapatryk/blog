+++
title = "VPN IPSec IKEv2 with Mikrotik router"
date = 2025-02-13

[taxonomies]
categories = ["Mikrotik"]
tags = ["code","mikrotik","network","vpn"]
+++

Mikrotik routers are powerful devices with a lot of features for various networking needs. In this post I describe how to setup IPSec IKEv2 VPN with RouterOS, that could be used to remotely access organizational or home network. 
<!-- more -->
Building your own home VPN might be very useful - for example if you have some PC connected to your router, you can access it anytime remotely. If your ISP provides you dynamic IP, you can use [Dynamic DNS](@/dyndns-ovh-mikrotik/index.md) to use DNS name to access your home network.

We start with generating certificates.

First we need to generate CA certificate - it will be used as a root certificate for server and client certificates later. The second certificate is server certificate - it will be used by the router to provide it's VPN identity. As an example domain I will use "example.org", use your own domain name here.

```bash
/certificate
add common-name="My CA" name=local-ca key-size=2048 days-valid=365 key-usage=key-cert-sign,crl-sign
sign local-ca 
add common-name="example.org" name=server-cert key-size=2048 days-valid=365 subject-alt-name=DNS:example.org key-usage=tls-server 
sign server-cert ca=local-ca

```
(note: using WebFig to provide subject-alt-name can alter the number of colons, make sure you have the right value here)

Client certificate - it will be used for client authentication:
```bash
/certificate
add name=client-cert common-name=client.local key-size=2048 days-valid=365 key-usage=tls-client
sign client-cert ca=local-ca

```
You can use WebFig to export PKCS#12 certificate file by using System > Certificates section and selecting your client certificate.

{{ responsive(src="assets/export-cert.png", width=400, height=220, alt="Certificate export window", caption="Certificate export window in RouterOS WebFig") }}

Next, configure IPSec settings. First we create an IP Pool for VPN clients.

```bash
/ip pool
add name=ipsec-pool ranges=192.168.100.10-192.168.100.100
```

Mode config

```bash
/ip ipsec mode-config
add name=ikev2-modecfg address-pool=ipsec-pool address-prefix-length=32 system-dns=yes
```
(Note: adding multiple split-include addresses might not work for some clients, as they use only the first address to tunnel connections)


IPSec profile

```bash
/ip ipsec profile
add name=ikev2-profile dh-group=modp2048 enc-algorithm=aes-256 hash-algorithm=sha256
```

IPSec peer

```bash
/ip ipsec peer
add name=ikev2-peer exchange-mode=ike2 passive=yes profile=ikev2-profile send-initial-contact=no
```

IPSec group

```bash
/ip ipsec policy group
add name=ikev2-group
```

IPSec proposal

```bash
/ip ipsec proposal
add name=ikev2-proposal auth-algorithms=sha256 enc-algorithms=aes-256-cbc lifetime=1h pfs-group=modp2048
```

IPSec identity

```bash
/ip ipsec identity
add auth-method=digital-signature certificate=server-cert remote-certificate=client-cert generate-policy=port-strict mode-config=ikev2-modecfg peer=ikev2-peer policy-template-group=ikev2-group match-by=certificate
```

IPSec policy

```bash
/ip ipsec policy
add group=ikev2-group template=yes dst-address=192.168.100.0/24 proposal=ikev2-proposal
```
(Note: adding bigger subnet for the ease of configuration and future modifications)

After configuring IPSec setting make sure your firewall permits IPSec connections

```bash
add action=accept chain=input protocol=ipsec-esp
add action=accept chain=input dst-port=500 protocol=udp
add action=accept chain=input dst-port=4500 protocol=udp
```

Router configuration should be ready, so it's time to configure client devices. To setup VPN in Ubuntu, you will need to export private key and certificate from PKCS#12 file created by Mikrotik router:

```bash
openssl pkcs12 -in exported-cert.p12 -out rdt.key -nodes -nocerts

openssl pkcs12 -in exported-cert.p12 -out rdt.crt -nokeys
```
Configuration in GUI:


{{ responsive(src="assets/ubuntu-vpn.png", width=720, height=120, alt="Ubuntu VPN settings image", caption="VPN settings in Ubuntu Linux") }}

