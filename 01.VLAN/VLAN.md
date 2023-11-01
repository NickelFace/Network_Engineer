 ## Laboratory work. Configuring Extended VLAN, VTP and STP networks

#### Topology

![img](img/VLAN.png)


#### Addressing table

| Table header | Interface | IP address   | Subnet Mask   |
| ------------ | --------- | ------------ | ------------- |
| S1           | VLAN 99   | 192.168.99.1 | 255.255.255.0 |
| S2           | VLAN 99   | 192.168.99.2 | 255.255.255.0 |
| S3           | VLAN 99   | 192.168.99.3 | 255.255.255.0 |
| PC-A         | NIC       | 192.168.10.1 | 255.255.255.0 |
| PC-B         | NIC       | 192.168.20.1 | 255.255.255.0 |
| PC-C         | NIC       | 192.168.10.2 | 255.255.255.0 |


#### **Homework**

##### VLAN

**Goal:** Setting up VTP, Setting up DTP, Adding VLANs and Assigning Ports, Setting up an Extended VLAN

In this lab, you will configure the trunk links between these switches
All switches will be configured to use VTP for VLAN updates. S2 will be configured as a server. 
Configuring Dynamic Trunking Protocol (DTP)
Adding VLANs and Assigning Ports
In Part 4, you will have to switch the VTP on the S1 switch to transparent mode and create an extended-range VLAN network on the S1 switch.

#### Creating a network and configuring basic device parameters:

1. Create a network according to the topology (already connected according to the scheme)

 <details>
<summary>S1</summary>
<pre><code>
Enable
Configure terminal
interface vlan 1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
hostname S1
do copy run start 
</code></pre>
</details>
 <details>
<summary>S2</summary>
<pre><code>
Enable
Configure terminal
interface vlan 1
ip address 192.168.1.2 255.255.255.0
no shutdown
exit
hostname S2
do copy run start 
</code></pre>
</details>
 <details>
<summary>S3</summary>
<pre><code>
Enable
Configure terminal
interface vlan 1
ip address 192.168.1.3 255.255.255.0
no shutdown
exit
hostname S3
do copy run start 
</code></pre>
</details>
2. To disable DNS lookup on each switch, we prescribe in configuration mode

   ```
   no ip domain-lookup
   ```
3. In the configuration mode, we will write : the password for the privileged mode, and for the device input and let logging synchronous for the console line. 
 <details>
<summary>S1,S2,S3</summary>
<pre><code>
no ip domain-lookup
enable secret cisco
line console 0
password cisco
login 
logging synchronous
</code></pre>
</details>

4. Set up a banner when you log in to the device
 <details>
<summary>S1,S2,S3</summary>
<pre><code>
Banner motd “**This is a secure system. Authorized Access Only!
</code></pre>
</details>
5. We start the VTP protocol on the switches

 <details>
<summary>S1</summary>
<pre><code>
vtp domain CCNA
vtp password cisco
vtp version 3
vtp mode client
</code></pre>
</details>
 <details>
<summary>S2</summary>
<pre><code>
vtp domain CCNA
vtp password cisco
vtp version 3
vtp mode server
end
vtp primary server force
</code></pre>
</details>
 <details>
<summary>S3</summary>
<pre><code>
vtp domain CCNA
vtp password cisco
vtp version 3
vtp mode client
</code></pre>
</details>
6. We will switch the ports to TRUNK mode in order for the VTP protocol to work

 <details>
<summary>S1,S2,S3</summary>
<pre><code>
interface  Ethernet 0/3
switchport trunk encapsulation dot1q
switchport mode trunk
interface  Ethernet 0/1
switchport trunk encapsulation dot1q
switchport mode trunk
</code></pre>
</details>

```
show interfaces trunk  
```

<details>
<summary>S3</summary>
<pre><code>
Port        Mode             Encapsulation  Status        Native vlan
Et0/1       on               802.1q         trunking      1
Et0/3       on               802.1q         trunking      1
</code></pre>
</details>
7. On the S2 switch (VTP domain server), we will create new VLANs

 <details>
<summary>S2</summary>
<pre><code>
vlan 999
name VTP_Lab
vlan 10 
name Red
vlan 20
name Blue
vlan 30
name Yellow
vlan 99
name Management  
</code></pre>
</details>
Let's see if VLANs have been added on other devices
 <details>
<summary>S1</summary>
<pre><code>
S3# show vlan brief  
!
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/2
10   Red                              active    
20   Blue                             active    
30   Yellow                           active    
999  VTP_Lab                          active    
1002 fddi-default                     act/unsup 
1003 trcrf-default                    act/unsup 
1004 fddinet-default                  act/unsup 
1005 trbrf-default                    act/unsup 
</code></pre>
</details>
 <details>
<summary>S3</summary>
<pre><code>
S3# show vlan brief  
!
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/2
10   Red                              active    
20   Blue                             active    
30   Yellow                           active    
999  VTP_Lab                          active    
1002 fddi-default                     act/unsup 
1003 trcrf-default                    act/unsup 
1004 fddinet-default                  act/unsup 
1005 trbrf-default                    act/unsup 
</code></pre>
</details>

8. Let's check the connectivity of PC A-C with the ping utility
<details>
<summary>PC A</summary>
<pre><code>
VPCS> show ip
NAME        : VPCS[1]
IP/MASK     : 192.168.10.1/24
GATEWAY     : 0.0.0.0
DNS         : 
MAC         : 00:50:79:66:68:04
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
!
VPCS> ping 192.168.10.2
!
84 bytes from 192.168.10.2 icmp_seq=1 ttl=64 time=0.722 ms
84 bytes from 192.168.10.2 icmp_seq=2 ttl=64 time=1.135 ms
84 bytes from 192.168.10.2 icmp_seq=3 ttl=64 time=1.121 ms
84 bytes from 192.168.10.2 icmp_seq=4 ttl=64 time=1.088 ms
84 bytes from 192.168.10.2 icmp_seq=5 ttl=64 time=1.197 ms
</code></pre>
</details>

 <details>
<summary>PC c</summary>
<pre><code>
VPCS> ping 192.168.10.1
84 bytes from 192.168.10.1 icmp_seq=1 ttl=64 time=0.822 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=64 time=1.187 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=64 time=1.427 ms
84 bytes from 192.168.10.1 icmp_seq=4 ttl=64 time=1.186 ms
84 bytes from 192.168.10.1 icmp_seq=5 ttl=64 time=1.085 ms
!
VPCS> show ip
!
NAME        : VPCS[1]
IP/MASK     : 192.168.10.2/24
GATEWAY     : 0.0.0.0
DNS         : 
MAC         : 00:50:79:66:68:06
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500
</code></pre>
</details>