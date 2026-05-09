# VLSM Example with OSPF – 5 Networks (100, 50, 25, 10, 5 hosts)

This repository demonstrates **Variable Length Subnet Masking (VLSM)** using the private Class C network `192.168.10.0/24`, and then shows how to enable **OSPF** dynamic routing between these subnets.

We efficiently allocate subnets for five networks with different host requirements – from largest to smallest – without wasting IP addresses, and then configure OSPF to advertise them.

## 📋 Host Requirements

| Network | Required Hosts |
|---------|----------------|
| A       | 100            |
| B       | 50             |
| C       | 25             |
| D       | 10             |
| F       | 5              |

> **Note**: Network E is skipped to avoid confusion with the original 4‑network example; F is added as the 5th network.

## 🧮 Step 1 – Determine Subnet Masks

For each requirement, find the smallest block size that provides enough usable hosts (`2ⁿ – 2 ≥ required hosts`).

| Hosts needed | Block size | Subnet mask     | CIDR |
|--------------|------------|-----------------|------|
| 100          | 128        | 255.255.255.128 | /25  |
| 50           | 64         | 255.255.255.192 | /26  |
| 25           | 32         | 255.255.255.224 | /27  |
| 10           | 16         | 255.255.255.240 | /28  |
| 5            | 16         | 255.255.255.240 | /28  |

> For 5 hosts, the smallest power of 2 is 16 (14 usable). So we need a `/28` subnet.

## 📍 Step 2 – Assign Subnets (Largest First)

Start from `192.168.10.0/24` and allocate **contiguous, non‑overlapping** subnets.

### Network A (/25, block size 128)
- **Network address**: `192.168.10.0/25`
- **Usable hosts**: `192.168.10.1 – 192.168.10.126`
- **Broadcast**: `192.168.10.127`

### Network B (/26, block size 64)
- **Network address**: `192.168.10.128/26`
- **Usable hosts**: `192.168.10.129 – 192.168.10.190`
- **Broadcast**: `192.168.10.191`

### Network C (/27, block size 32)
- **Network address**: `192.168.10.192/27`
- **Usable hosts**: `192.168.10.193 – 192.168.10.222`
- **Broadcast**: `192.168.10.223`

### Network D (/28, block size 16)
- **Network address**: `192.168.10.224/28`
- **Usable hosts**: `192.168.10.225 – 192.168.10.238`
- **Broadcast**: `192.168.10.239`

### Network F (/28, block size 16)
- **Network address**: `192.168.10.240/28`
- **Usable hosts**: `192.168.10.241 – 192.168.10.254`
- **Broadcast**: `192.168.10.255`

> All addresses from `0` to `255` are now used – no waste.

## 📊 Final Summary Table

| Network | CIDR | Subnet Mask         | Network Address | Usable Range                      | Broadcast     | Usable Hosts |
|---------|------|---------------------|-----------------|-----------------------------------|---------------|--------------|
| A       | /25  | 255.255.255.128     | 192.168.10.0    | 1 – 126                           | 192.168.10.127| 126          |
| B       | /26  | 255.255.255.192     | 192.168.10.128  | 129 – 190                         | 192.168.10.191| 62           |
| C       | /27  | 255.255.255.224     | 192.168.10.192  | 193 – 222                         | 192.168.10.223| 30           |
| D       | /28  | 255.255.255.240     | 192.168.10.224  | 225 – 238                         | 192.168.10.239| 14           |
| F       | /28  | 255.255.255.240     | 192.168.10.240  | 241 – 254                         | 192.168.10.255| 14           |

## 🧭 Step 3 – Adding OSPF Routing

To allow these five subnets to communicate, we can run **OSPF** on the routers that connect them.  
OSPF fully supports VLSM and will advertise each subnet with its correct mask.

### Example Topology

Assume three routers:

- **Router1** connects Net A (`192.168.10.0/25`) and has a link to Router2.
- **Router2** connects Net B (`192.168.10.128/26`) and Net C (`192.168.10.192/27`).
- **Router3** connects Net D (`192.168.10.224/28`), Net F (`192.168.10.240/28`), and links to Router2.

