# SimuLAN

An interactive network simulator for learning networking fundamentals: IPv4, CIDR, subnetting and VLANs.

SimuLAN is a browser-based network simulator built as a single HTML file. Design and visualize network topologies with routers, switches, PCs, servers, printers, and access points. Ships with a built-in tutorial network demonstrating VLSM subnetting across five VLANs with inter-VLAN routing and ACLs. No installation or dependencies required — just open `index.html` in a browser.

### Features

- Drag-and-drop topology builder
- VLAN assignment with access and trunk ports
- Router subinterfaces for inter-VLAN routing
- Deny rules (ACLs) on the router
- Route tracing with animated packet visualization
- Subnet calculator
- Save and load network configurations
- Built-in tutorial walkthrough

---

# Networking Fundamentals

## Table of Contents
1. [IPv4 Addressing](#ipv4-addressing)
2. [Subnet Masks & CIDR Notation](#subnet-masks--cidr-notation)
3. [Subnetting](#subnetting)
4. [IP Ranges per Subnet](#ip-ranges-per-subnet)
5. [Public vs Private IP Addresses](#public-vs-private-ip-addresses)
6. [VLANs](#vlans)
7. [Switches (Layer 2 & Layer 3)](#switches-layer-2--layer-3)
8. [Routers (Layer 3)](#routers-layer-3)
9. [Access Control Lists (ACLs)](#access-control-lists-acls)
10. [SimuLAN Tutorial Network](#simulan-tutorial-network)

---

## IPv4 Addressing

An IPv4 address is a 32-bit number that uniquely identifies a device on a network. It is written as four octets (bytes) separated by dots, each ranging from 0 to 255.

```
Example: 192.168.1.10

Binary:  11000000.10101000.00000001.00001010
Decimal: 192     .168     .1       .10
```

Every IPv4 address has two parts:
- **Network portion** — identifies which network the device belongs to
- **Host portion** — identifies the specific device within that network

Two devices can communicate directly (without a router) only if they share the same network portion. But how do we know where the network portion ends and the host portion begins? That's what subnet masks tell us.

---

## Subnet Masks & CIDR Notation

### Subnet Mask

A subnet mask is a 32-bit number that defines the boundary between the network and host portions of an IP address. The network bits are all `1`s and the host bits are all `0`s.

```
IP Address:  192.168.1.10    → 11000000.10101000.00000001.00001010
Subnet Mask: 255.255.255.0   → 11111111.11111111.11111111.00000000
                                |-------- Network --------|- Host -|

Network Address: 192.168.1.0  (all host bits = 0)
Broadcast:       192.168.1.255 (all host bits = 1)
```

### CIDR Notation

**CIDR** (Classless Inter-Domain Routing) is a compact way to express the same information. Instead of writing out the full subnet mask, you use a slash followed by the number of network bits.

```
192.168.1.0/24

  /24 means the first 24 bits are the network portion
  This equals a subnet mask of 255.255.255.0
```

CIDR notation is the standard way to represent subnets. You will see it everywhere in networking.

### CIDR to Subnet Mask Conversion

| CIDR | Subnet Mask | Network Bits | Host Bits | Usable Hosts |
|------|-------------|-------------|-----------|--------------|
| /8 | 255.0.0.0 | 8 | 24 | 16,777,214 |
| /16 | 255.255.0.0 | 16 | 16 | 65,534 |
| /24 | 255.255.255.0 | 24 | 8 | 254 |
| /25 | 255.255.255.128 | 25 | 7 | 126 |
| /26 | 255.255.255.192 | 26 | 6 | 62 |
| /27 | 255.255.255.224 | 27 | 5 | 30 |
| /28 | 255.255.255.240 | 28 | 4 | 14 |
| /29 | 255.255.255.248 | 29 | 3 | 6 |
| /30 | 255.255.255.252 | 30 | 2 | 2 |
| /31 | 255.255.255.254 | 31 | 1 | 2 (point-to-point) |
| /32 | 255.255.255.255 | 32 | 0 | 1 (single host) |

**Formula:** Usable hosts = 2^(32 - CIDR) - 2

The `-2` accounts for the **network address** (first) and **broadcast address** (last), which cannot be assigned to devices.

---

## Subnetting

Subnetting divides a large network into smaller, more manageable segments. This improves security, reduces broadcast traffic, and makes IP allocation more efficient.

### Example: Splitting a /24 into Subnets

Starting with `192.168.1.0/24` (256 addresses, 254 usable hosts):

**Split into 2 subnets (/25):**
```
192.168.1.0/25   → hosts .1 to .126   (126 usable)
192.168.1.128/25 → hosts .129 to .254 (126 usable)
```

**Split into 4 subnets (/26):**
```
192.168.1.0/26   → hosts .1 to .62    (62 usable)
192.168.1.64/26  → hosts .65 to .126  (62 usable)
192.168.1.128/26 → hosts .129 to .190 (62 usable)
192.168.1.192/26 → hosts .193 to .254 (62 usable)
```

**Split into 8 subnets (/27):**
```
192.168.1.0/27   → hosts .1 to .30    (30 usable)
192.168.1.32/27  → hosts .33 to .62   (30 usable)
192.168.1.64/27  → hosts .65 to .94   (30 usable)
192.168.1.96/27  → hosts .97 to .126  (30 usable)
192.168.1.128/27 → hosts .129 to .158 (30 usable)
192.168.1.160/27 → hosts .161 to .190 (30 usable)
192.168.1.192/27 → hosts .193 to .222 (30 usable)
192.168.1.224/27 → hosts .225 to .254 (30 usable)
```

### Variable-Length Subnet Masking (VLSM)

In practice, not every department needs the same number of hosts. VLSM allows subnets of different sizes from the same address block.

**Example: 192.168.1.0/24 for a small office**

| Department | Subnet | CIDR | Usable Hosts | Range |
|-----------|--------|------|-------------|-------|
| Sales | 192.168.1.0/26 | /26 | 62 | .1 – .62 |
| HR | 192.168.1.64/27 | /27 | 30 | .65 – .94 |
| IT | 192.168.1.96/28 | /28 | 14 | .97 – .110 |
| Guest | 192.168.1.112/28 | /28 | 14 | .113 – .126 |
| Services | 192.168.1.128/28 | /28 | 14 | .129 – .142 |

The largest department gets the biggest subnet (/26). Smaller departments use /27 or /28. No addresses are wasted.

---

## IP Ranges per Subnet

For any subnet, you can calculate the key addresses:

```
Given: 192.168.1.64/27

Step 1: Host bits = 32 - 27 = 5
Step 2: Block size = 2^5 = 32
Step 3: Network address = 192.168.1.64 (the given subnet)
Step 4: Broadcast = 192.168.1.64 + 32 - 1 = 192.168.1.95
Step 5: First usable host = 192.168.1.65
Step 6: Last usable host = 192.168.1.94
Step 7: Usable hosts = 32 - 2 = 30
```

### Quick Reference: /24 Subnet Blocks

| CIDR | Block Size | Number of Subnets in /24 |
|------|-----------|--------------------------|
| /25 | 128 | 2 |
| /26 | 64 | 4 |
| /27 | 32 | 8 |
| /28 | 16 | 16 |
| /29 | 8 | 32 |
| /30 | 4 | 64 |

### Default Gateway

The **first usable IP** in a subnet is typically assigned to the router interface (default gateway). All devices in the subnet use this address to reach other networks.

```
Subnet:          192.168.1.64/27
Default Gateway: 192.168.1.65  (router interface)
First Device:    192.168.1.66
Last Device:     192.168.1.94
Broadcast:       192.168.1.95
```

---

## Public vs Private IP Addresses

### Private IP Ranges (RFC 1918)

These addresses are reserved for internal networks and are **not routable** on the public internet.

| Class | Range | CIDR | Networks | Hosts per Network |
|-------|-------|------|----------|--------------------|
| A | 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | 1 | 16,777,214 |
| B | 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | 16 | 65,534 each |
| C | 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | 256 | 254 each |

The "Networks" column counts how many classful networks (Class A = /8, Class B = /16, Class C = /24) fit in each private range.

### Public IP Addresses

Everything outside the private ranges (and a few other reserved blocks) is considered public. Public IPs are assigned by Internet Service Providers (ISPs) and are globally unique.

### Why Private Addresses?

- There aren't enough IPv4 addresses for every device on Earth (only ~4.3 billion)
- Organizations use private addresses internally and share one or a few public IPs via **NAT** (Network Address Translation)
- Private addresses provide a layer of isolation from the internet

### Special Addresses

| Address | Purpose |
|---------|---------|
| 127.0.0.0/8 | Loopback (localhost) |
| 169.254.0.0/16 | Link-local (auto-assigned when no DHCP) |
| 0.0.0.0 | Default route / unspecified |
| 255.255.255.255 | Broadcast (all hosts on local network) |

---

## VLANs

A **VLAN** (Virtual Local Area Network) is a logical segmentation of a physical network at Layer 2 (Data Link). Devices on the same VLAN can communicate as if they were on the same physical switch, even if they are spread across multiple switches.

### Why VLANs?

- **Security** — Isolate sensitive departments (e.g., HR, Finance) from general traffic
- **Performance** — Reduce broadcast domains; broadcasts stay within the VLAN
- **Flexibility** — Group devices logically, not by physical location
- **Cost** — Use one physical switch for multiple logical networks

### VLAN IDs

- Valid range: **1 – 4094**
- VLAN 1 is the **default VLAN** (all ports belong to it initially)
- VLANs 1002–1005 are reserved for legacy protocols

### Port Types

**Access Port:**
- Belongs to a single VLAN
- Connects end devices (PCs, servers, printers)
- The device is unaware it's on a VLAN
- Frames are untagged

```
[PC] ----access (VLAN 10)---- [Switch]
```

**Trunk Port:**
- Carries traffic for multiple VLANs simultaneously
- Uses **802.1Q** tagging to identify which VLAN each frame belongs to
- Connects switches to switches, or switches to routers

```
[Switch1] ----trunk (VLANs 10,20,30)---- [Switch2]
[Switch]  ----trunk (VLANs 10,20,30)---- [Router]
```

### 802.1Q Tagging

When a frame travels over a trunk link, a 4-byte VLAN tag is inserted into the Ethernet header:

```
Normal frame:   [Dest MAC][Src MAC][Type][Data][FCS]
Tagged frame:   [Dest MAC][Src MAC][802.1Q Tag][Type][Data][FCS]
                                    ├─ TPID (0x8100)
                                    └─ VLAN ID (12 bits)
```

Access ports strip the tag before delivering to end devices.

---

## Switches (Layer 2 & Layer 3)

### Layer 2 Switch

A Layer 2 switch operates at the **Data Link layer** using **MAC addresses**. It learns which MAC addresses are on which ports and forwards frames accordingly.

**How it works:**
1. Device A sends a frame to Device B
2. Switch reads the source MAC and records it in its **MAC address table**
3. Switch looks up the destination MAC
4. If found → forward to that port. If not found → flood to all ports (except source)

**Key characteristics:**
- Forwards based on MAC addresses
- Creates separate **collision domains** per port
- All ports share one **broadcast domain** (unless VLANs are configured)
- Does NOT look at IP addresses

```
Device A (MAC: AA:AA) ──port 1──┐
                                │
Device B (MAC: BB:BB) ──port 2──├── Layer 2 Switch
                                │
Device C (MAC: CC:CC) ──port 3──┘

MAC Table:
  Port 1 → AA:AA
  Port 2 → BB:BB
  Port 3 → CC:CC
```

### Layer 2 Switch with VLANs

When VLANs are configured, the switch creates **separate broadcast domains**:

```
VLAN 10 (Sales):     ports 1, 2  → broadcast stays within VLAN 10
VLAN 20 (HR):        ports 3, 4  → broadcast stays within VLAN 20
Trunk (all VLANs):   port 24     → connects to router or another switch
```

Devices on VLAN 10 **cannot** communicate with devices on VLAN 20 through the switch alone — they need a **router** (Layer 3 device).

### Layer 3 Switch

A Layer 3 switch combines switching and routing in one device. It can:
- Forward frames by MAC address (Layer 2)
- Route packets by IP address (Layer 3)
- Have IP interfaces assigned to VLANs (SVIs — Switch Virtual Interfaces)
- Perform inter-VLAN routing without an external router

Layer 3 switches are common in enterprise networks where high-speed inter-VLAN routing is needed.

---

## Routers (Layer 3)

A router operates at the **Network layer** using **IP addresses**. Its job is to forward packets between different networks (subnets).

### How Routing Works

1. Device A (192.168.1.10) wants to reach Device B (192.168.2.20)
2. Device A checks: "Is 192.168.2.20 on my subnet (192.168.1.0/24)?" → **No**
3. Device A sends the packet to its **default gateway** (192.168.1.1 = router)
4. Router receives the packet on interface 192.168.1.1
5. Router looks up 192.168.2.20 in its **routing table**
6. Router finds that 192.168.2.0/24 is on another interface (192.168.2.1)
7. Router forwards the packet out that interface
8. Device B receives the packet

```
192.168.1.0/24                           192.168.2.0/24
                  ┌───────────┐
[PC1: .10] ────── │  Router   │ ────── [PC2: .20]
  gw: .1          │ .1    .1  │          gw: .1
                  │ eth0  eth1│
                  └───────────┘
```

### Router-on-a-Stick (Inter-VLAN Routing)

When using VLANs, a single physical router interface can handle multiple VLANs using **subinterfaces**. The router connects to the switch via a **trunk port**.

```
                        ┌──────────────┐
                        │    Router    │
                        │              │
                        │  G0/0.10 ──── 192.168.1.1/26  (VLAN 10 gateway)
                        │  G0/0.20 ──── 192.168.1.65/27 (VLAN 20 gateway)
                        │  G0/0.30 ──── 192.168.1.97/28 (VLAN 30 gateway)
                        │              │
                        └──────┬───────┘
                               │ trunk (VLANs 10,20,30)
                        ┌──────┴───────┐
                        │    Switch    │
                        │              │
              ┌─────────┼──────────────┼─────────┐
         port 1 (V10)  port 2 (V20)  port 3 (V30)
              │         │             │
           [PC-A]    [PC-B]        [PC-C]
```

**How a packet flows from PC-A (VLAN 10) to PC-C (VLAN 30):**
1. PC-A sends packet to its gateway 192.168.1.1
2. Switch forwards via trunk to router (tagged VLAN 10)
3. Router receives on subinterface G0/0.10
4. Router routes to subinterface G0/0.30 (VLAN 30)
5. Router sends back to switch (tagged VLAN 30)
6. Switch forwards to PC-C's access port (VLAN 30)

### Routing Table

The router maintains a table of known networks:

```
Destination        Gateway        Interface
192.168.1.0/26     directly       G0/0.10
192.168.1.64/27    directly       G0/0.20
192.168.1.96/28    directly       G0/0.30
0.0.0.0/0          ISP router     G0/1        (default route)
```

---

## Access Control Lists (ACLs)

An **ACL** is a set of rules on a router that filters traffic based on source and destination IP addresses (and optionally ports and protocols). ACLs act as a basic firewall at the network layer.

### How ACLs Work

ACLs are evaluated **top-down**. The first matching rule is applied, and the rest are skipped.

```
Rule 1: DENY   10.0.1.0/24  →  10.0.3.0/24     (Dept A → Servers)
Rule 2: DENY   10.0.2.0/24  →  10.0.1.0/24     (Dept B → Dept A)
Rule 3: DENY   10.0.4.0/24  →  10.0.0.0/16     (Guest → all internal)
Implicit: PERMIT everything else
```

### Types of ACLs

**Standard ACLs:**
- Filter based on **source IP only**
- Applied close to the destination
- Numbered 1–99

**Extended ACLs:**
- Filter based on **source IP, destination IP, protocol, and port**
- Applied close to the source
- Numbered 100–199

### ACL Design Principles

1. **Order matters** — Most specific rules first, general rules last
2. **Implicit deny** — Most systems deny everything not explicitly permitted (Cisco default). Some systems use implicit permit (everything allowed unless denied)
3. **Apply close to source** — For deny rules, apply as close to the traffic source as possible to avoid wasting bandwidth
4. **Be specific** — Use the narrowest subnet that matches your intent

---

## SimuLAN Tutorial Network

The SimuLAN tutorial network applies all the concepts above in a small office scenario. One /24 block is divided into five VLANs using VLSM, with inter-VLAN routing and ACLs for security.

### Network Topology

```
                         ┌──────────┐
                         │    R1    │
                         │ (Router) │
                         │          │
                    G0/0 │          │ G0/1
            ┌────────────┘          └───────────┐
            │ trunk                       trunk │
       ┌────┴────────────┐                 ┌────┴────────────┐
       │   SW1           │                 │   SW2           │
       │ (Switch)        │                 │ (Switch)        │
       └┬───┬───┬───┬───┬┘                 └┬───┬───┬───┬───┬┘
        │   │   │   │   │                   │   │   │   │
       V10 V10 V20 V20 V50                 V30 V30 V40 V50
        │   │   │   │   │                   │   │   │   │
       PC1 PC2 PC1 PC2 Printer            PC1  PC2 Ap  Server
        (Sales) (HR) (Service)              (IT)     (Service)
```

### Subnet Design (VLSM)

The entire office uses a single `192.168.1.0/24` block, divided as follows:

| Department | VLAN | Subnet | CIDR | Usable Hosts | Range |
|-----------|------|--------|------|-------------|-------|
| Sales | 10 | 192.168.1.0/26 | /26 | 62 | .1 – .62 |
| HR | 20 | 192.168.1.64/27 | /27 | 30 | .65 – .94 |
| IT | 30 | 192.168.1.96/28 | /28 | 14 | .97 – .110 |
| Guest | 40 | 192.168.1.112/28 | /28 | 14 | .113 – .126 |
| Services | 50 | 192.168.1.128/28 | /28 | 14 | .129 – .142 |

### Router Subinterfaces

R1 uses router-on-a-stick with one subinterface per VLAN. Each subinterface is the default gateway for its subnet.

| Interface | IP Address | Subnet Mask | VLAN | Role |
|-----------|-----------|-------------|------|------|
| G0/0.10 | 192.168.1.1 | 255.255.255.192 | 10 | Sales gateway |
| G0/0.20 | 192.168.1.65 | 255.255.255.224 | 20 | HR gateway |
| G0/1.30 | 192.168.1.97 | 255.255.255.240 | 30 | IT gateway |
| G0/1.40 | 192.168.1.113 | 255.255.255.240 | 40 | Guest gateway |
| G0/0.50 | 192.168.1.129 | 255.255.255.240 | 50 | Services gateway |

### Switch VLAN Assignments

**SW1** (VLANs 1, 10, 20, 50):
- Sales-PC1, Sales-PC2 → access VLAN 10
- HR-PC1, HR-PC2 → access VLAN 20
- Printer → access VLAN 50
- R1 → trunk (all VLANs)

**SW2** (VLANs 1, 30, 40, 50):
- IT-PC1, IT-PC2 → access VLAN 30
- Guest-WiFi (AP) → access VLAN 40
- Server → access VLAN 50
- R1 → trunk (all VLANs)

### Deny Rules (ACLs)

The router uses implicit permit (everything is allowed unless explicitly denied). These deny rules enforce security policies:

| # | Source | Destination | Purpose |
|---|--------|-------------|---------|
| 1 | 192.168.1.0/26 (Sales) | 192.168.1.128/28 (Services) | Sales cannot access Server or Printer |
| 2 | 192.168.1.0/26 (Sales) | 192.168.1.64/27 (HR) | Sales cannot access HR |
| 3 | 192.168.1.64/27 (HR) | 192.168.1.0/26 (Sales) | HR cannot access Sales |
| 4 | 192.168.1.112/28 (Guest) | 192.168.1.0/24 (all) | Guest WiFi fully isolated |

**What's allowed:**
- Sales ↔ Sales (same VLAN, doesn't go through router)
- IT → everything (no deny rule matches IT's subnet)
- HR ↔ IT (no deny rule between them)
- HR ↔ Services (no deny rule between them)

### Design Decisions

1. **VLSM** — One /24 block efficiently divided; Sales gets /26 (most users), smaller departments get /27 or /28
2. **VLANs** — Isolate departments at Layer 2; broadcasts stay within each VLAN
3. **Router-on-a-stick** — Single router handles all inter-VLAN routing via subinterfaces
4. **Trunk ports** — Switch-to-router links carry all VLAN traffic tagged with 802.1Q
5. **Access ports** — End devices connect on a single VLAN, unaware of VLAN tagging
6. **ACLs** — Sales is restricted from HR and Services; Guest is fully isolated; IT has full access for administration
