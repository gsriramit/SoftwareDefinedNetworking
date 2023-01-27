## Host Virtual Switch
Each of these physical compute machines in the SDN setup has a virtual switch created. The following sections describe the anatomy & capabilities of the virtual switch

### Capabilities of the vSwitch
- vSwitch (with the VFP in it) forms the Data plane of the SDN stack
- The switch has ports that the SDN components connect to. The ports are used to receive the traffic from the SDN components and send the same to the targeted logical network
  - The Network Controller (NC) connects to the management network through the management port in the switch
  - The customer virtual machines connect to the HNV PA network through the HNV port
  - The SLB Mux and s2s gateway instances connect to the Transit Network through the Transit port
  - The SLB Mux and s2s gateway instances also connect to the public & private VIP networks through the corresponding ports in the switch
- Mapping between the Consumer & Provider Addresses (CA <-> PA) are maintained in the virtual switch which handles the packet encapsulation. - Mapping between VIP and DIP of an SLB traffic is also maintained in the switch

### Virtual Filtering Platform (VFP)
- Policy/Data plane of the network
- The layers in a VFP provide core SDN functionalities that include ACLs, Metering, Virtual Network (mapping between address spaces), NAT for SLB
- VFP uses programmable rule/flow tables to perform per-packet actions. These are also called the Match Action Tables (MAT)
- Layers of the switch
  - The metering layer is used to billing and monitoring of the packets that are processed by the switch. 
  - The stateful ACL layer consists of the rules that help the switch in processing or dropping a packet. The rules are programmed on the switch when the NC receives the NSG rules from the management plane
  - NAT layer or the corresponding match action table maintains the mapping between public and private VIP(s) and the corresponding DIPs. The SLB Mux should be able to program the NAT data on the switch through the SLB host agent. It does the same when it advertises the routes for the traffic received on the VIP (edge router or TOR)
  - The VNET layer maintains data for the CA<->DA mapping that lets the switch perform the encapsulation and decapsulation based on NVGRE when the packets traverse between segments of a VM network through the HNV logical network   
From the perspective of public cloud (Azure) infrastructure, these would map to the Network Watcher as a monitoring service and the other associated services including network packet capture, NSG flow logs and connection troubleshoot.  


