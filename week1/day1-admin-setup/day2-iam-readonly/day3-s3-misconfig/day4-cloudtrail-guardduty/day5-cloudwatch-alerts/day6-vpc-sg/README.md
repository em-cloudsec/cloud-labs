# Day 6 – VPC & Security Groups Lab

## Objective
- Understand how AWS **Virtual Private Cloud (VPC)** and **Security Groups (SGs)** control traffic.
- Demonstrate **default deny** vs **explicit allow** firewall behavior.

---

## Steps Completed
1. Created Security Group `day6-sg-test` in default VPC:
   - Inbound: none (deny all)
   - Outbound: allow all (default)

2. Launched EC2 instance `day6-ec2-test`:
   - Type: t2.micro (Free Tier)
   - AMI: Amazon Linux 2
   - Attached SG: `day6-sg-test`

3. Connectivity Testing:
   - ❌ With no inbound rules → public IP timed out.
   - ✅ Added HTTP (80) inbound rule → public IP reachable.
   - ❌ Removed rule → blocked again.
   - (Optional) SSH:
     - Rule: port 22 from my IP
     - ✅ SSH login succeeded

---

## Key Learnings
- **VPC (Virtual Private Cloud):** Isolated cloud network space.
- **SG (Security Group):** Instance-level stateful firewall.
- **Default behavior:** inbound = deny, outbound = allow.
- **Stateful:** If inbound is allowed, return traffic is automatically allowed.
- Adding/removing SG rules instantly affects traffic without restarting the instance.

---

## Evidence
- Public IP timed out with no inbound rules.
- Public IP accessible after adding HTTP inbound rule.
- SSH connection worked only when SG allowed my IP.

---

## Acronyms
- **VPC (Virtual Private Cloud):** Private network within AWS.
- **SG (Security Group):** Firewall rules for EC2 instances.
- **EC2 (Elastic Compute Cloud):** AWS virtual machine service.
- **NACL (Network ACL):** Subnet-level firewall, stateless.
- **SSH (Secure Shell):** Secure remote login protocol (port 22).
