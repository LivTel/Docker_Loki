# Docker_Loki

A test bed for evaluating Loki for LT Operations. Not a mature installation.

This Docker contains Loki (queryable log database) and Alloy (ingestion agent). Typically these would probably be run in a single docker compose with grafana since they are all interdependent, but for testing and evaluation I have left the Grafana running in its own environment (https://github.com/LivTel/Docker_SDBgrafana).

Grafana docs include examples of merging the entire stack into single compose. See https://grafana.com/docs/loki/latest/setup/install/docker/.
```
wget https://raw.githubusercontent.com/grafana/loki/v3.4.1/production/docker-compose.yaml -O docker-compose.yaml
```

Uses the official docker images of Loki and Alloy from Grafana.

Only things here are the compose file and a single server config for each of Loki and Alloy. The compose mounts the external NAS and writes all data into there. That means all Loki databases persist externally and you can stop and start this docker whenever you like without loss of data.


# Instructions

``git clone https://github.com/LivTel/Docker_Loki``

``cd Docker_Loki``

``docker compose up -d``


Alloy will immediately start ingesting anything it can find in the directory specified in config.alloy.

This was developed on ltvmhost5. If you start it on some other host, there may be further config required.
For example
* adding the eng user to the docker group for permissions to run docker
* reverse proxy in the front-end web server
* mount the external RAID for the binds in the compose.yml

More information about how ltvmhost5 was configured to run docker on wiki GrafanaSDB.

External storage mount points are defined in compose.yml.

The location where Alloy looks for the log files to ingest is defined in config.alloy. Currently that is the big external RAID NAS, which is not where vlm are stored. You need to manually copy the vlm that you want ingested over from sdbserver.

# Still to be done
* Automatically scrape new vlm as they appear on sdbserver
* Automate 'this year' in the timestamp. Does Alloy default to now if we do not specify a year?




