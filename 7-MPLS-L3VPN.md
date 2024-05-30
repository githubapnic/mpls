![](apnic_logo.png)

# Module 6 - MPLS P2P L3 VPN Configuration
## Topology introduction:
-	The topology below shows 4 regional networks comprised of a core POP and 2 aggregation POPs (edge routers).
-	Edge routers aggregate downstream customers.
-	The regional networks are interconnected with redundant transport links.

![](LabTopology-MPLS-20240523.png)

<div style="page-break-after: always;"></div>

## Lab Tasks

After Module-6, you should now be able to reach All of our infrastructure devices via MPLS LDP.

In this module, participants will need to configure 3 types of L3 VPN as a separate sub-module.

Part A. Point-Point VPN
Part B. Extranet VPN (Member of another VPN for a specific purpose)
Part C. Hub and Spoke/Point-to-Multipoint VPN

Prerequisites: Knowledge of IGP, EGP, MPLS, LDP is required.     

Take note of the following:


<div style="page-break-after: always;"></div>

## Lab Exercise
### Part A - Point-Point VPN:
Here is an example COnfiguration for R4

MPLS L3 point to point VPN tunnel will be created between CPE routers:

1. r13-CAR1 to r15-CAR2 [Traffic from R13 E1/3 to R15 E1/3, Capture R1 E1/3 & R4 E1/3] (GREEN)
2. r14-CBR1 to r16-CBR2 [Traffic from R14 E1/3 to R16 E1/3, Capture R3 E1/3 & R6 E1/3] (RED)
3. r19-CAR4 to r17-CAR3 [Traffic from R19 E1/3 to R17 E1/3, Capture R10 E1/3 & R7 E1/3] (BLUE)
4. r20-CBR4 to r18-CBR3 [Traffic from R20 E1/3 to R18 E1/3, Capture R12 E1/3 & R9 E1/3] (BLACK)

Please spend some time getting familiar with the network topology and addressing plan

#### Lab Steps

To configure this module exercises we need to configure routers with following 4 steps:

1. VRF configuration on the PE routers
2. Adding PE-CE eBGP neighbour on the PE router VRF context of BGP
3. Activate VPNv4 address family on PE routers to send MPLS L3 VPN signalling to other P and PE routers.
4. After the PE routers configuration, we need to configure CE router regular eBGP configuration with the PE router


#### Part 1A - Configuration on PE Routers:

1. Intialise the VRF and enable routing for the VRF
This is an example for `R6`
```
conf t
vrf instance R16-RED
ip routing vrf R16-RED
```
2. Next we will configre the RD and Route Targets for the VRF and activate the VPN-IPv4 address family
Arista has deprecated the the use of RD in native VRFs and this is now done throough the BGP configurations

```
router bgp 17821
    vrf R16-RED
      rd 172.16.17.0:1
      route-target export vpn-ipv4 172.16.17.0:1
      route-target import vpn-ipv4 172.16.4.0:1
      router-id 172.16.0.6
      redistribute connected
      redistribute static

    address-family vpn-ipv4
      neighbor 172.16.0.3 activate
      neighbor default encapsulation mpls next-hop-self source-interface Loopback0
```
In the Above, we have declared that routes in this VRF will be identified by the Route Distinguisher of `172.16.17.0:1`.
We will Export those and import routes with a tag of `172.16.4.0:1` (CPE Routes from R14 in this case)
We have also added a VPN-IPv4 relationship to our Far-end peer

3. Next Step will be to add the Interface towards the CE (E1/3) into the VRF.
Once you enable the VRF on the interface, it will remove the IP address configuration, so you will need to re-add it.

I would advise doing the following before enabling the vrf on the interface
`show active` will show the current active configuration at any sub-level configuration component with Arista EOS

```
r6#conf t
r6(config)#int e1/3
r6(config-if-Et1/3)#show active
interface Ethernet1/3
   description R6-R16
   no switchport
   ip address 172.16.28.21/30
   ipv6 address 2406:6400:100:5::/127
```
This way, when you add the VRF, you can just copy and paste the IP address configs back in.

```
interface Ethernet1/3
   vrf R16-RED
   ip address 172.16.28.21/30
   ipv6 address 2406:6400:100:5::/127
```


4. Next we need to establish a iBGP neighbourship with MP-BGP for R3 (The far side PE router).  This will need to be done for ALL PE-PE pairs.

```
router bgp 17821
   router-id 172.16.0.6
   neighbor 172.16.0.3 remote-as 17821
   neighbor 172.16.0.3 update-source Loopback0
   neighbor 172.16.0.3 send-community extended    
```
In the above we are creating the neigbourship, using the loopback0 as our source IP, and enabling extended communities so that our Route Tags are passed as a BGP community attribute.

5. We will now add the PE-CE eBGP neighbours.  This example will be for `R7` 

```
router bgp 17821
   vrf R17-BLUE
      neighbor 172.16.28.10 remote-as 65003
      neighbor 172.16.28.10 as-path remote-as replace out
      redistribute connected
      redistribute static
```
What we have done above is add the BGP Peer in out VRF COntext.  We have also replaced the as-number as Both sides of the VPN connection have the same ASN, so the announcements would be filtered due to the BGP Loop-free mechinisims.

#### Part 1B - Configuration on CE Routers:

1. We will now create a standard eBGP peering from our CE->PE Router.  THis example will be for `R20` (R20-R12)

```
router bgp 65004
   neighbor 172.16.28.29 remote-as 17821
   !
   address-family ipv4
      neighbor 172.16.28.29 activate
      redistribute connected
```

#### Part1C - Validation:
Check the Vrf Routing Table (PE Routers) Examples from `R12`

`sh ip route vrf R20-BLACK`

`sh bgp vpn-ipv4 summary`

`sh bgp vpn-ipv4`


### Part 2 - Full Mesh VPN

!n this scenario no configuration template will be given. Participants need to build their own
configuration based on their understanding of RT manipulation so solve any given task. Try to build
your configuration based on following requirement specification:

1. Review and update your MP-BGP Configurations _**(Hint: Think of each sub-region with route reflectors and peer groups as an option)**_
2. Change the VPN membership and required RT import statement so that all CE routers can
access to all other CE routers.
3. Notice the traffic flow between CE routers to the other CE routers. Sometime the traffic does not need to go via the core routers.

### Part 3 -  Hub and Spoke

On this scenario customer requirement is to forward any traffic between CE routers via the central site
router. For example a bank would like to control its inter branch communication via the head office
router to make sure all the required policy has been enforced.
In this lab module there will be two central site configuration as follows:
1. Central site 1:
<BR>a. Hub Router: R13
<BR>b. Spoke Routers: R14, R15, R16
2. Central site 2:
<BR>a. Hub Router: R17
<BR>b. Spoke Router: R18, R19, R20

_**Before you start configuring any RT import statement on each VRF please remove any existing RT import and come up with a brand-new RT import strategy.**_

_**Keep in mind the exsiting Default routes for the Managememnt network on CE Routers(this can be removed)!!!**_


