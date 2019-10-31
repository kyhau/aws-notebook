# AWS Secrets Manager

([Limit info](
https://aws.amazon.com/about-aws/whats-new/2019/10/aws-secrets-manager-supports-increased-secret-size-api-request-rate/)
last updated on 2019-11-01)

- Secrets Manager supports larger secret size of up to 10 Kb, making it easier for customers to manage secrets
  such as certificates with a long chain of trust. 
- Secrets Manager supports resource policies of up to 20 Kb enabling customers to allow multiple users and applications
  access a single secret. 
- Secrets Manager supports request rates for the `GetSecretValue` API operation of up to 1,500 requests per second.
  These increased service quotas will be applied to your accounts automatically. No further action required on your end.
- Secrets Manager enables you to retrieve and manage secrets such as database credentials and API keys throughout their
  lifecycle. 
- Secrets Manager also makes it easier to follow the security best practice of using short-term secrets by rotating
  secrets safely on a schedule that you determine. E.g., you can configure Secrets Manager to rotate a database
  credential daily, turning a typical, long-term secret in to a short-term secret that is rotated automatically.  
- Increased service quotas are available in all regions where the service operates.
 