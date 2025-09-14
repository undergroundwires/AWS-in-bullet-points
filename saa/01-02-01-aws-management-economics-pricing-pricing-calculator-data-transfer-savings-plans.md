# AWS Pricing

## AWS Pricing Overview

- Services have often pay-as-you-go model.
  - E.g. you run a service for 2 minutes, you pay for two minutes.
- Free-tier
  - Most services have free usage per month
- More information about AWS service pricing, see [Cloud Services Pricing](https://aws.amazon.com/pricing/).

## AWS Pricing Calculator

- Provides an estimate of AWS fees and charges for different cases/services.
- ❗ Doesn't include any taxes
- Uses [AWS Price List API](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/price-changes.html)
- **Quick estimates**
  - Ballpark estimate while requiring minimal information and parameters
- **Advanced estimates**
  - Allows you to fine-tune the estimate
- See [calculator.aws](https://calculator.aws/)

## Data transfer charges

- Applies globally to all AWS services
- Ingress: Free
- Egress
  - Free within same region, costs extra for Internet and other regions.
  - 💡 Always double check, not so cheap

## Savings Plans

- Flexible pricing model offering up to 72% discount.
- Requires 1 or 3 year commitment
- **Compute Savings Plans**
  - Most flexible - apply to any EC2, Fargate and Lambda
- **EC2 Instance Savings Plans**:
  - Apply to specific instance family in a region
  - Flexible across size, OS, and tenancy within that family
- 💡 Automatically applies to existing usage - no need to modify instances
