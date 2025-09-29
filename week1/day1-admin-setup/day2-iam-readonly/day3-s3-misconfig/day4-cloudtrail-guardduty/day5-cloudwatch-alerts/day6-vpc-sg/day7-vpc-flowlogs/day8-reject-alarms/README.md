# Day 8 – VPC Flow Logs + CloudWatch Alarm (Rejected Traffic)

## Objective
- Turn ***REJECT*** events from VPC Flow Logs into a CloudWatch **metric** and **alarm**.
- Send notifications via **SNS** (Simple Notification Service) when blocked traffic occurs.

---

## Steps Completed

### 1) Metric Filter on Flow Logs
- Console: **CloudWatch → Log groups →** `/vpc/flowlogs/week1day7`  
- **Create metric filter**
  - **Filter pattern:**
    ```
    REJECT
    ```
  - **Metric namespace:** `Security`
  - **Metric name:** `RejectedConnections`
  - **Metric value:** `1`

### 2) CloudWatch Alarm
- Console: **CloudWatch → Alarms → Create alarm**
- **Select metric:** `Security / RejectedConnections`
- **Condition:** `RejectedConnections >= 1` for **1** datapoint in **5 minutes**
- **Actions:** Notify SNS topic `SecurityAlerts` (email subscription confirmed)

### 3) Generate a REJECT
- Console: **EC2 → Instances →** select instance `day6-ec2-test`
- **Security** tab → click **Security group** link (e.g., `day6-sg-test`)
- **Edit inbound rules** → ensure **no inbound rules**
- From your browser/terminal, attempt to reach the instance’s **public IP** (HTTP or SSH)
- Expect connection to **time out** (blocked by SG)

---

## Results
- ❌ Connection timed out (no inbound rule)  
- ✅ VPC Flow Logs recorded `REJECT`  
- ✅ CloudWatch alarm **RejectedTraffic** changed **OK → ALARM**  
- ✅ SNS email received

---

## Evidence

### Sample Flow Log (masked)

2 ******728494 eni-***e0786 ...149 172.31.40.188 51183 18572 6 1 44 1759101610 1759101636 REJECT OK

- Account ID: ***masked (******728494)***
- ENI: ***eni-******e0786***
- Source IP: ***masked external IP***
- Destination (private IP): ***172.31.40.188*** on TCP/***18572***
- Action: ***REJECT*** (blocked by Security Group)

### Alarm
- Name: `RejectedTraffic`
- Condition: `RejectedConnections >= 1` in 5 minutes
- State transitions: **OK → ALARM → OK** (after no new rejections)

### SNS
- Topic: `SecurityAlerts`
- Email subscription: **Confirmed**
- Subject example: `ALARM: "RejectedTraffic" in AWS/CloudWatch`

---

## Key Learnings
- **Security Groups (SGs):** default **deny** inbound until explicitly allowed.
- **VPC Flow Logs:** record **ACCEPT/REJECT** traffic metadata (not payload).
- **Metric Filter:** turns log matches into a metric value (here, `1` per REJECT).
- **Alarm + SNS:** real-time notifications on blocked traffic (helps spot scans/misconfig).

---

## Acronyms
- **VPC (Virtual Private Cloud):** isolated network in AWS  
- **SG (Security Group):** instance-level stateful firewall  
- **ENI (Elastic Network Interface):** virtual NIC on EC2  
- **SNS (Simple Notification Service):** alerting via email/SMS/webhook  
- **TCP/UDP:** transport protocols (reliable vs lightweight)

