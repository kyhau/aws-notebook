# Elastic Container Service for Kubernetes (EKS) vs. Elastic Container Service (ECS)

- Flow for each incoming request
   - EKS
     1. The client sends a request to ELB.
     2. ELB distributes the request to one of the **nodes** (aka. EC2 instances).
     3. A **proxy** running on the **node** is forwarding the request to one of the **pods** providing the service.
   - ECS
     1. The client sends a request to the ELB.
     2. ELB forwards request to one of the **tasks** providing the service.

- Load Balancing
   - EKS
      - EKS supports integration with **Network Load Balancer** and **Classic Load Balancer**.
      - The proxy running on each node is distributing requests randomly or based on the round robin algorithm among
        all pods running in the cluster. Doing so increases the network traffic between EC2 instances and between AZs
        which consumes network capacity and adds latency.
   - ECS
      - ECS supports integration with **Classic Load Balancer**, **Application Load Balancer** and 
        **Network Load Balancer**.
      - The tight integration between ECS and ALB does not require a third routing step and is, therefore, more
        efficient.

- VPC and ENI
   - An EC2 instance can attach one or more ENIs. 
    [The number of ENIs per EC2 instance is limited from 2 to 15 depending on the instance type](
       https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html#AvailableIpPerENI). 
   - EKS
      - Multiple **pods** share the same ENI.
      - Multiple private IP addresses are assigned to each ENI. 
      - EKS assigns each **pod** (a group of containers) a private IP address. 
      - However, some **pods** are sharing network interfaces with each other.
      - As EKS is sharing ENIs between pods; you can place up to **750 pods** per instance. 
      - You are NOT able to restrict traffic with a **security group** per **pod**, as the ENI and therefore the
       security group is shared with multiple **pods**.
   - ECS
      - Each **task** (a group of containers) on an EC2 instance, is assigned to a separate ENI.
      - You can place up to **15 tasks** per instance with ECS.


References:
1. https://docs.aws.amazon.com/eks/latest/userguide/load-balancing.html
2. https://cloudonaut.io/eks-vs-ecs-orchestrating-containers-on-aws/
