Article link - https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n40ec3fresyc0u49asch.png)

## Step 1: Create a VPC in the AWS Console

1. **Open the Amazon VPC Console**:
   - Go to the AWS Management Console and navigate to the Amazon VPC console at [https://console.aws.amazon.com/vpc/](https://console.aws.amazon.com/vpc/).

2. **Initiate VPC Creation**:
   - On the VPC dashboard, click on "Create VPC."

3. **Select Resources**:
   - For "Resources to create," select "VPC and more."

4. **Configure the VPC**:
   - **Name**: Enter a name for the VPC under "Name tag auto-generation."
   - **IPv4 CIDR Block**: Choose to keep the default suggestion or enter a specific CIDR block based on your application's requirements.
   - **IPv6 CIDR Block**: If your application uses IPv6, select "Amazon-provided IPv6 CIDR block."

5. **Configure the Subnets**:
   - **Availability Zones**: Choose "2" to launch instances in multiple Availability Zones for improved resiliency.
   - **Public Subnets**: Set the "Number of public subnets" to "2."
   - **Private Subnets**: Set the "Number of private subnets" to "2."
   - **Customize Subnet CIDR Blocks**: You can either keep the default CIDR block for the public subnet or expand "Customize subnet CIDR blocks" to enter a custom CIDR block.

6. **Configure NAT Gateways**:
   - Select "1 per AZ" to improve resiliency.

7. **Egress Only Internet Gateway (Optional)**:
   - If using IPv6, select "Yes" for "Egress only internet gateway."

8. **Configure VPC Endpoints**:
   - If your instances need access to an S3 bucket, keep the "S3 Gateway" as the default. Otherwise, choose "None." You can always add a gateway VPC endpoint later if needed.

9. **DNS Options**:
   - Uncheck "Enable DNS hostnames."

10. **Create the VPC**:
    - Review your configuration and click "Create VPC" to complete the setup.

---

## Step 2: Launch an Auto-Scaling Group

### 1. Create a Launch Template

1. **Access the EC2 Console**:
   - Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).

2. **Navigate to Launch Templates**:
   - In the left navigation pane, under **Instances**, click **Launch Templates**.

3. **Create a New Launch Template**:
   - Click **Create launch template**.
   - **Template Name**: Enter a name for your launch template.
   - **Template Version**: Provide a description for the version (optional).
   - **AMI ID**: Choose the Amazon Machine Image (AMI) that your instances will use.
   - **Instance Type**: Select the instance type (e.g., t2.micro).
   - **Key Pair**: Choose the key pair for SSH access.
   - **Security Groups**: Select one or more security groups.
   - **Storage (Volumes)**: Configure the storage as needed.
   - **IAM Role**: Assign an IAM role if necessary.
   - Configure additional settings such as **Network Interfaces**, **Monitoring**, and **Tags** as needed.

4. **Review and Create the Template**:
   - Review your settings, and then click **Create launch template**.

### 2. Create an Auto Scaling Group

