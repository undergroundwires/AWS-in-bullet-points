# Amazon S3 Overview

- Key / value store for objects
  - 💡 Great for big objects, not so great for small objects (latency)
  - It's object storage
    - Object => You rewrite entire object
    - Block store => You can just send changes to change the blob, can update incrementally as data is split into chunks.
- S3 is a global service (not region specific)
  - ❗ A bucket however is attached to a region, bucket is a regional-service.
- Supports IPv6, IPv4.
- 💡 Use cases: Static files, Key value store for big files e.g. movies, Website hosting

## S3 Objects

- Actual files, stored in [buckets](#s3-buckets) (directories)
- Object properties
  - Have a **key** which is FULL path
    - e.g. `<my_bucket>/my_file.txt` or `<my_bucket>/my_folder1_another_folder/my_file.txt`
    - ❗ There are no concepts of directories within buckets just key names with slashes.
  - ***Object values*** are the contents of the file
    - ❗ Max size is 5 TB, min size is 0 bytes (touched)
    - ❗ If file is more than 5 GB, you need to use multi part upload
    - ***Metadata*** is list of text key/value pairs.
      - Can be system or user metadata.
      - ❗ Cannot be modified after creation
  - ***Tags*** are unicode key/value pairs.
    - ❗ Up to 10
    - 💡 Useful for security / lifecycle / replication rules
- **Make an object publicly accessible**
  - On object level
    - Set a predefined grant (also known as a canned ACL) for a bucket in Amazon S3.
      - In Console -> Bucket -> Permissions -> Public access settings -> Untick *"Block new public bucket policies"* and *"Block public and cross-account access if the bucket has public policies"*
    - Update the object's ACL -> Ensure your object is set to be publicly accessible.
  - On bucket level
    - Use a [bucket policy](#s3-bucket-policy) that grants public read access to a specific object tag/prefix

### S3 Object Lock

- 📝 Help prevent Amazon S3 objects from being deleted or overwritten
- 📝 Lock types:
  - Retention period: Fixed amount of time or indefinitely
  - Legal hold: Lock object indefinitely
- ❗ Requires [S3 Versioning](#s3-versioning) to be enabled on bucket

## S3 Buckets

- ❗ Bucket names must be unique across all existing bucket names in Amazon S3.
- Buckets are defined at the region level and must have globally unique name as it'll follow URLs
  - `https://s3-<region>.amazonaws.com/<bucket>/<object>`
  - `https://<bucket>.s3.amazonaws.com/<object>` (virtual host style addressing)
- You can set default encryption
  - Options are: *None*, *AES-256* (SSE-S3) and *AWS-KMS* (SSE-KMS)
  - 📝Popular question: How to ensure that all objects can be encrypted at upload?
- ❗ 100 S3 buckets per account -> soft limit (can be increased using support request)
- **Folders**
  - Can be created under buckets
  - ❗ They are pseudo and will be treated as object by S3
    - S3 has a flat structure with no hierarchy
    - But presentation in console gives the sense of "Directories"

### S3 Versioning

- Can enable versioning at bucket level
  - ❗📝 Once enabled, it cannot be disabled but just suspended.
- Same key overwrite will increment the version
- Best practice as it allows to
  - Roll back to a previous version
  - Protection against **accidental deletion** i.e. unintended deletes.
    - Deleting files does not really delete versioned files.
    - It only puts **Delete marker** and files can still be downloaded.
    - Deleting a delete marker -> Restores the object & adds a new delete marker.
      - ❗ Only the owner of a bucket can permanently remove a delete marker e.g. delete a version.
    - 💡 Best practice: Allow only specific users to delete & enforce MFA
- 📝 Any file that is not versioned prior to enabling versioning will have the version `null`.

### S3 Bucket Policy

- Permissions applies to all objects in the bucket
- Resource-based policy: Who can access me, see [IAM Policies](./02-01-security-iam.md#iam-policies)
- Only the *bucket owner* can associate a policy with a bucket
- Use-cases:
  - Grants other accounts cross-account permissions to upload objects to your S3 bucket
  - Maintain full *bucket owner* control of uploaded objects
- ❗ Can't prevent deletions or transitions by an S3 Lifecycle rule

## S3 Event Notifications

- **Events**
  - Object created, object removed, object restored, object lost
  - Works on data plane as opposed to [RDS Event Notifications](./06-02-01-data-databases-rds.md#rds-event-notifications)
- **Destinations**
  - Determines where events will be published
  - Can be:
    - [SNS](./08-01-02-event-broadcasting-sns-eventbridge.md#amazon-sns-simple-notification-service)
    - [SQS](./08-01-02-integrations-queues-sqs.md)
    - [Lambda](./05-04-compute-lambda.md)

## S3 Lifecycle Policies

- Set of rules to automating moving data between different tiers, to save storage cost.
- **Transition actions**
  - When objects are transitioned to another storage class
  - E.g. General Purpose *--(60 days)-->* Infrequent Access *--(180 days)-->* [Glacier](#amazon-s3-glacier)
  - ❗ Objects must be stored at least 30 days in standard for standard to => Standard IA or OneZone IA
- **Expiration actions**
  - Expire & delete an object after a certain time period.
  - E.g. access log files can be set to delete after specified period of time
- You can combine transition & expiration actions.
- You can filter the resources by a key name prefix, e.g. `tax/` will only include `tax/doc1.txt`.
- You can configure it to
  - Clean-up expired object delete markers
  - Affect "Current version" and/or "Previous version"

## S3 Logging

- Object-level logging
- Can enable CloudWatch request metrics: e.g. total number of HTTP GET/PUT, 4XX etc.
- Can enable CloudTrail for auditing API calls and call stacks (not CloudWatch).

## S3 Consistency Model

- 📝 **Strong consistency** for all S3 operations (as of December 2020)
- Read-after-write consistency for all PUT and DELETE operations
- List operations are also strongly consistent
- No eventual consistency delays - data is immediately consistent
- Examples:
  - `PUT 200` -> `GET 200` (immediately consistent)
  - `DELETE 200` -> `GET 404` (immediately consistent)
  - `PUT 200` -> `PUT 200` -> `GET 200` (returns latest version immediately)

## S3 Pricing

- Charged mainly based on
  - Storage (per GB)
    - See [S3 storage classes](#s3-storage-classes)
  - Requests
    - Fee for each request made to S3 APIs for managing objects.
      - 💡 Usually very cheap, but can be a good idea to minimize the requests to optimize costs.
    - See also [Glacier](#amazon-s3-glacier) for Glacier retrieval fees.
  - [Data transfer](./01-02-01-aws-management-economics-pricing-pricing-calculator-data-transfer-savings-plans.md#data-transfer-charges)
  - Optional fees for region replication, transfer acceleration, management, and analytics
- See also:
  - [S3 pricing page](https://aws.amazon.com/s3/pricing/)
  - [AWS Pricing Calculator](./01-02-01-aws-management-economics-pricing-pricing-calculator-data-transfer-savings-plans.md#aws-pricing-calculator)

### Requester pays

- Requester pays for requests + storage in bucket instead of bucket owner.
- Requester is from AWS (not anonymous) & sends `requester-pays` flag and billed by AWS.

## S3 Storage Classes

- The storage class is assigned to individual objects within a bucket, not to the bucket as a whole.

- **S3 Storage Tiers**

  | Storage | Description | Use-cases | Retrieval latency | Cost |
  | ------- | ----------- | --------- | ----------------- | ---- |
  | **S3 Standard** | General purpose | Big Data analytics, mobile & gaming applications, content distribution... | ms | 💲💲💲💲💲💲 |
  | **S3 Standard IA** | Infrequent access | For data that's less frequently accessed but requires rapid access e.g. you'll get a file directly every month. | ms | 💲💲💲💲 |
  | **S3 One Zone-IA** | Low latency & high throughput performance | Storing secondary backup copies of on-premise data or storing data you can recreate over time | ms | 💲💲💲 |
  | **S3 Intelligent Tiering** | 📝 ML moves objects between *Standard IA* and *Standard* tiers based on changing access patterns | Unpredictable workloads, Changing access patterns, Lack of experience with storage optimization | ms | *(small)* Monitoring + auto-tiering fee |
  | **S3 Glacier Instant Retrieval** | Millisecond retrieval for archive data | Long-term data accessed quarterly (medical images, media assets) | ms | 💲💲💲 |
  | **S3 Glacier Flexible Retrieval** | Low cost cold storage | DR Back-ups, Media Asset Archival | Expedited: 1-5 min, Standard: 3-5 hr, Bulk: 5-12 hr | 💲💲 + retrieval fee |
  | **S3 Glacier Deep Archive** | Lowest cost coldest storage | Magnetic tapes / VTL / Regulatory archiving | 12-48 hours | 💲 + retrieval fee |
  | 🤗 **Reduced Redundancy Storage (RRS)** | [DISCONTINUED 2018], don't use it, use One Zone-IA | Noncritical, reproducible data at lower levels of redundancy than Amazon S3's standard storage | ms | 💲💲💲 |

- **S3 Storage Tiers Reliability**

  | | S3 Standard | S3 Standard IA | S3 One Zone-IA | S3 Intelligent Tiering | S3 Glacier Instant Retrieval | S3 Glacier Flexible Retrieval | S3 Glacier Deep Archive |
  |--|--|--|--|--|--|--|--|
  | **Durability** | 99.999999999% | 99.999999999% | 99.999999999% | 99.999999999% | 99.999999999% | 99.999999999% | 99.999999999% |
  | **Availability** | 99.99% | 99.9% | 99.5% | 99.9% | 99.9% | 99.99% | 99.99% |
  | **AZ** | ≥3 | ≥3 | 1 | ≥3 | ≥3 | ≥3 | ≥3 |
  | **Concurrent facility fault tolerance** | 2 | 2 | 0 | 2 | 2 | 1 | 1 |

### Amazon S3 Glacier

- Family of S3 storage classes that provide cost-effective solutions for long-term data archiving and backup
- Part of Amazon S3 and store data as objects in S3 buckets.
- All S3 Glacier storage classes offer the same durability and resiliency as the S3 Standard storage class, but at lower storage costs.
- ❗ To change the storage class of an archived object to another storage class, you must first restore it and then copy it
- ❗ S3 Glacier supports restore requests at a rate of 1000 transactions per second.
- 🤗 There is also a separate Amazon S3 Glacier service that stores data as archives within vaults
  - AWS recommends using the S3 Glacier storage classes within Amazon S3 for all long-term data.

#### Storage Classes Comparison

| Storage Class | Minimum Storage Duration | Recommended Access Frequency | Average Retrieval Times | Archival? | Storage Pricing | Retrieval Pricing |
|---------------|--------------------------|------------------------------|------------------------|-----------|-----------------|-------------------|
| [S3 Glacier Instant Retrieval](#s3-glacier-instant-retrieval) | 90 days | Quarterly | Milliseconds | No | 💲💲💲 | 💲💲💲💲 |
| [S3 Glacier Flexible Retrieval](#s3-glacier-flexible-retrieval) | 90 days | Semi-annually | Minutes to 12 hours | Yes | 💲💲 | 💲💲 |
| [S3 Glacier Deep Archive](#s3-glacier-deep-archive) | 180 days | Annually | 9 to 48 hours | Yes | 💲 | 💲 |

#### S3 Glacier Instant Retrieval

- Ideal for long-term data accessed once per quarter requiring millisecond retrieval
- Use cases: image hosting, file-sharing applications, medical records
- Offers real-time access with same latency and throughput as S3 Standard-IA
- Lower storage costs but higher data access costs than S3 Standard-IA
- Minimum object size: 128 KB
- ❗ Minimum storage duration: 90 days (charged for remainder if deleted earlier)

#### S3 Glacier Flexible Retrieval

- Ideal for archive data accessed 1-2 times per year that doesn't require immediate access
- Use cases: backup, disaster recovery
- 📝 Objects are archived and not available for real-time access - you must restore objects before accessing them
- 📝 Requires 40 KB of additional metadata per object:
  - 32 KB of metadata required to identify and retrieve data (charged at the storage class rate)
  - 8 KB of data to maintain user-defined name and metadata (charged at S3 Standard rate)
- ❗ When you restore an archive, you pay for both the archive storage and the temporary copy (S3 Standard rate)
- ❗ Minimum storage duration: 90 days (charged for remainder if deleted earlier)
- To access objects, initiate a restore request to create a temporary copy
- **Retrieval Options**

  | Attribute | Expedited | Standard | Bulk |
  | --------- | --------- | -------- | ---- |
  | Data | Subset of archives | Any archive | Large amounts |
  | Time | 1 - 5 minutes | 3 - 5 hours | 5 - 12 hours |
  | Pricing | 💲💲💲💲 | 💲💲 | 💲 |

  - **Expedited**: Access subset of archives in 1-5 minutes.
    - **On-demand**: Accepted except for rare high load situations.
    - **Provisioned**: Buy capacity to ensure retrieval capacity is available.
  - **Standard**: Access any archive in 3-5 hours.
  - **Bulk**: Access large amounts in 5-12 hours (free)

#### S3 Glacier Deep Archive

- Lowest-cost storage option in AWS
- Ideal for archive data accessed less than once per year
- Use cases: compliance requirements, backup, disaster recovery
- 📝 Objects are archived and not available for real-time access - you must restore objects before accessing them
- Requires 40 KB of additional metadata per object:
  - 32 KB of metadata required to identify and retrieve data (charged at the storage class rate)
  - 8 KB of data to maintain user-defined name and metadata (charged at S3 Standard rate)
  - 💡 Consider aggregating many small objects into a smaller number of large objects to reduce metadata overhead costs
- ❗ When you restore an archive, you pay for both the archive storage and the temporary copy (S3 Standard rate)
- ❗ Minimum storage duration: 180 days (charged for remainder if deleted earlier)
- ❗ The transition of objects to the S3 Glacier Deep Archive storage class can go only one way
- To access objects, initiate a restore request to create a temporary copy
- **Retrieval Options**
  - **Standard**: Typically restores within 12 hours (9-12 hours with Batch Operations)
  - **Bulk**: Typically restores within 48 hours at a fraction of Standard cost

## S3 Availability

- **Cross region replication (CCR)**
  - Copying is asynchronous replication
  - On bucket level (entire bucket) or object level (based on prefix/tags)
  - ❗📝 Must enable versioning (source and destination)
    - CRR is built on top of versioning functionality.
    - 🤗 As the replication is asynchronous, it needs 'non-changing' copy of data during replication
      - Versioning makes it easy so file can still be modified (upload, delete, etc.) while replication is in progress.
  - Buckets must be in different AWS regions & can be in different accounts
  - Requires IAM permissions to S3
  - Delete markers & deleting individual versions / delete markers are not replicated.
  - **Use cases**: compliance, lower latency access, replication across accounts
  - Flow:
    1. Create a bucket, and one more new one as replica
    2. Original bucket -> Management -> Replication -> Add rule
       - Apply to *entire bucket* or *"prefix or tags"*
    3. Choose if objects encrypted with AWS KMS will also be encrypted
       - Other account must also have access to AWS KMS as well
    4. For replicated objects you can change
       - Storage class
       - Ownership to destination bucket owner
    5. Set IAM rule
       - Role gets created automatically that gives replication permissions to the original bucket
    - ❗ Does not replicate:
      - Existing files are not replicated, just new files but with same metadata and ACLs.
      - Replicas of other objects, lifecycle actions and configurations, objects without bucket owner permissions, SSE-C & SSE-KMS encrypted object (can be configured to replicate)...

## S3 Performance

- 📝 **S3 Transfer Acceleration**
  - Also known as *S3TA*.
  - Helps speed-up long-distance transfer of larger objects
  - Behind-the-scenes, routes traffic via [CloudFront](./04-06-02-networking-edge-cloudfront.md)'s Edge Locations over AWS backbone.
  - Pricing: Pay only for transfers that are accelerated.
- **Multipart Upload**
  - Use if file is more than >100MB (required for >=5GB)
  - Allows uploading parts concurrently
    - Improved throughput
    - Quick recovery from any network failures
    - Pause and resume object uploads
    - Begin an upload before you know the final object size.
  - All storage that any parts the aborted multipart upload consumed is freed & deleted.
    - S3 supports a bucket lifecycle rule to abort multipart upload that don't complete within a specified number of days.

## S3 Websites

- S3 can host static websites and have them accessible on the www.
  - 📝 The website URL will be => bucket name + `s3-website` + region + `amazonaws.com`.
    - `<bucket-name>.s3-website-<AWS-region>.amazonaws.com`
    - `<bucket-name>.s3-website.<AWS-region>.amazonaws.com`
- If you get 403 (Forbidden) error; make sure bucket policy allows public reads!
- Supports redirects through a metadata tag.
- In bucket -> Properties -> Static website hosting
  - Select `Use this bucket to host a website` (optionally specify default index and error files)
  - Another option is to redirect requests (to another place)
  - **S3 CORS** (CORS=Cross Origin Resource Sharing)
    - 📝 If you request data from another S3 bucket, you need to enable CORS
    - Allows you to limit the number of websites that can request your files in S3 and this way limit costs.
    - E.g. `bucket1` has image in `bucket2`. `bucket2` must then has CORS headers for `bucket1`.
- **Prerequisites**
  - ❗ 📝 The bucket must have the same name as your domain or subdomain.
    - No need if CloudFront is used.
  - *(Optionally)* Use CloudFront for SSL/TLS
  - A registered domain name.
  - Route 53 as DNS service for the domain to configure DNS.
