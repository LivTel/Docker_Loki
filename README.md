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


# Notes on things that do and do not work
* Not yet automated for daily ingestion. To ingest a vlm, just unzip it into /mnt/newarchive1/Dockershare/Docker_Loki/vlm
* Possible Missing Data: I ingested 1.05e6 lines of var-log-messages log files and there only seem to be 950 thousand in the Loki database, so some lines seem to be getting lost. That is going to need investigation to find out which log lines are not getting ingested.
* Repeats: My current parser does not see these properly. It thinks the software process name is “last” and does not obviously flag to the user that this message has appeared hundreds of times. All you will see in Loki is that first instance which gets parsed properly. This is rather unfortunate. I do not yet have an elegant fix for this.
<code>
> May  8 20:14:34 mcc.lt.com Ept: <1e0020> More than 1 bit is set, state of OID 0x59 cannot be determined.
> May  8 20:15:05 mcc.lt.com last message repeated 31 times
> May  8 20:17:06 mcc.lt.com last message repeated 121 times
> May  8 20:22:52 mcc.lt.com last message repeated 346 times            
</code>
* Process PIDs: A small number of processes log both their name and their process ID (PID) in the log. That is a problem because Loki sees every instance of the process as a different label. Instead of seeing all sshd process as being sshd, it sees thousands of different sshd process. Every time anyone logs into the computer, it creates a new label. I think it is more use lump all the sshd messages together under a single “sshd” label. I have ‘fixed’ this by ignoring the PID number. I.e., all sshd messages are being stored in Loki as “sshd[xxx]” and the PID number has been completely lost. I think that is probably going to be OK. I doubt those PID numbers will ever be useful to us in the current context, but be aware I have discarded that particular datum. This effects sshd, bootp, telnetd, ftpd, inetd, ntpd messages.
> Jun 13 14:02:02 node<<1>> sshd[10722]: log: Connection from 192.168.1.30 port 57168
> Jun 13 14:02:02 node<<1>> sshd[10722]: log: RSA authentication for maintain accepted.
> Jun 13 14:02:02 node<<1>> sshd[10725]: log: executing remote command as user maintain
> Jun 13 14:02:02 node<<1>> sshd[10722]: log: Closing connection to 192.168.1.30
> Jun 13 14:02:02 node<<1>> sshd[10727]: log: Connection from 192.168.1.30 port 57169
> Jun 13 14:02:02 node<<1>> sshd[10727]: log: RSA authentication for maintain accepted.
> Jun 13 14:02:02 node<<1>> sshd[10730]: log: executing remote command as user maintain
but in Loki, we have
> Jun 13 14:02:02 node<<1>> sshd[xxx]: log: Connection from 192.168.1.30 port 57168
> Jun 13 14:02:02 node<<1>> sshd[xxx]: log: RSA authentication for maintain accepted.
> Jun 13 14:02:02 node<<1>> sshd[xxx]: log: executing remote command as user maintain
> Jun 13 14:02:02 node<<1>> sshd[xxx]: log: Closing connection to 192.168.1.30
> Jun 13 14:02:02 node<<1>> sshd[xxx]: log: Connection from 192.168.1.30 port 57169
> Jun 13 14:02:02 node<<1>> sshd[xxx]: log: RSA authentication for maintain accepted.
> Jun 13 14:02:02 node<<1>> sshd[xxx]: log: executing remote command as user maintain
* New Year: Overnight on the night of 31st Dec / New Year, the logs are not correctly ingested. If you look at the log format, there is no year! I know how to fix this, but it is only that one night. I will fix it at some point, but for now just do not use 31st Dec.
