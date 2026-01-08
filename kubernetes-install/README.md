# Install local Kubernetes Cluster 

## Environment:
* Ubuntu 22.04.5 LTS
* libvirt 8.0.0
* QEMU 6.2.0

## Prerequisites:
* Fully provisioned and updated hosts
* Linux host root access
* Internet access

### Assumptions and preamble

While this approach is perfectly valid for cloud deployments (e.g. onto EC2 instances in AWS), it's also one that's been thoroughly tested and refined within a local libvirt environment; thus, it's assumed that's how you'll try it first, with the expecation it'll be easier to succeed again, elsewhere, sooner.

An even faster way forward is to utilize [this Bash script](provision-k8s-nodes-ubuntu.sh)
for all installation operations, up to the very initial Control Plane and Worker node configuration, on each target host.

This micro build book draws primarily from the commands and operations in that script, then augments guidance by sharing next steps for completing the installation, such that the final result is a brand new and fully operational Kubernetes cluster.