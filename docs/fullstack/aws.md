---
title: AWS
---

## EC2

### User Guides

[EC2 User Guides](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/concepts.html)

Amazon Elastic Compute Cloud（Amazon EC2）在 Amazon Web Services（AWS）云中按需提供可扩展的计算容量。
使用 Amazon EC2 可以降低硬件成本，因此您可以更快地开发和部署应用程序。您可以使用 Amazon EC2 启动所需数量的虚拟服务器，配置安全性和联网以及管理存储。您可以添加容量（纵向扩展）来处理计算密集型任务，例如月度或年度进程或网站流量峰值。如果使用量减少，您可以再次减少容量（缩减）。

### Deploy Node App on AWS EC2

[Setting up an EC2 Instance Deploy Node app on AWS EC2 Amazon Linux 2](https://www.youtube.com/watch?v=oHAQ3TzUTro)

### EC2 Essentials

[EC2 Essentials](https://cloudcasts.io/course/ec2-essentials)

#### Instance Types

AWS has a pretty [well documented listing of the different EC2 instance types](https://aws.amazon.com/ec2/instance-types/).
Here are a the high-level instance types:

- General Purpose (T and M types, the most popular, and cheapest)
- Compute Optimized (CPUs - C types)
- Memory Optimized (RAM - R types)
- Accelerated (GPU and other - P, Inf, G, F, VT types)
- Storage Optimized (NVMe, fast EBS - I, D, H types)
- Naming Convention:

There are also variations within instance types - they share the same naming conventions. For example, you may see a `t3` type and `t3a` variation.
Here are the varations:

- a - AMD CPUs - cheaper than Intel CPUs, and not necessarily slower. I geneally recommend these.
- n - Increased networking performance
- d - NVMe ephemeral disks available
- g - Graviton2, ARM based CPUs. These are cheapest - I recommend them if you can use ARM
- z - Xeon processors, ideal for single-threaded applications
- b - Block storage (EBS) optimized

#### T3 Instance

[Amazon EC2 T3 Instances](https://aws.amazon.com/ec2/instance-types/t3/)

T3 instances **accumulate CPU credits** when a workload is operating **below the baseline** threshold and **uses credits** when running **above the baseline threshold**. 

#### Instance Config

[Instance Configuration](https://cloudcasts.io/course/ec2-essentials/instance-configurations)

Some of the options we cover:

1. [Spot Instances](https://aws.amazon.com/ec2/spot/?cards.sort-by=item.additionalFields.startDateTime&cards.sort-order=asc) - Save money using Spot instances, if your applications and infrastructure are fault-tolerant.
2. [Networks/VPC](https://cloudcasts.io/course/vpc-basics) - We cover this in the VPC Basics course
3. [Placement Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html) - Determine how EC2 instances are created within AWS's physical infrastructure
4. Domain join directory - Use [AWS Directory Services](https://aws.amazon.com/directoryservice/), essentially AWS's Active Directory
5. [IAM Roles](https://aws.amazon.com/iam/) - We mention this but don't cover it much, we'll be covering that in other courses
6. [CPU Options](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-optimize-cpu.html) - Often used if serving software whose licenses charge per CPU core or thread.
7. Behaviors - there's a few general options here, mostly self-explanatory
    - Monitoring in CloudWatch samples every 5 minutes by default, however "enabling monitoring" will sample every 1 minute
    - [EC2 Tenancy](https://docs.aws.amazon.com/autoscaling/ec2/userguide/auto-scaling-dedicated-instances.html)
    - [Elastic Inference](https://aws.amazon.com/machine-learning/elastic-inference/)
8. [Credit Specification](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/burstable-performance-instances-how-to.html)
9. [File systems (NFS)](https://aws.amazon.com/efs/)
10. [Enclave](https://aws.amazon.com/ec2/nitro/nitro-enclaves/)
11. [EC2 metadata/userdata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)

#### Select Disk Types

GP2 and GP3. Here are some resources comparing GP2 vs GP3:

- [EBS volume types](https://aws.amazon.com/ebs/volume-types/)
- [Reduce Costs by Taking Advantage of Amazon’s gp3 EBS Volumes With Cloudability Rightsizing](https://cloudwiry.com/ebs-gp3-vs-gp2-pricing-comparison/)
- [gp3 For AWS EBS Volumes: Why, When and How To Switch](https://cloudsoft.io/blog/gp3-for-aws-ebs-volumes-why-when-and-how-to-switch)

#### Security Groups

[Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) are essentially firewalls, but work in the network-layer in AWS. Unlike a firewall in a server (e.g. iptables), they are not configured within a specific resource.

The **default** for Security Groups is to **disallow** all ingress (inbound) and egress (outbound) traffic.
A Security Group defines what types of traffic is allowed in/out of the server.

The video shows creating a Security Groups.
Note that how we create a security group in this video implicitly sets an egress rule to allow all outbound traffic - that's not explicitly shown in the web console.

#### SSH Key Pairs

We discuss creating and using [SSK Key Pairs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) in AWS so we can SSH into our instances.
You can also security connect to a server in a private subnet [as outlined here](https://aws.amazon.com/blogs/security/securely-connect-to-linux-instances-running-in-a-private-amazon-vpc/).

Essentially you use an SSH Agent and forward the authentication agent when SSH'ing into a public instance. It looks a bit like this:

```bash
# List keys in the agent
ssh-add –L
 
ssh-add -K ~/.ssh/your-private-key.pem
 
# Get into the bastion host, use the -A flag
# to forward the authentication agent
ssh -A ubuntu@<ip-address>
 
# From within the public-network server, ssh into a private
# server, which works as our private ssh key was forwarded
# into the ssh agent on the public network server
ssh ubuntu@<private-network-ip>
```

### Up and Running with AWS EC2

[Up and Running with AWS EC2](https://cs.fyi/guide/up-and-running-with-aws-ec2)

## VPC

### Official Doc

[official doc](https://aws.amazon.com/vpc/)

![How it works](https://d1.awsstatic.com/Digital%20Marketing/House/Hero/products/ec2/VPC/Product-Page-Diagram_Amazon-VPC_HIW%402x.f33b1976a62ff4600a1a8fd65e94057b43d9d917.png)

### AWS VPC & Subnets For Beginners

[AWS VPC & Subnets For Beginners](https://www.youtube.com/watch?v=TUTqYEZZUdc)

[Other Reference: Nginx reverse proxy](https://www.youtube.com/watch?v=_EBARqreeao)

Summarized by Monica and ChatGPT:

A VPC is a virtual networking environment that manages everything we do with networking on AWS. The configuration of the VPC determines how we can connect different pieces of our infrastructure together and how we can connect those pieces to the public internet. A VPC is an entire private network, and we specify the private IP address range that we will use within the VPC. Inside the VPC, we create subnets with their own private IP cider blocks. We place the VPC within a region, and each subnet is placed within an availability zone. We can deploy EC2 instances or other compute services on these subnets. By default, **all resources** within the VPC can **communicate** with each other, even across **different availability zones**. 

However, by default, a VPC is completely private, and resources within a VPC **cannot** communicate with **other VPCs** or **access the public internet**. We can use other services to allow other types of networking. The most common is an **internet gateway**, which allows open networking on the public internet. We can attach an internet gateway to some of the subnets to allow connections over the public internet. We can keep **some of our subnets private**, so the **load balancer** could be on the **public** subnets, but the **database and the application instances** are all on **private** subnets with no way of communicating with the outside world directly. If resources need to remain private on private subnets but also need to communicate with other things outside of the VPC, there are other services we can use like an S3 gateway to connect privately to S3 buckets or a NAT gateway to allow connections out of the private subnet without allowing public access to that.

The author's objective is to establish a secure AWS environment using Virtual Private Cloud (VPC) components, consisting of public and private subnets. He intends to deploy two instances: one running **Nginx** to serve as a reverse proxy and another for a **Node.js application** that interacts with an S3 bucket. The deployment follows a step-by-step process:

1. VPC Setup: The author begins by creating a new VPC, naming it "My Files App VPC". He defines an IPv4 CIDR block (IP address range) for the VPC, this new VPC ensures controlled network segmentation.

2. Subnet Creation: The author proceeds to create subnets within the VPC, one public and one private. The **public subnet** will house the **Nginx** instance (used as a **load balancer** and **reverse proxy**) and is allocated a specific IP range within the VPC's CIDR block. Similarly, the **private subnet** is established for the **Node.js** application, using a distinct IP range.

3. Nginx Instance: He launches an EC2 instance in the public subnet, configuring it with Amazon Linux 2 and a t2.micro instance type. The VPC is set to "My Files App VPC," and a public IP address is enabled for external access. Security groups are defined to permit SSH and HTTP traffic. This instance is tailored to serve as an Nginx reverse proxy.

4. Internet Gateway: The author recognizes that the public subnet requires internet connectivity. To enable this, he creates an Internet Gateway named "My File App Internet Gateway" and associate it with the VPC. This gateway will **facilitate communication** between instances in the **VPC and the public internet**.

5. Route Table Configuration: A new route table is generated, labeled "My File App Public Route Table." This table is linked to the public subnet and configured to route traffic through the Internet Gateway, allowing instances within the public subnet to access external resources. (By deault there is a route table that allows private networking)

6. Subnet Route Association: The route table association for the public subnet is modified to point to the newly created public route table. This step ensures that **instances in the public subnet can leverage the internet connectivity established by the Internet Gateway**.

7. Nginx Installation: The author installs nginx, and starts it. And opens the browser, inputting the amazon ip address to check if nginx landing page is seen, which means public internet setting is successful.

8. Private Node.js Instance: An EC2 instance is launched in the private subnet. This instance uses the "My Files App VPC" and does not have a public IP address. Security groups are configured to allow traffic from within the VPC. This instance is set up to run the Node.js application, simulating Dropbox-like functionality by interacting with an S3 bucket.

9. Nginx Configuration: The author modifies the Nginx configuration file on the Nginx instance, configuring it to act as a reverse proxy. This setup will enable the Nginx instance to **accept incoming traffic from the internet and forward it to the private Node.js application**. The Nginx instance is restarted after configuration changes. It now acts as a reverse proxy, receiving external traffic from the internet and forwarding it to the private Node.js application instance.

10. Functionality Testing: The author verifies the setup by attempting to access the Nginx landing page and the Node.js application over the public internet. The Nginx instance successfully forwards traffic to the private application, mimicking a secure and controlled file-sharing service.

11. Private Application Access: The author demonstrates that the private Node.js application can connect to an S3 bucket, although external access is restricted. He clarifies that the private instance remains inaccessible from the public internet while still being functional within the VPC.

12. Future Steps: The author hints at forthcoming videos discussing how to establish connectivity from a private subnet to external services and the internet, ensuring secure and controlled access while maintaining a private infrastructure.

In conclusion, the author meticulously configures a VPC environment on AWS, creating public and private subnets, deploying instances, and implementing a reverse proxy setup to facilitate secure communication between the public internet and a private Node.js application. The result is a controlled and secure infrastructure that allows external access while maintaining robust security measures.


### Up and Running with AWS VPC

[Up and Running with AWS VPC](https://cs.fyi/guide/up-and-running-with-aws-vpc)

#### CIDR

Default CIDR block for this default VPC. This CIDR block defines the range of IP addresses that can be assigned to the instances in the VPC. You can define the CIDR block for a VPC when you create it.:

```
172.31.0.0/16
```

#### Route tables 

Route tables tell the AWS and VPC **how to route** the traffic depending on the source and destination. A route table contains a set of rules, called routes, that are used to determine where **network traffic** from your **subnet** is directed. Each subnet in your VPC must be associated with a route table.

### VPC Basics Tutorial

[VPC Basics](https://cloudcasts.io/course/vpc-basics/what-is-a-vpc)

## S3

### User Guides

[S3 User Guides](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)

## More resources

### Sam Meech-Ward’s AWS Videos

[Sam Meech-Ward’s AWS Videos](https://www.youtube.com/playlist?list=PL0X6fGhFFNTcU-_MCPe9dkH6sqmgfhy_M)

### Udemy

[Intro to Cloud Computing on AWS for Beginners [2023]](https://www.udemy.com/course/introduction-to-cloud-computing-on-amazon-aws-for-beginners/)
