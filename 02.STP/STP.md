## Laboratory work. Deployment of a switched network with redundant channels

### Topology

![Scheme](img/Scheme.png)

### Addressing table

| Device | Interface | IP address  | Subnet Mask   |
| ------ | --------- | ----------- | ------------- |
| S1     | VLAN 1    | 192.168.1.1 | 255.255.255.0 |
| S2     | VLAN 1    | 192.168.1.2 | 255.255.255.0 |
| S3     | VLAN 1    | 192.168.1.3 | 255.255.255.0 |

### Goals:

**Part1.** Creating a network and configuring basic device parameters.
**Part 2.** Choosing a Root bridge.
**Part 3.** Monitoring the process of choosing a port by the STP protocol, based on the cost of ports.
**Part 4.** Monitoring the process of selecting a port by the STP protocol, based on the priority of ports.

### Creating a network and configuring basic device parameters.

We will build a network according to the topology, write the device names, and also assign IP to VLAN 1 of the interface.

<details>
<summary>S1</summary>
<pre><code>
enable
conf t
host S1
line con 0
exec-t 0 0
exit
int vlan 1
ip add 192.168.1.1 255.255.255.0
no shut 
</code></pre>
</details>
 <details>
<summary>S2</summary>
<pre><code>
enable
conf t
host S2
line con 0
exec-t 0 0
exit
int vlan 1
ip add 192.168.1.2 255.255.255.0
no shut
</code></pre>
</details>
 <details>
<summary>S3</summary>
<pre><code>
enable
conf t
host S3
line con 0
exec-t 0 0
exit
int vlan 1
ip add 192.168.1.3 255.255.255.0
no shut 
</code></pre>
</details>
We will disable DNS search and also assign passwords to the privileged mode, as well as to the VTY console, logging synchronous for the console line.
<details>
<summary>S1,S2,S3</summary>
<pre><code>
no ip domain-lookup
enable secret cisco
line console 0
password cisco
login 
logging synchronous
banner motd “**This is a secure system. Authorized Access Only!**"'b'
</code></pre>
</details>

Let's check the connection between the switches

<details>
<summary>S1</summary>
<pre><code>
S1#ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/4/5 ms
S1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
S1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/3/5 ms
</code></pre>
</details>

### Root Bridge Definition.

Configuring ports in **trunk mode**

<details>
<summary>S1,S2,S3</summary>
<pre><code>
conf t
int ran e0/0-3
sw tr en d
sw m tr
exit
</code></pre>
</details>

Disable the extra ports, let it be e0/1 and e0/3

<details>
<summary>S1,S2,S3</summary>
<pre><code>
int e0/1
shut
int e0/3
shut
exit
</code></pre>
</details>

Displaying STP data.

<details>
<summary>S1</summary>
<pre><code>
S1(config)#do sh span
!
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
!
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec
!
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    P2p 
Et0/2               Desg FWD 100       128.3    P2p 
</code></pre>
</details>
<details>
<summary>S2</summary>
<pre><code>
S2(config)#do sh span
!
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
!
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec
!
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    P2p 
Et0/2               Desg FWD 100       128.3    P2p 
</code></pre>
</details>
<details>
<summary>S3</summary>
<pre><code>
S3(config)#do sh span
!
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
!
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec
!
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    P2p 
Et0/2               Root FWD 100       128.3    P2p 
</code></pre>
</details>

Now it's clearer :

![status](img/stp_status.png)

the red asterisk indicates that the port is disabled

Root is selected based on the MAC address and BID Priority (default 32769), the smallest MAC device address wins the election.   Port e0/0 on S3 has been blocked by STP and data will not go through it.

### Monitoring the process of choosing a port by the STP protocol, based on the cost of ports.

Since we have information on STP and its ports, now we will observe how the topology will change based on the cost of ports. By default, the cost of Ethernet = 100, let's see how the topology will change when we change the cost of the port to root on S3.
<details>
<summary>S3</summary>
<pre><code>
int e0/2
spanning-tree cost 90
exit
</code></pre>
</details>
Output the STP state in the order S1,S2,S3.

<details>
<summary>S1</summary>
<pre><code>
S1(config)#do sh span
!
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
!
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec
!
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    P2p 
Et0/2               Desg FWD 100       128.3    P2p 
</code></pre>
</details>
<details>
<summary>S2</summary>
<pre><code>
S2(config)#do sh span
!
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
!
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec
!
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    P2p 
Et0/2               Altn BLK 100       128.3    P2p 
</code></pre>
</details>
<details>
<summary>S3</summary>
<pre><code>
S3(config)#do sh sp
!
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        90
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
!
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec
!
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg LIS 100       128.1    P2p 
Et0/2               Root FWD 90        128.3    P2p 
</code></pre>
</details>

![change_cost](img/44res_cost.png)

As you can see , the ports have changed places . Now we will return it as it was.

<details>
<summary>S3</summary>
<pre><code>
int e0/2
no spanning-tree cost 90
exit
</code></pre>
</details>

### Monitoring the process of selecting a port by the STP protocol, based on the priority of ports.

Enabling previously disabled ports on switches

<details>
<summary>S1,S2,S3</summary>
<pre><code>
int e0/1
no shut
int e0/3
no shut
exit
</code></pre>
</details>

Let's check the STP status now

 <details>
<summary>S1</summary>
<pre><code>
S1(config)#do show spanning-tree
!
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
!
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.1000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec
!
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    P2p 
Et0/1               Desg FWD 100       128.2    P2p 
Et0/2               Desg FWD 100       128.3    P2p 
Et0/3               Desg FWD 100       128.4    P2p 
</code></pre>
</details>
 <details>
<summary>S2</summary>
<pre><code>
S2(config)#do show spanning-tree
!
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
!
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.2000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec
!
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    P2p 
Et0/1               Altn BLK 100       128.2    P2p 
Et0/2               Desg FWD 100       128.3    P2p 
Et0/3               Desg FWD 100       128.4    P2p 
</code></pre>
</details>
<details>
<summary>S3</summary>
<pre><code>
S3(config)#do show spanning-tree
!
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.1000
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
!
  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.3000
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec
!
Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    P2p 
Et0/1               Altn BLK 100       128.2    P2p 
Et0/2               Root FWD 100       128.3    P2p 
Et0/3               Altn BLK 100       128.4    P2p 
</code></pre>
</details>
![](img/port_status_after.png)

The S1 switch is root, which means that all its ports are desg. We also see that each switch in the direction of the root has chosen 1 port as root based on the port number and its cost.  And also highlighted an alternative and blocked port. As for the direction of S2 and S3, there is a choice of designed and alternated, so as not to create loops.

1. What value does the STP protocol use first after selecting the root bridge to determine the port selection?

   Search for ports in the direction of the root switch, that is, the shortest paths that depend on the connection speed and the number of intermediate switches.

2. If the first value on the two ports is the same, what is the next value that the STP protocol will use when selecting a port?

   If the port values are equal, the process compares the BID. If the bids are equal, port priorities are used to determine the root bridge. The default priority value is 128, the port with a large value is disabled.

3. If both values on two ports are equal, what will be the next value that the STP protocol uses when selecting a port?

   In this case , STP uses the smallest port number .
