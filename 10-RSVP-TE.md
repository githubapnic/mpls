![](apnic_logo.png)

# Module 10 - RSVP-TE Configuration
## Topology introduction:
-	The topology below shows 4 regional networks comprised of a core POP and 2 aggregation POPs (edge routers).
-	Edge routers aggregate downstream customers.
-	The regional networks are interconnected with redundant transport links.

![](LabTopology-MPLS-20240523.png)

<div style="page-break-after: always;"></div>

## Lab Tasks

THe Lab has been restored back to the defailt MPLS-LDP configuration that was configured in Module 6(OSPF/LDP)


MPLS TE Tunnels will be created according to the following scenarios.

For all pairings, take note of the following:
* Verify the current IGP Path from Head-end and End-Point nodes by looking at the existing LSP (Write it down somewhere)
* Look at the diagram  and find other possibible paths from HE to EP Nodes<BR>
* Constrain the current path by lowering the reserved Bandwidth to **BELOW** the tunnel bandwidth on an interface in the path 
* This information will be propagated to other routers via OSPF which will trigger the change in IGP path and MPLS LDP labels.

1. r1 <=> r4

    * Midpoints will be r2 and r5.<BR>
    * Recommended constraint Point `E1/2` on `r4`

2. r3 <=> r6
    * Midpoints will be r2 and r5.<BR>
    * Recommended constraint Point `E1/2` on `r3`
3.  rR10 <=> r7
    * Midpoints will be r11 and r8.<BR>
    * Recommended constraint Point `E1/2` on `r7`

4. r12 <=> r9
    * Midpoints will be r11 and r8.<BR>
    * Recommended constraint Point `E1/2` on `r12`


### Lab configuration Steps
_**The following will need to be done on all PE and P routers**_

1. Prepare the routers for RSVP-TE.  The following is an example for `R11`<BR>
* Enable RSVP-TE
  ```
  conf t
  router traffic-engineering
  router-id ipv4 X.X.X.X ! Insert your IPv4 Loopback address here
   rsvp
      local-interface Loopback0
  !
  mpls rsvp
   no shutdown
  ```
* Configure OSPF for RSVP Signalling
  ```
  router ospf 17821
     traffic-engineering
      no shutdown
  ```
* Configure MPLS participating interfaces for traffic Engineering 
  ```
  interface Ethernet1/1
     traffic-engineering
     traffic-engineering bandwidth 100 percent
  interface Ethernet1/2
     traffic-engineering
     traffic-engineering bandwidth 100 percent
  interface Ethernet1/4
     traffic-engineering
     traffic-engineering bandwidth 100 percent
   interface Ethernet1/5
     traffic-engineering
     traffic-engineering bandwidth 100 percent
   interface Loopback0
     traffic-engineering
  ```
_**DO NOT** configure the PE<=>CE Links on the PE Routers_

2. **FOR TUNNEL HEAD-END and END POINTS** <BR>Build and configure the tunnel and Path.  On Arista EOS this is done under the `router traffic-engineering` stanza:<BR>
* The following example is for `r1`
    ```
    router traffic-engineering
        rsvp
            local-interface Loopback0
            !
            path To-R10 dynamic
            !
            tunnel Tunnel1
                destination ip 172.16.0.4
                path To-R10
                bandwidth 20.00 mbps
                no shutdown
    ```
In the Above, we have 
* Configured the Loopback interface for signalling     
* Created a dynamic path(No explicit hops)
* Defined our tunnel, set a path and the Reserved bandidth for the tunnel and brought it up

3. **ALL PE and P ROUTERS**
* Check the status of the tunnels, RSVP<BR>
`sh mpls rsvp`<BR>
`sh mpls rsvp neighbor`<BR>
`sh mpls rsvp bandwidth` <BR>
`sh mpls rsvp session`<BR>
`sh traffic-engineering rsvp tunnel`<BR>

Explore some of the other `show` command and observe the output

3. **Constrain Bandwidth**<BR>
Add a bandwidth constraint on either the current ingress or egress path, as per recomendataions above(Or choose your own)<BR>
<BR>
The following example is for `r4`<BR>

```
interface Ethernet1/2
  traffic-engineering bandwidth 10 mbps
```
  Now Observe the change in tunnel paths.
  
  _**REMEMBER** Build the tunnel from both directions.  RSVP tunnels are UNI-DIRECTIONAL_


### Step 3 - Verify Configuration:
Run the Verification commands above again.

***
