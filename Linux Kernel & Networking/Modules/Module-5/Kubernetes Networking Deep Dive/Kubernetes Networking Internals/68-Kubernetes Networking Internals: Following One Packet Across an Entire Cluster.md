Most Kubernetes developers know:

```yaml
kind: Pod
```

They deploy it:

```bash
kubectl apply -f pod.yaml
```

A few seconds later:

```
STATUS

↓

Running
```

Everything works.

But very few understand what actually happens when one Pod sends a packet to another Pod on a different machine.

For example:

```
Pod A

(Node 1)

↓

Pod B

(Node 2)
```

Questions:

- How does Pod A know where Pod B is?
- How does the packet leave Node 1?
- Who forwards it?
- Does NAT happen?
- Does kube-proxy participate?
- Does CNI participate?
- Where do Netfilter and conntrack come in?
- How do eBPF-based clusters differ?

This chapter combines **everything you've learned so far** into one complete networking journey.

After this chapter, you'll understand Kubernetes networking from the application's socket all the way to the destination Pod.

---

# 👑 Linux Kernel & Networking Mastery

# Module 11 — Kubernetes Networking Deep Dive

# Chapter 68 — Kubernetes Networking Internals: Following One Packet Across an Entire Cluster

> **Learning Objectives**
>
> By the end of this chapter, you'll understand:
>
> - Kubernetes Networking Model
> - Pod Network Namespace
> - Pod veth Pair
> - CNI
> - Linux Bridge
> - Routing
> - Overlay Networks
> - kube-proxy
> - Netfilter
> - conntrack
> - Service Routing
> - Complete End-to-End Packet Journey

---

# 📖 Kubernetes Networking Philosophy

Kubernetes has one simple networking goal:

> Every Pod should be able to communicate with every other Pod.

No matter:

- Same Node
- Different Node
- Different Rack
- Different Availability Zone

Applications should not care.

---

# Kubernetes Networking Rules

Kubernetes defines four important rules.

✔ Every Pod has its own IP.

✔ Pods communicate directly.

✔ Nodes can reach Pods.

✔ Applications never need NAT between Pods.

Notice:

Kubernetes defines the rules.

CNI plugins implement them.

---

# Example Cluster

```
             Kubernetes Cluster

     +-----------------------------+

Node 1                      Node 2

Pod A                       Pod B

10.244.1.5                  10.244.2.8

     +-----------------------------+
```

Question:

```
How does

10.244.1.5

reach

10.244.2.8 ?
```

---

# High-Level Journey

```
Application

↓

Socket

↓

TCP

↓

Pod eth0

↓

veth

↓

Node Network

↓

Physical Network

↓

Remote Node

↓

veth

↓

Pod eth0

↓

Application
```

Let's explore every step.

---

# Step 1 — Application

Application sends:

```text
Destination

↓

10.244.2.8
```

Example:

```bash
curl http://10.244.2.8
```

The application only knows:

```
Destination IP
```

Nothing about Kubernetes.

---

# Step 2 — Socket

Linux creates:

```
Socket

↓

TCP/IP Stack
```

Packet is generated.

---

# Step 3 — Pod Network Namespace

Every Pod has:

```
Own Network Namespace
```

Inside:

```
eth0

↓

IP

↓

10.244.1.5
```

To the application,

this looks like a normal Linux machine.

---

# Step 4 — eth0

Packet leaves:

```
Application

↓

eth0
```

Inside the Pod,

`eth0` is one end of a `veth` pair.

---

# Step 5 — veth Pair

```
Pod

↓

eth0

↓

veth Pair

↓

Host
```

Packet leaves the Pod namespace.

Now it's inside the Node.

---

# Step 6 — CNI Network

Depending on the plugin:

```
Bridge
```

or

```
VXLAN
```

or

```
eBPF
```

Packet enters the node networking layer.

---

# Bridge Example

Using Linux Bridge:

```
Pod

↓

veth

↓

Bridge

↓

Host
```

