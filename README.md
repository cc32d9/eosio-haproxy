# HAProxy check script for eosio

`haproxy_check_block_time` is a simple script that sends a `get_info`
request to the backend `nodeos` instance and reports a failure status
if the node is more than 60 seconds behind the host time. The script also sends detailed error messages to syslog.

`h_check_eosio_ship` is checking the health of EOSIO state history
plugin. It compares its head block with the head block that a provided
EOSIO API is returning, and it returns an error if the state history
is older than the critical value.


## Installation

```
apt install -y git libdatetime-format-iso8601-perl libjson-xs-perl libjson-perl libwww-perl

curl -fsSL https://deb.nodesource.com/setup_14.x | bash -
apt-get install -y nodejs

git clone https://github.com/cc32d9/eosio-haproxy.git /opt/eosio-haproxy

cd /opt/eosio-haproxy/ship
npm install

cat >/etc/default/h_check_eosio_ship_waxshipbe.json <<'EOT'
{
  "critical": 15,
  "apiurl": "https://wax.eu.eosamsterdam.net"
}
EOT


```

Example HAProcy configuration

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

```






## Copyright and License

Copyright 2019-2021 cc32d9@gmail.com

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
