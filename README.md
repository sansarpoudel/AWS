# AWS

# AWS Networking

This documentation will go over the different components of AWS Networking.

### Amazon VPC

Amazon Virtual Private Cloud (Amazon VPC) lets you provision a logically isolated section of the AWS Cloud where you can launch AWS resources in a virtual network that you define. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways. You can use both IPv4 and IPv6 in your VPC for secure and easy access to resources and applications.

You can easily customize the network configuration for your Amazon VPC. For example, you can create a public-facing subnet for your web servers that has access to the Internet, and place your backend systems such as databases or application servers in a private-facing subnet with no Internet access. You can leverage multiple layers of security, including security groups and network access control lists, to help control access to Amazon EC2 instances in each subnet.

Additionally, you can create a Hardware Virtual Private Network (VPN) connection between your corporate data center and your VPC and leverage the AWS Cloud as an extension of your corporate data center.

### Availability Zones

Each AWS region can have multiple availability zones. Availability Zones can be considered as a seperate datacenter. Due to multiple availability zones in one region, the chances for an instance to fail is minimized.  If you distribute your instances across multiple Availability Zones and one instance fails, you can design your application so that an instance in another Availability Zone can handle requests.When an instance is launched, we can select an Availability Zone or let AWS choose one for us automatically. 

You can also use Elastic IP addresses to mask the failure of an instance in one Availability Zone by rapidly remapping the address to an instance in another Availability Zone. For more information, see Elastic IP Addresses.

### Subnets

A subnet is an identifiably separate part of an organization's network. Typically, a subnet may represent all the machines at one geographic location, in one building, or on the same local area network (LAN). Having an organization's network divided into subnets allows it to be connected to the Internet with a single shared network address. Without subnets, an organization could get multiple connections to the Internet, one for each of its physically separate subnetworks, but this would require an unnecessary use of the limited number of network numbers the Internet has to assign. It would also require that Internet routing tables on gateways outside the organization would need to know about and have to manage routing that could and should be handled within an organization.

Subnet in AWS is a subset of a VPC. Each VPC can have multiple subnets. One or more subnets in each Availability Zone can be created. When you create a subnet, you specify the CIDR block for the subnet, which is a subset of the VPC CIDR block. Each subnet must reside entirely within one Availability Zone and cannot span zones. Availability Zones are distinct locations that are engineered to be isolated from failures in other Availability Zones. By launching instances in separate Availability Zones, you can protect your applications from the failure of a single location. 

Subnets can be internal or external. For example, our DMZ Subnet is external while App or DB Subnet is internal. 

