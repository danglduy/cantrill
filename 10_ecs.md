# Introduction to containers

- Virtualization
  - Multiple OS running on a same machine via virtualization hypervisor
  - Each OS takes a fixed amount of resources (RAM and disk)
- Containerization
  - Containers run on Container engine, each in a process
  - Each container does not take a fixed amount of resources, all managed by the Container engine
  - Does not take much resources => More containers can be run than virtual machines
- Container Image
  - Stacked layers images, defined by Dockerfile
  - Each line of Dockerfile creates a new layer of image, contains differential changes with the previous layer
  - When running a container, a new layer is added called "R/W layer", allowing the container to see the file system
  - Can be used to launch multiple containers, each is independent
- Container registry
  - To be used to store container images
  - Can be public/private
  - User upload container images to registry, then others pull the images and launch containers on container host

# ECS - Concepts

- Container definition
  - Define image & port
- Task definition:
  - Define number of containers, different containers for different applications (web, app, db containers...) => Wrapper of a whole application
  - Define CPU, memory specifications
  - Define Network specifications
  - Compatibility: EC2 mode or Fargate
  - Task role: The IAM role for the task to assume to interact with AWS resources
  - Doesn't define scalability
- Service definition
  - Define scalability
  - Number of tasks (copies)
  - Load balancer
  - HA
  - Auto scaling
  - ...

# ECS - Cluster

ECS management components manage where to run ECS containers

- EC2 mode
  - An ECS cluster is created within a VPC
  - ECS containers run inside EC2 instances, which are controlled by Auto Scaling Group (ASG)
  - Owner is responsible for paying & managing EC2 instances whether or not the ECS containers are running or not
- Fargate mode
  - ECS containers are run on Fargate infrastructure platform
  - Then the shared Fargate platform inject the ECS containers into the VPC
- EC2 vs ECS(EC2) vs Fargate
  - Container => ECS
  - Large workload, price conscious => EC2 mode
  - Large workload, overhead conscious => Fargate
  - Small / Burst workloads => Fargate
  - Batch / Periodic workloads => Fargate
  
# Elastic Container Registry (ECR)

- Container image regsitry
- Like Dockerhub
- Each AWS account has a public and private regsitry
- Each registry can have many repositories
- Each repository can contain many images
- Images can have several tags
- Public = public R/O... R/W requires permissions
- Private = permissions require for any R/O // R/W
- Integrated with IAM
- Image scanning, basic and enahcned (inspector)
- nr real-time Metrics => CW (auth, push, pull)
- API actions = Cloudtrail
- Events => EventBridge
- Replication... Cross-region and cross-account

# Kubernetes <= Rewatch

# EKS <= Rewatch

