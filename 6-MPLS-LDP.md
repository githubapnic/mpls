![](apnic_logo.png)

# Module 6 - MPLS LDP Configuration
## Topology introduction:
-	The topology below shows 4 regional networks comprised of a core POP and 2 aggregation POPs (edge routers).
-	Edge routers aggregate downstream customers.
-	The regional networks are interconnected with redundant transport links.

![](LabTopology-MPLS-20240523.png)

<div style="page-break-after: always;"></div>

## Lab Tasks

After Module-2, you should now be able to reach All of our infrastructure Point to Point and Loopback Interfaces.

The Next Phase will be to deploy MPLS LDP.
We will need to configure MPLS LDP on all relevant
interface of infrastructure routers. After finishing the configuration please ensure that required
LSP has been built for all the infrastructure prefixes and you can do LSP ping and LSP
traceroute from PE routers to the other PE routers within the Lab.
Prerequisites: Knowledge of IGP, EGP, MPLS, LDP is required.     

Take note of the following:


1.	After the LDP configuration there will be 26 LSP build for the 26 IGP prefix according the table below:

| Loopback |
|:-------------:|
|R1 => 172.16.0.1/32| 
|R2 => 172.16.0.1/32 | 
|R3 => 172.16.0.3/32 | 
|R4 => 172.16.0.4/32|
|R5 => 172.16.0.5/32|
|R6 => 172.16.0.6/32|
|R7 => 172.16.0.7/32|
|R8 => 172.16.0.8/32|
|R9 => 172.16.0.9/32|
|R10 => 172.16.0.10/32|
|R11 => 172.16.0.11/32|
|R12 => 172.16.0.12/32|


<div style="page-break-after: always;"></div>

## Lab Exercise
### Step 1 - MPLS LDP Configuration:
Here is an example COnfiguration for R4

First we need to tell the router that we are going to use MPLS IP for routing

```
conf t
mpls ip
```

Next we configure some parameters to tell LDP how we are going to work.
```
mpls ldp
   router-id 172.16.0.4
   transport-address interface Loopback0
   interface disabled default
   no shutdown
```
We define the 32-bit router-id.
We then use the Loopback as our transport interface and
Diable MPLS from running on all interfaces by default(We will enable it PER-Interface in the next steps)
By default in Arista EOS, the  MPLS LDP process is shutdown, so we need to start it with `no shutdown`

WE will attempt modify the label ranges that each router is using.
THis is a bit more hit and miss than say a Cisco, as Arista creates a very large pool based on protocol types.

```
mpls label range static 16 83
mpls label range dynamic 400 131072
```
So what we are doing here is moving the static range to somthing small to free up space, then setting the start of the dynamic pool (there are 2 dynamic pools, 1 for LDP and 1 for BGP)

_**The starting pool number will depend on your router**_ 

| Router Number | Pool Start    |
|:-------------:|:-----------   |
|   R1          |   100         |
|   R2          |   200         |
|   R3          |   300         |
|   ------      |   ------      |
|   snip        |   snip        |
|   ------      |   ------      |
|   R10         |   1000        |
|   R11         |   1100        |
|   R12         |   1200        |


The next step will be to enable MPLS on our Infrastructre interfaces (Only those facing towrds P and PE routers, not the CE ROuters) and Loopback interface.

The Following is an example from R2

```
interface Ethernet1/1
    mpls ldp interface
!
interface Ethernet1/2
    mpls ldp interface
!
interface Ethernet1/4
    mpls ldp interface
!
interface Ethernet1/5
    mpls ldp interface
!
interface Loopback 0
    mpls ldp interface
```

_Please remember to save the configuration._ `wr`

### Step 2 - Verify MPLS configuration:

`show mpls label ranges` 	 Look at the assigend local Label ranges

`show mpls ldp discovery`   SHow the interfaces configured for LDP discovery

`show mpls ldp bindings`  	 Check the local and remote bindings for Prefixes

`show mpls route`    Show the MPLS routing table

`show mpls lfib route`  Show the MPLS LFIB table

`show mpls ldp neighbor`    Check the LDP Neighbour information





### Step 3 - Verify Reachability:

Make sure you can reach (ping and tracroute) all other routers Loopbacks in the infrastructure network.

`ping mpls ldp ip X.X.X.X/32`
`traceroute mpls ldp ip x.x.x.x/32`

***
