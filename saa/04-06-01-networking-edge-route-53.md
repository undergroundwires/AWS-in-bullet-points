# Route 53

- Route53 is a Managed DNS (Domain Name System)
  - DNS = Collection of rules and records to guide clients to reach a server through URLs.
  - 🤗 DNS is on port 53 that's where the name comes from.
- Route 53 can use both public and private domain names:
  - e.g. `application1.mypublicdomain.com`
  - e.g. `application1.company.internal` (can be resolved by your instances in your VPCs).
- Route 53 has advanced features:
  - **Load balancing** -> through DNS - also called client load balancing
  - **Health checks** -> although limited
  - **Routing policy** -> simple, failover, geolocation, latency, weighted, multi value
- **Hosted Zone**
  - Container that holds information about how you want to route traffic for a domain
    - E.g. `example.com` and its subdomains.
  - Can be public or private zone for public & private domain names.
    - Public hosted zone for public domain names
      - Accessible from www e.g. `application1.mypublicdomain.com`
    - Private hosted zones for private domain names
      - Gets resolved by instances in VPC e.g. `application1.company.internal`
      - 📝 ❗ If you use custom DNS domain names in a private zone in Route 53, you must set both attributes to true in VPC: `enableDnsSupport`, `enableDnsHostnames`.
  - ❗ You need to delete all record sets in order to delete Hosted Zone
- **Pricing**
  - $0.50 per month per hosted zone
  - A domain name (.com) costs 12$ per year
- **TTL (time to live)**
  - Computer caches DNS request for TTL duration
    - E.g. if `domainname.com` points to an IP, computer caches it.
  - Cache is only updated after TTL is expired, computer then talks to Route 53 again.
  - High TTL vs Low TTL
    - High TTL: e.g. 24 hours
      - Less traffic to DNS so it's cheaper
      - Possible change of out-dated DNS
    - Low TTL e.g. 60 seconds -> more traffic to DNS & easy to change records
- You can test DNS queries to see if it resolves right with `nslookup domainname.com` in Windows and `dig domainname.com` in Mac.

## DNS Records

- **DNS Records**
  - **SOA** (Start Of Authority record)
    - Every DNS starts with authority record SOA
    - Stores name of the server that supplied the data for the zone, the administrator of the zone, current version of the data file, default number of seconds for TTL file on resource records.
  - **NS** (Name Server Records)
    - Used by Top Level Domain servers to direct traffic to the Content DNS server which contains the authoritative DNS records.
  - In AWS, the most common records are:

    | Record | Description |
    |:------:|-------------|
    | A     | URL to IPv4 |
    | AAAA  | URL to IPv6 |
    | CNAME | URL to URL |
    | Alias | URL to AWS resource |

  - Other common records are **MX** (e-mail) and **PTR** (reversed A, look up name against IP address) records.
  - DNSSEC is not supported, these all supported are: A (address record), AAAA (IPv6 address record), CNAME (canonical name record), CAA (certification authority authorization), MX mail exchange record), NAPTR (name authority pointer record), NS (name server record), PTR (pointer record), SOA (start of authority record), SPF (sender policy framework), SRV (service locator), TXT (text record).

- **How does DNS work?**
  1. ***Query NS***: *Browser* makes DNS request to *Route 53 (DNS Server)* with e.g. `http://myapp.mydomain.com`
     - Top Level Domain Server then returns NS record to the name server.
  2. ***Query DNS***: *Browser* then queries NS server and gets SOA (start of authority) record where all DNS records exists
     - *Route 53 (DNS Server)* sends back IP e.g. `32.45.67.85` (= a record: URL to IP)
  3. ***Request content***: *Browser* makes HTTP Request to *application server* with e.g. `IP: 32.45.67.85` and with header `Host : http://myapp.mydomain.com`.
  4. ***Get content***: *Application server* returns with HTTP response
