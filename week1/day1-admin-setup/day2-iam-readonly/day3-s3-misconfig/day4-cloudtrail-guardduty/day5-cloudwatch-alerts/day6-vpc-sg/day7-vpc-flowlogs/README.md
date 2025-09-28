# Day 7 – VPC Flow Logs

## Objective
- Capture and analyze AWS VPC Flow Logs.
- Understand the difference between ***ACCEPT*** vs ***REJECT*** traffic.
- Link Security Group (Day 6) rules to actual network-level visibility.

---

## Steps Completed
1. Enabled VPC Flow Logs on default VPC:
   - Filter: ***ALL*** (captured ACCEPT + REJECT traffic)
   - Destination: CloudWatch Logs (`/vpc/flowlogs/week1day7`)
   - IAM Role: auto-generated for publishing to CloudWatch

2. Generated traffic:
   - Added and removed Security Group rules on `day6-ec2-test`
   - Browsed to public IP with HTTP allowed vs denied
   - Observed ACCEPT/REJECT actions in Flow Logs

---

## Evidence (Flow Logs)

### Accepted Traffic

2 ******728494 eni-***e0786 172.31.40.188 ...106 34927 123 17 1 76 1759101610 1759101636 ACCEPT OK

- Account ID: ***masked (******728494)***
- Interface ID: ***eni-******e0786*** (EC2 network interface)
- Source IP: ***masked external IP***
- Destination (private IP): ***172.31.40.188*** on ***port 123 (UDP/NTP)***
- Action: ***ACCEPT*** (allowed by Security Group)

---

### Rejected Traffic

2 ******728494 eni-***e0786 ...149 172.31.40.188 51183 18572 6 1 44 1759101610 1759101636 REJECT OK

- Account ID: ***masked (******728494)***
- Interface ID: ***eni-******e0786***
- Source IP: ***masked external IP***
- Destination (private IP): ***172.31.40.188*** on ***port 18572 (TCP)***
- Action: ***REJECT*** (blocked by Security Group)

---

## Key Learnings
- **Flow Logs** provide ***metadata*** (not packet payloads) about network traffic.
- They can be created at ***VPC, subnet, or ENI*** level.
- ***ACCEPT*** traffic = passed SG/NACL rules.
- ***REJECT*** traffic = blocked by SG/NACL rules.
- Flow Logs + CloudWatch give visibility into what’s happening inside a cloud network.

---

## Acronyms
- **VPC (Virtual Private Cloud):** Isolated cloud network environment.
- **SG (Security Group):** Instance-level firewall rules.
- **NACL (Network ACL):** Subnet-level firewall rules.
- **ENI (Elastic Network Interface):** Virtual NIC attached to EC2.
- **NTP (Network Time Protocol):** Common protocol on port 123 (used for time sync).
- **UDP (User Datagram Protocol):** Lightweight transport protocol.
- **TCP (Transmission Control Protocol):** Reliable transport protocol.
