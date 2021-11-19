# Deploying FortiGate A-P HA cluster in load balancer sandwich with Deployment Manager
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
