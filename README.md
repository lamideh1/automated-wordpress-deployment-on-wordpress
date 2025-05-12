# automated-wordpress-deployment-on-wordpress
Thanks for the clarification. Here is a complete A–Z guide for your **Terraform Capstone Project: Automated WordPress Deployment on AWS**, detailing **what** to do, **where** to do it, and **how** each component fits in.

---

## **Project Directory Structure**

```
terraform-wordpress/
│
├── main.tf
├── variables.tf
├── terraform.tfvars
├── outputs.tf
│
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── nat_gateway/
│   ├── rds/
│   ├── efs/
│   ├── alb/
│   ├── autoscaling/
│   ├── ec2/
│   └── security_group/
│
└── userdata.sh
```

---

## **Step-by-Step Instructions**

### **Step 1: Initialize Project**

Create a directory:

```bash
mkdir terraform-wordpress && cd terraform-wordpress
mkdir -p modules/{vpc,nat_gateway,rds,efs,alb,autoscaling,ec2,security_group}
```

---

## **Step 2: VPC Setup**

### **modules/vpc/main.tf**

Define:

* VPC
* Public and private subnets
* Route tables

### **Main config (main.tf)**

```hcl
module "vpc" {
  source = "./modules/vpc"
  vpc_cidr = "10.0.0.0/16"
  public_subnet_cidrs = ["10.0.1.0/24"]
  private_subnet_cidrs = ["10.0.2.0/24"]
  availability_zones = ["us-east-1a", "us-east-1b"]
}
```

---

## **Step 3: NAT Gateway Module**

### **modules/nat\_gateway/main.tf**

* Create NAT Gateway in a public subnet
* Elastic IP
* Route table for private subnet

### **main.tf**

```hcl
module "nat_gateway" {
  source = "./modules/nat_gateway"
  public_subnet_id = module.vpc.public_subnet_id
  private_route_table_id = module.vpc.private_route_table_id
}
```

---

## **Step 4: RDS (MySQL)**

### **modules/rds/main.tf**

* RDS instance
* Parameter group
* Subnet group
* Security group

### **main.tf**

```hcl
module "rds" {
  source = "./modules/rds"
  db_name     = "wordpress"
  db_username = "admin"
  db_password = "YourSecurePassword"
  vpc_id      = module.vpc.vpc_id
  subnet_ids  = module.vpc.private_subnet_ids
}
```

---

## **Step 5: EFS Setup**

### **modules/efs/main.tf**

* EFS File System
* Mount targets in private subnets

### **main.tf**

```hcl
module "efs" {
  source = "./modules/efs"
  subnet_ids = module.vpc.private_subnet_ids
  vpc_id = module.vpc.vpc_id
}
```

---

## **Step 6: Security Group**

### **modules/security\_group/main.tf**

Create SGs for:

* EC2 (allow HTTP, SSH)
* RDS (allow MySQL)
* ALB (HTTP from internet)

### **main.tf**

```hcl
module "security_group" {
  source = "./modules/security_group"
  vpc_id = module.vpc.vpc_id
}
```

---

## **Step 7: Application Load Balancer**

### **modules/alb/main.tf**

* ALB in public subnets
* Target group (EC2)
* Listener on port 80

### **main.tf**

```hcl
module "alb" {
  source = "./modules/alb"
  subnet_ids = module.vpc.public_subnet_ids
  vpc_id = module.vpc.vpc_id
  ec2_sg_id = module.security_group.ec2_sg_id
}
```

---

## **Step 8: Auto Scaling and Launch Template**

### **modules/autoscaling/main.tf**

* Launch template with EFS + RDS configs
* Auto Scaling group with min/max/desired capacity
* Target ALB

### **main.tf**

```hcl
module "autoscaling" {
  source = "./modules/autoscaling"
  launch_template_name = "wp-launch"
  image_id = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  key_name = "your-keypair"
  alb_target_group_arn = module.alb.target_group_arn
  subnet_ids = module.vpc.private_subnet_ids
  efs_id = module.efs.efs_id
  user_data = file("userdata.sh")
}
```

---

## **Step 9: EC2 User Data Script**

### **userdata.sh**

```bash
#!/bin/bash
yum update -y
amazon-linux-extras install -y php7.4
yum install -y httpd mysql php php-mysqlnd
systemctl start httpd
systemctl enable httpd

# Mount EFS
yum install -y amazon-efs-utils
mkdir -p /var/www/html
mount -t efs ${efs_id}:/ /var/www/html

cd /var/www/html
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
cp -r wordpress/* .
rm -rf wordpress latest.tar.gz
chown -R apache:apache /var/www/html
```

---

## **Step 10: Outputs**

### **outputs.tf**

```hcl
output "alb_dns_name" {
  value = module.alb.alb_dns
}
```

---

## **Step 11: Initialize and Deploy**

```bash
terraform init
terraform plan
terraform apply
```

---

## **Step 12: Verify & Document**

* Access the WordPress site using `alb_dns_name`.
* Simulate traffic (e.g., Apache Benchmark or `curl`) to test auto-scaling.
* Capture screenshots and logs.
* Document:

  * VPC CIDRs
  * Security Group rules
  * Scaling policy
  * Terraform command output
  * Challenges & resolution

---

Would you like a downloadable project folder with all the module files prepared?
