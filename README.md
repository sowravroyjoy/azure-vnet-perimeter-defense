# Cloud Security Lab: Hardening the Cloud Perimeter (Azure VNet & NSGs)

## Project Overview
This project demonstrates the implementation of an enterprise-grade, multi-tier cloud network architecture inside Microsoft Azure, focused on architectural isolation and zero-trust perimeter defense. By separating front-end web utilities from sensitive backend data vaults, this infrastructure actively mitigates unauthorized lateral movement and reduces the overall public attack surface.

---

## Architectural Design & Real-World Analogy
Think of this infrastructure deployment like a **high-security bank branch**:
* **Virtual Network (VNet):** `sowravit-prod-vnet` acts as the secure building structure.
* **Public Subnet (`10.0.1.0/24`):** The **Front Lobby**. Hosts public-facing gateway assets (`sowravit-web-vm`) that legitimate traffic interacts with.
* **Backend Subnet (`10.0.2.0/24`):** The **Heavy Cash Vault**. Hosts deeply isolated database assets (`sowravit-db-vm`) with 0% public internet visibility.

---

## Implemented Security Controls (Network Security Groups)

### 1. Management Plane Lockdown (Locking the Lobby Door)
* **Mechanism:** Restricted inbound Remote Desktop Protocol (RDP) management traffic (`Port 3389`) on the Web NSG strictly to a designated administrator whitelisted public IP address.
* **Impact:** Prevents global automated internet scanners and malicious bots from attempting brute-force authorization attacks on the public entry point.

### 2. Subnet Isolation Policy (Securing the Vault Room)
* **Mechanism:** Completely stripped the backend Database VM of any Public IP address resource. Configured the database's Network Security Group (NSG) inbound rules to reject all external traffic by default, enforcing an explicit priority rule that *only* permits ingress requests originating from the private IP space of the front-end web tier.
* **Impact:** Eliminates direct public attack vectors. The database is entirely dark to the outside world.

---

## Verification & Penetration Testing Results

To validate the security posture of the deployed network, a multi-path penetration and connectivity test was conducted to verify rule efficacy.

### 🟢 Path A & B: Authorized Ingress & Internal Lateral Traversal
* **The Test:** Initiated an RDP session from the authorized administrator workstation to the Web VM's public IP, followed by an internal nested RDP "hop" to the Database VM using its private network address (`10.0.2.4`).
* **The Result:** Successful authentication. This confirms that authorized administrators can successfully access the perimeter and safely traverse into internal segments over secure, private routing pathways.

### Multi-Tier Network Lateral Jump Verification
<img width="1512" height="982" alt="Network-verification" src="https://github.com/user-attachments/assets/6470d805-9f3b-4d59-a324-9757421f9934" />


---

### 🔴 Path C: External Intrusion Simulation (Unauthorized Boundary Breach)
* **The Test:** Attempted to initiate a direct, external RDP connection from the public internet directly to the backend database network space, simulating an outside threat actor bypassing the front-end gateway.
* **The Result:** The connection request was dropping immediately at the cloud boundary, timing out with an **Error Code `0x204`**. This visually validates absolute network boundary fortification and confirms that the database layer is entirely hidden from unauthenticated external vectors.

### Perimeter Defense & Intrusion Block Verification
<img width="1512" height="982" alt="network-verification-02" src="https://github.com/user-attachments/assets/cc508603-f311-4545-9ac0-d4d8f7bdb6f8" />
