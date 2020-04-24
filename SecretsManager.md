# AWS Secrets Manager

### AWS Parameter Store vs. AWS Secrets Manager
[Ref - February 28th, 2019](https://www.1strategy.com/blog/2019/02/28/aws-parameter-store-vs-aws-secrets-manager/)

1. **Password Generation**
   - Secrets Manager is able to generate random secrets through the AWS CLI or SDK. 
     E.g., when creating a new RDS instance through a CloudFormation template, you can also create a randomly generated
     password and reference it in the RDS configuration since it requires a master username and password. 
     The CloudFormation can store the username and password in an AWS Secrets Manager secret that can be only accessed
     by Database Admins.
   - Password generation is not only useful in CloudFormation templates, but applications (through the SDK) can also
     leverage this feature. The functionality to generate random strings is only available to AWS Secrets Manager and
     not available in SSM Parameter Store.
2. **Secrets Rotation**
   - Secrets Manager provides full key rotation integration with RDS. This means that AWS Secrets Manager can rotate
     keys and actually apply the new key/password in RDS for you. 
   - For services other than RDS, AWS allows you to write custom key rotation logic using an AWS Lambda function.
   - See https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html
3. **Cost**
   - There are no additional charges for using SSM Parameter Store. However, there are limit of 10,000 parameters per
     account. 
   - AWS Secrets Manager does accrue additional costs ($ per secret stored and $ for 10,000 API calls).
4. **Cross Account Access**
   - Secrets Manager can be shared across accounts. 
     E.g., IAM users and application resources in one development or production AWS account will be able access secrets
     stored in a different AWS account (e.g. Security AWS Account). 
     Such functionality is also beneficial for use cases where a customer needs to share a particular secret with a
     partner.
   - See https://aws.amazon.com/blogs/security/how-to-access-secrets-across-aws-accounts-by-attaching-resource-based-policies/

---
#### [Limit info](https://aws.amazon.com/about-aws/whats-new/2019/10/aws-secrets-manager-supports-increased-secret-size-api-request-rate/)
last updated on 2019-11-01

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
 