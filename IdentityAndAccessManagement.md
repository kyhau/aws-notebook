# Identity and Access Management

Table of Contents

- [IAM](#iam)
- [Cognito](#cognito)
- [STS](#sts)

## IAM
- IAM Users can be used to login to the AWS Console and can contain long-term AWS Credentials.
- IAM Roles provide short-term credentials and can be used with the AWS CLI via profiles.
- IAM Roles CANNOT be used to login to the AWS Console.
- IAM User is best used for a "service account" running one or more application processes in AWS. The application needs
  long-term credentials.
- **Permissions boundaries** allow control over WHAT an IAM user with delegated permissions can do.


### Policy Evaluation
- DENY always wins against ALLOW. Policy evaluation looks at all user/group permissions. 


### IAM Best Practices
- IAM users can be restricted via **resource AND identity policies**.
- ROOT users can be restricted using **service control policies** as long as accounts are inside an
  **AWS organisation**.


## Cognito
- Cognito is involved in identity federation and can create pooled identities based on isolated web identities.


## STS
- STS provides short-term AWS credentials to access AWS resources as part of ID Federation.
- STS does NOT provide long-term credentials, which are generally contained on IAM users, which are NOT involved in ID
  federation.


## Web Identity Federation
- Login occurs to remove IDP; Token is generated and provided to AWS and swapped for temporary credentials which are
  used to access AWS resources.
- SAML is NOT involved in Web Identity Federation


---

## IAM Use Cases

- Use case: Account A users to access resources in Account B.
   - Soln 1: In Account B, create an IAM role, edit the TRUST policy, adding account A.
   - Soln 2: In Account B, create resource policies, configure for account A access, and apply the policies to
             resources in Account B.

- Use case: A partner AWS account has been exploited, that account had a large number of users which assumed an IAM
  role in an account you manage. You need to cancel any access to your account by those users as quickly as possible,
  but with as least impact to the other accounts who use that role.
   - Soln: Adjust the role TRUST policy and revoke any role sessions.
