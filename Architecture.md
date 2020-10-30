# Architecture and Design Patterns

1. [Amazon Builders' Library](https://aws.amazon.com/builders-library)
1. [AWS Architecture Blog](https://aws.amazon.com/blogs/architecture/)
1. [Data, Analytics, and Machine Learning Resource Hub](
   https://resources.awscloud.com/aws-data-analytics-machinelearning)

---
1. [Serverless Microservice Patterns for AWS](https://www.jeremydaly.com/serverless-microservice-patterns-for-aws/) - Jeremy Daly, 2020-07-23
1. [Using API Gateway as a Single Entry Point for Web Applications and API Microservices](https://aws.amazon.com/blogs/architecture/using-api-gateway-as-a-single-entry-point-for-web-applications-and-api-microservices/) - Anandprasanna Gaitonde and Mohit Malik on 2019-10-22
1. [One to Many: Evolving VPC Design](https://aws.amazon.com/blogs/architecture/one-to-many-evolving-vpc-design/) - Androski Spicer, 2019-10-09
1. [Top Resources for API Architects and Developers](https://aws.amazon.com/blogs/architecture/top-resources-for-api-architects-and-developers/) - George Mao, 2019-09-10
1. [Best Practices for Developing on AWS Lambda](https://aws.amazon.com/blogs/architecture/best-practices-for-developing-on-aws-lambda/) - George Mao, 2019-07-09
1. [What should we know about AWS Networking](https://salerno-rafael.blogspot.com/2019/06/what-should-we-know-about-aws-networking.html) - Rafael Salerno, 2019-06-13
1. [Building Multi-Region Active-Active Architecture in AWS using containerised microservices](https://medium.com/@sajidniazi/building-multi-region-active-active-architecture-in-aws-using-containerised-microservices-7b1d40a7063f) - Sajid Niazi, 2019-06-07
1. [Standardizing infrastructure delivery in distributed environments using AWS Service Catalog](https://aws.amazon.com/blogs/mt/standardizing-infrastructure-delivery-in-distributed-environments-using-aws-service-catalog/) - Kristopher Lippe, 2019-05-17
1. [The 5 Pillars of the AWS Well-Architected Framework](https://aws.amazon.com/blogs/apn/the-5-pillars-of-the-aws-well-architected-framework/) - Derek Belt, 2019-05-15
1. [Security Pillar of AWS Well-Architected Framework](https://d1.awsstatic.com/whitepapers/architecture/AWS-Security-Pillar.pdf) - AWS, 2018-07
1. [How to build a multi-region active-active architecture on AWS](https://read.acloud.guru/why-and-how-do-we-build-a-multi-region-active-active-architecture-6d81acb7d208) - Adrian Hornsby, 2018-02-25
    - Read also [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem).
1. [Building a Multi-region Serverless Application with Amazon API Gateway and AWS Lambda](https://aws.amazon.com/blogs/compute/building-a-multi-region-serverless-application-with-amazon-api-gateway-and-aws-lambda/) - Stefano Buliani, 2017-11-13
1. [CloudFront Origin Protection with AWS WAF & Shield](https://www.metaltoad.com/blog/how-to-protect-origin-with-aws-waf-shield) - Dylan Tack, 2017-10-12
    > Amazon has been steadily improving their CloudFront CDN offering with WAF capabilities.
      This is a great feature, however it's ineffective if origin servers can be attacked directly, bypassing
      CloudFront. With a little extra work, access to the origin can be restricted.
      The solution is to add a secret header value at the edge, and configure the load balancer to block requests that
      are missing this secret. This is necessary because CloudFront distributions are not associated with security
      groups, nor are fixed IPs available (unlike higher-priced competitors like Kona Site Shield).


---
(Non AWS specific)

1. [Open Web Application Security Project  (OWASP) API Security Top 10](https://apisecurity.io/encyclopedia/content/owasp/owasp-api-security-top-10.htm) - APIsecurity.io, 2019-12-31
