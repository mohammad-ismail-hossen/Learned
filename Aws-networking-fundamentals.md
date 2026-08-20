# AWS Networking Fundamentals

All of cloud networking:

- *Who can access this?*
- *How does traffic move?*
- *How do we keep it secure?*

## Index

 - [Internet Basics](#part-1-internet-basics)
 - [AWS networking basics](#part-2---aws-networking)
 - [How it all fits together](#part-3---how-it-all-fits-together)

## Part 1: Internet Basics

### IP Address

An IP address is a unique address for a device on a network. Example:

```
192.168.1.10
```

To find each other, computers need IP addresses.

**Public IP**: Visible and reachable from the internet.

> Metaphor: A storefront address anyone can walk up to.

**Private IP**: Only works inside a private network, invisible to the outside world.

> Metaphor: A room number inside a building. Meaningless for someone outside.

---

### Public vs. Private Servers

**Public server**: exposed to the internet (web servers, load balancers, public APIs).

**Private server**: hidden from the internet (databases, backend logic).

---

### Subnet

A subnet is a smaller, isolated section of a larger network.

> Metaphor: **Network = City**, **Subnet = Neighborhood**

- **Public subnet** - holds resources that need internet access
- **Private subnet** - holds resources that should stay internal

---

### CIDR (Classless Inter-Domain Routing )

CIDR notation defines how large a network is. A CIDR block divides a giant network into smaller, specific IP ranges so that we can hand them out efficiently without wasting any addresses.

```
10.0.0.0/16
```

<details><summary>ipv4 address?</summary>

An IPv4 address is a 32-bit address (bit:0,1). 

Example: 11000000101010000000000100000001

Four octets: This 32 bits are divided into four groups of 8 bits, which are called octets. And every octet is a decimal number between 0-255 because 2⁸ = 256. So the decimal representation of the given example is: 192.168.1.1 which is familiar :)

The total number of ipv4 adress is 2³² = 4,294,967,296 ; rougly 4 billion.

</details>

The number after `/` is called prefix length. It controls size:

| CIDR | Network Size |
|------|-------------|
| `/16` | Large (65,536 addresses) |
| `/24` | Medium (256 addresses) |
| `/32` | Single IP address |

> **smaller number = bigger network**.


<details><summary>Prefix length math?</summary>

In `10.0.0.0/24`:

10.0.0.0 is the network address. /24 is the prefix. It means the first 24 bits (3 octets) are used for the network portion (network prefix) and remaining 8 bits (last octet) are available for addresses within that network (host bits).

Because IPv4 has 32 bits:

32 - 24 = 8 host bits

2⁸ = 256 addresses

For an IPv4 CIDR block:

Total addresses = 2^(32 - prefix length)

For /24:

2^(32 - 24)
= 2⁸
= 256 addresses

Because for /24 there is left only one octet, and we know one octed in decimal value is upto 256

For /32:

2^(32-32)
= 2⁰
= 1 address

So, for 192.168.0.104/32 all of it is network prefix, and none of it is host bits. Thats why the whole address is just one address~ 

</details>

---

### DNS (Domain Name System)

DNS translates human-readable names into IP addresses that computers use.

```
google.com  to  142.250.80.46
```

---

### Ports

A port is a specific entry point on a server.

| Port | Purpose |
|------|---------|
| 80 | HTTP |
| 443 | HTTPS |
| 22 | SSH (remote terminal access) |

> Metaphor: **IP address = apartment building**, **port = individual apartment door**.

<details><summary>HTTP vs HTTPS</summary>

| Protocol | What it is |
|----------|-----------|
| HTTP | Regular web traffic (unencrypted) |
| HTTPS | Encrypted web traffic (HTTP + TLS) |

Modern applications use HTTPS because with HTTP anyone between user and the server can read traffic.

</details>

---

### Routing

Routing is the process of deciding where traffic should go.

> Metaphor: Maps for internet traffic.

---

### Firewalls

A firewall decides what traffic is allowed in or out of a network or server.

> Metaphor: A security guard

### Networking Layers (OSI model)

| Layer | Name | Key concepts |
|-------|------|--------------|
| **1** | Physical layer | Cables, Wi-Fi signals |
| **2** | Data Link | Local network (MAC addresses, switches) |
| **3** | Network layer | IP addresses, routing |
| **4** | Transport layer | TCP/UDP protocols, ports |
| **5–6** | Session & Presentation | Session management, encoding |
| **7** | Application layer | Actual data contents (HTTP/HTTPS,  URLs, headers, cookies) |


#### Layer 3: Network (IP & Routing)
- `Where` a packet is going?
- AWS services: Route Tables, Internet Gateway (IGW), NAT Gateway, VPC, NACLs, Gateway load balancer (GLB)

#### Layer 4: Transport (TCP/UDP + Ports)
- `How` data is being delivered?
- TCP = connection-oriented (checks before sending), reliable (e.g. HTTPS on port 443)
- UDP = connectionless (throws data), faster (e.g., DNS on port 53)
- AWS services: Security Groups, Network load balancer (NLB)

#### Layer 7: Application (HTTP/HTTPS + Content)
- `What` is being sent?
- AWS services: Application load balancer (ALB) (path/host-based routing), WAF, CloudFront, API Gateway

---

## Part 2 - AWS Networking

### VPC (Virtual Private Cloud)

A VPC is a private, isolated network inside AWS. Inside a VPC we can control: IP ranges, subnets, routing and security rules.

```
AWS Account
 └── VPC  (your private network)
      ├── Public Subnet  (internet-facing)
      ├── Private Subnet (internal only)
      └── Your resources (EC2, RDS, etc.)
```

A VPC exists within one Region and can span across multiple Availability Zones in that Region. The subnets spans across the AZ's. Note that a subnet belongs to one AZ it can't be multiple, where an AZ can have multiple subnets.

```
Region
└── VPC
    ├── Subnet A (AZ-a)
    ├── Subnet B (AZ-b)
    └── Subnet C (AZ-c)
```

For EFS (Elastic file system) in aws:

```

          _________EFS___________ ----------vpc/region bound
         /          |            \
 Mount A         Mount B       Mount C------subnet/AZ bound
 (AZ-a)          (AZ-b)        (AZ-c)-------|
    |               |              |        |
 EC2-A           EC2-B         EC2-C ------─┘
 (AZ-a)          (AZ-b)        (AZ-c)
``` 
EFS are VPC bound where mount targets are subnet bound.

EFS exists inside VPC and where the mount targets (doors) exist in subnets. So data directly goes from subnets to efs, it never travels between AZ's because subnets are AZ bound and VPC is region bound.


---

### Public & Private Subnets

**Public subnet** - connected to the internet via an Internet Gateway. Holds anything that needs to be publicly reachable (load balancers, public servers).

**Private subnet** - no direct internet connection. Holds anything that should stay internal (databases, backend services).

A typical real-world architecture:

```
Internet
   ↓
Load Balancer--------- Public Subnet
   ↓
Application Server---- could be public or private
   ↓
Database-------------- Private Subnet
```

Traffic goes to load balancer, the load balancer sends traffic to the app and the app works with the database. The database is not directly reached from outside.

---

## Gateways

### IGW (Internet Gateway)

An Internet Gateway is a 1:1 bi-directional bridge for vpc and internet. It allows the vpc to be connected to the internet.

```
VPC  ←→  Internet Gateway  ←→  Internet
```

A public subnet is "public" precisely because its route table points to an IGW.

---

### NAT Gateway

A NAT Gateway is a oneway valve for a private server. When a private server needs to reach the internet (e.g., downloading software updates) we use nat gateway.

| Direction | Allowed? |
|-----------|---------|
| Private server → Internet | Yes |
| Internet → Private server | No |

Only the private server's initiated requests can go out, and come back in. So, ouside traffic can't initiate connection.

> Metaphor: a one-way valve. Traffic can flow out, but not in.

---

### Route Tables

A route table is a set of rules that tells traffic where to go.

Example rule:

To make a subnet public, we route traffic to IGW.

```
0.0.0.0/0  →  Internet Gateway
```

This means: "send all traffic coming from the internet through the IGW."

Private subnets route tables don't point to an IGW. That's why they can't be reached from the internet directly.

---

## Firewalls

### Security Groups

Security Groups are the primary firewall in AWS. Applied at the instance level (EC2 instance, RDS instance, etc.).

We define rules like:

```bash
Inbound:
Allow port 443 from 0.0.0.0/0
# HTTPS traffic from

Allow port 22 from 203.0.113.5
# SSH from only 203.0.113.5 ip

Outbound:
Allow all traffic
# default
```

**Key concept - Stateful:**

If you allow inbound traffic on port 443, the response automatically goes back out without needing a separate outbound rule. The Security Group tracks the connection state.

> Metaphor: a smart bouncer who remembers faces. If they let you in, they'll let you out.

---

### NACLs (Network Access Control Lists)

NACLs are an additional firewall layer, but they operate at the **subnet level**.

**Key concept - Stateless:**

NACLs don't track connection state. If you allow inbound traffic, you must also explicitly allow the return traffic outbound. Both directions need rules.

> Metaphor: a less smart bouncer who doesn't remember faces. He has to be told who to let in and also who to let out.

**Security Groups vs. NACLs at a glance:**

| Feature | Security Groups | NACLs |
|---------|----------------|-------|
| Applied to | Individual instances | Entire subnets |
| State | Stateful | Stateless |
| Default inbound behavior | Deny all | Allow all |
| Default outbound behavior | Allow all | Deny all |
| Primary use | Main firewall | Extra subnet-level filter |
| Rule evaluation | All rules checked | Rules checked in number order |

For most cases, Security Groups are sufficient. NACLs are an extra layer for stricter environments.

---

### Load Balancer

A Load Balancer creates a single endpoint for the users and distributes incoming traffic across multiple instances.

```
10,000 users
      ↓
 Load Balancer
  ↙    ↓    ↘
 S1   S2   S3   - traffic spread evenly
```

Benefits: **scalability** (handle more traffic), **high availability** (if one server dies, others keep running), **fault tolerance** (no single point of failure).



---

## Part 3 - How It All Fits Together

### The AWS Security Layer Model

Security in AWS is a stack of layers.

```
┌──────────────────────────────────────────┐
│  Layer 1: Edge                           │
│  - AWS Shield (DDoS protection) & WAF    │
│  - Filters malicious traffic globally    │
└─────────────────────┬────────────────────┘
                      ↓
┌──────────────────────────────────────────┐
│  Layer 2: Network Gateway                │
│  - Internet Gateway + Route Tables       │
│  - Controls what can enter/exit the VPC  │
└─────────────────────┬────────────────────┘
                      ↓
┌──────────────────────────────────────────┐
│  Layer 3: Subnet                         │
│  - NACLs                                 │
│  - Filters traffic at the subnet boundary│
└─────────────────────┬────────────────────┘
                      ↓
┌────────────────────────────────────────────┐
│  Layer 4: Instance                         │
│  - Security Groups                         │
│  - Filters traffic per individual resource │
└─────────────────────┬──────────────────────┘
                      ↓
┌──────────────────────────────────────────┐
│  Layer 5: OS / Application               │
│  - iptables, app-level auth, encryption  │
│  - Last line of defense on the machine   │
└──────────────────────────────────────────┘
```

---

### Typical Architecture - (3 tier)

```
Internet
   ↓
[Internet Gateway]
   ↓
[Load Balancer]  - (Public Subnet) (Security Group (allows 80/443))
   ↓
[App Servers]    - (Private Subnet) (Security Group (allows from LB only))
   ↓
[Database]       - (Private Subnet) (Security Group (allows from app only))
   ↑
[NAT Gateway]   - Lets private servers reach internet for updates but blocks inbound connections from internet
                   
```

This is a standard 3-tier architecture.

---

## Quick Reference Cheat Sheet

### Core Concepts

| Concept | One-Line Summary |
|---------|-----------------|
| IP Address | Unique address for a device on a network |
| Public IP | Reachable from the internet |
| Private IP | Only reachable inside a private network |
| DNS | Translates domain names to IP addresses |
| Port | A specific entry point on a server (80=HTTP, 443=HTTPS, 22=SSH) |
| CIDR | Defines network size |
| Routing | Rules that decide where traffic is sent |
| Firewall | Gatekeeper that controls what traffic is allowed |

### AWS-Specific Concepts

| Concept | One-Line Summary |
|---------|-----------------|
| VPC | Your private isolated network inside AWS |
| Public Subnet | Has a route to the internet via IGW |
| Private Subnet | No direct internet access |
| Internet Gateway | The door between your VPC and the internet |
| NAT Gateway | Lets private servers reach internet - blocks inbound |
| Route Table | Rules that tell traffic where to go |
| Security Group | Instance-level firewall - stateful |
| NACL | Subnet-level firewall - stateless |
| Load Balancer | Distributes traffic across multiple servers |

### Common Port Numbers

| Port | Protocol | Used for |
|------|----------|---------|
| 22 | SSH | Remote server access |
| 80 | HTTP | Unencrypted web traffic |
| 443 | HTTPS | Encrypted web traffic |
| 3306 | MySQL | Database connections |
| 5432 | PostgreSQL | Database connections |
