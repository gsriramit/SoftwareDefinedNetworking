## Network Controller

- The Microsoft Network Controller is the “brains” of the SDN Stack
- It is a highly available and scalable server role, and provides one application programming interface (API) that allows Network Controller to communicate with the network, and a second API that allows you to communicate with Network Controller
  - APIs that let you interact with the controller are the called the North-bound APIs and the ones that let the controller interact with the SDN components are called the South-bound APIs
  - The NC Northbound API provides you with the ability to gather network information from Network Controller and use it to monitor and configure the network.
    - The Network Controller Northbound API allows you to configure, monitor, troubleshoot, and deploy new devices on the network by using Windows PowerShell, the Representational State Transfer (REST) API, or a management application with a graphical user interface, such as System Center Virtual Machine Manager.
  - Network Controller communicates with network devices, services, and components by using the Southbound API.
    - With the Southbound API, Network Controller can discover network devices, detect service configurations, and gather all of the information you need about the network. In addition, the Southbound API gives Network Controller a pathway to send information to the network infrastructure, such as configuration changes that you have made.
  - Example to relate to the practical application of these APIs
    - Azure Network Security Group (NSG) rules applied through the portal/ARM template/PS are then passed to the NC by Azure Resource Manager through its North-bound APIs
    - The Network Controller in this case would pass the rules aka ACLs through its South-bound APIs. The network host agent is supposed to query these south-bound APIs to retrieve the new config settings
  - The deployment of NC Virtual Machines in the host machines also deployes the NC host agent in each of the hosts
  - NC Host Agent receives from NC the configuration settings and programs the same on the local host virtual switch. The same has been illustrated in the fig shown in the next section
  - NC Virtual Machines are connected to the Management Logical Network and use this network to pass on the settings to the Host Machines.The NC host agent and the SLB host agent are supposed to be interacting with the NC through mgmt network.The significance of the logical networks is detailed in a separate document. 

### Illustration of NC's interation with the Management Plane and Data Plane
![NC-API-Illustration](https://user-images.githubusercontent.com/13979783/214771268-5bc7d3da-a18a-4fce-b59f-659af99b2120.png)


### Network Controller Features
The following Network Controller features allow you to configure and manage virtual and physical network devices and services.

- Firewall Management
- Software Load Balancer Management
- Virtual Network Management
- RAS Gateway Management  

More information how NC configures and manages these software based networking components is available in the link provided in the next section


## References
1. https://learn.microsoft.com/en-us/previous-versions/windows/server/dn859239(v=ws.12)?redirectedfrom=MSDN
