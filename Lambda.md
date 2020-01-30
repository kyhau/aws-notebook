# Lambda

- [Shave 99.93% off your Lambda bill with this one weird trick](
  https://medium.com/@hichaelmart/shave-99-93-off-your-lambda-bill-with-this-one-weird-trick-33c0acebb2ea)
  (2019-12-10)
- [Improved VPC networking for AWS Lambda functions](
  https://aws.amazon.com/blogs/compute/announcing-improved-vpc-networking-for-aws-lambda-functions/)
  (2019-09-03)
- [AWS Lambda supports SQS FIFO (First-In-First-Out) as an Event Source](#aws-lambda-supports-sqs-fifo-as-an-event-source)
- [Best Practices for Developing on AWS Lambda](#best-practices-for-developing-on-aws-lambda)

---
## Best Practices for Developing on AWS Lambda

[Ref-1 AWS Architecture Blog](https://aws.amazon.com/blogs/architecture/best-practices-for-developing-on-aws-lambda/)

1. When to VPC-Enable a Lambda Function
    - You should only enable your functions for VPC access when you need to interact with a private resource located in a private subnet. An RDS instance is a good example.
    - Once your function is VPC-enabled, all network traffic from your function is subject to the routing rules of your VPC/Subnet. If your function needs to interact with a public resource, you will need a route through a NAT gateway in a public subnet.

2. Deploy Common Code to a Lambda Layer (i.e. the AWS SDK)

3. Watch Your Package Size and Dependencies

    - Lambda functions require you to package all needed dependencies (or attach a Layer) - the bigger your deployment package, the slower your function will cold-start. 
    - Remove all unnecessary items, such as documentation and unused libraries. 
    - If you are using Java functions with the AWS SDK, only bundle the module(s) that you actually need to use - not the entire SDK.
    
4. Monitor Your Concurrency (and Set Alarms)

5. Over-Provision Memory (in some use cases) but Not Function Timeout

---
## AWS Lambda supports SQS FIFO as an Event Source

- See https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html
- Make sure that you configure the dead-letter queue on the source queue, not on the Lambda function. 
  The dead-letter queue that you configure on a function is used for the function's asynchronous invocation queue,
  not for event source queues. 