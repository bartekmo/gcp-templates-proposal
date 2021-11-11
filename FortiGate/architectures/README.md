# Deployment templates for FortiGate in Google Cloud

In this directory you will find templates and reference architectures for running a FortiGate Next-Generation Firewall VM in Google Cloud (GCE).

## Common Architectures

### [Active-Passive HA Cluster with SDN Connector Failover](ha-active-passive-sdn/)
<p align="center">
<img width="500px" src="https://lucid.app/publicSegments/view/09e00569-7fab-4e9f-92f1-de8fd9148623/image.png" alt="FortiGate A-P HA with SDN Connector">
</p>

This design deploys 2 FortiGate VMs in 2 zones and preconfigure an Active/Passive cluster using unicast FGCP HA protocol. FGCP synchronizes the configuration and session table. One (directly) or more (using Protocol Forwarding) External IP addresses can be assigned to the cluster. On failover the newly active FortiGate takes control and issues API calls to GCP Compute API to shift the External IPs and update the route(s) to itself. Shifting the EIPs and routes takes about 30 seconds, but preserves existing connections.

Multiple IPs are available in this design with FortiGate version 7.0.2 and later.
This design is subject to 99.99% GCP Compute SLA.

### [Peered Security Services Hub](peered-security-hub/)
/upcoming/

## Other Architectures

### [Single VM](single-vm/)
This single FortiGate VM will process all the traffic and as such become a single point of failure during operations as well as upgrades. This block can also be used in an architecture with multiple regions where a FortiGate is deployed in each region.
Single instance is subject to 99.5% GCP compute SLA.

/upcoming/

### [Active-Passive HA in Load Balancer Sandwich](ha-active-passive-lb-sandwich/)
This design will deploy 2 FortiGate VMs in 2 zones, preconfigure an Active/Passive cluster using unicast FGCP HA protocol, and place them between a pair of external and internal load balancers. On failover load balancers will detect failure of the primary instance using active probes on port 8008 and will switch traffic to the secondary instance. The failover time is noticeably faster than using Fabric Connector and is configurable in Health Check settings. This design supports multiple public IPs.
This design is subject to 99.98% GCP Compute SLA.

/upcoming/

### [IDS with Packet Mirroring](ids-packet-mirroring/)
/upcoming/

### [Network Connectivity Center](network-connectivity-center/)
/upcoming/

### [Active-Active HA](ha-active-active/)
/upcoming/

## Choosing HA Architecture
Fortinet recommends two base building blocks when designing GCP network with an Active-Passive FortiGate cluster:
1. [A-P HA with SDN Connector](architectures/ha-active-passive-sdn/)
2. [A-P HA in LB Sandwich](architectures/ha-active-passive-lb-sandwich/)

Make sure you understand differences between them and choose your architecture properly. Also remember that templates and designs provided here are the base building blocks and you can modify or mix them to match your individual use case.

| Feature | with SDN Connector | in LB Sandwich |
| --------|--------------------|----------------|
| Failover time | 30-40 secs | ~10 secs |
| Protocols supported | UDP, TCP, ICMP, ESP, AH, SCTP | UDP, TCP |
| Max. external addresses | hundreds* (ver. 7.0.2+) / 1 (ver. 7.0.1 and earlier) | hundreds* |
| Tag-based internal GCP routes| supported | supported |
| Preserves connections during failover | yes | no |
| SDN Connector privileges | read-write | none |

\* - subject to external forwarding rules quota per GCP project and set of forwarded protocols

## How to Deploy...
- [...using Deployment Manager](../../howto-dm.md)
- [...using Terraform](../../howto-tf.md)
