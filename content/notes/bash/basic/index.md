---
title: Bash Commands
weight: 10
---

<!-- Variable -->
{{< note title="Variable" >}}
```bash
NAME="John"
echo $NAME
echo "$NAME"
echo "${NAME}
```
{{< /note >}}

<!-- Condition -->
{{< note title="Condition" >}}
```bash
if [[ -z "$string" ]]; then
  echo "String is empty"
elif [[ -n "$string" ]]; then
  echo "String is not empty"
fi
```
{{< /note >}}

<!-- Filter -->
{{< note title="Filter" >}}
```bash
# Filter comment and empty lines from a file
grep -v '^$\|^\s*\#' temp

# Filter certs
awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}’ cacerts.pem
```
{{< /note >}}

<!-- Validate WSS -->
{{< note title="WSS" >}}
```bash
# Verify endpoint is using secure web sockets
curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" -H "Host: ikotak.com" -H "Origin: https://ikotak.com" https://ikotak.com -k
```
{{< /note >}}

<!-- Convert pfx file to pem file -->
{{< note title="Convert pfx file to pem file" >}}
```bash
# Conversion to a combined PEM file
openssl pkcs12 -in filename.pfx -out cert.pem -nodes

# Extract the private key form a PFX to a PEM file
openssl pkcs12 -in filename.pfx -nocerts -out key.pem

# Export the certificate only
openssl pkcs12 -in filename.pfx -clcerts -nokeys -out cert.pem

# Remove the password from the extracted private key
openssl rsa -in key.pem -out server.key
```
{{< /note >}}

<!-- Latency Troubleshooting -->
{{< note title="Latency Troubleshooting" >}}
```bash
curl -kso /dev/null https://www.ikotak.com -w "==============\n\n 
| dnslookup: %{time_namelookup}\n 
| connect: %{time_connect}\n 
| appconnect: %{time_appconnect}\n 
| pretransfer: %{time_pretransfer}\n 
| starttransfer: %{time_starttransfer}\n 
| total: %{time_total}\n 
| size: %{size_download}\n 
| HTTPCode=%{http_code}\n\n" ; 
```
{{< /note >}}

<!-- Network Emulator -->
{{< note title="Network Emulator" >}}
```bash
# Add a fixed amount of delay to all packets going out of the local Ethernet
tc qdisc add dev bond0 root netem delay 100ms

# Delete delay
tc qdisc list
tc qdisc del dev bond0 root netem

# Add delay to be 100ms ± 10ms with the next random element depending 25% on the last one
tc qdisc change dev eth0 root netem delay 100ms 10ms 25%

# Packet loss
tc qdisc change dev eth0 root netem loss 0.1%

# Causes the random number generator to be less random and can be used to emulate packet burst losses.
tc qdisc change dev eth0 root netem loss 0.3% 25%

# Packet duplication
 tc qdisc change dev eth0 root netem duplicate 1%

# Packet corruption
tc qdisc change dev eth0 root netem corrupt 0.1%

# Packet re-ordering, 25% of packets (with a correlation of 50%) will get sent immediately, others will be delayed by 10ms.
tc qdisc change dev eth0 root netem delay 10ms reorder 25% 50%
```
{{< /note >}}

<!-- PostgreSQL -->
{{< note title="PostgreSQL" >}}
```bash
sudo -u postgres psql
postgres=# create database mydb;
postgres=# create user myuser with encrypted password ‘mypass’;
postgres=# grant all privileges on database mydb to myuser;

# Quit psql
\q

# Choose database
Sql
\l
\c database_name

# List tables
\dt+

SELECT *
FROM pg_catalog.pg_tables
WHERE schemaname != 'pg_catalog’ AND 
    schemaname != ‘information_schema’;

SELECT * FROM pg_catalog.pg_tables;

# Drop sessions
SELECT pg_terminate_backend(pg_stat_activity.pid)
FROM pg_stat_activity
WHERE pg_stat_activity.datname = ‘TARGET_DB';
```
{{< /note >}}
