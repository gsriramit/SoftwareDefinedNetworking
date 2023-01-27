## SDN Routing Basics

### Network Virtualization Generic Routing Encapsulation (NVGRE)
- Hyper-V Network Virtualization supports Network Virtualization using Generic Routing Encapsulation (NVGRE) as the mechanism to virtualize IP addresses
- In NVGRE, the virtual machine's packet (with IP addresses from Consumer Area, i.e. CA) is encapsulated inside another packet. The header of this new, NVGRE-formatted packet has the appropriate source and destination provider area (PA) IP addresses. In addition, it has a 24-bit Virtual Subnet ID (VSID), which is stored in the GRE header of the new packet.  

The following figure shows a GRE-encapsulated packet. On the wire, NVGRE-encapsulated packets look like IP-over-Ethernet packets, except that the payload of the outer IP header is a GRE-encapsulated IP packet (including the Ethernet header)  
**Image**
src:https://learn.microsoft.com/en-us/windows-hardware/drivers/network/network-virtualization-using-generic-routing-encapsulation--nvgre--task-offload  

#### Application of NVGRE
The L2 overlay virtual networks associated with the Consumer IP networks will only have network traffic originating from, or destined to VMs attached to the network. Packets on the virtual network will then be encapsulated with a header, dependent upon the network overlay technology chosen: VxLAN (default) or NVGRE, and placed on the wire as shown in Figure below. IP addresses associated with the Provider Address (PA) are on the outside of the encapsulation envelope and IP addresses associated with the Customer Address (CA)  are on the inside