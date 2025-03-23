---
title: BGP Concepts
weight: 50
menu:
  notes:
    name: BGP
    identifier: notes-bgp-concepts
    parent: notes-network
    weight: 50
---

<!-- BGP Summary-->
{{< note title="BGP Summary" >}}

Protocol type: Path Vector  
Type: EGP (External Gateway Protocol)  
Packet Types: Open, Update, KeepAlive, Notification  
Administrative Distance: eBGP: 20; iBGP: 200  
Transport: TCP port 179  
Keepalive 60 sec, hold timer 180  
Neighbor States: Idle -> Active -> Connect -> Open Sent -> Open Confirm -> Established  

1 – Idle: the initial state of a BGP connection. In this state, the BGP speaker is waiting for a BGP start event, generally either the establishment of a TCP connection or the re-establishment of a previous connection. Once the connection is established, BGP moves to the next state.  
2 – Connect: In this state, BGP is waiting for the TCP connection to be formed. If the TCP connection completes, BGP will move to the OpenSent stage; if the connection cannot complete, BGP goes to Active.  
3 – Active: In the Active state, the BGP speaker is attempting to initiate a TCP session with the BGP speaker it wants to peer with. If this can be done, the BGP state goes to OpenSent state.  
4 – OpenSent: the BGP speaker is waiting to receive an OPEN message from the remote BGP speaker  
5 – OpenConfirm: Once the BGP speaker receives the OPEN message and no error is detected, the BGP speaker sends a KEEPALIVE message to the remote BGP speaker.  
6 – Established: All of the neighbor negotiations are complete. You will see a number, which tells us the number of prefixes the router has received from a neighbor or peer group.  

Path Selection Attributes: (highest) Weight > (highest) Local Preference > Originate > (shortest) AS Path > Origin > (lowest) MED > External > IGP Cost > eBGP Peering > (highest) Router ID

(Originate: prefer routes that it installed into BGP by itself over a route that another router installed in BGP)

Authentication: MD5

BGP Origin codes: i – IGP (injected by “network” statement), e – EGP, ? – Incomplete

AS number range: Private AS range: 64512 – 65535, Globally (unique) AS: 1 – 64511

IBGP - can form over hops but use full mesh other routing to next hop does not work (unless IGP is setup, bgp session between 2 loopbacks). IGP synchronization must be IGP is not used for forwarding transit packets. (Redistribute ebgp to igp then back to ebgp). peer between loopbacks for redundancy using update-source lo0 command.

EBGP multihop command with TTL. also if update-source for lo0 add static route for connectivity

^ is mutually exclusive with ttl-security hops x

Inject prefix using network (local, static or igp route in table) or redistribute
if router does not know how to reach the next hop of prefix, it won’t advertise to other router. make external subnets known internal is risk, use next-hop-self

Origin has not much use as EGP is obsolete. IGP(network), EGP, incomplete(redistribute)

Aggregate route using Null0 static address or aggregate-address command

Atomic_aggregate attribute - indicates prefix is aggregate and not actual destination. not much use

aggregator attribute - as number and router id originating router

AS_SET - unordered list of all ASN of all prefix belonging to aggregate rarely used

ip prefix-list  filter
ip as-path access-list  filter

Originator_id and cluster_list prevent loops when using route reflector

confiderations is similar to route reflector. break into sub-AS. But local-pref, med and next hop are preserved within whole confideration

{{< /note >}}

<!-- BGP Path Selection -->
{{< note title="BGP Path Selection" >}}

1. Prefer the path with the highest WEIGHT. 
2. Prefer the path with the highest LOCAL_PREF.
3. Prefer the path that was locally originated via a network or aggregate BGP subcommand or through redistribution from an IGP. Local paths that are sourced by the network or redistribute commands are preferred over local aggregates that are sourced by the aggregate-address command.
4. Prefer the path with the shortest AS_PATH.
5. Prefer the path with the lowest origin type.
6. Prefer the path with the lowest multi-exit discriminator (MED).
7. Prefer eBGP over iBGP paths.
8. Prefer the path with the lowest IGP metric to the BGP next hop.
9. Determine if multiple paths require installation in the routing table for BGP Multipath.
10. When both paths are external, prefer the path that was received first (the oldest one).
11. Prefer the route that comes from the BGP router with the lowest router ID.
12. If the originator or router ID is the same for multiple paths, prefer the path with the minimum cluster list length.
13. Prefer the path that comes from the lowest neighbor address.

“We Love Oranges AS Oranges Mean Pure Refreshment”

{{< /note >}}

<!-- Multicast -->
{{< note title="Multicast" >}}

PIM dense mode (PIM-DM) uses a push model to flood multicast traffic to every corner of the network. This push model is a brute-force method of delivering data to the receivers. This method would be efficient in certain deployments in which there are active receivers on every subnet in the network. PIM-DM initially floods multicast traffic throughout the network. Routers that have no downstream neighbors prune the unwanted traffic. This process repeats every 3 minutes.

PIM Sparse Mode (PIM-SM) uses a pull model to deliver multicast traffic. Only network segments with active receivers that have explicitly requested the data receive the traffic. PIM-SM distributes information about active sources by forwarding data packets on the shared tree. Because PIM-SM uses shared trees (at least initially), it requires the use of an RP. The RP must be administratively configured in the network.

A rendezvous point (RP) is required only in networks running Protocol Independent Multicast sparse mode (PIM-SM). By default, the RP is needed only to start new sessions with sources and receivers.
The concept of joining the rendezvous point (RP) is called the RPT (Root Path Tree) or shared distribution tree. The RP is the root of our tree which decides where to forward multicast traffic to. Each multicast group might have different sources and receivers so we might have different RPTs in our network.

PIM-DM supports only source trees – that is, (S,G) entries–and cannot be used to build a shared distribution tree.

IGMP is used to dynamically register individual hosts in a multicast group on a particular LAN. Enabling PIM on an interface also enables IGMP. IGMP provides a means to automatically control and limit the flow of multicast traffic throughout your network with the use of special multicast queriers and hosts.

A querier is a network device, such as a router, that sends query messages to discover which network devices are members of a given multicast group.

A host is a receiver, including routers, that sends report messages (in response to query messages) to inform the querier of a host membership. Hosts use IGMP messages to join and leave multicast groups.

Hosts identify group memberships by sending IGMP messages to their local multicast device. Under IGMP, devices listen to IGMP messages and periodically send out queries to discover which groups are active or inactive on a particular subnet.

{{< /note >}}