# High Availability Web Cluster on Docker

This project demonstrates a robust **High Availability (HA)** infrastructure implementation for a web server, deployed entirely on Docker containers.

The system ensures zero downtime by utilizing a **Floating IP** mechanism managed by a Pacemaker & Corosync cluster. If the active server fails, traffic is automatically and instantly routed to a passive standby node.

## 🏗 Architecture

The infrastructure consists of 4 Ubuntu containers:

* **Cluster Nodes (`webz-001`, `webz-002`, `webz-003`):**
    * Active/Passive configuration.
    * Running **Apache2** web server.
    * Managed by **Pacemaker** resource manager & **Corosync** messaging layer.
* **Monitoring Node (`webz-004`):**
    * Running **Jenkins** automation server.
    * Executes periodic health checks and logs the active node's identity.

## 🚀 Key Features

* **Automatic Failover:** Detection of node failure within seconds and automatic resource migration.
* **Floating IP (VIP):** A single entry point (`172.20.0.20`) that always points to the healthy, active node.
* **Docker-Compatible Networking:** Solved the common Docker Multicast limitation by configuring Corosync to use **Unicast (UDPU)** transport protocol.
* **Resource Colocation:** Ensures the Web Server service always runs on the same node holding the Floating IP.
* **Automated Monitoring:** A Jenkins job monitors the cluster's health status and response origin every 5 minutes.

## 🛠 Tech Stack

* **Infrastructure:** Docker, Docker Compose
* **OS:** Ubuntu 20.04 (Optimized)
* **Cluster Management:** Pacemaker, Corosync (pcs)
* **Web Server:** Apache HTTP Server
* **Automation:** Jenkins (Headless installation)
* **Scripting:** Bash

## ⚙️ How It Works

1.  **The Cluster:** The three nodes communicate constantly via Corosync.
2.  **The VIP:** Pacemaker assigns the Virtual IP (`172.20.0.20`) to the primary node.
3.  **The Client:** Users (or the monitoring script) access the web service via the VIP, unaware of which physical container is serving the request.

### Healthy Cluster Status
Initially, all nodes are online and resources are stable on the primary node:

![Healthy Cluster Status](images%20pcs/pcs%20status.png)

## 💥 Failure Scenario (Failover Test)

To test the system's resilience, we simulated a catastrophic failure by killing the active node:
`docker kill webz-001`

**The Result:**
The cluster immediately detected the loss of heartbeat. Pacemaker marked the node as `OFFLINE` and automatically migrated the VIP and Apache service to the standby node (`webz-002`) within seconds.

![Cluster Status After Kill](images%20pcs/pcs%20status%20after%20kill.png)

## 📊 Monitoring & Verification

The project includes a dedicated **Jenkins** instance running a custom monitoring pipeline. The job executes a `curl` request to the VIP every 5 minutes and logs the timestamp and the **Active Server Name**.

### Proof of Seamless Transition
The logs below capture the exact moment of failover. Note the transition from `webz-002` to `webz-001` (Active/Passive switch) with zero service interruption:

![Jenkins Logs POC](images%20pcs/logs%20poc.png)

