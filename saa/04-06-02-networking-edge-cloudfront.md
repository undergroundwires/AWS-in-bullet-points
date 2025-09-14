# CloudFront

- Content Delivery Network (CDN)
- Improves read performance, content is cached at the edge locations (+136 point of presence globally)
- 📝 Popular with S3 but works with EC2, Load balancing
- Can help protect against network attacks
- You use domain name that CloudFront assigns to your distribution, e.g. cannot attach Elastic IP.
- Can serve both
  - static (e.g., images, videos),
  - and dynamic content (e.g., APIs, real-time data)
- A ***distribution*** is a CloudFront instance (collection of edge locations) that can be:
  - ***Web distributions*** -> Web pages
  - 📝 ***RTMP*** -> a video/media (streaming) protocol over SSL
    - For RTMP CloudFront distributions files must be stored in an S3 bucket
- You set up ***origin*** (domain name & path)
  - Origin can be S3, EC2, ELB, or Route53.
  - **Origin Failover**: Set-up primary + secondary origins on selection of 500, 502, 503, 504, 404, or 403.
    - 💡 Enables high availability.
- ***Cache Behavior settings***
  - Can set-up viewer Protocol Policy: HTTP & HTTPS, Redirect HTTP to HTTPS, HTTPS only
  - Can allow HTTP Methods: GET & HEAD & PUT & OPTIONS & PATCH & DELETE.
  - Set minimum maximum & default TTL
  - Forward cookies and/or query strings
  - And more: e.g. Path Pattern, Compress objects
