---
title: Networking Concepts
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

<!-- OSPF Overview -->
{{< note title="OSPF Overview" >}}
OSPF router ID selection:

OSPF uses the following criteria to select the router ID:
1. Manual configuration of the router ID (via the “router-id x.x.x.x” command under OSPF router configuration mode).
2. Highest IP address on a loopback interface.
3. Highest IP address on a non-loopback and active (no shutdown) interface.

OSPF forms neighbor relationship with other OSPF routers on the same segment by exchanging hello packets. The hello packets contain various parameters. Some of them should match between neighboring routers. These include:

+ Hello and Dead intervals
+ Area ID
+ Authentication type and password
+ Stub Area flag
+ Subnet ID and Subnet mask

When OSPF is run on a network, two important events happen before routing information is exchanged:
+ Neighbors are discovered using multicast hello packets.
+ DR and BDR are elected for every multi-access network to optimize the adjacency building process. All the routers in that segment should be able to communicate directly with the DR and BDR for proper adjacency (in the case of a point-to-point network, DR and BDR are not necessary since there are only two routers in the segment, and hence the election does not take place).
For a successful neighbor discovery on a segment, the network must allow broadcasts or multicast packets to be sent.

In an NBMA network topology, which is inherently nonbroadcast, neighbors are not discovered automatically. OSPF tries to elect a DR and a BDR due to the multi-access nature of the network, but the election fails since neighbors are not discovered. Neighbors must be configured manually to overcome these problems

Each OSPF area only allows some specific LSAs to pass through. Below is a summarization of which LSAs are allowed in each OSPF area:

Area -- Restriction  
Normal -- None  
Stub -- No Type 5 AS-external LSA allowed  
Totally Stub -- No Type 3, 4 or 5 LSAs allowed except the default summary route  
NSSA -- No Type 5 AS-external LSAs allowed, but Type 7 LSAs that convert to Type 5 at the NSSA ABR can traverse  
NSSA Totally Stub -- No Type 3, 4 or 5 LSAs except the default summary route, but Type 7 LSAs that convert to Type 5 at the NSSA ABR are allowed

{{< /note >}}

<!-- OSPF States -->
{{< note title="OSPF States" >}}

When OSPF neighbor relationship is formed, a router goes through several state changes before it becomes fully adjacent with its neighbor. The states are Down -> Attempt (optional) -> Init -> 2-Way -> Exstart -> Exchange -> Loading -> Full. Short descriptions about these states are listed below:

𝐃𝐨𝐰𝐧: no information (hellos) has been received from this neighbor

𝐀𝐭𝐭𝐞𝐦𝐩𝐭: only valid for manually configured neighbors in an NBMA environment. In Attempt state, the router sends unicast hello packets every poll interval to the neighbor, from which hellos have not been received within the dead interval

𝐈𝐧𝐢𝐭: specifies that the router has received a hello packet from its neighbor, but the receiving router’s ID was not included in the hello packet

𝟐-𝐖𝐚𝐲: indicates bi-directional communication has been established between two routers

𝐄𝐱𝐬𝐭𝐚𝐫𝐭: Once the DR and BDR are elected, the actual process of exchanging link state information can start between the routers and their DR and BDR

𝐄𝐱𝐜𝐡𝐚𝐧𝐠𝐞: OSPF routers exchange and compare database descriptor (DBD) packets

𝐋𝐨𝐚𝐝𝐢𝐧𝐠: In this state, the actual exchange of link state information occurs. Outdated or missing entries are also requested to be resent

𝐅𝐮𝐥𝐥: routers are fully adjacent with each other

{{< /note >}}

<!-- OSPF Summarization -->
{{< note title="OSPF Summarization" >}}

OSPF offers two methods of route summarization:
1) Summarization of internal routes performed on the ABRs
2) Summarization of external routes performed on the ASBRs

1) To summarize routes at the area boundary (ABRs), use the command:
area area-id range ip-address mask [advertise | not-advertise] [cost cost]

An internal summary route is generated if at least one subnet within the area falls in the summary address range and the summarized route metric is equal to the lowest cost of all the subnets within the summary address range. Interarea summarization can only be done for the intra-area routes of connected areas, and the ABR creates a route to Null0 to avoid loops in the absence of more specific routes.

