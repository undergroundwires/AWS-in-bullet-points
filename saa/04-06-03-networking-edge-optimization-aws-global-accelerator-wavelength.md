
# Edge Optimization

## AWS Global Accelerator

- Improves application availability, performance, and security.
- Proxies packets at the edge to applications running in one or more AWS Regions.
- It can front endpoints such as
  - load balancers (NLB),
  - Amazon EC2 instances,
  - and Elastic IP addresses.
- Route traffic to your applications using the AWS global network instead of the internet.
- Use-cases:
  - 📝 Non-HTTP use cases, such as gaming (UDP), IoT (MQTT), or Voice over IP
  - HTTP use cases that specifically require static IP addresses or deterministic, fast regional failover.
- Helps with:
  - 📝 Instant regional failover by routing to next available healthy app endpoint
  - High availability by providing two static IP addresses for resiliency
  - Improved performance by routing via AWS global network
  - Easy manageability with fixed static IP addresses
- Empowers Amazon S3 Multi-Region Access Points
- AWS Global Accelerator vs [Amazon CloudFront](./04-06-02-networking-edge-cloudfront.md)
  - Both use AWS global network and edge locations
  - CloudFront is optimized for content delivery (cacheable content) for HTTP
  - Global Accelerator is optimized for performance of wide range of applications over TCP/UDP

## AWS Wavelength

- Brings AWS compute and storage services to the edge of 5G networks
- Enables **ultra-low latency applications** (single-digit millisecond latency)
- Use cases:
  - Real-time gaming and media streaming
  - AR/VR applications
  - Industrial IoT and autonomous vehicles
  - Real-time video analytics
- How it works:
  - AWS infrastructure deployed inside telecommunications providers' 5G networks
  - Applications run at the **mobile network edge**
  - Traffic stays within the carrier's network (doesn't traverse internet)
- Complements Global Accelerator:
  - [Wavelength](#aws-wavelength) places compute at edge
  - [Global Accelerator](#aws-global-accelerator) optimizes network routing
- ❗ Limited to specific metro areas where carrier partners have deployed Wavelength Zones
