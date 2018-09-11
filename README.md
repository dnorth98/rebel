# Websites in a box

This repo contains several AWS cloudformation templates to implement "serverless" web site hosting and some basic metrics gathering.  The solution is implemented as a few templates as follows:

# route53-zone.yaml
This template creates a route53 zone.  It's only parameter is the root (apex) domain name for the zone to be created.  After creation you MUST delegate this zone for it to be usable.

# log-bucket.yaml
This template creates an S3 bucket to gather access logs and assigns the correct ACL to it.  It also creates a DynamoDB table which is used to store basic metrics for how many times a unique IP address has accessed a specific site.

It also creates a lambda function which watches for files in the log bucket, processes new log files and writes to the DynamoDB table.

This template exports the name of the log bucket which can then be used in other related cloudformation stacks. Specifically, the website-stack below imports this value and uses it to configure access logging for each site

# website-stack.yaml
This template is based on the [standard AWS example](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/quickref-s3.html#scenario-s3-bucket-website-customdomain) and has been modified to enable access logging and reference the logging bucket from a cloudformation export.

# Sample Usage
* Create N stacks using the route53 template to create a new zone (one stack for each required zone)
* Create a stack using the log bucket template to add a new bucket for logs and configure the lambda log watcher
* Create N stacks using the website-stack template (one for each website you want to setup)

Profit from the serverless web hosting joy!

# A note on S3 log delivery

[As documented by AWS](https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerLogs.html), S3 log delivery to the configured bucket may take some time. In my own testing, the frequently took a while to show up in the bucket and of course, this delays the lambda function from firing to perform the tracking.

> Server access log records are delivered on a best effort basis. Most requests for a bucket that is properly configured for logging result in a delivered log record. Most log records are delivered within a few hours of the time that they are recorded, but they can be delivered more frequently.