- ***Distribution Settings*** are • Price Class • AWS WAF Web ACL • Alternate Domain Names (CNAMEs), SSL certificate • Supported HTTP versions, logging (with log prefix & option to include cookies) • Enable IPv6
- 📝 You can clear cache objects by creating an ***invalidation*** but you will be charged.
  - 💡 AWS recommends versioned file names instead of invalidating if files are updated frequently.
  - Integrates with [AWS Shield](./04-04-networking-security.md#aws-shield) for DDoS protection.
  - Integrates with **AWS WAF** for application layer security.
- **Regional Edge Cache**: If data is infrequently accessed, instead of CloudFront sends request back to your origin, it caches your data in a regional edge caches and gets from there (faster).
- 💡 Use cases
  - Accelerate uploads -> Users can upload to edge locations
  - Can stream videos from edge locations
  - Used in general to accelerate other services e.g. S3 for e.g. faster upload.
  - Allows Lambda with Lambda@Edge to run in edge locations.
  - Integrates with API gateway to run on edge locations.
  - Use with any text, blob (e.g. `.pdf`)
- `Cache-Control` and `Expires` headers control how long objects stay in the cache.
  - `Cache-Control max-age` lets you specify how long (in seconds) you want an object to remain in the cache before CloudFront gets the object again from the origin server.
    - Minimum 0, Max 3600

## Origin Groups

- 📝 Provide high availability and automatic failover for CloudFront distributions
- Combines multiple origins (primary + secondary) into a single group
- Automatically switches to secondary origin on specific HTTP status codes indication failure
  - E.g. `500`, `502`, `503`, `504`, `404`, `403`
- 💡 **Use cases**:
  - Multi-AZ EC2 instances for dynamic content
  - S3 bucket with cross-region replication as backup
  - Load balancer with backup origin
- **Benefits**:
  - Automatic failover without manual intervention
  - Improved application availability and reliability
  - Seamless user experience during origin failures
- **Best practices**:
  - Place origins in different Availability Zones or regions
  - Use health checks to monitor origin availability
  - Configure appropriate cache behaviors for both origins

## CloudFront Security

- Can provide SSL encryption (HTTPS) at the edge using ACM (AWS Certificate Manager)
  - Supports: **Perfect Forward Secrecy** (new SSL key for each session)
- Can protect against DDoS attacks.
- ***Geo Restriction*** allows you to specify a list of whitelisted or blacklisted countries in your CloudFront distribution.

### CloudFront Signed URLs

- 📝 Secure method to provide temporary access to private content through CloudFront
- Creates time-limited URLs with cryptographic signatures using RSA private/public key pairs
- More advanced than [S3 pre-signed URLs](./06-01-02-data-s3-security-macie.md#s3-pre-signed-urls):
  - Supports IP restrictions, custom policies
  - Works with all CloudFront features
- **S3 Integration**:
  - When S3 bucket is private with OAC/OAI, CloudFront signed URLs are required since S3 pre-signed URLs won't work through CloudFront
- 💡 Use cases: Premium content, paid video streaming, temporary file sharing
- Can specify expiration times, IP ranges, and custom access policies
- Generated using CloudFront API/SDK with private key signing

### Origin Access Control (OAC)

- Feature that secures origins by permitting access only to designated CloudFront distributions
  - Prevents direct origin access
- Ensures users can only access content through CloudFront URLs, not direct origin URLs
- **Supported origins**:
  - 📝 **Amazon S3 buckets** (regular buckets, not website endpoints)
  - **Lambda Function URLs** - uses SigV4 authentication
- **Not supported**: ALB, NLB, EC2, API Gateway, other custom origins (use VPC Origins, Security Groups, or custom headers instead)
- Uses AWS IAM service principals for authentication
- **Configuration**: Choose signing behavior - "Sign requests" (recommended), "Do not sign", or "Do not override authorization header"
- 🤗 Successor to older *Origin Access Identity (OAI)* with enhanced capabilities
  - OAI continues to work, but OAC recommended for new deployments

### CloudFront Signed Cookies

- Alternative to [signed URLs](#cloudfront-signed-urls) for controlling access to multiple files
- Set cookies in user's browser instead of generating individual signed URLs
- **Use cases**:
  - Access to entire sections of private content
  - Multiple files/videos in a subscription area
  - When you don't want to change URLs
- Same security model as signed URLs: expiration, IP restrictions, custom policies
- Signed URLs vs Signed Cookies:
  - Signed URLs: Individual file access
  - Signed Cookies: Multiple files or site sections

## Origin Shield

- Extra caching layer between CloudFront edge locations and your origin
- Reduces origin load, improves availability, and cuts operating costs
- **Benefits**:
  - Better cache hit rates with additional caching layer
  - Fewer origin requests - collapses multiple regional requests into one
  - Better performance - traffic stays on AWS network to Origin Shield
- **Configuration**: Choose AWS Region closest to your origin
- 💡 **Use cases**: Live streaming, image processing, compute-heavy origins
- 💡 **Best for**: High-compute origins or limited bandwidth; less useful for basic S3 buckets

## S3 & CloudFront

- Allows to directly upload to and read from edge locations.
- **Transfer acceleration**
  - In S3 enables faster transfers through routing from CloudFront's edge locations.
  - Does not provide caching, use [CloudFront](./04-06-02-networking-edge-cloudfront.md#cloudfront) for caching
  - 💡 Costs extra but not always fastest.
  - Supports multipart uploads.
- **CloudFront vs S3 Cross Region Replication**

  | Attribute | CloudFront | S3 Cross Region Replication |
  | --------- | ---------- | --------------------------- |
  | Cache location | Global edge network | Each unique region you set up for replication |
  | Cache duration | TTL | Updated in near real time |
  | Can write? | Yes, can put object | Read-only |
  | Use case | Static content that must be available everywhere | Dynamic content that needs to be available at low-latency in few regions |

- **Secure S3 Access**:
  - 📝 Use [Origin Access Control (OAC)](#origin-access-control-oac) to restrict bucket access to CloudFront only
  - You must manually add a bucket policy that allows the CloudFront service principal to access the bucket
  - 🤗 Legacy *Origin Access Identity (OAI)* still supported but OAC recommended
- 📝 **CloudFront Signed URL / Signed Cookies**
  - Commonly used to give access to paid content
    - E.g. distribute paid shared content to premium users over the world & content lives in S3
    - ❗ If S3 can only be accessed through CloudFront, pre-signed S3 URLs cannot be used
      - 💡 Use [CloudFront Signed URLs](#cloudfront-signed-urls) or [signed cookies](#cloudfront-signed-cookies) instead
  - **Choice between URLs vs Cookies**:
    - Signed URLs: Individual file access
    - Signed Cookies: Multiple files or entire content sections
  - ❗ Can only be created using the AWS SDK
  - **Flow**:
    - Attach a policy with URL expiration, IP ranges, and trusted signers
    - 💡 URL expiration best practices:
      - Shared content (movie, music): make it short e.g. a few minutes
      - Private content (private to the user): make it last for years
    - Application generates signed URLs: *Client* gets *signed URL* from *application* where *application* talks to *Amazon CloudFront* using SDK
  - **Example architecture**: API Gateway **-->** Lambda *(Creates signed URL & verifies if user is premium with DynamoDB)* **-->** CloudFront *(issues signed URL)* **<--** [OAC](#origin-access-control-oac) **-->** S3 (serves videos + authorizes only from OAC with bucket policy)

## API Gateway & CloudFront

- CloudFront in front is generally not a good idea.
  - Most of the functionality in CloudFront can be found in API Gateway.
    - You can use ***edge-optimized*** endpoints in API Gateway.
      - CF usage is already included on API GW pricing.
- Deploy it if you want to fully control CloudFront distribution
  - E.g. manage WAF, add custom behaviors (static files on S3, some paths going to an ALB or other API), host API on multiple regions, set-up CF access logs etc.
