# Host-based Security

Table of Contents

- [AWS Host/Hypervisor Security (disk/memory)](#aws-hosthypervisor-security-diskmemory)
- [Host Proxy Servers  (aka. Instance Proxy Servers)](#host-proxy-servers--aka-instance-proxy-servers)
- [Host-based IDS/IPS (Intrusion Detection/Prevention System)](#host-based-idsips-intrusion-detectionprevention-system)
- [SSM (Systems Manager)](#systems-manager-ssm)
- [Packet Capture on EC2](#packet-capture-on-ec2)


## AWS Host/Hypervisor Security (disk/memory)

Prevent data leakage between clients within their shared environment.

1. Isolation
   1. Instances are always isolated from other customer instances.
   2. Instances have **no direct access to hardware**, and must go via the hypervisor and firwall system**.
   3. Use **Dedicated Host** instead if you need the instances run on a physical server that is dedicated for your uses.
2. Memory
   1. Host memory is allocated to an instance **scrubbed** (i.e. zero'd out).
   2. When un-allocating memory, the **hyervisor** zeros it out *before* returning it to the pool.
3. Disk
   1. EBS volumes are provided to instance in a **zero'd** state; i.e. this zeroing occurs *immediately before use*.
   2. If you have specific deletion requirements, you need to do this *before terminating* the instance/deleting the
      volume.


## Host Proxy Servers  (aka. Instance Proxy Servers)

## Host-based IDS/IPS (Intrusion Detection/Prevention System)

## Systems Manager (SSM)

## Packet Capture on EC2

