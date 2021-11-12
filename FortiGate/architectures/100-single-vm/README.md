# Single FortiGate VM

Single VM is a basic setup good to start exploring capabilities of the next-generation firewall. It will allow you to protect your workloads running in Google Cloud as well as inspect the outbound traffic. As a single VM is subject to lower GCP Compute SLA (99.5%), it is very rarely used in production deployments.

## Design

![FGT Single VM details](https://lucid.app/publicSegments/view/4e56ef05-671c-47f3-a2cd-65cca6185f20/image.png)

A single FortiGate VM instance with 2 network interfaces in external and internal VPC. Although it is technically possible to deploy with just one NIC, Fortinet recommends using port1 as external interface and port2 as internal for convenience.

External NIC (port1) is connected to Internet using its public IP address and optionally can use protocol forwarding to utilize more external addresses.

Internal NIC (port2) is configured as target for a custom route added to internal VPC Network, which causes all outbound traffic to be routed via FortiGate appliance.

## Templates
### Deployment Manager

#### singlevm-no-template.yaml

This configuration file includes plain YAML declaration of a FortiGate instance and a log disk. It will not create any additional resources and is provided as a basis for anyone wishing to create their own templates. As it references VPC Networks and subnets, they need to be created before deploying the config file. Custom route and cloud firewall rules need to be added manually.

#### singlevm2.jinja
This file provides a highly flexible template for deploying a single FortiGate instance with all additional necessary resources and is meant to be included in customer Infrastructure-as-Code. singlevm2 is used in some other architectures provided in this repository.

Created resources:
- 1 FortiGate VM instance (BYOL or PAYG, depending on license.type property) with variable number of network interfaces (depending on networks property)
- 1 log disk
- external IP addresses - for each externalIP and additionalExternalIPs
- forwarding rules - for each additionalExternalIPs entry
- target instance - if at least one additionalExternalIPs entry exists
- cloud firewall rules - allowing all traffic to FGT VM
- custom route towards port2 of FGT VM

Required configuration:
- region
- list of networks to connect FGT VM to

(see [config.minimal.yaml](config-minimal.yaml) for an example)

### Terraform
