# EC2 AMIs

- AMI = Amazon Machine Images
- AWS has base images for Ubuntu, Fedora, Red Hat, Windows, Amazon Linux etc.
- AMI can be found and published on the Amazon Marketplace
- **AMI Pricing**: Cheap as you only pay for space it takes in S3.
- AMI stores:
  - Boot volume
  - Data volumes
  - Permissions
  - Block Device Mapping

## Custom AMIs

- AMIs can be built for Linux/Windows machines.
- Removes need for EC2 user data with faster boot time.
- Can be created from volumes or snapshots.
- Can use pre-configured & maintained public AMIs from other people by paying by hour.
- ❗ AMIs are built for a specific AWS region
  - An AMI created for a region can only be seen in that region.
  - 📝 Copying makes it available in different regions for e.g. disaster recovery.
- ❗ Do not use AMI you don't trust, they may be insecure or contain malware.
- VM migrations
  - **VM Import/Export** allows you to import OVA file (or VMDK, VHD, or RAW) as AMI then create EC2 based on it.
  - 💡 Use [AWS Application Migration Service](./09-migrations-discovery-migration-hub-migration-evaluator-application-discovery-service-database-migration-service-application-migration-service-app2container-datasync.md#aws-application-migration-service) instead for lift-and-shift migrations.
- Permissions options:
  - Public access
  - Owner only
  - Specific AWS accounts

## AMI Storage

- S3 but you won't see them in the S3 console
- By default, your AMIs are private and locked for your account / region.
  - You can make AMIs public in order to share with other accounts or sell in AMI Marketplace.

### Common AMI tasks

- **Copying an AMI**
  - ❗ AWS does not batch copy following, need manual work:
    - User-defined tags
    - Launch permissions
    - S3 bucket permissions
- **Create AMI from EC2**
  - Right click on EC2 -> Image -> Create Image
  - When it's done it appears on EC2 dashboard -> Images -> AMIs
  - You can right click and
    - *Copy* it to another region
    - *Deregister* it in order to remove it.
    - In *image permissions*:
      - Make it publicly available, or share it with other AWS account numbers.
      - Create volume permissions option allows others accessing AMI to make a copy of the AMI.
    - *Launch* a new EC2 based on it.
- 📝**Cross account / cross region AMI Copy**
  - Share AMI with another AWS account
    - Sharing an AMI does not affect the ownership of the AMI.
      - ❗ However if you copy an AMI that has been shared with your account, you are the owner of the target AMI in your account.
    - Flow:
      1. Right click on EC2 -> Image Permissions
      2. Keep it private & add account number.
      3. Select *Add "create volume permissions" to snapshot* to allow other accounts to make a copy.
  - Ensure you have read permissions for AMI storage:
    - Associated EBS snapshot (for an Amazon EBS-backed AMI)
    - Associated S3 bucket (for an instance store-backed AMI)
  - ❗ You can't copy:
    - Encrypted AMI shared from another account
      - 💡 Instead you can copy the snapshot (if it's shared with encryption key) and re-encrypt it & register as a new AMI you own.
    - AMI with an associated **billingProduct** shared from another account
      - They're Windows AMIs and AMIs from the AWS marketplace
      - 💡 You can launch an EC2 instance in your account using the shared AMI -> Create an AMI from the instance

### AMI Virtualization Types

#### HVM

- HVM = Hardware Virtual Machine
- Utilizes special hardware extensions as it's on bare metal hardware.
- Enhanced Networking
- Only option for Windows.
- Current generation instance types support HVM only.

#### PV

- PV - Paravirtual Virtual Machine
- Only available for Linux for some previous generation instance types.
- 💡 HVM is recommended because same or better performance than paravirtual guests.
  - ***Before***: Better storage & network in PV VM because of special I/O drivers.
  - ***Now***: PV drivers are available for HVM VM (instead of emulating hardware)