2) To summarize external routes on the domain boundary (ASBRs), use the command:
summary-address {{ip-address mask} | {prefix mask}} [not-advertise] [tag tag]
The ASBR will summarize external routes before injecting them into the OSPF domain as type 5 external LSAs.

Note: An exception of using the “summary-address” is at the boundary of a NSSA area.

In both methods of route summarization described above, a summarized route is only generated if at least one subnet in the routing table falls in the summary address range.

Summarization in EIGRP and OSPF

Unlike OSPF where we can summarize only on ABR or ASBR, in EIGRP we can summarize anywhere.

Manual summarization can be applied anywhere in EIGRP domain, on every router, on every interface via the ip summary-address eigrp as-number address mask [administrative-distance ] command (for example: ip summary-address eigrp 1 192.168.16.0 255.255.248.0). Summary route will exist in routing table as long as at least one more specific route will exist. If the last specific route will disappear, summary route also will fade out. The metric used by EIGRP manual summary route is the minimum metric of the specific routes.

{{< /note >}}

<!-- OSPF LSA Types -->
{{< note title="OSPF LSA Types" >}}

𝐑𝐨𝐮𝐭𝐞𝐫 𝐥𝐢𝐧𝐤 𝐋𝐒𝐀 (𝐓𝐲𝐩𝐞 𝟏) – Each router generates a Type 1 LSA that lists its active interfaces, IP addresses, neighbors and the cost to each. LSA Type 1 is only flooded inside the router’s area, it does not cross ABR.
E bit indicates boundary router. used when connected to another domain/ redistribution
B bit - indicate if it is ABR
V bit - indicate endpoint for virtual link

𝐍𝐞𝐭𝐰𝐨𝐫𝐤 𝐥𝐢𝐧𝐤 𝐋𝐒𝐀 (𝐓𝐲𝐩𝐞 𝟐) – is sent out by the designated router (DR) and lists all the routers on the segment it is adjacent to. Types 2 are ﬂooded within its area only; does not cross ABR. Type 1 & type 2 are the basis of SPF path selection.

𝐒𝐮𝐦𝐦𝐚𝐫𝐲 𝐥𝐢𝐧𝐤 𝐋𝐒𝐀 (𝐓𝐲𝐩𝐞 𝟑) – ABRs generate this LSA to send between areas (so type 3 is called inter-area link). It gathers information it has learned on one of its attached areas and summarizes them before sending out to another area. LSAs Type 3 are injected by the ABR from the backbone area into other areas and from other areas into the backbone area.

𝐒𝐮𝐦𝐦𝐚𝐫𝐲 𝐀𝐒𝐁𝐑 𝐋𝐒𝐀 (𝐓𝐲𝐩𝐞 𝟒) – Generated by the ABR to describe an ASBR to routers in other areas so that routers in other areas know how to get to external routes through that ASBR.

𝐄𝐱𝐭𝐞𝐫𝐧𝐚𝐥 𝐋𝐢𝐧𝐤 𝐋𝐒𝐀 (𝐋𝐒𝐀 𝟓) – Generated by ASBR to describe routes redistributed into the area and point the destination for these external routes to the ASBR. These routes appear as O E1 or O E2 in the routing table.
O E1 - external cost and internal ospf cost added
O E2 - static cost. cost does not change

𝐌𝐮𝐥𝐭𝐢𝐜𝐚𝐬𝐭 𝐋𝐒𝐀 (𝐓𝐲𝐩𝐞 𝟔)  – specialized LSAs that are used in multicast OSPF applications. Cisco does not support it.

𝐍𝐒𝐒𝐀 𝐄𝐱𝐭𝐞𝐫𝐧𝐚𝐥 𝐋𝐒𝐀 (𝐓𝐲𝐩𝐞 𝟕) – Generated by an ASBR inside a Not So Stubby Area (NSSA) to describe routes redistributed into the NSSA. LSA 7 is translated into LSA 5 as it leaves the NSSA. These routes appear as N1 or N2 in the routing table inside the NSSA. Much like LSA 5, N2 is a static cost while N1 is a cumulative cost that includes the cost upto the ASBR.

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