# Known Issues


1. [Service error (code 500) when invoking Lambda function from API Gateway](
https://forums.aws.amazon.com/message.jspa?messageID=806142)
   
    ```
    Endpoint response body before transformations:
    {
      "Message": "An error occurred and the request cannot be processed.",
      "Type": "Service"
    }
    ```
    - By design, API Gateway maps 429 errors from Lambda to 500 responses. There is no way to map Lambda's 429 to any
      status code. It will always be considered as 500. See [this post](
      https://stackoverflow.com/questions/36363280/map-aws-lambda-429-errors-to-api-gateway-2xx-response).
    - And it is possible to have a 200 code returned for certain function errors. See [this documentation](
      https://docs.aws.amazon.com/lambda/latest/dg/retries-on-errors.html).

2. CloudWatch Event Rules
    - When creating a Event Rule with CloudFormation, it somehow changes the digital value (e.g. `exitCode: 1`) in
     EventPattern to `exitCode: '1'`.
    - You need to manually change the value back from `'1'` to `1` from the Console.
    - Then when you update the CloudFormation stack later, the value will not get changed back to `'1'`.

