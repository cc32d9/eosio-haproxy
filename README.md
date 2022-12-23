# HAProxy check script for eosio

`haproxy_check_block_time` is a simple script that sends a `get_info`
request to the backend `nodeos` instance and reports a failure status
if the node is more than 60 seconds behind the host time. The script also sends detailed error messages to syslog.

`h_check_eosio_ship` is checking the health of EOSIO state history
plugin. It compares its head block with the head block that a provided
EOSIO API is returning, and it returns an error if the state history
is older than the critical value.

`haproxy_check_aa_health` is checking the AtomicAssetts API health.

`haproxy_check_hyperion_health` is checking the health of Hyperion API
by EOS Rio. It fails if the API is behind the head block or if the
number of unindexed blocks is increasing.


## Installation

```
apt install -y git libdatetime-format-iso8601-perl libjson-xs-perl libjson-perl libwww-perl

curl -fsSL https://deb.nodesource.com/setup_14.x | bash -
apt-get install -y nodejs

git clone https://github.com/cc32d9/eosio-haproxy.git /opt/eosio-haproxy

# state history monitor requires a configuration file that
# specifies the EOSIO API URL
cd /opt/eosio-haproxy/ship
npm install

cat >/etc/default/h_check_eosio_ship_waxshipbe.json <<'EOT'
{
  "critical": 15,
  "apiurl": "https://wax.eu.eosamsterdam.net"
}
EOT

# AtomicAssets monitor configuration is optional, and allows tuning
# the critical thresholds
cat >/etc/default/haproxy_check_aa_health_waxaabe.json <<'EOT'
{
        "critical_blocks_behind": 100
}
EOT
```


Example HAProxy configuration

```
global
        log /dev/log    local1 notice
        stats socket /run/haproxy/admin.sock mode 666 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon
        external-check

defaults
        log     global
        timeout connect 5000
        timeout client  50000
        timeout server  50000

frontend apife
    bind 127.0.0.1:8888
    mode http
    use_backend apibe
    mode    http
    option  dontlognull
    option log-separate-errors

backend apibe
    mode http
    balance leastconn
    option external-check
    external-check command /opt/eosio-haproxy/haproxy_check_block_time
    server node1 10.0.3.20:8801 check
    server node2 10.0.3.20:8802 check

frontend waxshipfe
    bind 127.0.0.1:8080
    mode tcp
    use_backend waxshipbe
    option log-separate-errors

backend waxshipbe
    mode tcp
    balance roundrobin
    option allbackups
    option external-check
    external-check command /opt/eosio-haproxy/ship/h_check_eosio_ship
    default-server check inter 15s on-marked-down shutdown-sessions on-marked-up shutdown-backup-sessions
    server ship01 1.2.3.4:8080
    server ship02 4.3.2.1:8080


frontend waxaafe
    bind 127.0.0.1:9000
    mode http
    use_backend waxaabe
    mode    http
    option  dontlognull
    option  log-separate-errors

backend waxaabe
    mode http
    option external-check
    external-check command /opt/eosio-haproxy/haproxy_check_aa_health
    server aa-api01 aa-api01.example.com:9000 check
    server aa-api02 aa-api02.example.com:9000 check backup
    server aa-api03 aa-api03.example.com:9000 check backup

frontend waxhyperfe
    bind 127.0.0.1:7000
    mode http
    use_backend waxhyperbe
    option  dontlognull
    option log-separate-errors

backend waxhyperbe
    mode http
    option external-check
    external-check command /opt/eosio-haproxy/haproxy_check_hyperion_health
    server hyper03.example.com hyper03.example.com:7000 check inter 10s
    server hyper04.example.com hyper04.example.com:7000 check inter 10s
    server hyper05.example.com hyper05.example.com:7000 backup
```

In the Hyperion proxy, backup is not performing the checks, so that it
always stays online as a last resort.




## Copyright and License

Copyright 2019-2022 cc32d9@gmail.com

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