Very similar to Docker bridge networking.

---

# Step 7 — Routing

Linux checks:

```
Destination

↓

10.244.2.8
```

Question:

```
Local Node?

or

Remote Node?
```

Routing table decides.

---

# Same Node Example

If Pod B is on:

```
Node 1
```

Flow:

```
Bridge

↓

veth

↓

Pod B
```

Done.

No physical network required.

---

# Different Node Example

Destination:

```
Node 2
```

Linux forwards packet toward:

```
Physical NIC
```

---

# Overlay Network

Flannel example:

```
Original Packet

↓

VXLAN

↓

New UDP Packet

↓

Internet

↓

Remote Node
```

Packet travels inside an overlay tunnel.

---

# Native Routing

Calico example:

```
Original Packet

↓

Routing

↓

Remote Node
```

No VXLAN required (depending on configuration).

---

# eBPF Example

Cilium:

```
Packet

↓

eBPF

↓

Routing

↓

Remote Pod
```

Packet processing can bypass large iptables rule sets.

---

# Step 8 — Physical Network

Packet leaves:

```
Node 1

↓

Switch

↓

Router

↓

Node 2
```

Normal networking.

Nothing Kubernetes-specific here.

---

# Step 9 — Remote Node

Packet arrives.

Linux performs:

```
Routing

↓

Destination

↓

10.244.2.8
```

---

# Step 10 — CNI

Plugin knows:

```
10.244.2.8

↓

Pod B
```

Packet forwarded correctly.

---

# Step 11 — veth

Packet enters:

```
Host

↓

veth Pair

↓

Pod
```

---

# Step 12 — eth0

Packet appears on:

```
eth0
```

inside Pod B.

---

# Step 13 — TCP/IP Stack

Linux processes:

```
IP

↓

TCP

↓

Socket
```

---

# Step 14 — Application

Application receives:

```
HTTP Request
```

Communication complete.

---

# Complete Journey

```
Application

↓

Socket

↓

TCP

↓

eth0

↓

veth

↓

Bridge / eBPF

↓

Routing

↓

Physical Network

↓

Remote Node

↓

Routing

↓

Bridge / eBPF

↓

veth

↓

eth0

↓

Application
```

---

# Where Does kube-proxy Fit?

Suppose application connects to:

```
ClusterIP

↓

10.96.0.15
```

This isn't a Pod.

It's a:

```
Service
```

kube-proxy translates:

```
ClusterIP

↓

Real Pod IP
```

using:

- iptables
- IPVS
- nftables (depending on implementation)
- or is bypassed by eBPF implementations such as Cilium.

---

# Service Example

Application:

```
10.96.0.20
```

kube-proxy:

```
↓

10.244.2.8
```

Packet reaches the Pod.

---

# Where Does conntrack Fit?

Suppose reply returns.

conntrack remembers:

```
Original Flow

↓

Reply Flow
```

State maintained automatically.

---

# Where Does Netfilter Fit?

Packet typically passes through:

```
PREROUTING

↓

Routing

↓

FORWARD

↓

POSTROUTING
```

depending on the networking implementation.

---

# Where Does eBPF Fit?

Modern clusters:

```
Packet

↓

eBPF

↓

Routing

↓

Pod
```

Many networking operations happen without relying heavily on iptables.

---

# Complete Kubernetes Networking Stack

```
Application

↓

Socket

↓

TCP/IP

↓

Network Namespace

↓

veth

↓

CNI

↓

Routing

↓

Netfilter

↓

conntrack

↓

Physical Network

↓

Remote Node

↓

CNI

↓

veth

↓

Pod
```

---

# Why Kubernetes Networking Feels Simple

Applications only know:

```
IP Address
```

Everything else is hidden by:

- Linux Kernel
- CNI
- kube-proxy
- Routing
- conntrack
- Netfilter
- eBPF

---

# Real Companies

Large platforms use these concepts every second.

