---
title: "Hardware requirements"
linkTitle: "Hardware Requirements"
description: "Define the hardware requirements for your Cozystack use case."
weight: 5
---

Cozystack utilizes Talos Linux, a minimalistic Linux distribution designed solely to run Kubernetes.
Usually it means that it will **not** be possible to share a server with services other than Cozystack runs.
The good news is that whichever service you need, Cozystack will run it prefectly: securely, efficiently, and
in a fully containerized or virtualized environment.

Hardware requirements depend on your usage scenario.
Below are several common deployment options, review them to determine which setup fits best to your needs.

## Small lab

Here are the baseline requirements for running a tiny installation.

Recommended hardware configuration for each node:
```yaml
CPU: 4 cores
CPU Type: host
RAM: 16 GB
DISK1: 32 GB*
DISK2: 100 GB** (raw)
```
<small><i>
*Minimum: 32 GB, (Approximately 26 GB is used in a standard Cozystack setup). Talos install expects /dev/sda as system disk (virtio drive usually use /dev/vda)[see system disk install issues](https://github.com/cozystack/cozystack/issues/723#issuecomment-2762374751).

**Suggested: 100 GB. Disk path (usually /dev/sdb) will be defined on storage configuration. Do not affect Talos install. [Learn more about configuring Linstor StorageClass here.](https://cozystack.io/docs/getting-started/first-deployment/#configure-storage)
</i></small>

**Compute:**

- Three physical or virtual servers with amd64/x86_64 architecture, with at least 16 GB RAM and 4 CPU cores each.
- Virtualized servers need nested virtualization enabled and the CPU model set to `host` (without emulation).
- PXE installation requires an extra management instance connected to the same network, with any Linux system able to run a Docker container.
  It should also have `x86-64-v2` architecture, which most probably may be achieved by setting CPU model to `host` in case of a VM.

**Storage:**

Storage in a Cozystack cluster serves two primary purposes: one for the system and one for your workloads. Understanding the role of each ensures the stability and scalability of your environment.

- Primary Disk: Contains the Talos Linux operating system, essential system kernel modules and Cozystack system base pods, and logs, and base container images.

- Secondary Disk: Dedicated to workload data and can be increased based on workload requirements. Used for provisioning volumes via PersistentVolumeClaims (PVCs).


**Networking:**

- Machines must be allowed to use additional IPs, or an external load balancer must be available.
  Using additional IPs is disabled by default and must be enabled explicitly in most public clouds.
- Additional public IPs for ingress and virtual machines may be needed. Check if your public cloud provider supports floating IPs.


## Production Cluster

For a production environment, consider the following:

**Compute:**

- Having at least **three worker nodes** is mandatory for running highly available applications.
  If one of the three nodes becomes unavailable due to hardware failure or maintenance, you’ll be operating in a degraded state.
  While database clusters and replicated storage will continue functioning, starting new database instances or creating replicated volumes won’t be possible.
- Having separate servers for Kubernetes master nodes is highly recommended, although not required.
  It’s much easier to take a pure worker node offline for maintenance or upgrades, than if it also serves as a management node.

**Networking:**

- In a multi-datacenter setup, it’s ideal to have direct, dedicated optical links between datacenters.
- Servers must support out-of-band management (IPMI, iLO, iDRAC, etc.) to allow remote monitoring, recovery, and management.

## Distributed Cluster

You can build a [distributed cluster]({{% ref "/docs/operations/stretched/" %}}) with Cozystack.

**Networking:**

- Distributed cluster requires both fast and reliable network, and it **must** have low RTT (Round Trip Time), as
  Kubernetes is not designed to operate efficiently over high-latency networks.

  Datacenters in the same city typically have less than 1 ms latency, which is ideal.
  The *maximum acceptable* RTT is 10 ms.
  Running Kubernetes or replicated storage over a network with RTT above 20 ms is strongly discouraged.
  To measure actual RTT you can use the `ping` command.

- It's also recommended to have at least 2–3 nodes per datacenter in a distributed cluster.
  This ensures that the cluster would be able to survive one datacenter loss without major disruption.

- If it's hard to keep a single address space between datacenters, instead of using some external VPN,
  you can enable **KubeSpan**, a Talos Linux feature that creates a WireGuard-backed full-mesh VPN between nodes.

## Highly Available Applications

Achieving high availability adds to the basic production environment requirements.

**Networking:**

- It is recommended to have multiple 10 Gbps (or faster) network cards.
  You can separate storage and application traffic by assigning them to different network interfaces.

- Expect a significant amount of horizontal, inter-node traffic inside clusters.
  It is usually caused by multiple replicas of services and databases deployed across different nodes exchanging data.
  Also, virtual machines with live migration require replicated volumes, which further increases the amount of traffic.
  
