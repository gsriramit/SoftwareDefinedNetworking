## Logical & Virtual Machine Networks

### Logical networks

#### Management and HNV Provider
- All physical compute hosts must access the management logical network and the HNV Provider logical network
- For IP address planning purposes, each physical compute host must have at least one IP address assigned from the management logical network
- The Network Controller requires a reserved IP address from this network to serve as the Representational State Transfer (REST) IP address
- The HNV Provider network serves as the underlying physical network for East/West (internal-internal) tenant traffic, North/South (external-internal) tenant traffic, and to exchange BGP peering information with the physical network

#### Gateways and the Software Load Balancer (SLB)
Additional logical networks are required to use gateways and the SLB. It is required to obtain the correct IP prefixes, VLAN IDs, and gateway IP addresses for these networks  

**Logical network Description**  
| Logical network  | Description |
| ------------- | ------------- |
| Public VIP logical network  | The Public virtual IP (VIP) logical network must use IP subnet prefixes that are routable outside of the cloud environment (typically internet routable). These are the front-end IP addresses that external clients use to access resources in the virtual networks, including the front-end VIP for the site-to-site gateway or the SLB. You don’t need to assign a VLAN to this network.  |
| Private VIP logical network (Private Transit) | The Private VIP logical network is not required to be routable outside of the cloud. This is because only VIPs that can be accessed from internal cloud clients use it, such as private services. You don’t need to assign a VLAN to this network.  |
|GRE VIP logical network | The Generic Routing Encapsulation (GRE) VIP network is a subnet that exists solely to define VIPs. The VIPs are assigned to gateway VMs running on your SDN fabric for a site-to-site (S2S) GRE connection type. You don't need to preconfigure this network in your physical switches or router, or assign a VLAN to it.|

### Routing infrastructure 

#### SLB and VPN Gateway Route advertisement 
- Routing information (such as next-hop) for the VIP subnets is advertised by the SLB/MUX and Remote Access Server (RAS) gateways into the physical network using internal BGP peering
- The VIP logical networks do not have a VLAN assigned and they are not preconfigured in the Layer-2 switch (such as the Top-of-Rack switch)
- A BGP peering needs to be created on the router or the TOR (top of rack switch) that your SDN infrastructure uses to receive routes for the VIP logical networks, advertised by the SLB/MUXes and RAS Gateways
- BGP peering only needs to occur one way (from the SLB/MUX or RAS Gateway to the external BGP peer)
- As previously stated, the IP subnet prefix for the VIP logical networks do need to be routable from the physical network to the external BGP peer
- The BGP router peer in the network infrastructure must be configured to use its own Autonomous System Numbers (ASN) and allow peering from an ASN that is assigned to the SDN components (SLB/MUX and RAS Gateways)
- The following information from the physical router or the TOR are required to complete the peering
  - Router ASN
  - Router IP address
- After the peering is done, the next-hop for the VIP in the routing table will be the SLB mux and hence the router will be able to send the packets to the Mux instance
- **Note:** The SLB Mux and the Internal NIC of the physical routers or TORs are connected to the Transit Network. It is through this transit network that the Muxs reach the public/private routers for BGP peering and route advertisement

**Illustration of the BGP Route advertisement from Mux to the TOR Switch**
![RouteAdvertisementFromMuxToTOR](https://user-images.githubusercontent.com/13979783/214899215-cd74456d-417a-4d73-9be5-f8d7a2cb1131.png)


**Illustration of the BGP ASNs for peering**
![BGPPeeringASN](https://user-images.githubusercontent.com/13979783/214899262-8307693a-dee4-4e48-b1f8-3c7daa6206f1.png)




#### HNV Logical Network for Tenant Traffic
The HNV logical network, also know as the Provider Network is used by the Tenant Virtual Machines to send traffic between each other and also to the machines in the connected networks. The HNV overlay network receives and carries the encapsulated packet (following the NVGRE) where the outer packet contains the IP and MAC of the host machines that the packet egresses from. NVGRE has been detailed in a separate document. The following are examples of traffic that use the HNV Provider Network 
- Traffic between VMs that are in different subnets of a consumer network and could also be spread among multiple host machines
- Traffic from the SLB Mux to the DIP of the backend tenant VMs - The SLB mux receives the traffic from the edge routers and then uses the HNV provider network to send the packets to the identified backend tenant VM

### Virtual Machine Network 
Tenant workloads are deployed within dedicated Virtual Networks and Subnets that are allocated to each of the tenants. Following are the characteristics of these Virtual Networks
-	One or more networks per tenant that are spread across multiple host machines. In the Azure cloud, the Virtual Networks span Availability zones which means that the underlying physical machines can be spread across multiple data centres in different Availability zones 
- SDN makes it possible for multiple tenants to use the same VM network CIDR. This implies that these consumer networks would not have to have different IP spaces and can exist in the same set of host machines
  - An address space has to be assigned each of these consumer networks and this would be space from which the VMs would be assigned IPs. These would be addresses that would be seen by the receiving appliance, either in the same network or a connected network
- The Virtual Machines in the Consumer network are connected to the Host Virtual Switch using the Consumer Address space (CA) vNIC. The switch in turn connects to the HNV PA logical network


## References
1. https://learn.microsoft.com/en-us/azure-stack/hci/concepts/plan-software-defined-networking-infrastructure