1. **Access the Auto Scaling Console**:
   - Open the Amazon EC2 Auto Scaling console at [https://console.aws.amazon.com/ec2autoscaling/](https://console.aws.amazon.com/ec2autoscaling/).

2. **Create Auto Scaling Group**:
   - Click **Create Auto Scaling group**.

3. **Configure the Auto Scaling Group**:
   - **Auto Scaling Group Name**: Enter a name for your Auto Scaling group.
   - **Launch Template**: Select the launch template you created earlier.
   - **Version**: Choose the version of the launch template.

4. **Configure the Network**:
   - **VPC**: Select the VPC where your instances will run.
   - **Subnets**: Choose one or more subnets in different Availability Zones for better redundancy.

5. **Configure the Group Size and Scaling Policies**:
   - **Desired Capacity**: Enter the desired number of instances.
   - **Minimum Capacity**: Set the minimum number of instances.
   - **Maximum Capacity**: Set the maximum number of instances.
   - Configure **Scaling Policies** if you want the group to scale dynamically.

6. **Configure Advanced Options (Optional)**:
   - You can configure load balancing, health checks, and other advanced options as per your requirements.

7. **Add Notifications (Optional)**:
   - Set up notifications to receive alerts when instances are launched, terminated, etc.

8. **Add Tags (Optional)**:
   - Add any tags to the Auto Scaling group for identification and organization.

9. **Review and Create**:
   - Review your configuration, and click **Create Auto Scaling group**.

---

## Step 3: Launch and Access the Bastion Host

### 1. Create a Security Group for Bastion Host

1. **Go to EC2 Console**:
   - Navigate to **Security Groups**.

2. **Create a Security Group**:
   - **Name**: `Bastion-SG`.
   - **VPC**: Select `Cup`.

3. **Configure Inbound Rules**:
   - **Type**: SSH.
   - **Protocol**: TCP.
   - **Port Range**: 22.
   - **Source**: `My IP` (recommended for security).

4. **Outbound Rules**:
   - Leave the default rule to allow all traffic.

### 2. Launch EC2 Instance (Bastion Host)

1. **Go to EC2 Console**:
   - Click **Launch Instance**.

2. **Choose AMI**:
   - Select an Amazon Linux 2 or Ubuntu Server AMI.

3. **Instance Type**:
   - Choose `t2.micro`.

4. **Configure Instance Details**:
   - **Network**: Select `Cup`.
   - **Subnet**: Select `Public-Subnet-1`.
   - **Auto-assign Public IP**: Enable.
   - **IAM role**: If needed, select an IAM role for additional permissions.

5. **Add Storage**:
   - Use default storage settings or customize as needed.

6. **Configure Security Group**:
   - Select the security group `Bastion-SG`.

7. **Key Pair**:
   - **Key Pair Name**: Select an existing key pair or create a new one. This is critical for SSH access.

8. **Launch** the instance.

### 3. SSH into Bastion Host

1. **Open Terminal**:
   - For Linux/Mac, use the terminal; for Windows, use PuTTY.

2. **Execute SSH Command**:
   ```bash
   ssh -i /path_to_your_key_pair.pem ubuntu@<Bastion-Host-Public-IP>
   ```

### 4. Copy the Key Pair to Bastion Host

1. **Copy Key Pair**:
   ```bash
   scp -i /path_to_your_key_pair.pem /path_to_your_key_pair.pem ubuntu@<Bastion-Host-Public-IP>:/home/ubuntu/
   ```

### 5. SSH from Bastion Host to Private Subnet Instance

1. **Execute SSH Command**:
   ```bash
   ssh -i /home/ubuntu/your_key_pair.pem ubuntu@<Private-Instance-Private-IP>
   ```

---

## Step 4: Install and Run Python Application in Private Subnet

### 1. Install Python on Private Instance

1. **Update Package List**:
   ```bash
   sudo apt-get update
   ```

2. **Install Python3**:
   ```bash
   sudo apt-get install python3 -y
   ```

### 2. Create an HTML File

1. **Create `index.html`**:
   ```bash
   echo "<html><body><h1>Hello from Private Subnet</h1></body></html>" > index.html
   ```

### 3. Start Python HTTP Server

1. **Run the Server**:
   ```bash
   python3 -m http.server 8000
   ```

---

## Step 5: Create and Attach a Load Balancer

### 1. Create a Load Balancer

#### **Access the EC2 Console**
- **Navigate to the Amazon EC2 Console**:
  - Log in to your AWS Management Console.
  - From the AWS services menu, click on **EC2** to access the Amazon EC2 console.

#### **Create a Load Balancer**
- **Load Balancers Section**:
  - In the left-hand navigation pane of the EC2 console, scroll down to the **Load Balancing** section.
  - Click on **Load Balancers** to view existing load balancers or to create a new one.

- **Initiate Load Balancer Creation**:
  - Click the **Create Load Balancer** button located at the top of the Load Balancers page.

- **Select Load Balancer Type**:
  - AWS provides different types of load balancers, such as **Application Load Balancer (ALB)**, **Network Load Balancer (NLB)**, and **Classic Load Balancer**. Choose the appropriate type based on your application’s requirements.

- **Load Balancer Name**:
  - Enter a name for your load balancer. This name will help you identify the load balancer later. The name should be unique within your AWS account.

- **Scheme**:
  - **Internet-facing**: Select this option if you want the load balancer to route requests from clients over the internet to your instances.
  - **Internal**: Choose this option if the load balancer will only route requests within your VPC, for instance, between microservices.

- **Network Mapping**:
  - **VPC Selection**: From the drop-down list, select the Virtual Private Cloud (VPC) where your load balancer will operate. This should be the same VPC where your instances are running.
  - **Subnets**: Select the subnets where your load balancer will deploy. To ensure high availability, select at least two subnets across different Availability Zones. This configuration allows the load balancer to distribute traffic across multiple instances in different zones, enhancing fault tolerance.

- **Security Groups**:
  - **Assign Security Groups**: Choose one or more security groups that allow the necessary traffic (e.g., HTTP/HTTPS) to pass through the load balancer. If you don’t have a security group set up yet, you can create a new one here.
    - **Example**: If you're setting up an HTTP/HTTPS application, ensure your security group allows inbound traffic on port 80 (HTTP) and/or port 443 (HTTPS).
  - Security groups act as virtual firewalls for your load balancer, controlling the traffic that is allowed in and out.

This step ensures that your load balancer is properly configured and securely connected to the network resources it needs to route traffic efficiently and securely to your EC2 instances.
