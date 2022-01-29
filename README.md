# aws-certification-notes

Notes for myself, focused on the Solutions Architect Professional Exam.

# Table of Contents
- [aws-certification-notes](#aws-certification-notes)
- [AWS Accounts](#aws-accounts)
  * [Budgets](#budgets)
- [IAM](#iam)
  * [Policy Evaluation](#policy-evaluation)
  * [Advanced Identities and Federation](#advanced-identities-and-federation)
    + [SAML 2.0 Identity Federation](#saml-20-identity-federation)
    + [AWS SSO](#aws-sso)
- [Amazon Cognito](#amazon-cognito)
- [AWS Organizations](#aws-organizations)
- [AWS Resources Access Manager (RAM)](#aws-resources-access-manager-ram)
- [Service Quotas](#service-quotas)
- [Amazon Workspaces](#amazon-workspaces)
- [AWS Directory Service](#aws-directory-service)
- [Advanced VPCs](#advanced-vpcs)
  * [DHCP](#dhcp)
- [Encryption Fundamentals](#encryption-fundamentals)

# AWS Accounts

* Billing & Cost Management Dashboard or “My Billing Dashboard” has quick links to billing and cost management products.
  * Can show month-to-month spending and forecast the current month spend. This includes free-tier usage.
  * Under “billing preferences” you can enable PDF invoices via email, free tier usage alerts, and billing alerts.
  * Cost Explorer

## Budgets
* Need to enable Cost Explorer first to enable all budgets. The basic cost budget only works against dollar amounts.
* You can specify a budget period
* Daily budgets do not support budget planning or forecast alerts
* You can choose to budget a Fixed amount. This tracks against a single month, regardless of the configured period.
* You can specify filters like Usage Type (egress), Service, Linked Account (if organization), Purchase Option, etc.
* You can also create up to 5 different alerts based on the budgeted amount.
* Alerts can go to an email, SNS topic, or Amazon Chatbot (includes Slack and Chime)
* Budget Actions let you define an action to occur when a budget threshold (actual or forecasted) is exceeded. Action types include IAM policies, SCPs, or target running instances (EC2 or RDS).
* Actions can run immediately or execute a request on your behalf. Each budget threshold can have up to 10 actions associated with it.
* IAM and SCP action types reset at the beginning of each budgeted period.

# IAM
  * IAM Access Analyzer can identify resources (e.g. S3 buckets, IAM roles) that are shared with an external entity.
  * At role creation time, you specify what type of entities will use this role. AWS service, other AWS account, web identity (Cognito or any OpenID provider), SAML 2.0 federation
  * An IAM role has three places where it uses policies
    * Permissions policies (inline and attached) - define permissions that the assuming principal is able (or restricted to perform and on what resources
    * Permissions boundary - another layer of security on top of a user/roles permission policy. If the permissions boundary exists, then it must specify which actions the entity is allowed to perform. That is, they don’t GRANT any access, only define the maximum set of permissions that can be received. For a user/role to be able to do something, both the permission policy _and_ the permissions boundary must allow for it. Permissions boundaries only affect identity permissions, not resource permissions.
      * One use case for this is creating a boundary that affects all users. Then create another boundary that affects admins stating that all users they create must be affecting with the standard boundary.
      ![image](https://user-images.githubusercontent.com/9027551/151602642-aede770d-0c44-4be4-bace-a6a39e3ef572.png)

    * Trust relationship (trust policy)- defines which principals can assume the role, and under which conditions.
  * In the event that IAM role credentials are leaked, you can update the permissions policy with a `AWSRevokeOlderSessions` inline DENY for any sessions older than `<current time>`

## Policy Evaluation

![chrome_4gcB68QWDJ](https://user-images.githubusercontent.com/9027551/151603608-8767c287-f539-4c39-9412-f000b3264af8.png)
* The interesting thing to note here is the short-circuiting that resource policies perform here.
* When accessing resources (e.g. S3 bucket) is another account, we perform the same evaluation logic, but the other account must _also_ allow access.

## Advanced Identities and Federation

### SAML 2.0 Identity Federation
![image](https://user-images.githubusercontent.com/9027551/151604891-7ba378fe-b7c6-441b-b569-886e5bf3262c.png)
* The idea is pretty similar for AWS Console Access. The SAML assertion goes straight to an assertion Sign-In URL within the SAML/SSO endpoint, which gives back another sign-in url.
* Remember that only AWS Creds can be used to access AWS resources. Any other sort of identifying token (SAML 2.0 assertion, Google Token, JWT from user pool, etc.) must be exchanged for AWS creds before we can interact with AWS.

### AWS SSO
* More-or-less replaces direct SAML 2.0 identity federation.
* Manage SSO Access to AWS Accounts and External Applications
* Identity store implementation is abstracted, options:
  * Built-in identity store (provides additional benefits of cross-account permissions)
  * AWS Managed Microsoft AD
  * On-premises Microsoft AD (Two way trust or AD Connector)
  * External Identity Provider that implements SAML2.0
* You can configure various MFA options like
  * When to prompt users for MFA
  * What users can authenticate with
  * Provide a means for users for sign in when they don’t yet have an MFA device (skip MFA, require to register after first sign in, block sign in, require emailed code)
  * Deny sign in without MFA
  * Who can add/manage MFA devices
  * Lets you apply handy managed policies to users, grouping them into sets
* No inherent cost.

![image](https://user-images.githubusercontent.com/9027551/151605579-a28d0559-8b6f-4648-881a-d69b1ff958a0.png)

# Amazon Cognito

* Authentication, Authorization, and user management for web/mobile apps (not workforce)
* Don’t confuse user pools and identity pools!
* User pool - Sign-in and get a JSON Web Token (not IAM creds!)
    * User directory management and profiles, sign-up & sign-in (customizable web UI), MFA and other security features
    * Provides a joined-up user management experience. Does not authorize use of most AWS resources (except API Gateway!)
* Identity pool - allows you to offer access to Temporary AWS Credentials
  * Swap OpenID provider’s token for an IAM role creds
  * Or allow access without authentication
  * Recall that API Gateway can accept JWTs directly
* User pools and Identity pools can be combined (Web identity federation)

![image](https://user-images.githubusercontent.com/9027551/151606046-68e0ce46-00fe-4115-a295-4be1b7392e4a.png)


# AWS Organizations
  * Existing Accounts must be invited.
  * New accounts can be created from directly within Organizations. These accounts will have an email, but will not come with a password. At creation time, you will be asked for an IAM role name. The resulting IAM role can be used by the management account to access the new account’s resources.

# AWS Resources Access Manager (RAM)

* Alternative to VPC-peering.
* Allows sharing of AWS resources between AWS Accounts. Some resources can be shared across any principal (accounts, OUs, or organization) while some can only be shared between accounts within the same organization.
* Shared resources can be accessed natively.
* No charge.
* Note: AWS rotates which physical facilities are used for AZs. That is, my us-east-1a might not be the same as your us-east-1a. This makes coordinating resources between accounts from a performance or HA perspective difficult.
  * Introduce AZ IDs (ex. use1-az1 and usee1-az2). These _are_ consistent between accounts.
* How it works: Owner account creates a share, providing a name
  * Owner account retains full ownership, non-owners get a limited set of permissions.
  * Defines the principal with whom to share
  * If the participant account is inside an organization with sharing enabled, its accepted automatically. Otherwise, invites must be accepted.
  * Participant accounts can create resources in a subnet shared by the owner, but the participant still owns those resources

![image](https://user-images.githubusercontent.com/9027551/151604324-46dfdd4d-4257-42ea-8d66-0b9a2f9b009a.png)

# Service Quotas

* Each service has a default per-region quota
* Some services have a per account quota
* Most service quotas can be increased as needed
* Some can’t like the number of IAM users in an account (5000)
* Higher increases = more process & time needed (plan the time for it)
* Can request service quota increases through Service Quotas, can also do it from Support Center
* Can create cloudwatch alarms based on percentage of the quota value

# Amazon Workspaces
* Desktop as a Service product. Similar to Citrix.
* With or without custom images and AWS related apps
* Monthly or Hourly pricing (plus base infrastructure cost!)
* Uses Directory Service for authentication and user management.
* Workspaces run in an AWS managed VPC, but inject ENIs into customer-managed VPCs
* Connection bandwidth for maintaining a remote desktop connection is at no extra cost.
* System volume and user volume, both are based on EBS, can be encrypted with CMKs
* Susceptible to AZ-failure since individuals VMs exist within a single AZ.

![image](https://user-images.githubusercontent.com/9027551/151606314-227da43e-da01-45f5-b4a4-6054f816658d.png)

# AWS Directory Service
* Managed Microsoft AD option
  * Injected into customer VPC (deployed across 2+ AZs)
  * Includes monitoring, recovery, replication, snapshots and maintenance performed by AWS
  * Can trust on-prem AD
  * Continues to work even if connection to on-prem fails
![image](https://user-images.githubusercontent.com/9027551/151606586-a3cd8bf7-190c-4bf1-8053-1234813097ac.png)

* AD Connector
  * Provides a pair of directory endpoints running in AWS (ENIs in a VPC)
  * Supports directory-aware AWS products
  * Redirects directory-related request within AWS to some external AD service
  * Like Managed Microsoft AD mode, requires 2 subnets within a VPC that are in different AZs

![image](https://user-images.githubusercontent.com/9027551/151606864-33b73327-32e8-4feb-aa39-a608480fc769.png)

# Advanced VPCs

## DHCP
* Concept: automatic configuration of network resources
  * Starts with a Layer2 broadcast to get info from DHCP server
  * Gives back your IP address, subnet mask, and default gateway
  * Tells you which DNS Servers to use and allows custom domain name within subnet
  * Configures NTP Servers for time synchronization
  * You can provide your own DNS names (need custom DNS server) or use the Amazon-provided public and private ones
  * In the event you want to roll your own custom DNS servers, you do this with DHCP option sets
![image](https://user-images.githubusercontent.com/9027551/151608085-15d7c76c-1cfe-4a9a-a1b7-183fceec364d.png)

## VPC Router
* Every VPC is created with a Main route table (RT) - the default RT for every subnet in the VPC
* Custom route tables be created and associated with subnets in the VPC - removing the association between those subnets and the main RT
* Keep in mind that subnets are only associated with exactly one RT. This means that deassociating a custom RT retores the association with the main RT
* Routes can be associated with IPs or AWS resources (including gateways)

## NACLs
* Affect subnets, not components within
* Rules can only target IPs/CIDR, ports and protocols, not logical resources
* Stateless
* Rules are processed by rule number order, rather than by specificity
* The default NACL is allow all
* Custom NACL by default is deny all
* A NACL can be associated with multiple subnets, but a sunbet can only have one NACL

## Security Groups (SGs)
* Stateful
* No explicity deny, meaning SGs can't be used to block specific IPs
* Unlike NACLs, also supports logical resources, including SGs and gateways

## Border Gateway Protocol (high level)
* Useful for dynamic route discovery
* DX and Dynamic VPNs utilize this
* Composed of Autonomous Systems (AS) -- single blackbox networks. 
 * Each is assigned an ASN. Public and private ASNs are available.
* Operates over tcp/179
* Peering between AS's is manually configured
* After being peered, AS's will communicate network topology and communicate its findings with its other peers. Specifically it communicates shortest paths (by AS hops).

## VPNs
* AWS Site-to-Site VPN is a logical connection between a VPC and an on-premises network ecrypted using IPSec
* VPN connections and VPGW are limited to 1.25Gbps.
![image](https://user-images.githubusercontent.com/9027551/151620999-1572c3c4-373c-4f06-84bd-a655171e7537.png)
* Accelerated Site-to-Site VPN has a fixed accerlator cost plus a transfer fee. Also can only be used with TGW attachments, not VGWs.

# Transit Gateway
* Can be attached to VPCs, VPGWs (site-to-site vpn), and DXGWs.
* Has a route table which, by default, contains all BGP-learned and VPC routes. This means all attachments can communicate to all other attachments!
* You can peer TGWs with other TGWs. However, they won't share routes. Use static routes to achievement communication. Inter-region and account possible. Much like VPC peering, traffic is encrypted over the AWS network.
* You can configure additional route tables. Specifically, you can configure specific attachements to 
 * propogate connections to these route tables and/or
 * use these route tables
* Route tables can be assigned to VGWs and IGWs too! Ex. rerouting all incoming traffic to a security appliance

# Direct Connect
* Private VIF - connects a single VGW to a DX Gateway
* Private VIF => DX Gateway => Transit Gateways = conenct private VIF to multiple VPCs in multiple regions
* Remember that DXGW can only be used for VPCS & Private VIFs ... OR ... TGW and a Transit VIF .. NOT BOTH
* Each transit VIF supports up to 3 TGWs

# Encryption Fundamentals
  * Symmetric Encryption -- Both parties use the same algorithm and secret key to encrypt/decrypt data. 
  * Asymmetric encryption
    * Both parties agree on an algorithm.
    * Both parties create public and private keys for two-way communication.
    * Public key is used to generate ciphertext. Public key cannot decrypt. Only the corresponding private key can decrypt.
    * Party 2 uses party 1’s public key to generate ciphertext. Party 1 can then decrypt using its own private key.
    * Usually a single key is communicated and agreed upon using asymmetric encryption so that the connection can be switched to using symmetric encryption (part of SSL/TLS)
  * Signing - Party 2 uses its private key to sign a message. Party 1 uses Party 2’s public key to confirm that message actually came from party 2.

# Challenges
* Can you evole a monolithic Wordpress architecture to be HA?
* Can you design and implement hybrid DNS?
* Can you use Systems Manager to keep EC2 instances patched?
* Can you design a serverless application?
* Do you understand how to set up web identity federation? Understand the differences between user pools and identity pools?
* Can you implement a VPN that connects on-prem and AWS networks?
* Can you set up VPC routing between 
 * A VPC in one region containing two subnets, A and B
 * Two other VPCs that have the same CIDR range, VPC B and C
 Such that subnet A can contact VPC-B and subnet B can contact VPC-C? What if subnet B needs to be able to route to a specific address in VPC-B? See Advanced VPC Routing part 2 for answer.
* Draw the architecture for a Client VPN where
 * clients should be able to connect to both private IPs on the host VPC (VPC A)
 * clients should be able to connect to other IPs on the public internet
 * clients should be able to connect to a second VPC, VPC-B
 * (part 2) Say we have a second client computer on the same network. How can we snure that the first client device can connect to the second without having to (more slowly) go over the VPN?
 * See AWS Client VPN video for solution
* Draw a basic DX architecture that connects a single VPC to a single on premises network. After that, how can we secure the connect such that the provider running the DX location is unable to access traffic sent through the location? 

# TODO: Make sure everything discusses cost where relevant
