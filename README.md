# Simple VPC Design on AWS 🌐

**Intern ID:** CITS5659

A no-jargon walkthrough of how to design and build a basic **Virtual Private Cloud (VPC)** on AWS — with public and private subnets, an Internet Gateway, and a NAT Gateway.

---

## 🧠 What Is a VPC, Really?

Think of a **VPC** as your own private, isolated section of the AWS cloud — like renting an entire floor of a building instead of just a desk. Inside that floor, you decide the rooms (subnets), who can enter from outside (gateways), and who can talk to whom (route tables, security groups).

**Why it matters:**
- 🔒 Keeps your resources isolated and secure
- 🧩 Lets you control exactly what's public-facing vs. private
- 🛣️ Gives you full control over networking (IP ranges, routing, access)

---

## 🏗️ The Design (Big Picture)

```
                          Internet
                             │
                    ┌────────▼────────┐
                    │ Internet Gateway │
                    └────────┬────────┘
                             │
        ┌────────────────────┴─────────────────────┐
        │                    VPC                    │
        │   ┌───────────────┐   ┌────────────────┐  │
        │   │ Public Subnet │   │ Private Subnet │  │
        │   │  (web server) │   │  (database)    │  │
        │   └───────┬───────┘   └────────┬───────┘  │
        │           │                    │           │
        │     Route Table          Route Table       │
        │     → IGW (0.0.0.0/0)    → NAT Gateway      │
        └────────────────────────────────────────────┘
                             │
                     ┌───────▼───────┐
                     │  NAT Gateway  │  (sits in public subnet)
                     └───────────────┘
```

**In plain terms:**
- **Public subnet** → resources here can be reached directly from the internet (e.g., a web server).
- **Private subnet** → resources here are hidden from the internet (e.g., a database), but can still reach *out* to the internet (for updates, patches) through the NAT Gateway.
- **Internet Gateway (IGW)** → the "front door" that lets public subnet traffic in and out.
- **NAT Gateway** → lets private subnet resources talk to the internet *outbound only* — nothing from the internet can initiate a connection back in.

---

## 🪜 Step-by-Step Setup

### 1. Create the VPC
- Go to the **VPC console** → **Create VPC**.
- Give it a name (e.g., `simple-vpc`).
- Set an IPv4 CIDR block, e.g., `10.0.0.0/16` (this gives you ~65,000 IP addresses to divide up).

### 2. Create Subnets
Create two subnets inside your VPC:
| Subnet | CIDR Block | Purpose |
|---|---|---|
| Public Subnet | `10.0.1.0/24` | Web-facing resources |
| Private Subnet | `10.0.2.0/24` | Databases, internal services |

> Tip: Put them in different Availability Zones for better fault tolerance.

### 3. Create and Attach an Internet Gateway
- Create an **Internet Gateway**.
- Attach it to your VPC.

### 4. Set Up Route Tables
- **Public route table**: Add a route `0.0.0.0/0 → Internet Gateway`. Associate it with the public subnet.
- **Private route table**: Add a route `0.0.0.0/0 → NAT Gateway`. Associate it with the private subnet.

### 5. Create a NAT Gateway
- Launch a NAT Gateway **inside the public subnet**.
- Allocate an Elastic IP for it (NAT Gateways need a public IP to function).
- Point the private route table to this NAT Gateway (done in step 4).

### 6. Launch Resources
- Put a web server (EC2 instance) in the **public subnet** → give it a public IP.
- Put a database in the **private subnet** → no public IP, only reachable from inside the VPC.

### 7. Test It
- Confirm the public EC2 instance is reachable from the internet.
- Confirm the private instance can reach the internet (e.g., run `yum update`) but **cannot** be reached directly from outside.

---

## 🔑 Key Concepts Cheat Sheet

| Term | Plain-Language Meaning |
|---|---|
| **VPC** | Your own private network inside AWS |
| **Subnet** | A smaller section/division within the VPC |
| **CIDR block** | The range of IP addresses assigned to a network |
| **Internet Gateway** | Lets a subnet talk directly to the internet (two-way) |
| **NAT Gateway** | Lets a private subnet reach the internet (one-way, outbound only) |
| **Route Table** | The "map" telling traffic where to go |
| **Security Group** | A firewall attached to instances (controls traffic in/out) |

---

## ⚠️ Common Gotchas

- **NAT Gateway must live in a public subnet** — a common mistake is placing it in the private one, which breaks everything.
- **Elastic IP required** — NAT Gateways won't launch without one attached.
- **Forgetting route table associations** — creating a route table isn't enough; you must associate it with the right subnet.
- **NAT Gateways cost money per hour + data processed** — remember to delete them after testing/labs to avoid charges.

---

## 📁 Suggested Repo Structure

```
simple-vpc-design/
├── README.md
├── diagrams/
│   └── vpc-architecture.png
└── screenshots/
    ├── 01-vpc-created.png
    ├── 02-subnets-created.png
    ├── 03-igw-attached.png
    ├── 04-route-tables.png
    ├── 05-nat-gateway.png
    └── 06-connectivity-test.png
```

## 📸 Screenshots

| Step | Preview |
|---|---|
| VPC created | `screenshots/01-vpc-created.png` |
| Subnets created | `screenshots/02-subnets-created.png` |
| Internet Gateway attached | `screenshots/03-igw-attached.png` |
| Route tables configured | `screenshots/04-route-tables.png` |
| NAT Gateway set up | `screenshots/05-nat-gateway.png` |
| Connectivity test | `screenshots/06-connectivity-test.png` |

> Tip: Embed these directly with `![VPC Created](screenshots/01-vpc-created.png)`

---

## ✅ Summary

A simple VPC design boils down to one idea: **separate what's public from what's private, and control exactly how traffic flows between them.** With a public subnet (fronted by an Internet Gateway) and a private subnet (routed out through a NAT Gateway), you get a secure, standard network layout used in real-world production environments.

---

*Feel free to fork this repo and adapt the CIDR ranges/resources to your own project.*
