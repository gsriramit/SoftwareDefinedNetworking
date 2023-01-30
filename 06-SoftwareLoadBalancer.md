## Software Load Balancer

**Note:** The official MS documentation for SLB has a lot of the important details captured. I am not reinventing the wheel here again. I have used a few sections from the documentation and other references for the purposes of elaborated documentation.   

### Routing of Traffic 
Reference: https://learn.microsoft.com/en-us/azure-stack/hci/concepts/software-load-balancer#how-software-load-balancer-works 
Initial setup & dynamic updates
- NC has set up a public IP (20.20.20.100) on all the mux instances spread across multiple physical hosts
- The SLB configuration has been fed to the NC through the management plane entity, ARM template
  - The configuration is then programmed on the mux instances through the SLB host agent that is deployed on every host in the SDN cluster
  - The NC also programs the virtual switch with all the necessary information including ACLs, NAT, Metering and Virtual Network. In this case, the Virtual Network matching table would contain the mapping information between the PA and the CA. This will be required in step#4 of the traffic flow explained below
  - The virtual switch would supposedly be updated by the mux instances for every *unique initial packet* processed. The NAT match table should now contain information sufficient for the vSwitch to handle the response back to the client, without relying on the mux
- Mux instances have completed the eBGP peering with the internet facing edge routers. With this peering the routing table of the routers would have been updated with 3 Transit IP addresses of the mux instances for the traffic received on the public VIP address. The same has been shown in the diagram
- ECMP would be used by the edge routers when trying to identify a mux to send the received traffic to  

### Flow

1. The client sends an HTTP request to the VIP (20.20.20.100). If there is a FQDN mapped to this publicIP address, then the client would be able to reach this IP after the DNS resolution completes
2. The physical network has multiple paths available to reach the VIP located on any MUX. Each router along the way uses ECMP to pick the next segment of the path until the request arrives at a MUX.
3. The MUX that receives the request (192.168.30.53 on HOst#3) checks configured policies, and sees that there are three DIPs available, 10.10.10.11, 10.10.10.12 and 10.10.10.13 on a virtual network to handle the request to the VIP 20.20.20.100. The SLB uses either the 5 tuple, 3 tuple or the 2 tuple algorithm to determine the tenant VM that would handle the request
   - The MUX selects DIP 10.10.10.12 and encapsulates the packets using VXLAN so that it can send it to the host containing the DIP using the host's physical network address.
4. The host receives the encapsulated packet and inspects it. It removes the encapsulation and rewrites the packet so that the destination is now DIP 10.10.10.12 instead of the VIP, and then sends the traffic to DIP VM.
5. The request reaches the targeted virtual machines in the Customer network. The server generates a response and sends it to the client, using its own IP address as the source.
6. The host intercepts the outgoing packet in the virtual switch which remembers that the client, now the destination, made the original request to the VIP. The host rewrites the source of the packet to be the VIP so that the client does not see the DIP address.
7. The host forwards the packet directly to the default gateway for the physical network which uses its standard routing table to forward the packet on to the client, which eventually receives the response. The host would supposedly use the transit network connection from its physical NIC to send the packet to the edge router or the gateway

Note:  
1. The Mux and the physical hosts use the HNV provider network for the MUX to send the received request to the host where the targeted DIP VM is located  
2. The host switch should use the HNV provider interface to send the traffic to the tenant VM

### Load balancing internal datacenter traffic

- When load balancing network traffic internal to the datacenter, such as between tenant resources that are running on different servers and are members of the same virtual network, the Hyper-V virtual switch to which the VMs are connected performs NAT.
- With internal traffic load balancing, the first request is sent to and processed by the MUX, which selects the appropriate DIP, and then routes the traffic to the DIP. From that point forward, the established traffic flow bypasses the MUX and goes directly from VM to VM

### DSR
When tenant VMs respond and send outbound network traffic back to the internet or remote tenant locations, because the NAT is performed by the Hyper-V host, the traffic bypasses the MUX and goes directly to the edge router from the Hyper-V host. This MUX bypass process is called Direct Server Return (DSR).

### HA ports & Floating IP
TBD

### Outbound NAT settings