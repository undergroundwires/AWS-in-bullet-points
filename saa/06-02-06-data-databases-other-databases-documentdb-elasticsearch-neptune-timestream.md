# Other databases

## Amazon DocumentDB

- Document database with Mongo DB compatibility.
- More hands-on than DynamoDB => Middle step between MongoDB and DynamoDB

## Amazon Neptune

- Fully managed graph database
- Highly available across 3 AZ ***(Multi-AZ)*** , with up to 15 read replicas ***(Clustering)***
- Point-in-time recovery, continuous backup to Amazon S3
- Support for KMS encryption at rest + HTTPS
- 💡 Use-cases:
  - High relationship data
  - Social networking: e.g. Users friends with Users, replied to comment on post of user and likes other comments
  - Knowledge graphs (e.g. Wikipedia)

## Amazon Keyspaces (for Apache Cassandra)

- Fully managed, serverless Cassandra-compatible wide-column NoSQL database service
- Compatible with Cassandra Query Language (CQL) and existing Cassandra drivers
- On-demand and provisioned capacity modes
- 💡 Use-cases: IoT device data, time-series data, messaging applications, personalization engines, fraud detection systems, gaming leaderboards

## Amazon Quantum Ledger Database (Amazon QLDB)

- Fully managed ledger database with immutable, transparent, and cryptographically verifiable transaction log
- Central trusted authority that maintains complete and verifiable history of all application data changes
- ACID transactions with SQL-like query language (PartiQL)
- Journal-first architecture - every change is written to an append-only journal
- Cryptographic hashing creates tamper-evident audit trail
- 2-3x performance improvement over common ledger blockchain frameworks
- 💡 Use-cases: Financial transactions, supply chain tracking, insurance claims, regulatory compliance, audit trails, digital identity verification

## Amazon Timestream

- Serverless time series database service that
- 💡 Use-cases: IoT and operational applications
