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

<!-- Prevent MacBook from sleeping -->
{{< note title="Prevent MacBook from sleeping" >}}
```bash
caffeinate -imsu -t 6000 & # Set time in seconds to stay awake
pmset displaysleepnow
```
{{< /note >}}

<!-- Keep foreground program running after logout -->
{{< note title="Keep program running after logout" >}}
```bash
# Forgot to use nohup myprogram &
# want to move foreground process to background 
Ctrl + Z
[1]+ Stopped   myprogram
$ disown -h %1
$ bg 1
[1]+ myprogram &
$ logout
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

time for i in {1..50}; do curl -s https://ikotak.com > /dev/null; done
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

<!-- Fiber Channel -->
{{< note title="Fiber Channel" >}}
```bash
# Troubleshooting Fiber Channel connections
$ multipath -ll
mpatha (000219fb) dm-0 PURE,FlashArray
size=500G features='0' hwhandler='1 alua' wp=rw
`-+- policy='service-time 0' prio=50 status=active
  |- 14:0:0:1 sdb 8:16  active ready running
  |- 14:0:1:1 sdd 8:48  active ready running
  |- 12:0:0:1 sdf 8:80  active ready running
  `- 12:0:1:1 sdh 8:112 active ready running
mpathb (000219fc) dm-1 PURE,FlashArray
size=250G features='0' hwhandler='1 alua' wp=rw
`-+- policy='service-time 0' prio=50 status=active
  |- 14:0:0:2 sdc 8:32  active ready running
  |- 14:0:1:2 sde 8:64  active ready running
  |- 12:0:0:2 sdg 8:96  active ready running
  `- 12:0:1:2 sdi 8:128 active ready running 

$ grep -v "zZzZ" -H /sys/class/fc_host/host*/port_state
/sys/class/fc_host/host12/port_state:Linkdown
/sys/class/fc_host/host13/port_state:Linkdown
/sys/class/fc_host/host14/port_state:Online
/sys/class/fc_host/host15/port_state:Linkdown 

$ ll /sys/class/fc_host/
total 0
lrwxrwxrwx. 1 root root 0 Aug 2 17:35 host12 -> ../../devices/pci0000:20/0000:20:03.1/0000:21:00.0/host12/fc_host/host12
lrwxrwxrwx. 1 root root 0 Aug 2 17:35 host13 -> ../../devices/pci0000:20/0000:20:03.1/0000:21:00.1/host13/fc_host/host13
lrwxrwxrwx. 1 root root 0 Aug 2 17:35 host14 -> ../../devices/pci0000:80/0000:80:01.1/0000:81:00.0/host14/fc_host/host14
lrwxrwxrwx. 1 root root 0 Aug 2 17:35 host15 -> ../../devices/pci0000:80/0000:80:01.1/0000:81:00.1/host15/fc_host/host15 

$ cat /sys/class/fc_host/host*/port_name
0x100000620b416b60
0x100000620b416b61
0x100000620b416b75
0x100000620b416b76
```
{{< /note >}}

<!-- PCIe devices -->
{{< note title="PCIe devices" >}}
```bash
# Get Vendor ID and Device ID of all PCI Devices
grep PCI_ID /sys/bus/pci/devices/*/uevent

# vGPU mdev devices
ls /sys/class/mdev_bus/*/mdev_supported_types
```
{{< /note >}}

<!-- Mellanox NIC -->
{{< note title="Mellanox NIC" >}}
```bash
# Upgrade firmware on NIC
/mlxup -u -y -i fw-ConnectX5-rel-16_25_4518-MCX516A-CCA_Ax-UEFI-14.18.19-FlexBoot-3.5.701.bin

# Upgrade using mstflint utility
mstflint -d <lspci-device-id> -i <image-file> b

# Query SRIOV VFs
yum install -y mstflint
for adapter in $(lspci -D| grep -i mel |awk '{print $1}');do
    mstconfig -d ${adapter} query |egrep -i "sriov|vfs”
done

# Set mellanox adapters enable SRIOV set #vfs
for adapter in $(lspci -D| grep -i mel |awk '{print $1}');do
    mstconfig -y -d ${adapter} set SRIOV_EN=1
    mstconfig -y -d ${adapter} set NUM_OF_VFS=64
done

# eSwitch
$ devlink dev eswitch show pci/0000:88:00.0
pci/0000:88:00.0: mode legacy inline-mode none encap enable
```
{{< /note >}}


<!-- Manual sosreport script -->
{{< note title="Manual sosreport script" >}}
```bash
#!/bin/bash
export LANG=C

# If this script hangs, un-comment the below two entries and note the command that the script hangs on.  Then comment out that command and re-run the script.
# set -x
# set -o verbose

[[ -d /tmp/sosreport ]] && rm -rf /tmp/sosreport
mkdir /tmp/sosreport && cd /tmp/sosreport && mkdir -p  var/log etc/lvm etc/sysconfig network storage sos_commands/networking

echo -e "Gathering system information…”
hostname &> hostname  
cp -a /etc/redhat-release  ./etc/ 2>> error_log
uptime &> uptime 

echo -e "Gathering application information…”
chkconfig --list &> chkconfig
top -bn1 &> top_bn1
service --status-all &> service_status_all
date &> date
ps auxww &> ps_auxww
ps -elf &> ps_-elf
rpm -qa --last &> rpm-qa
echo -e "Running 'rpm -Va'. This may take a moment.”
rpm -Va &> rpm-Va