Examples:

- Google GKE
- Amazon EKS
- Azure AKS
- OpenShift
- DigitalOcean Kubernetes

Different implementations,

same Linux foundations.

---

# Hands-on Lab

## Lab 1 — View Pod IP

```bash
kubectl get pods -o wide
```

---

## Lab 2 — View Node Routes

```bash
ip route
```

Observe Pod CIDRs.

---

## Lab 3 — Enter Pod

```bash
kubectl exec -it <pod> -- sh
```

Run:

```bash
ip addr
```

Observe:

```
eth0
```

---

## Lab 4 — Ping Another Pod

```bash
ping <pod-ip>
```

Observe direct Pod communication.

---

## Lab 5 — Inspect CNI

```bash
ls /etc/cni/net.d
```

Determine which plugin your cluster uses.

---

# Interview Questions

## Does Kubernetes implement networking itself?

No.

It defines the networking model and delegates implementation to CNI plugins.

---

## Why does every Pod receive its own IP?

Because Kubernetes networking is based on direct Pod-to-Pod communication.

---

## What connects a Pod to the Node?

A `veth` pair.

---

## What is the role of kube-proxy?

It implements Service networking by directing traffic from Service IPs to backend Pods.

---

## What is the role of CNI?

It configures Pod networking, interfaces, routes, and IP addresses.

---

## Does every Kubernetes cluster use VXLAN?

No.

It depends on the CNI plugin.

Examples:

- Flannel → Commonly VXLAN
- Calico → Routing or Overlay
- Cilium → eBPF

---

## Which Linux technologies power Kubernetes networking?

- Network Namespaces
- veth
- Linux Bridge
- Routing
- Netfilter
- conntrack
- CNI
- eBPF (in some implementations)

---

# Summary

A Kubernetes packet is ultimately just a Linux packet.

```
Application

↓

Socket

↓

Network Namespace

↓

eth0

↓

veth

↓

CNI

↓

Routing

↓

Physical Network

↓

Remote Node

↓

CNI

↓

veth

↓

eth0

↓

Application
```

Key concepts:

- Every Pod is an isolated Linux network namespace.
- Pods connect to Nodes using `veth` pairs.
- CNI plugins configure networking.
- Linux routing moves packets between nodes.
- kube-proxy provides Service networking.
- conntrack tracks active flows.
- Netfilter processes packets.
- Modern CNIs like Cilium accelerate networking with eBPF.

Congratulations! 🎉

You have now completed the **Linux Kernel & Networking Mastery** journey from basic Linux internals to Kubernetes networking.

You now understand the complete path of a packet through:

- Linux Kernel
- Docker
- Container Runtimes
- Netfilter
- Routing
- conntrack
- Traffic Control
- eBPF
- XDP
- CNI
- Kubernetes

This is the level of understanding expected of senior platform engineers, cloud infrastructure engineers, Kubernetes networking engineers, and Linux kernel networking specialists.

---

# Next Module

# Module 12 — Distributed Systems & Cloud Networking

Now that you've mastered networking **inside one Linux kernel and across Kubernetes clusters**, it's time to expand beyond a single cluster.

We'll explore:

- Chapter 69 — Why Distributed Systems Exist
- Chapter 70 — CAP Theorem
- Chapter 71 — Consistency Models
- Chapter 72 — Leader Election
- Chapter 73 — Consensus Algorithms (Raft & Paxos)
- Chapter 74 — Service Discovery
- Chapter 75 — API Gateways
- Chapter 76 — Load Balancers (L4 vs L7)
- Chapter 77 — Reverse Proxies
- Chapter 78 — Service Mesh (Istio & Linkerd)
- Chapter 79 — gRPC & HTTP/2
- Chapter 80 — Cloud Networking (AWS VPC, Azure VNet, GCP VPC)

> **This module will connect Linux networking with real-world distributed systems used by Google, Amazon, Netflix, Uber, and other large-scale platforms.**