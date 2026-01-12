---
title: "OpenvSwitch CLI Guide"
date: 2020-12-08T06:15:40+06:00
hero: ovs.svg
description: OpenvSwitch CLI Guide
menu:
  sidebar:
    name: OpenvSwitch Guide
    identifier: ovs-cli-guide
    weight: 20
---

This article provides information about OVS commands that can be used to troubleshoot OpenvSwitch related issues.
<!--more-->

### What is Open vSwitch?

Open vSwitch is a software implementation of a virtual multilayer network switch, designed to enable effective network automation through programmatic extensions, while supporting standard management interfaces and protocols such as NetFlow, sFlow, SPAN, RSPAN, CLI, LACP and <abbr title="IEEE standard which defines protocols and practices for OAM (Operations, Administration, and Maintenance) for paths through 802.1 bridges and local area networks (LANs)">802.1ag</abbr>. 


---------------------------------------
### OVS Base Commands
OVS is a feature rich vSwitch with different configuration commands, but the majority of the configuration and troubleshooting can be accomplished with the following four commands:

* **ovs-vsctl** : Used for configuring the ovs-vswitchd configuration database (known as ovs-db)
* **ovs-ofctl** : A command line tool for monitoring and administering OpenFlow switches
* **ovs-dpctl** : Used to administer Open vSwitch datapaths
* **ovs−appctl** : Used for querying and controlling Open vSwitch daemons


---------------------------------------
#### ovs-vsctl
This tool is used for configuration and viewing OVS switch operations. Port configuration, bridge additions/deletions, bonding, and VLAN tagging are just some of the options that are available with this command. Below are the most useful show commands:

<kbd>ovs-vsctl –V</kbd> : Prints the current version of openvswitc

<kbd>ovs-vsctl show</kbd> : Prints a brief overview of the switch database configuration

<kbd>ovs-vsctl list-br</kbd> : Prints a list of configured bridges

<kbd>ovs-vsctl list-ports &lt;bridge&gt;</kbd> : Prints a list of ports on a specific bridge

<kbd>ovs-vsctl list interface</kbd> : Prints a list of interfaces

The above should be fairly self explanatory. Below are the common switch configuration commands:

<kbd>ovs-vsctl add-br &lt;bridge&gt;</kbd> : Creates a bridge in the switch database.

<kbd>ovs-vsctl add-port &lt;bridge&gt; &lt;interface&gt;</kbd> : Binds an interface (physical or virtual) to a bridge.

<kbd>ovs-vsctl add-port &lt;bridge&gt; &lt;interface&gt; tag=&lt;VLAN number&gt;</kbd> : Converts port to an access port on specified VLAN (by default all OVS ports are VLAN trunks).

<kbd>ovs-vsctl set interface &lt;interface&gt; type=patch options:peer=&lt;interface&gt;</kbd> : Used to create patch ports to connect two or more bridges together.


---------------------------------------
#### ovs-ofctl
This tool is used for administering and monitoring OpenFlow switches. Even if OVS isn't configured for centralized administration, ovs-ofctl can be used to show the current state of OVS including features, configuration, and table entries.

Below are common show commands:

<kbd>ovs-ofctl show &lt;bridge&gt;</kbd> : Shows OpenFlow features and port descriptions.

<kbd>ovs-ofctl snoop &lt;bridge&gt;</kbd> : Snoops traffic to and from the bridge and prints to console.

<kbd>ovs-ofctl dump-flows &lt;bridge&gt; &lt;flow&gt;</kbd> : Prints flow entries of specified bridge. With the flow specified, only the matching flow will be printed to console. If the flow is omitted, all flow entries of the bridge will be printed. 

<kbd>ovs-ofctl dump-ports-desc &lt;bridge&gt;</kbd> : Prints port statistics. This will show detailed information about interfaces in this bridge, include the state, peer, and speed information. Very useful for viewing port connectvity and detecting errors in NIC to bridge bonding.

<kbd>ovs-ofctl dump-tables-desc &lt;bridge&gt;</kbd> : Similar to above but prints the descriptions of tables belonging to the stated bridge.

Below are the common configurations used with the ovs-ofctl tool:

<kbd>ovs-ofctl add-flow &lt;bridge&gt; &lt;flow&gt;</kbd> : Adds a static flow to the specified bridge. Useful in defining conditions for a flow (i.e. prioritize, drop, etc).

<kbd>ovs-ofctl del-flows &lt;bridge&gt; &lt;flow&gt;</kbd> : Deletes the flow entries from flow table of stated bridge. If the flow is omitted, all flows in specified bridge will be deleted.

The above commands can take many arguments regarding different field to match. They can be used for simple source/destination flow additions to complex L3 rewriting like SNAT, DNAT, etc. You can even build a functional router with them!


---------------------------------------
#### ovs-dpctl
ovs-dpctl is very similar to ovs-ofctl in that they both show flow table entries. The flows that ovs-dpctl prints are always an exact match and reflect packets that have actually passed through the system within the last few seconds. ovs-dpctl queries a kernel datapath and not an OpenFlow switch. This is why it's useful for debugging flow data.

OVS uses a single datapath that is shared by all bridges of that type. In order to create a new datapath, use the following:

<kbd>ovs-dpctl add-dp dp1;
ovs-dpctl add-if dp1 eth0</kbd>

<kbd>ovs-dpctl dump-flows</kbd> : Use the following to view flow table data.


---------------------------------------
#### ovs-appctl
OVS is comprised of several daemons that manage and control an Open vSwitch switch. ovs-appctl is a utility for managing these daemons at runtime. It is useful for configuring log module settings as well as viewing all OpenFlow flows, including hidden ones.

The following are useful commands to use:

<kbd>ovs-appctl bridge/dump-flows &lt;bridge&gt;</kbd> : Dumps OpenFlow flows, including hidden flows. Useful for troubleshooting in-band issues.

<kbd>ovs-appctl dpif/dump-flows &lt;bridge&gt;</kbd> : Dumps datapath flows for only the specified bridge, regardless of the type.

<kbd>ovs-appctl vlog/list</kbd> : Lists the known logging modules and their current levels. Use ovs-appctl vlog/set to set/change the module log level.

<kbd>ovs-appctl ofproto/trace</kbd> : Used to show entire flow field of a given flow (flow, matched rule, action taken).

<kbd>ovs-appctl fdb/show &lt;bridge&gt;</kbd> : Lists each MAC address/VLAN pair learned by the specified bridge, along with the port on which it was learned and the age of the entry, in seconds.

<kbd>ovs-appctl fdb/flush &lt;bridge&gt;</kbd> : Flushes bridge MAC address learning table, or all learning tables if no bridge is given.

<kbd>ovs-appctl fdb/stats-show &lt;bridge&gt;</kbd> : Shows MAC address learning table statistics for the specified bridge.

<kbd>ovs-appctl fdb/stats-clear &lt;bridgegt;</kbd> : Clears bridge MAC address learning table statistics, or all statistics if no bridge is given.

---------------------------------------
#### Bonus: Multicast on OpenvSwitch
Use these commands to enable IGMP-based multicast snooping on br-int and inspect the multicast forwarding table, helping diagnose multicast flooding or missing group subscriptions.

<kbd>ovs-vsctl set Bridge br-int mcast_snooping_enable=true

<kbd>ovs-vsctl set Bridge br-int other_config:mcast-snooping-disable-flood-unregistered=true

<kbd>ovs-vsctl get Bridge br-int other_config

<kbd>ovs-appctl mdb/show br-int

<kbd>ovs−vsctl clear bridge ovs-br other_config

