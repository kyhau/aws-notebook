# Design Edge Security

Table of Contents
- [CloudFront](#cloudfront---global-content-delivery-network-cdn)
- [Protecting Web Applications](#protecting-web-applications)
- [AWS WAF](#aws-waf-web-application-firewall)
- [AWS Shield](#aws-shield-standard)

---

## CloudFront - Global Content Delivery Network (CDN)

- CloudFront is a global CDN operating from AWS Edge Locations.
- Connections to a CloudFront distribution can utilize HTTP or HTTPS.
- Connections from CloudFront to your content (origin server) can occur using HTTP or HTTPS.
- Viewer protocol policy (per distribution) allows redirection of HTTP->HTTPS.
- Supports file access control and signed URLs and cookies.
- Can integrate with 3rd party solution using signed URLs and cookies.
- Provides basic white/blacklist geo-restriction per distribution.
- Use Field-level Encryption to help protect sensitive data.
- Integrates with **AWS WAF (Web Application Firewall)**.
- It also removes many invalid HTTP requests at the edge - basic filtering.
- Supports Lambda at the edge.

### SNI (Server Name Identifier)
- CloudFront supports SNI - Edge Location IPs can be shared.
- Dedicated IP SSL is supported in ALL browsers, but cost extra.
- SNI has no extra cost, but browsers need to support it.

### Restricting S3 to CloudFront
- By default, when using CloudFront with S3, CloudFront is optional, and S3 can be accessed directly. This can be changed by creating
  an OAI (Origin Access Identity).
- OAI is a virtual identity. A distribution can be configured to use it, so when accessing S3, CloudFront assumes this identity.
- How to configure CloudFront to use an OAI?
  - Add the OAI to the bucket policy as the only READ entry.

### Field-level encryption
- Field-level encryption allows you to securely upload user-submitted sensitive information to your web servers. 
  The sensitive information provided by your clients is encrypted at **the edge closer to the user and remains encrypted**
  **throughout your entire application stack**, ensuring that only applications that need the data - and have the
  credentials to decrypt it - are able to do so.
- You can encrypt up to 10 data fields in a request. *You can't encrypt all of the data in a request with field-level*
  *encryption; you must specify individual fields to encrypt.*

### Signed URLs and Cookies

CloudFront signed URLs and signed cookies allow you to control who can access your content. 
- Use signed URLs in the following cases:
  - You want to restrict access to individual files, for example, an installation download for your application.
  - You want to use an RTMP (Real-Time Messaging Protocol) distribution. Signed cookies aren't supported for RTMP distributions.
  - Your users are using a client (for example, a custom HTTP client) that doesn't support cookies.
- Use signed cookies in the following cases:
  - You want to provide access to multiple restricted files, for example, all of the files for a video in HLS format or all of the
    files in the subscribers' area of website.
  - You don't want to change your current URLs.
- If you are not currently using signed URLs and if your URLs contain any of the following query string parameters, you cannot use   
  either signed URLs or signed cookies:
  - Expires
  - Policy
  - Signature
  - Key-Pair-Id
- CloudFront assumes that URLs that contain any of those query string parameters are signed URLs and therefore won't look at
  signed cookies.
- If you use both signed URLs and signed cookies to control access to the same files and a viewer uses a signed URL to request a file,
  CloudFront determines whether to return the file to the viewer based only on the signed URL.

### Features and Limits
- Signed URLs and cookies are linked to an existing identity (Role/User), and they have the permissions of that entity.
- They can have their own validity period: default is 60 minutes.
- They expire either at the end of the period, or until the entity on which they are based expires.
- If you use a role, this is when the role’s temp credentials expire.
- Anyone can create a signed URL, even if they do not have permissions on the object.
- With CloudFront you defined the accounts which can sign; the key pair TrustedSigners is needed for CloudFront.
- Signed Cookies do NOT work with RTMP (Real-Time Messaging Protocol) distributions.
	
### Geo Restriction
1. Option 1:  CloudFront can restrict content using CloudFront Geo Restriction.
   - Whitelist OR Blacklist and it works on country restriction ONLY.
   - Location is based on IP country location - acked by a GeoIP Database (~99.8% accuracy).
   - No restrictions on ANYTHING ELSE - session/cookie/content/browser etc.
2. Option 2:  CloudFront can restrict content using a Third-Party Geolocation Service.
   - Third-Party Geo Restriction needs a server/serverless application - signed URLs are used.
   - A Third-Party Geolocation Service is used - extra accuracy.
   - Location can be much more accurate (city, locale, lat/long in some cases).

---

## Protecting Web Applications

### AWS WAF (Web Application Firewall)
- WAF allows for conditions or rules to be set on CloudFront web traffic or an Application Load Balancer.
- WAF can watch for cross-site scripting, IP addresses, the location of requests, query strings, and SQL injection.
- When multiple conditions exist in a rule, the result must include all conditions:
  - Example Rule: Block requests from 2.2.0.0/16 that appears to have an SQL code.
    Both conditions must match for a block.

### Denial of service attacks
- Flooding a system with traffic to overwhelm and prevent legitimate traffic access to resources.
- Distributed DoS (DDoS) is that same attack from multiple sources or systems.
- AWS provides resilience for network and transport layer attacks using AWS Shield. 

### AWS Shield Standard
- The basic level of DDoS protection for your web applications.
- Included with WAF with no additional cost.

### AWS Shield Advanced
- Expands services protected to include 
  - Elastic Load Balancers
  - CloudFront Distributions
  - Route53 hosted zones
  - resources with Elastic IPs
- Contact 24x7 DDoS Response Team (DRT) for assistance during an attack.
- Financial protection against DDoS attacks (cost protection against spiles in a bill from DDoS attack).
- Expanded protection against many types of attacks.
