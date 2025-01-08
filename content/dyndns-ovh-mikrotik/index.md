+++
title = "Dynamic DNS for OVH with RouterOS scripts"
date = 2025-01-08

[taxonomies]
categories = ["Mikrotik"]
tags = ["code","mikrotik","network","dns"]
+++

Assigning DNS domain to dynamic public IP addresses provided by ISP can make remote access to your home network easier. In this post I will present how to configure Dynamic DNS script for OVH domain provider using RouterOS scripts.
<!-- more -->

Whether you’re managing a remote network, accessing wireless IoT device, or hosting a website from your home, having a reliable way to connect is essential. This is where a Dynamic DNS (DDNS) solution can make life much easier. DDNS can be provided by third party companies, but usually it's provided for free with your DNS domain bought from hosting company. In my case I use OVH APIs, that offer to update DDNS mapping by sending HTTP requests.

To make DDNS work without interruptions, requests should be sent frequently 24/7 (for example every 15 minutes). This can be done by configuring server to send HTTP requests using crontab jobs, but also you can use Mikrotik router to do it using RouterOS scripts. I prefer using router for that, because if server responsible for updating DNS goes down, entire network can be lost. If router goes down, I lose the access to my network anyway :).

Crontab jobs bash script. Replace **LOGIN** and **PASSWORD** variables with values provided by OVH, **domainArray** with a list of your desired domains and subdomains and **PATH_LOG** with path to log file that will store status of DDNS requests. 

```bash

#!/bin/bash

domainArray=(example.org, www.example.org)

LOGIN=yourusername
PASSWORD=yourpassword

PATH_LOG=/home/user/maintenance/dynhost.log

echo "Run dyndns" >> $PATH_LOG
date >> $PATH_LOG

for item in ${domainArray[*]}
do
        UPDATE_IP=false

        HOST_IP=`dig +short $item`
        CURRENT_IP=`curl ifconfig.co`
        sleep 2

        echo "Current domain" >> $PATH_LOG
        echo "$item" >> $PATH_LOG
        echo "Current IP" >> $PATH_LOG
        echo "$CURRENT_IP" >> $PATH_LOG
        echo "Host IP" >> $PATH_LOG
        echo "$HOST_IP" >> $PATH_LOG

        if [ -z $CURRENT_IP ] || [ -z $HOST_IP ]
        then
                        echo "No IP retrieved" >> $PATH_LOG
        else
                        if [ "$HOST_IP" != "$CURRENT_IP" ]
                        then
                                        RES=`curl --user "$LOGIN:$PASSWORD" "https://www.ovh.com/nic/update?system=dyndns&hostname=$item&myip=$CURRENT_IP"`
                                        echo "DynHost request result:" >> $PATH_LOG
                                        echo "$RES" >> $PATH_LOG
                                        echo "IP has changed" >> $PATH_LOG
                        else
                                        echo "IP has not changed" >> $PATH_LOG
                        fi
        fi
done
```

For RouterOS you can use script provided below:

```bash

:local ovhddnsuser "yourusername"
:local ovhddnspass "yourpassword"
:local theinterface "eth1"
:local ovhddnshost "example.org"
:local ipddns [:resolve $ovhddnshost]
:local ipfresh [ /ip cloud get public-address ]
:if ([ :typeof $ipfresh ] = nil ) do={
   :log info ("OVHDynDNS: NO IP address on $theinterface")
} else={
   :for i from=( [:len $ipfresh] - 1) to=0 do={ 
      :if ( [:pick $ipfresh $i] = "/") do={ 
    :set ipfresh [:pick $ipfresh 0 $i]
      } 
}
 
:if ($ipddns != $ipfresh) do={
   :log info ("OVHDynDNS: $ovhddnshost DNS RECORD IP = $ipddns")
   :log info ("OVHDynDNS: $theinterface CURRENT IP = $ipfresh")
   :log info ("OVHDynDNS: UPDATING $ovhddnshost -> $ipfresh")
   :local str "nic/update?system=dyndns&hostname=$ovhddnshost&myip=$ipfresh&wildcard=OFF&backmx=NO&mx=NOCHG"

   /tool fetch address=www.ovh.com host=www.ovh.com src-path=$str mode=https user=$ovhddnsuser password=$ovhddnspass dst-path=("/disk1/OVHDynDNS.".$ovhddnshost)
   :delay 1

   :local ovhresult [/file get "OVHDynDNS.$ovhddnshost" contents]
   :log info "OVHDynDNS: OVH response: $ovhresult"

   :local str [/file find name="OVHDynDNS.$ovhddnshost"]
   /file remove $str
   
   :if ($ovhresult = "good $ipfresh\n" ) do={
      :log info "OVHDynDNS: SUCCESS"
   } else={
       :log info "OVHDynDNS: FAILED"
   }

   :local ipddns $ipfresh
   :log info "OVHDynDNS: $ovhddnshost DNS RECORD = $ipfresh!"
    } else={
     :log info "OVHDynDNS: it works!"
    }
}
```