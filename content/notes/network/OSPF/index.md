---
title: OSPF Concepts
weight: 60
menu:
  notes:
    name: OSPF
    identifier: notes-ospf-concepts
    parent: notes-network
    weight: 60
---

<!-- OSPF Overview -->
{{< note title="OSPF Overview" >}}
A classless routing protocol.  
Supports VLSM, CIDR, manual route summarization, equal cost load balancing.  
Incremental updates are supported.  
Uses only one parameter as the metric – the interface cost.  
The administrative distance of OSPF routes is, by default, 110.  
Uses multicast addresses 224.0.0.5 and 224.0.0.6 for routing updates.  

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