- **CNAME vs Alias**
  - AWS resources (load balancer, [CloudFront](./04-06-02-networking-edge-cloudfront.md), etc..) expose an AWS URL e.g. `lb1-1234.us-east-2.elb.amazonaws.com` and you want it to be `myapp.mydomain.com`
  - Two options
    1. CNAME
       - Points a URL to any other URL (app.mydomain.com => blabla.anything.com)
       - ❗ Only for non root domain (aka something.mydomain.com)
    2. Alias (AWS concept)
       - Points a URL to an AWS resource (app.mydomain.com => blabla.amazonaws.com)
       - 💡 Works for root domain and non root domain (e.g. mydomain.com)
       - Free of charge
       - Native health check
       - Good for pointing to e.g. load balancer / EC2 that'll have IP changing all the time.

## Health Checks

- If an instance is unhealthy, Route 53 will not send traffic to that address.
- It decides if the instance is healthy after running X check (default 3).
- Default **health check interval**: 30s (can set to 10s -> costs more)
- **Health checker**
  - You can customize which regions checkers will come from.
  - About 15 health checkers will check the endpoint health.
    - So if health check interval is 30s, it's checked every 2 secs on average,
- Can have HTTP, TCP and HTTPS health checks (no SSL verification)
- Can be integrated with CloudWatch
- Health checks can be linked to Route53 DNS queries
  - DNS failover -> It'll change behavior of Route 53
- Can have string matching (compare response) to decide if healthy.
- You can monitor:
  - Endpoint (can be IP address or domain name)
  - Status of other health checks (calculated health check)
  - State of CloudWatch alarm
- You can enable **Latency Graph** to track latencies.
- You can set-up SNS notification for failed health-checks.

## Routing policies

- Associated in a DNS record set.

  | Policy | Description | Use cases | Creation | Notes |
  | ------ | ----------- | --------- | -------- | ----- |
  | **Simple routing policy** | One domain to one/more IP/URL | Single instance | One set to target(s) | ❗ You can't attach health checks to simple routing policy<br>• Multiple IP handling is not standard, most clients do round robin<br>• Route 53 returns in random order |
  | **Weighted routing policy** | Control % of request to which targets | AB testing, split traffic between regions | Record set per endpoint with single weight | Only selected address is calculated returned to the client by Route 53 |
  | **Latency routing policy** | Redirect to least latency target | Low latency requirement | Record set per endpoint & region to the same DNS name. | Not based on region e.g. Germany may be directed to the US (if that's the lowest latency) |
  | **Failover routing policy** | To healthy target(s) | Active-passive failover | 2 record sets, one primary and one secondary for same target | • Secondary is activated when primary fails<br>• Primary must have health check, optional for secondary |
  | **Geolocation routing policy** | Based on user location | E.g. traffic from UK goes to IP X | Record set per location (country, region or default) | 💡 Should create "default" policy if there's no match |
  | **Geoproximity routing policy** | Based on user and resource location | Shift traffic from resources in one location to resources in another | For each role, AWS resource? AWS region, not? latitude and longitude  | • You must use Route 53 traffic flow<br>• Uses configurable biases to route more or less<br>• Bias expands or shrinks zone from where traffic is routed to a resource |
  | **Multi Value routing policy** | Improves *simple routing policy* with health checks | • Traffic to multiple resources<br>• Must have health checks | Record set for each target | • Enables client side load balancing |

## 3rd Party Domains

- **Domain names**
  - Top level: last part e.g. `.gov`
    - IANA [has a database](https://www.iana.org/domains/root/db) of all available top-level domains
  - Second level *(optional)*: e.g. if it's `.gov.uk` then `.gov` is second level `.uk` is top level domain name.
- Domain name **registrar** is an organization that manages the reservation of Internet domain names under one or more domain names.
  - E.g. GoDaddy, Google Domains, Gandi...
    - You can also buy from AWS with Route 53.
      - It can take up to 3 days for registration to be completed.
  - Registers domains with InterNIC, a service of ICANN which enforces uniqueness of domain names across the internet.
  - Each domain name becomes registered in a central database known as the WhoIS database.
- You can also use 3rd party registrar with Route 53.
  1. Create a public Hosted Zone in Route 53
  2. Update NS records on 3rd party website to use Route 53 ***name servers*** (NS Records).
     - You get NS (name server) records from Hosted Zone & Route 53 that you copy & paste to you registrar.
- ❗ Soft limit of up to 50 domains managed by Route 53.
