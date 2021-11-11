# Active-Passive HA FortiGate cluster in LB Sandwich
This template deploys 2 FortiGate instances in an Active-Passive HA cluster between two load balancers ("load balancer sandwich" pattern). LB Sandwich design enables use of multiple public IPs and provides faster, configurable failover times when compared to SDN-connector based. Due to Google Cloud Load Balancer limitations only UDP and TCP traffic is supported.

HA multi-zone deployments provide 99.99% Compute Engine SLA vs. 99.5-99.9% for single instances. See [Google Compute Engine SLA](https://cloud.google.com/compute/sla) for details.

Deployment Manager Template file: [modules/fgcp-ha-ap-elbilb.jinja](../modules/deployment-manager/fgcp-ha-ap-elbilb.jinja)
Deployment Manager Schema file: [modules/fgcp-ha-ap-elbilb.jinja.schema](../modules/deployment-manager/fgcp-ha-ap-elbilb.jinja.schema)
Terraform module: [modules/](../modules/)

## Overview
FGCP protocol natively does not work in L3 overlay networks. For cloud deployments it must be configured to use unicast communication, which slightly limits its functionality (only Active-Passive between 2 peers is possible) and enforces use of dedicated management interface. In this template port3 is used as heartbeat and FGCP sync interface and port4 is used as dedicated management interface.

As cloud networks do not allow any network mechanisms below IP layer (e.g. gratuitous arp) usually used in HA scenarios, this template adds a pair of load balancers and configures health probes to detect currently active instance. Passive peer will be detected as unhealthy by the load balancers and will not receive any traffic. External load balancer is configured to forward all UDP and all TCP ports, while internal load balancer is used as a next hop in the custom route (note that despite using a single port in the ILB, it will route all UDP, TCP and ICMP traffic).

## Active-Passive HA Design Options Comparison
Read [here](../README.md#choosing-ha-architecture) more about differences between different HA designs in Google Cloud.

## Diagram
As unicast FGCP clustering of FortiGate instances requires dedicated heartbeat and management NICs, 2 additional VPC Networks need to be created (or indicated in configuration file). This design features 4 separate VPCs for external, internal, heartbeat and management NICs. Both instances are deployed in separate zones indicated in **zones** property to enable GCP 99.99% SLA.

Additional resources deployed include:
- default route for the internal VPC Network pointing to the internal load balancer rule
- external IPs - by default one floating IP for incoming traffic from Internet and 2 management IPs, to add more external IPs simply list them in the **publicIPs** property
- Cloud Router/Cloud NAT service is used to allow outbound traffic from port1 of secondary FortiGate instance (e.g. for license activation)

![ELBILB Sandwich diagram](https://app.lucidchart.com/publicSegments/view/b1ee079a-3c64-4e75-acb7-a42e3b6f8982/image.png)

## Deployed Resources
- 2 FortiGate VM instances with 4 NICs each
- 2 VPC Networks: heartbeat and management (unless provided)
- External Load Balancer
    - External addresses
    - Target pool
    - Legacy HTTP Health Check
    - 2 Forwarding Rules for each IP (UDP and TCP)
- Internal Load Balancer
    - 2 unmanaged Instance Groups (one in each zone)
    - Backend Service
    - Internal Forwarding Rule (using ephemeral internal IP)
    - HTTP Health Check
    - route(s) via Forwarding Rule
- Cloud NAT

## Prerequisites and Requirements
You MUST create the external and protected VPC networks and subnets before using this template. External and protected subnets MUST be in the same region where VMs are deployed.

All VPC Networks already created before deployment and provided to the template using `networks.*.vpc` and `networks.*.subnet` properties, SHOULD have first 2 IP addresses available for FortiGate use. Addresses are assigned statically and it's the responsibility of administrator to make sure they do not overlap.

## How to deploy
See example deployment templates for different platforms:
- [Terraform](examples-terraform)
- [Deployment Manager](examples-dm)

For instructions on how to use Terraform/Deployment-Manager check:
- [Terraform](../../../howto-tf.md)
- [Deployment Manager](../../../howto-dm.md)

## See also
[Other FortiGate designs](../README.md)