echo -e "Gathering memory information…”
free -m &> free  
vmstat 1 10 &> vmstat

echo -e "Gathering network information…”
ifconfig &> ./network/ifconfig  
netstat -s &>./network/netstat_-s
netstat -agn &> ./network/netstat_-agn
netstat -neopa &> ./network/netstat_-neopa
route -n &> ./network/route_-n
for i in $(ls /etc/sysconfig/network-scripts/{ifcfg,route,rule}-*) ; do echo -e "$i\n..................."; cat $i;echo " ";  done &> ./sos_commands/networking/ifcfg-files    
for i in $(ifconfig | grep "^[a-z]" | cut -f 1 -d " "); do echo -e "$i\n..................." ; ethtool $i; ethtool -k $i; ethtool -S $i; ethtool -i $i;echo -e "\n" ; done &> ./sos_commands/networking/ethtool.out
cp /etc/sysconfig/network ./sos_commands/networking/ 2>> error_log
cp /etc/sysconfig/network-scripts/ifcfg-* ./sos_commands/networking/ 2>> error_log
cp /etc/sysconfig/network-scripts/route-* ./sos_commands/networking/ 2>> error_log
cat /proc/net/bonding/bond* &> ./sos_commands/networking/proc-net-bonding-bond 2>> error_log
iptables --list --line-numbers &> ./sos_commands/networking/iptables_--list_--line-numbers
ip route show table all &> ./sos_commands/networking/ip_route_show_table_all
ip link &> ./sos_commands/networking/ip_link

echo -e "Gathering Storage/Filesystem information…”
df -l &> df
fdisk -l &> fdisk
parted -l &> parted
cp -a /etc/fstab  ./etc/ 2>> error_log
cp -a /etc/lvm/lvm.conf ./etc/lvm/ 2>> error_log
cp -a /etc/lvm/backup/ ./etc/lvm/ 2>> error_log
cp -a /etc/lvm/archive/ ./etc/lvm/ 2>> error_log
cp -a /etc/multipath.conf ./etc/ 2>> error_log
cat /proc/mounts &> mount  
iostat -tkx 1 10 &> iostat_-tkx_1_10
parted -l &> storage/parted_-l
vgdisplay -v &> storage/vgdisplay
lvdisplay &> storage/lvdisplay
pvdisplay &> storage/pvdisplay
pvs -a -v &> storage/pvs
vgs -v &> storage/vgs
lvs -o +devices &> storage/lvs
multipath -v4 -ll &> storage/multipath_ll
pvscan -vvvv &> storage/pvscan
vgscan -vvvv &> storage/vgscan
lvscan -vvvv &> storage/lvscan
lsblk &> storage/lsblk
lsblk -t &> storage/lsblk_t
dmsetup info -C &> storage/dmsetup_info_c
dmsetup status &>  storage/dmsetup_status 
dmsetup table &>  storage/dmsetup_table
ls -lahR /dev &> storage/dev

echo -e "Gathering kernel information…”
cp -a /etc/security/limits.conf ./etc/ 2>> error_log
cp -a /etc/sysctl.conf ./etc/ 2>> error_log
ulimit -a &> ulimit
cat /proc/slabinfo &> slabinfo
cat /proc/interrupts &> interrupts 
cat /proc/iomem &> iomem
cat /proc/ioports &> ioports
slabtop -o &> slabtop_-o
uname -a &> uname
sysctl -a &> sysctl_-a
lsmod &> lsmod
cp -a /etc/modprobe.conf ./etc/ 2>> error_log
cp -a  /etc/sysconfig/* ./etc/sysconfig/ 2>> error_log
for MOD in `lsmod | grep -v "Used by"| awk '{ print $1 }'`; do modinfo  $MOD 2>&1 >> modinfo; done;
ipcs -a &> ipcs_-a
ipcs -s | awk '/^0x/ {print $2}' | while read semid; do ipcs -s -i $semid; done &> ipcs_-s_verbose
sar -A &> sar_-A
cp -a /var/log/dmesg dmesg 2>> error_log
dmesg &> dmesg_now

echo -e "Gathering hardware information…”
dmidecode &> dmidecode
lspci -vvv &> lspci_-vvv
lspci &> lspci
cat /proc/meminfo &> meminfo  
cat /proc/cpuinfo &> cpuinfo

echo -e "Gathering kump information…”
cp -a /etc/kdump.conf ./etc/ 2>> error_log
ls -laR /var/crash &> ls-lar-var-crash
ls -1 /var/crash | while read n; do mkdir -p var/crash/${n}; cp -a /var/crash/${n}/vmcore-dmesg* var/crash/${n}/ 2>> error_log; done

echo -e "Gathering container related information…”
mkdir container
rpm -q podman || alias podman=“docker”
podman ps &> container/ps
podman image list &> container/image_list
podman ps | awk '$1!="CONTAINER" {print $1}' | while read id; do podman inspect $id &> container/inspect_${id}; done

echo -e "Gathering logs…”
cp -a /var/log/{containers*,message*,secure*,boot*,cron*,yum*,Xorg*,sa,rhsm,audit,dmesg} ./var/log/ 2>> error_log
cp -a /etc/*syslog.conf ./etc/ 2>> error_log

echo -e "Compressing files…”
tar -cjf /tmp/sosreport.tar.bz2 ./

echo -e "Script complete."
```
{{< /note >}}
