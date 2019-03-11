# Elastic Beanstalk 

See https://github.com/kyhau/eb-app-template for deployment and configuration template.

## Why use Elastic Beanstalk, not Elastic Load Balancer?

Amazon Elastic Beanstalk is a PaaS: you only provide it with your artifact and it manages deployments and scaling automatically.
Elastic Beanstalk uses ELB internally, but you don't have to manage it.

Amazon Elastic Load Balancer is a part of IaaS. You will have to manage your own instances and deployments to use it.

## Elastic Beanstalk logging

AWS Elastic Beanstalk provides two ways for you to regularly view logs from the Amazon EC2 instances that run your application:

1. You can configure your Elastic Beanstalk environment to upload rotated instance logs to the environment's Amazon S3 bucket.
2. You can configure the environment to stream instance logs to Amazon CloudWatch Logs.

When you configure CloudWatch Logs, Elastic Beanstalk creates log groups for proxy and deployment logs on the Amazon EC2 instances and transfers these log files to CloudWatch Logs in real time.
