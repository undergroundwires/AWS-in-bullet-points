# Networking - IP Addresses

- AWS supports both types of IPs
  - IPv4 e.g. `1.160.10.240`
    - Allows 3.7 billion addresses that are almost running out today.
      - `[0-255].[0-255].[0-255].[0-255]`
  - IPv6 e.g. `3ffe:1900:4545:3:200:f8ff:fe21:67cf`
    - Newer & commonly used for IoT (internet of things) as it solves a lot of problems.
    - 💡 IPv6 are all public addresses (global unicast addresses)
- 💡❗ Use private IP whenever possible: Communication using Public, Elastic IP or Elastic Load Balancer inside EC2 network costs Data Transfer charges even in same AZ.
- **Bring Your Own IP Addresses (BYOIP)**
  - You can bring part or all of your public IPv4 address range from your on-premises network to your AWS account
  - You need to create a ROA (**Route Origin Authorization**)
    - A document that you can create that contains the address range, the ASNs that are allowed to advertise the address range, and an expiration date.

## CIDR (IPv4)

- Classless Inter-Domain Routing
- Used for Security Groups rules, or AWS networking in general
- ❗ Max CIDR size in AWS is `/16`
- CIDR has two components:
  - The base IP (`XX.XX.XX.XX`)
    - The base IP represents an IP contained in the range
  - The subnet mask (`/26`)
    - Defines how many bits can change in the IP
    - Can take two forms:
      1. `255.255.255.0`: Less common
      2. `/24`: More common
    - Basically allows part of the underlying IP to get additional values from the base IP

      | Range | Total Allowed IPs | Calculation |
      |:-----:|:-----------------:|:-----------:|
      | /32 | 1 IP | 2 ^ 0 |
      | /31 | 2 IP | 2 ^ 1 |
      | /30 | 4 IP | 2 ^ 2 |
      | /29 | 8 IP | 2 ^ 3 |
      | /28 | 16 IP | 2 ^ 4 |
      | /27 | 32 IP | 2 ^ 5 |
      | /26 | 64 IP | 2 ^ 6 |
      | /25 | 128 IP | 2 ^ 7 |
      | /24 | 256 IP | 2 ^ 8 |
      | /16  | 65,536 IP | 2 ^ 16 |
      | /0 | all IPs | 2 ^ 32 |

    - 📝 So just know that `32 - CIDR number = X power of 2`
      - E.g `192.168.0.0/24`:
        - `32 - 24 = 8` => `2 ^ 8 = 256` IP addresses
        - Between `192.168.0.0` - `192.168.0.255` (256 IP)
        - Decimal subnet mask = 255.255.255.**0**
      - E.g. `192.168.0.0/16`:
        - `32 - 16 = 16` => `2 ^ 16 = 65,536` IP addresses
        - Between `192.168.0.0` - `192.168.255.255`
        - Decimal subnet mask = 255.255.**0**.**0**
    - 💡📝 Memo
      - Each 8 points from 32 represents which IP number can change totally:
        - /32 - no IP number can change
        - /24 - last IP number can change
        - /16 - last IP two numbers can change
        - /8 - last IP three numbers can change
        - /0 - all IP numbers can change

## Public & Private IP

- **Private vs Public IP (IPv4)**
  - IANA had defined blocks of IPv4 addresses for private (LAN) and public (Internet) access
    - IANA: The Internet Assigned Numbers Authority
  - **Public IP**
    - Machine can be identified on internet (WWW)
    - Must be unique across the whole web (two machines cannot have same public IP)
    - Can be geo-located easily
  - **Private IP**
    - Machine can only be identified on a private network only
    - IP must be unique across the private network.
    - Machines connect to WWW using an ***internet gateway*** (a proxy)
    - Only a specified range of IPs can be used as private IP
      - `10.0.0.0` - `10.255.255.255` (`10.0.0.0/8`) <- in big networks
      - `172.16.0.0` - `172.31.255.255` (`172.16.0.0/12`) <- default AWS one
      - `192.168.0.0` - `192.168.255.255` (`192.168.0.0/16`) <- e.g. home networks
    - AWS registers 5 IP addresses per subnet, see [Subnets](./04-03-networking-vpc-subnets.md) for more information.
  - **Other reserved ranges**
    - **🤗Trivia** <sup>[StackOverflow](https://stackoverflow.com/questions/42314029/whats-special-about-169-254-169-254-ip-address-for-aws)</sup>
      - `169.254.0.0/16` exists only on the directly connected network.
      - It's a reserved IPv4 Link Local Address.
      - AWS choose its metadata server to make it reachable within this range (169.254.169.254).
    - `127.0.0.0` – `127.255.255.255` is used for addresses to the local host.

- **Elastic IP**
  - Fixed public IPv4 IP address for attaching to EC2 instances (to their Elastic Network Interfaces).
    - Because when you stop & start an EC2 instance, it'll change its public IP.
  - You own it as long as you don't delete it.
  - Can help you mask the failure of an instance by remapping the address to another instance.
  - You can attach it to a one instance at a time
  - ❗ Max 5 Elastic IP per account (soft limit)
  - 💡 Elastic IP reflects often poor architectural decisions
    - Horizontal scaling becomes much harder
    - Instead you can:
      - Use a random public IP and register a DNS name to it.
      - Load balancer (best pattern).
  - **Pricing**
    - Elastic IPs are free as long as
      - They are being used by an instance i.e. attached to instance and instance is running.
        - ❗ Amazon will charge you for each EIP that you reserve and do not use (check current AWS pricing for rates).
      - The instance has only one Elastic IP address attached to it.
    - ❗ You will be charged if you ever remap an EIP more than 100 times in a month.

## Essential Ports

- Web Traffic
  - `80` - HTTP (unencrypted web traffic)
  - `443` - HTTPS (encrypted web traffic, SSL/TLS)
- Remote Access:
  - `22` - SSH (Secure Shell for Linux/Unix systems)
  - `3389` - RDP (Remote Desktop Protocol for Windows)
- Database Ports:
  - `3306` - MySQL/Aurora MySQL
  - `5432` - PostgreSQL/Aurora PostgreSQL
  - `1433` - Microsoft SQL Server
  - `1521` - Oracle Database
- Other Important Ports:
  - `53` - DNS (Domain Name System)
  - `25` - SMTP (Simple Mail Transfer Protocol)
  - `110` - POP3 (Post Office Protocol)
  - `143` - IMAP (Internet Message Access Protocol)
  - `21` - FTP (File Transfer Protocol)
  - `20` - FTP Data Transfer
