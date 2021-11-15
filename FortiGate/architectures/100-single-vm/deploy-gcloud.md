# Single FortiGate VM
## How to deploy using gcloud

### Prerequisites
Before you deploy a FortiGate instance, you need to create
- 2 VPC Networks (external and internal)
- with 1 subnet in each

Note: subnets must be in the same region where you plan to deploy your FortiGate instance.

You also must know the following values:
1. URLs of external VPC and subnet
1. URLs of internal VPC and subnet
1. URL of image you're going to deploy (see [below](#how-to-find-image) for more details)
1. zone where you want to create your FortiGate VM

to not get lost, it's a good idea to save the values in shell variables. See the example below:

```
VPC_EXT_NAME=vpc-external
SB_EXT_NAME=sb-external-euwest1
VPC_INT_NAME=vpc-internal
SB_INT_NAME=sb-internal-euwest1

# find newest BYOL image
FGT_IMG_URL=$(gcloud compute images list --project fortigcp-project-001 --filter="name ~ fortinet-fgt- AND status:READY" --format="get(selfLink)" | sort -r | head -1)

ZONE=europe-west1-c
```

### Deploy
The commands below will deploy a FortiGate instance with 2 NICs and a 100GB logdisk

```
# create log disk
gcloud compute disks create my-fgt-logdisk --size=100 --type=pd-ssd --zone=$ZONE

# create FortiGate instance
gcloud compute instances create my-fortigate --zone=$ZONE \
  --machine-type=e2-standard-2 \
  --image=$FGT_IMG_URL --can-ip-forward \
  --network-interface="network=$VPC_EXT_NAME,subnet=$SB_EXT_NAME" \
  --network-interface="network=$VPC_INT_NAME,subnet=$SB_INT_NAME,no-address"
  --disk="auto-delete=yes,boot=no,device-name=my-fgt-logdisk,mode=rw,name=my-fgt-logdisk"
```

### [optional] Additional external IP addresses

[Protocol Forwarding]() is a GCP feature, which allows attaching additional external addresses to a VM. Before creating and attaching an address, a targetInstance resource must be created:

```
gcloud compute target-instance create my-fgt-target
```


### How to find the image
You can either deploy one of the official images published by Fortinet or create your own image with disk image downloaded from [support.fortinet.com](https://support.fortinet.com). We recommend you use official images unless you need to deploy a custom image.

Fortinet publishes official images in *fortigcp-project-001* project. This is a special public project and every GCP user can list images available there using command

`gcloud compute images list --project fortigcp-project-001`

Official images for FortiGate have names starting with *fortinet-fgt-[VERSION]* (BYOL images) or *fortinet-fgtondemand-[VERSION]*. It is your responsibility to select the correct image if deploying using gcloud or templates (Deployment Manager templates in this repository automatically find image name based on version and licenses properties). Use filter and format options of gcloud command to get a clean list, eg.
`gcloud compute images list --project fortigcp-project-001 --filter="name ~ fortinet-fgtondemand AND status:READY" --format="get(selfLink)"`

will get you a list of image URLs for FortiGate PAYG, and

`FGT_IMG=$(gcloud compute images list --project fortigcp-project-001 --filter="name ~ fortinet-fgt- AND status:READY" --format="get(selfLink)" | sort -r | head -1)`

will save the URL of the newest BYOL image into FGT_IMG variable
