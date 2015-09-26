# AWSM CloudFront
[awsm](https://github.com/awsm-org/awsm) module for putting a CloudFront distribution in front of your JAWS project.

This allows for hosting a web application using Just AWS Without Servers! Using S3 static website hosting and API 
Gateway and Lambda for backend APIs. Using CloudFront allows you to perform all of this on the same domain and avoid
any same origin problems, no need for CORS.

Define the resources necessary for creating a CloudFront distribution in front of a static S3 bucket and API Gateway endpoints
