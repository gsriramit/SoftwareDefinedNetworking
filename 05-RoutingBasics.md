## SDN Routing Basics

### Network Virtualization Generic Routing Encapsulation (NVGRE)
- Hyper-V Network Virtualization supports Network Virtualization using Generic Routing Encapsulation (NVGRE) as the mechanism to virtualize IP addresses
- In NVGRE, the virtual machine's packet (with IP addresses from Consumer Area, i.e. CA) is encapsulated inside another packet. The header of this new, NVGRE-formatted packet has the appropriate source and destination provider area (PA) IP addresses. In addition, it has a 24-bit Virtual Subnet ID (VSID), which is stored in the GRE header of the new packet.  

The following figure shows a GRE-encapsulated packet. On the wire, NVGRE-encapsulated packets look like IP-over-Ethernet packets, except that the payload of the outer IP header is a GRE-encapsulated IP packet (including the Ethernet header)  
![image](https://user-images.githubusercontent.com/13979783/215036217-6ca69052-00d1-4ddb-98fd-7589ab5e01c4.png)

src:https://learn.microsoft.com/en-us/windows-hardware/drivers/network/network-virtualization-using-generic-routing-encapsulation--nvgre--task-offload  

#### Application of NVGRE
The L2 overlay virtual networks associated with the Consumer IP networks will only have network traffic originating from, or destined to VMs attached to the network. Packets on the virtual network will then be encapsulated with a header, dependent upon the network overlay technology chosen: VxLAN (default) or NVGRE, and placed on the wire as shown in Figure below. IP addresses associated with the Provider Address (PA) are on the outside of the encapsulation envelope and IP addresses associated with the Customer Address (CA)  are on the inside
![SLBEncapsulation](https://user-images.githubusercontent.com/13979783/215036246-883a6bc6-22f2-4e63-9bd1-5d8c4ddf20c4.jpg)  

### BGP Route Advertisement
- BGP peering and the route advertisement in the 03-Networks.md page. Peering between the SLB Mux and the internet facing routers is a mandatory prerequisite to make public load balancing work
- The edge router needs to know that a specific public VIP (assigned to all of the Mux instances and operated based on ECMP) is in fact an address that they need to receive traffic for
- It is with this VIP advertisement, the edge router can continue to advertise this to other BGP peers in the WAN. So when a client tries to reach the public VIP of the Load balancer, then the client's routers would find the SDDC edge router after the destined number of hops. 
- The edge routers on receiving the traffic would forward the same to one of the healthy Mux instances (again ECMP would make all the Mux instances valid destinations of the received traffic)

### Example Scenarios
#### S2S VPN Gateway Routing
![image](https://user-images.githubusercontent.com/13979783/215037618-f133d2e1-2914-47b7-8279-4929626c8ac1.png)

1. The source VM in the CA address space has an IP 10.1.1.2 and is in a physical host with IP 10.1.1.5 (in reality the PA address space would be different from CA)
2. The destination is in a different but connected network with a CA space of 10.2.0.9. The connection between these networks is through a SD S2S VPN Gateway in this case
3. The vNIC in the VM gets programmed with the routes of the connected network with the next hop as the private IP of the s2s gateway i.e., 10.1.2.1 residing on a host with Ip 10.1.1.7
4. The VFP (in the host switch) in each of the hosts is programmed by the NC host agent to perform the encapsulation & decapsulation of the packets based on the protocol chosen, NVGRE or VXLAN
5. The packets from the source VM now traverse the HNV PA network with the outer packet containing srcIP:10.1.1.5 and destIP: 10.1.1.7
6. VFP on the 10.1.1.7 physical host decapsulates the packet to read the IP pair of the inner packet, i.e., srcIP:10.1.1.2 and destIP: 10.2.0.9
7. The s2s VPN gateway should supposedly use the GRE VIP logical network to send the packets via the encrypted VPN tunnel (Note: The packets still traverse the public internet)
8. The host switch in the gateway host would have performed the encapsulation before sending the packets to the local network gateway on the connected network
