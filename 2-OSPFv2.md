![](apnic_logo.png)

# Module 2 - OSPF(v2) Configuration
## Topology introduction:
-	The topology below shows 4 regional networks comprised of a core POP and 2 aggregation POPs (edge routers).
-	Edge routers aggregate downstream customers.
-	The regional networks are interconnected with redundant transport links.

![](LabTopology-MPLS-20240523.png)

<div style="page-break-after: always;"></div>

## Lab Tasks

After Module-1, you can only reach the directly connected neighbours (only the directly connected interfaces), but not the other routers in your network (R1-R12).

To be able to reach all routers within your network, we need to deploy interior gateway routing (IGP) protocols like OSPF and IS-IS.

In this lab, we will run OSPF to carry both IPv4 and IPv6 address families. This helps consolidate both address families under a single OSPF process (as opposed to running two separate OSPF processes, OSPFv2/v3 for IPv4 and IPv6).      

Take note of the following:

1.	To scale, it is advisable to only carry your infrastructure prefixes in IGP (loopbacks, point-to-point addresses and transport links), but not customer routes/prefixes (which includes the customer point-to-point links).

2.	The whole network (R1-R12) runs in OSPF area-0 (backbone area).

3.	After finishing OSPF configuration we should see the following 26 new prefixes in all infrastructure routers’ routing table.

| Loopback | Point-to-Point | Transport|
|:-------------:|:---------------:|:-------------:|
|R1 => 172.16.0.1/32| R1-R2 => 172.16.1.0/30|Purple=>172.16.1.48/29|
|R2 => 172.16.0.1/32 | R2-R3 => 172.16.1.4/30|Green=>172.16.1.56/29|
|R3 => 172.16.0.3/32 | R1-R3 => 172.16.1.8/30| |
|R4 => 172.16.0.4/32|R4-R5 => 172.16.1.12/30||
|R5 => 172.16.0.5/32|R5-R6 => 172.16.1.16/30||
|R6 => 172.16.0.6/32|R4-R6 => 172.16.1.20/30||
|R7 => 172.16.0.7/32|R7-R8 => 172.16.1.24/30||
|R8 => 172.16.0.8/32|R8-R9 => 172.16.1.28/30||
|R9 => 172.16.0.9/32|R7-R9 => 172.16.1.32/30||
|R10 => 172.16.0.10/32|R10-R11 => 172.16.1.36/30||
|R11 => 172.16.0.11/32|R11-R12 => 172.16.1.40/30||
|R12 => 172.16.0.12/32|R10-R12 => 172.16.1.44/30|&nbsp;|





<div style="page-break-after: always;"></div>

## Lab Exercise
### Step 1 - OSPF Configuration:

We will be using OSPF for our undlying IGP.
Even though we have IPv6 configured, we will not be including this in our Configuration as we will be using other methods for transporting IPv6 in a later lab.

* If there are no active IPv4 addresses configured, we need to manually configure a 32-bit router-ID
* In OSPF, `passive-interface default` will be configured. The function is to not create hello message by default to all interface. This approach is very effective if we would like to build OSPF adjacency on the selected interface. Specially if there is an external facing interface and no OSPF routing to them. We will disable `passive-interface` on those interfaces which are running OSPF.

Example OSPF config on a R1:
```
conf t
router ospf 17821
   passive-interface default
   no passive-interface Ethernet1/1
   no passive-interface Ethernet1/2
   exit
```
To advertise networks into OSPF, you need to enable OSPF directly on the interfaces
```
interface Loopback 0
    ip ospf area 0
```

Since we are using ethernet links (which are broadcast/multi-access interfaces) as point-to-point links, to avoid the election of DR/BDR, we need to explicitly tell OSPF process that they are point-to-point links

On edge PE(POP) routers, for example, R4:
```
interface Ethernet1/1
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Ethernet1/2
   ip ospf network point-to-point
   ip ospf area 0.0.0.0
!
interface Loopback0
   ip ospf area 0.0.0.0
```

On Core(P) routers, the transport links are still broadcast, therefore we only configure `point-to-point` on P2P interfaces, not on transport links.

For example, R2:
```
interface Ethernet 1/1
    ip ospf area 0
!
ip ospf network point-to-point
    interface Ethernet 1/2
    ip ospf area 0
    ip ospf network point-to-point
!
interface Ethernet 1/4
    ip ospf area 0
!
interface Ethernet 1/5
    ip ospf area 0
!
interface Loopback0
    ip ospf area 0
```

_Please remember to save the configuration._ `wr`

### Step 2 - Verify OSPF configuration:

`show ip ospf neighbor` 	 Check OSPF neighbor table

`show ip ospf database`  	 Check OSPF topology table

`show ip route ospf`    Check ipv4 routes/prefixes learned through OSPF




### Step 3 - Verify Reachability:

Make sure you can reach (`ping`) all other routers in the network (loopbacks and point-to-points).

***
