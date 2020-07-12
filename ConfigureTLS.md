# Disable TLS 1.0 and use TLS 1.1 or higher

## CloudFront

Can be done (need to check [Security Policy](
https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-values-specify.html#DownloadDistValues-security-policy)).

See also [What's New 2017-09](
https://aws.amazon.com/about-aws/whats-new/2017/09/amazon-cloudfront-now-lets-you-select-a-security-policy-with-minimum-tls-v1_1-1_2-and-security-ciphers-for-viewer-connections/).

## API Gateway

[Q: Can I configure my REST APIs in API Gateway to use TLS 1.1 or higher?](
https://aws.amazon.com/api-gateway/faqs/)

Not managed in API Gateway. You can set up a CloudFront distribution with custom SSL certificate in your account and use it with Regional APIs in API Gateway. You can then configure the Security Policy for the CloudFront distribution with TLS 1.1 or higher based on your security and compliance requirements.

**Updated:**

See [Choose a Minimum TLS Version for a Custom Domain in API Gateway](
https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-custom-domain-tls-version.html).

## Load Balancer

AWS recommends "We recommend the ELBSecurityPolicy-2016-08 policy for general use".

See [Create an HTTPS Listener for Your Application Load Balancer - Security Policies](
https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html#describe-ssl-policies).