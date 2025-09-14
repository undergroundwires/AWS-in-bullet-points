# EC2 Administration

## Common EC2 Tasks

- **Hibernating**
  - Means persisting all of its data and RAM so you can pre-warm an instance before resuming.
- **Stopping and starting**
  - Most likely will migrate EC2 to new underlying host.
- **Retiring**:
  - An instance is scheduled to be **retired** when AWS detects irreparable failure of the underlying hardware hosting the instance.
  - When an instance reaches its scheduled retirement date
    - It is stopped (if it has EBS root volume) or
    - Terminated (if it has instance store root volume) by AWS

### Launching EC2

#### Launching EC2 using AWS Console

1. Go to EC2 -> Launch an instance -> Choose Amazon Machine Image (AMI)
2. Choose type for machine (e.g. `t2.nano`, `t2.micro`)
3. Configurations
   - Storage: EBS is where the data will be stored (SSD, HDD etc.)
   - You can have different tags
     - *Name* is important as it appears in UI.
4. Launch
   - For SSH you need to have key pair, you can generate new or use existing.

#### EC2 Launch templates

- Launch templates allow configuring and provisioning EC2 instances.
- Stores launch parameters so that you don't specify them every time you launch an instance.
  - E.g. AMI ID, instance type, key pairs and Security Groups.
- Useful for example Auto Scaling

#### EC2 Launch Agent

- Included in AWS Windows Server by default.
- PowerShell script that makes the computer AWS-friendly
- Helps configuring Windows machines to:
  - sends instance information to EC2 console,
  - sets computer name & wallpaper,
  - adds DNS suffixes,
  - executes user data,
  - provides persistence static routes to reach the metadata & KMS services
  - dynamically extends OS partition to include any unpartitioned space and more

### EC2 Connection

- SSH with e.g. `ssh -i privatekey.pem <user-name>@<ip-address>`
  - Username can be `ec2-user` (Amazon Linux, RHEL, SUSE, FreeBSD), `ubuntu` (Ubuntu, NanoStack), `admin` (Debian), `root` (RHEL, TurnKey, OmniOS), `fedora` (Fedora), `centos` (Centos), `bitnami` (BitNami)
  - IP address is the public IP address of the machine, you cannot use private IP to connect.
  - Private key `privatekey.pem` is enough as AWS puts public key automatically to `~/.ssh/authorized_keys`
- 💡 When you first download key-pair file permission is 0644 which is to open -> private key can leak as it's accessible by others.
  - 📝Fix with `chmod 0400 awsEc2.pem` -> only root user can see it.
  - 📝chmod 400 is for Linux only
- On Linux / Mac we use SSH, on Windows we use PuTTy (on Windows 10 SSH can be used as well)

### EC2 Termination

- Termination = deletion
- 💡 An instance may immediately terminate if:
  - You've reached your EBS volume limit
  - An EBS snapshot is corrupt
  - The root EBS volume is encrypted and you do not have permissions to access the KMS key for decryption
  - The instance store-backed AMI that is attached is missing a required part (an `image.part.xx` file)
- You can enable ***termination protection***
  - Prevents an instance from being accidentally terminated by requiring that you disable the protection before terminating the instance

## EC2 Remote Administration

### Bastion Host

- Also known as *Jump Boxes*
- Designed to withstand security attacks and usually hosts only a proxy server
- Used to administer EC2 (SSH/RDP)
  - Does not replace NAT gateway/instance as they are not used for SSH/RDP but to provide internet traffic to private subnets.
- The bastion is in the public subnet (DMZ) which is then connected to all other private subnets
- 💡 Small EC2 instances are enough as they do not require much processing power.
- Bastion Host security group must be tightened
  - 📝💡 Make sure the bastion host only has port 22 traffic from the IP you need, not from the security groups of your instances

### SSM Session Manager

- Also known as *AWS Systems Manager Session Manager*
- Allows you to remotely execute commands on managed hosts without using a bastion host
- 📝 Best-practice, replacing complexity of bastion hosts
  - You don't have to manage SSH keys.
- Free & serverless AWS managed service
- IAM Integration
  - Requires an IAM policy that allows users or roles to execute commands remotely.
  - Agents require an IAM role and policy that allow them to invoke the Systems Manager service.
    - The Systems Manager agent is an open-source executable that runs on supported versions of Linux and Windows
- Systems Manager immutably logs every executed command

## EC2 Hibernation

- Allows you to suspend your EC2 instances to disk and resume them later
- Key benefits:
  - Faster boot times compared to a cold start
  - Preserves in-memory state and running processes
  - Can help reduce costs since you're not charged for compute time while hibernated
- Hibernation flow:
  1. The instance's RAM contents are saved to the root EBS volume
  2. The instance shuts down
  3. When you start it again, the RAM contents are restored and applications resume from where they left off
- ❗📝 Cannot disable/enable on same instance.
  - Not available: stop → enable/disable hibernation → start
  - Hibernation must be enabled at launch time only
  - The hibernation setting is baked into the instance configuration when it's first created
  - Even when an instance is stopped, you cannot modify the hibernation setting.
  - To modify:
    1. Stop the instance
    2. Create an AMI (snapshot) of the instance
    3. Launch a new instance from that AMI with hibernation enabled
    4. Terminate the old instance
