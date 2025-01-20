Here’s a detailed guide to **set up a custom VPC with public and private subnets, configure an Internet Gateway and NAT Gateway for internet access, and launch instances connected via a Bastion Host**:

---

### **Objective**
- Create a custom VPC with both public and private subnets.
- Configure internet access for public subnets via an Internet Gateway and for private subnets via a NAT Gateway.
- Set up a Bastion Host to allow secure SSH access to private instances.

---

### **Step-by-Step Instructions**

#### **Step 1: Create the Custom VPC**
1. **Navigate to the VPC Dashboard**:
   - Open AWS Management Console > **VPC Dashboard**.

2. **Create a VPC**:
   - Click **Create VPC**.
   - Name: `CustomVPC`.
   - IPv4 CIDR block: `10.0.0.0/16`.
   - IPv6 CIDR block: No IPv6 for now.
   - Tenancy: Default.
   - Click **Create**.

---

#### **Step 2: Create Subnets**
1. **Create a Public Subnet**:
   - Go to **Subnets** > **Create Subnet**.
   - Name: `PublicSubnet`.
   - VPC: Select `CustomVPC`.
   - Availability Zone: `us-east-1a` (or your preferred zone).
   - CIDR block: `10.0.1.0/24`.
   - Enable **Auto-assign public IPv4** for the subnet (Modify later if not available here).
   - Click **Create Subnet**.

2. **Create a Private Subnet**:
   - Go to **Subnets** > **Create Subnet**.
   - Name: `PrivateSubnet`.
   - VPC: Select `CustomVPC`.
   - Availability Zone: `us-east-1b`.
   - CIDR block: `10.0.2.0/24`.
   - Click **Create Subnet**.

---

#### **Step 3: Create an Internet Gateway**
1. **Go to Internet Gateways**:
   - Click **Create Internet Gateway**.
   - Name: `CustomInternetGateway`.
   - Click **Create**.

2. **Attach the Internet Gateway to the VPC**:
   - Select `CustomInternetGateway`.
   - Click **Actions** > **Attach to VPC**.
   - Select `CustomVPC` and attach.

---

#### **Step 4: Configure Route Tables**
1. **Create a Public Route Table**:
   - Go to **Route Tables** > **Create Route Table**.
   - Name: `PublicRouteTable`.
   - VPC: `CustomVPC`.
   - Add a route:
     - Destination: `0.0.0.0/0`.
     - Target: `Internet Gateway`.
   - Associate the **PublicSubnet** with this route table:
     - Go to **Subnet Associations** > **Edit Subnet Associations**.
     - Select `PublicSubnet`.

2. **Create a Private Route Table**:
   - Go to **Route Tables** > **Create Route Table**.
   - Name: `PrivateRouteTable`.
   - VPC: `CustomVPC`.
   - Associate the **PrivateSubnet** with this route table:
     - Go to **Subnet Associations** > **Edit Subnet Associations**.
     - Select `PrivateSubnet`.

---

#### **Step 5: Create a NAT Gateway**
1. **Allocate an Elastic IP Address**:
   - Go to **Elastic IPs** > **Allocate Elastic IP Address**.
   - Click **Allocate**.

2. **Create the NAT Gateway**:
   - Go to **NAT Gateways** > **Create NAT Gateway**.
   - Name: `CustomNATGateway`.
   - Subnet: Select `PublicSubnet`.
   - Elastic IP: Select the allocated EIP.
   - Click **Create**.

3. **Update the Private Route Table**:
   - Add a route in the **PrivateRouteTable**:
     - Destination: `0.0.0.0/0`.
     - Target: `NAT Gateway`.

---

#### **Step 6: Launch EC2 Instances**
1. **Public Instance (Bastion Host)**:
   - Go to **EC2 Dashboard** > **Launch Instance**.
   - Name: `BastionHost`.
   - Subnet: `PublicSubnet`.
   - Security Group:
     - Allow SSH (port 22) from your IP (`0.0.0.0/0` for testing only, restrict for production).
   - Assign a public IP.

2. **Private Instance**:
   - Go to **EC2 Dashboard** > **Launch Instance**.
   - Name: `PrivateInstance`.
   - Subnet: `PrivateSubnet`.
   - Security Group:
     - Allow SSH (port 22) only from the Bastion Host’s private IP address.

---

#### **Step 7: Configure Bastion Host Access**
1. **Connect to the Bastion Host**:
   - Use the public IP of the Bastion Host:
     ```bash
     ssh -i "your-key.pem" ec2-user@<BastionHost-Public-IP>
     ```

2. **Connect to the Private Instance via Bastion Host**:
   - Use the private IP of the Private Instance:
     ```bash
     ssh -i "your-key.pem" ec2-user@<PrivateInstance-Private-IP>
     ```

---

### **Optional: Enable Application Traffic**
- **Public Instance**:
  - Update the Bastion Host security group to allow HTTP/HTTPS traffic (e.g., port 80/443).
- **Private Instance**:
  - Allow application traffic from the Bastion Host’s IP or other application servers as needed.

---

### **Clean Up**
- Terminate instances, NAT Gateway, and associated resources when testing is complete to avoid unnecessary charges.

---
