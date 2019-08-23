# API Gateway

- [Lambda Authorizer (aka. Custom Authorizer) ](#lambda-authorizer-formerly-known-as-a-custom-authorizer)
  - [Control error message and code](#lambda-authorizer-control-error-message-and-code)

## Lambda Authorizer (formerly known as a Custom Authorizer) 

1. There are two types of Lambda authorizers:

    1. A token-based Lambda authorizer (also called a TOKEN authorizer) receives the caller's identity in a
       bearer token, such as a JSON Web Token (JWT) or an OAuth token.

    2. A request parameter-based Lambda authorizer (also called a REQUEST authorizer) receives the caller's identity
       in a combination of headers, query string parameters, stageVariables, and $context variables.
       a.  For WebSocket APIs, only request parameter-based authorizers are supported.

https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html


### Lambda Authorizer: Control error message and code

Ref-1: [stackoverflow.com - api-gateway-custom-authorizer-control-error-message-and-code](
https://stackoverflow.com/questions/50686003/api-gateway-custom-authorizer-control-error-message-and-code)

Ref-2: [forums.aws.amazon.com - Custom Authorizer, how to return 401 http status code](https://forums.aws.amazon.com/thread.jspa?threadID=226689)

- **error code**: AWS does not fully allow a Custom Authorizer implementation to dictate the error code sent back to
  caller.
  - If the Custom Authorizer returns an Auth Policy which does not have resource/method that was invoked in one of the
    statements with action **Allow**, then user gets a 403 with something like "Not authorized to access resource".
  - If the Custom Authorizer returns an Auth Policy which has statements with action **Deny** that contains
    resource/method that was invoked, then user gets a 403 with something like "access denied explicitly with a Deny".
  - If the Exception raised by Custom Authorizer has message **"Unauthorized"** then user gets 401 with message
    "Unauthorized".
  - If Custom Authorizer throws an exception with any other message then user gets HTTP-500 internal server error
    (Authorizer Configuration Error) and call is rejected/not-authorized.

- **error message**: Only static control is allowed via Body Mapping Template in Gateway Responses.
  - E.g. you can update the Body Mapping Template for "Unauthorized [401]" in "Gateway Responses" to say "My service
    does not like you for some unknown reason" and then whenever Custom Authorizer throws "Unauthorized" exception the
    end user gets HTTP 401 with "My service doesn't like you for some unknown reason".
  - Similarly you can also update "Access Denied [403]" or "Authorizer Configuration Error [500]". But the message is
    static and can not be controlled from Custom Authorizer implementation.
  - It is **NOT** possible to have different 401 messages like:
  - 401: Unauthorized due to expired token.
  - 401: Unauthorized due to missing scope.

Other unrelated thing: Because the Custom Authorizer throws an exception in certain conditions to convey auth failure,
 from a metric point of view this increments the Lambda ErrorCount metric. So that metric is not reliable to identify
 "application errors".
