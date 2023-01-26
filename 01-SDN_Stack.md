## Microsoft's Software Defined Networking Stack

### Representation of the Stack


- The most important thing for cloud scale networking setup requiring flexibility and scale is a host-based SDN model
- This means that each entity in the network platform must be programmable. Specific configuration settings are applied using a centralized controller which is reachable through the conventional REST APIs
  - The method through which the configuration settings are passed can vary between CLIs, GUI and JSON payloads
- For instance, consider a simple access policy rule: using a host based model, policies are transformed into a specific set of instructions, pushed out by a centralized controller all the way down to the hosts
- In the diagram shown above,
  - The Management plane can be Azure Resource Manager if the networking setup is done for Azure public cloud or SCVMM (System Center Virtual Machine Manager) for environment setup done on-premises or can also be powershell scripts. In all these modes, the configuration settings (imperative or declarative) is passed on to the Control Plane component in the next step
  - The Control Plane component is Network Controller (NC) and it takes cares of the heavy lifting of the SDN deployments. This is referred to as the brain of SDN as a whole.
    - NC can be deployed to SDN stack as a Virtual Machine and multiple instances of the same are spread across the host machines in the cluster for high-availability reasons
    - The functionalities of NC will be detailed out in a separate document
 - As mentioned in point#1, SDN uses a host based model to implement the requested configuration and also the commands
   - The Virtual switch that is created in each of the host machines receives the configuration data from the NC, through the Network Host Agent. The Switch then programs the Virtual Filtering Platform (VFP). This will be detailed out in a separate document
   - The Network Host agent and the SLB host agent are programmed agents are deployed to each of the host machines through which the NC and the Software Load Balancer (SLB Mux) implement the SDN settings  
