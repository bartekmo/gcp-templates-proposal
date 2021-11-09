# Fortigate Building Blocks
This directory contains templates for the following Fortigate in GCP deployment building blocks:

### [Single VM](architectures/singlevm.md)
This single FortiGate VM will process all the traffic and as such become a single point of failure during operations as well as upgrades. This block can also be used in an architecture with multiple regions where a FortiGate is deployed in each region.
Single instance is subject to 99.5% GCP compute SLA.

### [Active-Passive HA Cluster with Fabric Connector Failover](architectures/fgcp-ha-ap-sdn.md)
This design will deploy 2 FortiGate VMs in 2 zones and preconfigure an Active/Passive cluster using unicast FGCP HA protocol. This protocol will synchronize the configuration. On failover the passive FortiGate takes control and will issue api calls to GCP API to shift the External IP and update the route(s) to itself. Shifting the public IP and gateway IPs of the routes will take some time and depends on the number of routes. This design supports only a single External IP.
This design is subject to 99.99% GCP Compute SLA.

### [Active-Passive HA in Load Balancer Sandwich](architectures/fgcp-ha-ap-elbilb.md)
This design will deploy 2 FortiGate VMs in 2 zones, preconfigure an Active/Passive cluster using unicast FGCP HA protocol, and place them between a pair of external and internal load balancers. On failover load balancers will detect failure of the primary instance using active probes on port 8008 and will switch traffic to the secondary instance. The failover time is noticeably faster than using Fabric Connector and is configurable in Health Check settings. Routing via GCP Internal Load Balancer does not support tag-based routes. This design supports multiple public IPs.
This design is subject to 99.99% GCP Compute SLA.

## How to Deploy
[Deployment Manager](howto-dm.md)
[Terraform](howto-tf.md)
