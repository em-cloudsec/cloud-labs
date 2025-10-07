# Day 10 – Controlled Inbound Access (SSH) & VPC Flow Logs (ACCEPT)

## Objective
- Prove that **allowed** inbound traffic appears as **ACCEPT** in VPC Flow Logs.
- Compare this with earlier **REJECT** behavior (Day 8).
- Practice secure access by restricting SSH to **My IP** only.

---

## Steps Completed
1. **Identify instance**
   - EC2 → Instances → selected test instance (from Day 6).
   - Noted **Public IPv4** and **Private IPv4** (e.g., `172.31.40.188`).

2. **Security Group (SG) – add controlled SSH**
   - EC2 → Instance → **Security** tab → clicked the attached SG.
   - **Edit inbound rules**:
     - Type: **SSH**
     - Protocol: **TCP**
     - Port: **22**
     - Source: **My IP** (auto-filled to my current public IP)
     - Description: `SSH access for developers - restricted to specific IPs only`
   - Saved.

3. **Generate traffic (PowerShell)**
   ```bash
   ssh ec2-user@<public-ip>

parse @message "* * * * * * * * * * * * *"
  as version, accountId, interfaceId, srcAddr, dstAddr, srcPort, dstPort, protocol, packets, bytes, start, end, action, logStatus
| filter dstAddr = "172.31.40.188" and dstPort = 22
| sort @timestamp desc
| limit 50


2 ******728494 eni-******e0786 ***.***.***.193 172.31.40.188 52632 22 6 9 2013 1759808374 1759808402 ACCEPT OK


$ ssh ec2-user@<public-ip>
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '<public-ip>' (ED25519) to the list of known hosts.
ec2-user@<public-ip>: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
