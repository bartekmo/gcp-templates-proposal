# Peered Security Hub

## Introduction
GCP limitations related to deployment of multi-NIC instances make the [usual architecture](https://cloud.google.com/solutions/best-practices-vpc-design#multi-nic) for deploying firewalls very static and costly (a classic 3-tier application would require 8-core FGT instances). Peered Security Hub architecture provides flexibility of securing 25 segments using only one internal NIC.

## Design
Hub-and-spoke design puts firewalls in the hub VPC Network and connects all VPC Networks to be inspected for traffic using peering. Default route defined in the Hub is propagated to the spokes using exportCustomRoutes/importCustomRoutes properties set on peerings (hub exports, spoke imports), thus enforcing traffic flow between spoke VPCs and from spokes to the Internet to be routed via firewalls.

![Peered Security Hub diagram](https://lucid.app/publicSegments/view/cdc1dc90-2ab4-4488-841a-92e2795ea630/image.png)

Note that the Security Hub design focuses on the cloud architecture, while the Fortigate part is flexible - you can use any building block from single VM to FGCP A-P HA cluster with LB Sandwich.

## How to Deploy
You can turn any FortiGate deployment into a Peered Security Hub, but peering internal VPC Network with Spoke VPCs. Make sure you divide your resources properly into separate VPCs as only the traffic between separate Spoke VPCs will be inspected.

### FortiGate configuration
Peered Security Hub architecture requires FortiGates to be additionally configured with a static route towards all spokes (use supertnetting if possible) via port2. Without this settings all connections between Spoke VPCs will be dropped because of failed RPF checks.

Note that all spoke-to-spoke traffic will be arriving and leaving via port2, make sure your firewall policies are configured properly.

### Templates
- [Deployment Manager](deployment-manager/)
- [Terraform](terraform/)
