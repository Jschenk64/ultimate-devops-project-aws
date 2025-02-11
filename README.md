Here is the breakdown of each Terraform resource:

### **1. `aws_vpc.main` (VPC)**
- **Creates a Virtual Private Cloud (VPC)** to logically isolate resources within AWS.
- **CIDR block:** Defined by `var.vpc_cidr`, which specifies the IP range for the VPC.
- **Enables DNS hostnames and support** for instances within the VPC.
- **Tags:** Identifies the VPC with a name and Kubernetes cluster association.

---

### **2. `aws_subnet.private` (Private Subnets)**
- **Creates multiple private subnets** based on `var.private_subnet_cidrs`.
- **Belongs to the VPC** (`aws_vpc.main.id`).
- **CIDR block and Availability Zone:** Assigned dynamically using `count.index`.
- **Tags:** Used for Kubernetes and internal Elastic Load Balancer (ELB) identification.

---

### **3. `aws_subnet.public` (Public Subnets)**
- **Creates multiple public subnets** based on `var.public_subnet_cidrs`.
- **CIDR block and Availability Zone:** Assigned dynamically using `count.index`.
- **Belongs to the VPC** (`aws_vpc.main.id`).
- **Maps public IPs on launch**, allowing instances in public subnets to be directly accessible from the internet.
- **Tags:** Used for Kubernetes and external ELB identification.

---

### **4. `aws_internet_gateway.main` (Internet Gateway)**
- **Provides internet access** for resources in the public subnets.
- **Attached to the VPC** (`aws_vpc.main.id`).
- **Tags:** Identifies the internet gateway.

---

### **5. `aws_eip.nat` (Elastic IPs for NAT Gateways)**
- **Creates an Elastic IP (EIP) for each public subnet**.
- **Domain set to `vpc`**, meaning it's within the VPC scope.
- **Tags:** Identifies the NAT EIPs.

---

### **6. `aws_nat_gateway.main` (NAT Gateway)**
- **Creates a NAT Gateway for each public subnet**, allowing private subnet instances to access the internet without exposing them.
- **Uses the Elastic IP (`aws_eip.nat[count.index].id`)** to ensure a static outbound IP.
- **Attached to a public subnet (`aws_subnet.public[count.index].id`)**.
- **Tags:** Identifies the NAT Gateways.

---

### **7. `aws_route_table.public` (Public Route Table)**
- **Creates a route table** for public subnets.
- **Default route (`0.0.0.0/0`) to the internet gateway (`aws_internet_gateway.main.id`)** to allow external access.
- **Tags:** Identifies the public route table.

---

### **8. `aws_route_table.private` (Private Route Tables)**
- **Creates separate route tables for each private subnet**.
- **Routes outbound traffic through the corresponding NAT Gateway** (`aws_nat_gateway.main[count.index].id`), ensuring instances in private subnets can access the internet securely.
- **Tags:** Identifies each private route table.

---

### **9. `aws_route_table_association.private` (Private Route Table Associations)**
- **Associates each private subnet** with its corresponding private route table.

---

### **10. `aws_route_table_association.public` (Public Route Table Associations)**
- **Associates each public subnet** with the public route table.

---

### **Summary**
- **VPC:** Defines the network.
- **Subnets:** Divides the VPC into public and private sections.
- **Internet Gateway:** Enables internet access for public subnets.
- **NAT Gateway & EIP:** Enables internet access for private subnets securely.
- **Route Tables:** Directs traffic.
- **Route Table Associations:** Ensures subnets use the correct route tables.

This configuration **sets up a highly available network** for a Kubernetes cluster in AWS, ensuring secure connectivity and scalability. 🚀
