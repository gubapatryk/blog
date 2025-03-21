+++
title = "IPFIX monitoring with Docker ELK dashboards" 
date = 2025-03-21

[taxonomies]
categories = ["DevOps"] 
tags = ["ipfix","elastic","network","docker","mikrotik"]
+++

Almost nobody needs a full-blown network monitoring stack for a home lab — but here we are. If you’re running servers, you should probably know what’s moving across your network before something else does. Using IPFIX into ELK gives you actual visibility instead of vibes, so you’re not blind to weird traffic, misconfigurations, or your own mistakes. Let’s set it up.

<!-- more -->

With more and more services running on my home lab, I started wondering — how would I even know if something was acting up? The pivotal point came when I bought a WiFi lamp. I realized I had no idea what it was doing on my network. Was it harmlessly checking for updates, or was it quietly exfiltrating data to some mystery server? Or maybe it will start sending 3,6GB of data per day like this LG washing machine? 

{{ responsive(src="lg-iot.png", width=691, height=682, alt="LG network traffic") }}

Well - I wasn’t going to just hope my devices were behaving, I needed real visibility. It was time to deploy a network monitoring dashboard. The plan was to collect packets sent by Mikrotik Traffic Flow tool to a small server in my home lab. It had to balance performance and memory usage, and avoid excessive RAM consumption - this server wasn't a beefy machine I would use for a real life SIEM, but just enough for a simple tool with one source. Among the tools that I tried, the following can be mentioned:

- Grafana and Prometheus - due to high memory and compute usage I couldn't run them stable on my server (it didn't support Docker resource constraints either)
- Graylog - IPFIX worked fine, but I prefer ElasticSearch over OpenSearch and GeoIP plugin didn't work perfectly with my Mikrotik IPFIX schema - I had to add some changes manually using lookup tables to for IP addresses
- Akvorado - it was the first time I heard about this tool and I really loved the stack and had a lot of hopes... but somehow it didn't work with my server (maybe the issue was that it required 100% of server resources to run :D), so I decided that I should go with an old beloved elastic stack
- ELK stack - it provides NetFlow integration by default, but can be upgraded using [ElastiFlow](https://www.elastiflow.com/). In this post I use default NetFlow integration provided by elastic.

First we need to set up our Mikrotik. Go to IP > Traffic Flow > Targets and define a target that will receive IPFIX packets. In my example it will be an ELK Fleet Agent listening at machine with 192.168.1.10 address at port 2055. Apply those settings, go back to Traffic Flow and set "Enabled" to True. You can also select which information fields you want to send with IPFIX packets.


{{ responsive(src="ipfix-mikrotik.png", width=691, height=682, alt="Mikrotik configuration") }}

Then we need to run our ELK stack. I'm using a modified docker compose configuration based on [github.com/deviantony/docker-elk](https://github.com/deviantony/docker-elk), with only Kibana container exposed to the local network, you can find my version [here](https://github.com/gubapatryk/docker-elk/). 

If you don't want to use 30 day trial license, you can change it to basic at **elasticsearch/config/elasticsearch.yml**
```yml
xpack.license.self_generated.type: basic
```

Change passwords at .env (or better, configure some secret manager to store them).

Add port 2055:2055/udp to agent apmserver docker compose file at **extensions/fleet/agent-apmserver-compose.yml** to open port for Netflow

First run you need to setup environment with line below:
```sh
docker compose up setup
```

After setup, you can use the line below to deploy all required containers:
```sh
docker compose -f docker-compose.yml -f extensions/fleet/fleet-compose.yml -f extensions/fleet/agent-apmserver-compose.yml up -d
```

Login at http://localhost:5601/ to Kibana.

Go to Management > Fleet > Agent Policies, choose Agent Policy APM Server
Click "Add integration" button, find Netflow integration and fill in the form.
Set host IP (for example listening at 0.0.0.0) and port (2055)

Restart docker compose (go down, then up) if required. You should be able to see some default NetFlow dashboards.


{{ responsive(src="ipfix-dashboard.png", width=1339, height=672, alt="Kibana dashboard") }}



