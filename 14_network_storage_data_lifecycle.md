# Elastic File System (EFS) Architecture

- Provide shared network file storage
- Implementation of NFSv4
- Can be mounted in Linux, officially supported in Linux
- Shared between many EC2 instances
- Private service, via mount targets inside a VPC
- Can be accessed from hybrid networks - VPN or direct connect
- Performance modes
  - General purpose: Latency sensitive
  - Max I/O: High throughput, higher latency
- Throughput modes
  - Bursting: Burse, scales with the size of the data ~ gp2
  - Provisioned: ~ io1
- Storage classes
  - Standard
  - Infrequent Access (IA)
- Lifecycle policies can be used to move data between storage classes

# Demo Implementing EFS

## Create EFS

- Choose performance modes
- Choose throughput modes
- Setup storage lifecycle policy
- Enable AWS encryptions (at rest/in transit)
- Choose VPC
- Choose AZ, subnets
- Choose security groups for each subnet

## Use/mount EFS

- Connect to EC2 instance
- Create a new folder
- Install (yum/apt) amazon-efs-utils
- Edit /etc/fstab
- Mount the folder
- Check df -k to see attached efs volumes

## Delete EFS

- Delete mount targets
- Delete file system

# AWS Backup <= **Rewatch**

- Managed data backup service
- Consolidate management in one place, across accounts & regions
- Support many AWS products
  + Computer
  + Block storage
  + File storage
  + Databases
  + Object storage
- Backup plans
  + frequent, window, lifecycle, vault, region copy
- Resources
- Vaults
- Vault Lock
- On-Demand
- Point in time recovery
