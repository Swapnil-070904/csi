# Azure Virtual Network Setup

## Introduction
This document outlines key Azure Virtual Network (VNet) concepts, including VNet, CIDR, subnets, and VNet peering, followed by a step-by-step guide to create two VNets, each with a subnet hosting a Linux VM, configure VNet peering, and ensure the VMs can ping each other.

### Definitions
- **Virtual Network (VNet)**: An Azure VNet is a logically isolated network in the Azure cloud that enables secure communication between Azure resources, such as virtual machines, databases, and applications. It provides a private IP address space for resource organization and connectivity.

- **CIDR (Classless Inter-Domain Routing)**: CIDR is a method for allocating IP addresses and routing using a notation like `10.0.0.0/16`. The `/16` indicates the number of bits in the network portion, defining the range of available IP addresses (e.g., 65,536 addresses for `/16`).

- **Subnets**: Subnets are smaller segments of a VNet’s IP address space, each assigned a unique CIDR block (e.g., `10.0.1.0/24`). Subnets organize resources, improve security, and control traffic flow within the VNet.

- **VNet Peering**: VNet peering connects two VNets, allowing resources in each to communicate as if they were in the same network. It supports low-latency, private connectivity without requiring gateways or public internet.
  - **Types of VNet Peering**:
    - **Same-Region Peering**: Connects VNets within the same Azure region for high-speed, low-latency communication.
    - **Global VNet Peering**: Connects VNets across different Azure regions, enabling cross-region resource communication.

## Implementation Steps
### Step 1: Create a Resource Group
1. Log in to the Azure Portal.
2. Navigate to **Resource Groups** > **Create**.
3. Configure:
   - Resource Group Name: `csi`
   - Region: `Central India`
4. Click **Review + Create** > **Create**.
![Rsgrp](img/rsgrp.png)

### Step 2: Create First Virtual Network
1. Navigate to **Virtual Networks** > **Create**.
2. Configure:
   - Resource Group: `csi`
   - Name: `VNet-1`
   - Region: `Central India`
3. Under **IP Addresses**:
   - IPv4 address space: `10.0.0.0/16`
   - Subnet:
     - Name: `Subnet1`
     - Address range: `10.0.1.0/24`
4. Click **Review + Create** > **Create**.
![vnet1](img/vnet1.png)

### Step 3: Create Second Virtual Network
1. Navigate to **Virtual Networks** > **Create**.
2. Configure:
   - Resource Group: `csi`
   - Name: `VNet-2`
   - Region: `Central India`
3. Under **IP Addresses**:
   - IPv4 address space: `10.1.0.0/16`
   - Subnet:
     - Name: `Subnet2`
     - Address range: `10.1.0.0/24`
4. Click **Review + Create** > **Create**.
![vnet2](img/vnet2.png)

### Step 4: Deploy First Linux VM
1. Navigate to **Virtual Machines** > **Create** > **Azure virtual machine**.
2. Configure:
   - Resource Group: `csi`
   - VM Name: `vnet1-VM`
   - Region: `Central India`
   - Image: Ubuntu Server 20.04 LTS
   - Size: Standard B1s
   - Authentication: SSH public key (generate or provide key)
   - Public inbound ports: Allow SSH (port 22)
3. Under **Networking**:
   - Virtual Network: `VNet-1`
   - Subnet: `Subnet1 (10.0.1.0/24)`
   - Public IP: Create new
4. Click **Review + Create** > **Create**.
5. Configure the rule:
   >Go to network settings edit inbound rule

   - Source: Any
   - Source port ranges: *
   - Destination: Any
   - Destination port ranges: *
   - Protocol: ICMP
   - Action: Allow
   - Priority: 100 
   - Name: Allow-ICMP
![vnet1VM](img/vnet1vm.png)

### Step 5: Deploy Second Linux VM
1. Navigate to **Virtual Machines** > **Create** > **Azure virtual machine**.
2. Configure:
   - Resource Group: `csi`
   - VM Name: `vnet2-VM`
   - Region: `Central India`
   - Image: Ubuntu Server 20.04 LTS
   - Size: Standard B1s
   - Authentication: SSH public key (generate or provide key)
   - Public inbound ports: Allow SSH (port 22)
3. Under **Networking**:
   - Virtual Network: `VNet-2`
   - Subnet: `Subnet1 (10.1.0.0/24)`
   - Public IP: Create new
4. Click **Review + Create** > **Create**.
5. Configure the rule:
   >Go to network settings edit inbound rule

   - Source: Any
   - Source port ranges: *
   - Destination: Any
   - Destination port ranges: *
   - Protocol: ICMP
   - Action: Allow
   - Priority: 100 
   - Name: Allow-ICMP
![vnet1VM](img/vnet2vm.png)

### Step 6: Validate Connectivity
1. **Connect to LinuxVM1**:
   - Use SSH to connect via its public IP (e.g., `ssh -i publickey azureuser@<LinuxVM1-public-IP>`).
   - Run `ping <LinuxVM2-private-IP>`.
2. **Connect to LinuxVM2**:
   - Use SSH to connect via its public IP (e.g., `ssh -i publickey azureuser@<LinuxVM2-public-IP>`).
   - Run `ping <LinuxVM1-private-IP>`.
3. Confirm successful ping responses in both directions.
![notping](img/notping.png)

***As we can see VMs cannot ping each other without Peering connections***

### Step 7: Configure VNet Peering
1. Navigate to **Virtual Networks** > `VNet1` > **Peerings** > **Add**.
2. Configure:
   - Peering Name: `VNet1-to-VNet2`
   - Remote Virtual Network: `VNet2`
   - Enable **Allow virtual network access**.
   - Enable **Allow forwarded traffic** if needed.
3. Click **Add**.<br>
***It creates a two way connection***
![peering](img/peering.png)
4. Repeat step 6 and check connectivity again.
![ping](img/ping.png)
***After creating peering connection VMs are able to ping each other***