Please see this [diagram](https://docs.aws.amazon.com/AmazonVPC/latest/GettingStartedGuide/images/getting-started-1-diagram.png) to better undertand the concept of VPC, Availibility Zones, and Subnets. 

### VPC Peering

VPC Peering is the networking connection between two VPCs that enables us to route traffic between them.  VPC peering can be between CmEng and HiEng. Also, there is a VPC Peering between CMProd and HiProd. This [diagram](https://docs.aws.amazon.com/AmazonVPC/latest/PeeringGuide/images/peering-intro-diagram.png) shows VPC Peering between two VPC.

### Internet Gateways

An Internet gateway is a VPC component that allows communication between instances in your VPC and the Internet. It therefore imposes no availability risks or bandwidth constraints on your network traffic.

An Internet gateway serves two purposes: to provide a target in your VPC route tables for Internet-routable traffic, and to perform network address translation (NAT) for instances that have been assigned public IPv4 addresses.

An Internet gateway supports IPv4 and IPv6 traffic.

### NAT 

NAT enables instances in a private subnet to connect to the internet or other AWS services, but prevent the internet from initiating connections with the instances. A NAT device forwards traffic from the instances in the private subnet to the Internet or other AWS services, and then sends the response back to the instances. When traffic goes to the Internet, the source IPv4 address is replaced with the NAT device’s address and similarly, when the response traffic goes to those instances, the NAT device translates the address back to those instances’ private IPv4 addresses.

AWS offers two kinds of NAT devices.
1. NAT gateway
2. NAT instance

AWS recommends NAT gateways, as they provide better availability and bandwidth over NAT instances.

### Route Tables

A route table contains a set of rules, called routes, that are used to determine where network traffic is directed.

Each subnet in your VPC must be associated with a route table; the table controls the routing for the subnet. A subnet can only be associated with one route table at a time, but you can associate multiple subnets with the same route table.

Each VPC comes up with a main route table that can be modified if needed.

### Elastic IP

An Elastic IP address is a public IPv4 address, which is reachable from the internet. If your instance does not have a public IPv4 address, you can associate an Elastic IP address with your instance to enable communication with the internet. We have used elastic ip adress for our Postfix servers, Network Load Balancer, and NAT Gateways.We use Cloudformation templates to reserve elastic ip. Take a look at this [cloudformation Template](https://github.cerner.com/HealtheLife/healthelife_aws-repo/blob/master/cloudformation_templates/postfix_nlb_external_template.json#L55-L60) to understand how we have specified to use elastic ip.

### VPN (Virtual Private Network)

We have the option to connect to the Amazon VPC over VPN to make the connection more secure. It is like connecting to Cerner Network from your home using cerner VPN. We currently have the VPN connection between our CA CHO Clients and AWS VPC. We also have the VPN connection between Cerner Data Centers and AWS VPC. 

### Hosted Zones 

A hosted zone is a collection of records for a specified domain. You create a hosted zone for a domain (such as example.com), and then you create records to tell the Domain Name System how you want traffic to be routed for that domain.

In Order to create the Hosted Zone, you will need to submit a PR to the [ets_alpha_cloudformation repo](https://github.cerner.com/ETS-Alpha/ets_alpha_cloudformation). Under this repo, there are different folders for different region. Each region has its own config.yml file. You will submit a PR to add the information about your zone on the specific config.yml file. 

Here is the [example](https://github.cerner.com/ETS-Alpha/ets_alpha_cloudformation/pull/280/files) of how wasdev.camh.ca was added to the config.yml file under cerner-healtheintent-ets-ca. We had to create a hosted zone called wasdev.camh.ca instead of just camh.ca because Client's EAWas url was named wasdev.camh.ca which isn't publicly resolvable.  If we create dns zone camh.ca then any requests for camh.ca will only go to the private zone. Therefore, we must create the exact name of the record to the zone.

Please, follow this guidelines below before submitting the PR to [ets_alpha_cloudformation repo](https://github.cerner.com/ETS-Alpha/ets_alpha_cloudformation).

|Region | Account Name |submit a PR to config.yml under this folder | 
|---|---|---|
|us-west-2|HiEng|cerner-healtheintent-ets-staging|
|us-west-2|HiProd|cerner-healtheintent-ets-us|
|ca-central-1|HiProdCA|cerner-healtheintent-ets-ca|
|ap-southeast-2|HiProdAus|cerner-healtheintent-ets-au|


You will also need to submit another PR to [ets_alpha_aws_dns repo](https://github.cerner.com/ETS-Alpha/ets_alpha_aws_dns) to update the DNS records. 

Please take this [PR](https://github.cerner.com/ETS-Alpha/ets_alpha_cloudformation/pull/280/files) as a reference.

Also, follow this guidelines below before submitting the PR.

|Region | Account Name |submit a PR under this folder | 
|---|---|---|
|us-west-2|HiEng|cerner-hieng|
|us-west-2|HiProd|cerner-hiprod|
|ca-central-1|HiProdCA|cerner-hiprodca|
|ap-southeast-2|HiProdAus|cerner-hiprodaus|

Once the PR is submitted, Rundeck job has to be executed to make the changes. Rundeck job is currently configured to run every 5 minutes. So, you should not worry about running it manually. But, if you see the zone is not created within AWS console within next 10-15 minutes, please login to the following Rundeck instance and execute the job manually.

#### Execute rundeck job Healtheintent DNS

1) Login to the Canada Jumpgate (https://ssh.ca.cernerasp.cernerops.com)
2) go to https://rundeck.ca.healtheintent.net/project/aws-hiprod-ca/job/show/652b9d1d-8932-4b5a-944c-8d420c674f1b
3) Ensure the Rundeck job Healtheintent DNS is setup for hiprodca
4) Run the Healtheintent DNS to upload the records.


