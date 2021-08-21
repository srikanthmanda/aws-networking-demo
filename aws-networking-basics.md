# AWS Networking Basics
## VPC

A block of IP addresses that can be used to group/administer various components in AWS as a single entity, from networking perspective.

+ Can be associated with one AWS region.
+ The components within the VPC can be hosted on either dedicated tenancy or shared tenancy.
+ Can be peered with other VPCs of non-overlapping IP addresses.
+ The [CIDR](https://cidr.xyz/) block size of a VPC's IP addresses can be between `/16` [netmask](http://unixwiz.net/techtips/netmask-ref.html) and `/28` netmask.

**Note**: Private IP addresses in [RFC 1918](http://www.faqs.org/rfcs/rfc1918.html) range are [recommended](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html#VPC_Sizing), though public IP addresses can be used too.

VPCs come with various components like Subnets (SN), Route Tables (RT), Network ACLs (NACL), Security Groups (SG), various Gateways and Endpoints. These components can be configured to group, connect and isolate resources like EC2 instances, EFS file systems, databases.

Reference: [What is Amazon VPC?](https://docs.aws.amazon.com/vpc/latest/userguide/what-is-amazon-vpc.html)

### Default VPC
Each AWS account comes with a default VPC in the default region.

Features:
+ Default NACL that allows all inbound and outbound traffic.
+ An Internet Gateway (IGW).
+ Default RT with a route to Internet, via the IGW.
+ Default SG that allows all traffic on all ports.
+ One SN in each AZ of the region, all associated with the default NACL and with the default RT (implicitly).
+ All the above components can be customised.
+ Auto public IP addresses are on for default SNs.

Reference: [Default VPC and default subnets](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html#detecting-platform)

### Custom VPC
While the default VPC can be customised, it might be more convenient to create a new VPC with stricter/secure configuration settings.

Custom VPC features:
+ Default NACL that allows all traffic.
+ Default RT with only local route, and no route to internet.
+ The first 4 and the last (broadcast) IP addresses of the VPC's CIDR range (5 in total) are reserved and are unusable.

Custom VPCs can be tailored by creating new NACLs, RTs, SNs, GWs as required.

### Network Access Control List (NACL)
NACLs are akin to barriers that allow only certain traffic in certain directions.

+ NACLs control traffic in and out of SNs associated with them.
+ NACLs are list of numbered rules which either allow or deny traffic of a [protocol](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml) over a range of port numbers.
+ Stateless: NACLs have separate rules for inbound traffic and outbound traffic.
+ Inbound rules specify traffic sources and outbound rules specify destinations.
+ Rules are evaluated in ascending order of priority and the traffic is either allowed or denied based on the first match.
+ A NACL can be associated with only one VPC.

Reference: [Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

#### Ephemeral Ports
[Ephemeral ports](https://en.wikipedia.org/wiki/Ephemeral_port) are temporary ports used by clients in a client-server connection. My understanding is that their use enables a client to communicate with multiple servers over different connections of same type (HTTP, HTTPs, etc) at the same time, by using different temporary ports.

Since NACLs work on port ranges, ephemeral ports should be considered while setting up both inbound and outbound traffic rules. 

For example, while allowing inbound HTTP traffic (i.e., requests) over port 80, outbound traffic (i.e., responses) should be allowed over ports in the range 32768 - 65535. This range covers [IANA range](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml) which is [adopted](https://docs.microsoft.com/en-US/troubleshoot/windows-server/networking/default-dynamic-port-range-tcpip-chang) by recent versions of Windows and the ephemeral port ranges of various Linux and Unix OS.

Another example: while allowing outbound HTTP requests from within VPC *to* port 80, inbound responses should be allowed over the ephemeral port range of NAT instances i.e., 1024-65535.

Reference: [Ephemeral Ports](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#nacl-ephemeral-ports)

### Route Table (RT)
Routes Tables are akin signboards at roundabouts telling which exits go where.

+ They are lists of routes associating destination IP addresses with targets.
+ Targets can be various types of Gateways or VPC endpoints.
+ Multiple SNs can be associated with an RT.
+ A RT can be associated with only one VPC.

Reference: [Route tables for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)

### Internet Gateway (IGW)
VPC component that enables communication with the internet.

+ An IG can be associated with only one VPC.
+ Once associated with a VPC, an IG can be added as a route target in RTs of that VPC.
+ Performs network address translation (NAT) for instances with public IP4 addresses.

Reference: [Internet Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)

### NAT Gateway (NGW)
Network Address Translation (NAT) Gateway allow connections from private SN to outside VPC while blocking connections in the opposite direction.

+ NGWs should be placed in the SNs that have a route out of the VPC (via Internet/Transit/Virtual Private Gateways).
+ Then they can added as a route target in RTs associated with private SNs.
+ _Public_ NGW allow communication with internet, in one direction, via IGW. They need an EIP. 
+ _Private_ NGW allow communication with other VPCs or on-premise network (again, only in one direction) via Transit Gateways and Virtual Private Gateways.

Reference: [NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)

### Subnet (SN)
A logical sub-grouping of a block of IP addresses within VPC's CIDR range.

+ Can be associated with one NACL and one RT of the VPC, for tailored network conditions/restrictions.
+ SNs not explicitly associated with any RT are associated with the VPC's default RT.
+ Similarly SNs not explicitly associated with any NACL are associated with the VPC's default NACL.
+ SNs are treated as public or private based on their association with an RT that has a route to Internet via an IG.
+ A SN can be associated with only one AZ of the VPC's region.

Reference:  [VPCs and subnets](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)

### Security Groups (SG)
Security groups are to instances (EC2 and DB) what NACLs are to SNs - they control traffic in and out instances they are associated with.

But they differ from NACLs in a few ways:
+ SG rules can only allow--not deny. However, absence of a rule allowing a specific traffic type would result in that traffic type being denied.
+ SG rules are stateful: Allowing a traffic type in one direction would result in it being allowed in the reverse direction too, even if there is no rule explicitly allowing it in that reverse direction.

Reference: [Security groups for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)

#code #aws