# HAProxy check scripts for eosio

This is a simple script that send a `get_info` request to the backend
`nodeos` instance and reports a failure status if the node is more
than 60 seconds behind the host time.

The script also sends detailed error messages to syslog.


## Installation

```
apt install -y git libdatetime-format-iso8601-perl libjson-xs-perl libjson-perl libwww-perl

git clone https://github.com/cc32d9/eosio-haproxy.git /opt/eosio-haproxy

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
```






## Copyright and License

Copyright 2019 cc32d9@gmail.com

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.


