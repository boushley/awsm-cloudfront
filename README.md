# AWSM CloudFront

This is an [awsm](https://github.com/awsm-org/awsm) module that puts a CloudFront distribution in front of your JAWS project.

## Now: Web Applications using Just AWS Without Servers!

### The Pieces

#### Amazon S3 Static Website Hosting
Amazon has documentation about [static website hosting](http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html)
that explains how to use S3 to host a static website.

#### API Gateway and Lambda with JAWS
We then use [JAWS](https://github.com/jaws-framework/JAWS) to host our backend API. JAWS makes deploying to API Gateway
and Lambda simple and repeatable. It also integrates with the awsm module system to make coordinating with tools like
this one easy!

#### CloudFront to merge the two domains
The above mentioned technologies are great, but the best you can do is host your web application at www.example.com and
then host your API at api.example.com. This can work, but relies on CORS and generally just adds a bunch of extra work.

Using CloudFront you can use both of these setups on the same domain. We create a CloudFront distribution with two
origins. The first (default) origin is our static S3 bucket, then we add our API Gateway endpoint as our second origin.
We then setup a Cache Behavior that routes traffic for a specific set of routes, like `/api` to our API Gateway
endpoint.

## Gotchas

### Watch your paths
There are many paths involved here, keeping them all straight is of paramount importance.

Here's the rundown:
* The API Gateway origin has a config named `OriginPath`.
* The Cache Behavior uses a `PathPattern` to determine which requests are sent to our API Gateway.
* There is the path of the request the user makes to our CloudFront distribution.
* Then there is the path of the request that CloudFront makes to our API Gateway.

There are a few correct ways to configure these, but I'll walk through just one.

You can set `OriginPath` to you JAWS / API Gateway stage, `"/dev"` or `"/production"`. This means that calls from our
frontend to our backend don't need to know about which stage they're operating on. **Note:** If you don't want to
specify an origin path then use `""` not `"/"`.

Next you can configure your Cache Behavior to use `"api/*"` as the `PathPattern`. This means all requests to your
CloudFront distribution that have `api/` at the start of their route will be redirected to our API Gateway.

Then the paths on API Gateway should all start with `api/`. This matches with the `PathPattern` we specified for our
Cache Behavior. For the time being this requires you to manually update the awsm.json for each API Gateway endpoint
and add `api/` to the front, hopefully that will change soon. **Note:** Multiple people have expected that the `api/`
portion of our request would be trimmed when sent to API Gateway, but it is not.

#### An Example Request
We run something like `curl https://d1z31i778g9an.cloudfront.net/api/users/list` (that is not a real CloudFront
distribution).

CloudFront uses the Cache Behavior with `api/*` since it matches and forwards this request to API Gateway. The
request that is sent to API Gateway is equivalent to:
`curl https://hga136gdlt.execute-api.us-west-2.amazonaws.com/dev/api/users/list`

Note how the `OriginPath` was prepended to the request path, and the `PathPattern` was not removed.

### HTTPS Only
The API Gateway is HTTPS only. You must create the API Gateway origin to have `match-viewer` for the protocol policy. You
must then ensure that the `CacheBehavior` for the API Gateway uses `redirect-to-https` for the `ViewerProtocolPolicy`.
Luckily the CloudFormation resource defined in this module has all of those settings built in, so these settings are all
taken care of.
