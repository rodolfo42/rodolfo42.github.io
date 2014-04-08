---
layout: post
title: Script to delete all objects in a AWS S3 bucket
date: 2013-05-02
comments: true
---

## Why?

This is mainly for *test* cases, when you're new to AWS S3 and you create a *test* bucket to start *testing* things out. Soon you discover there's a lot of extra objects like `FooBar2013-03-29-02-54-37-E56AFCD9348BBA9D776F6C4` (that's ok, it's just the logging), but when you try to delete this entire test bucket, you can't, because **S3 buckets need to be empty before they are deleted**.

Since I particularly had many many objects like these, I started going through all of them in the AWS S3 web interface deleting, waiting, then selecting again (because the web interface doesn't have a "select all" feature to date - I think for security/performance reasons).

<!-- more -->

**Update (11/06/2013):** you should use `bucket.delete!` instead as it does all the work of clearing the bucket beforehand - [see here][AWSS3BucketDelete]

## Deleting all objects through a ruby script using AWS SDK

The script below was adapted from [this answer][StackOverflowAnswer]. I first tried to use it to delete my test bucket, but the original script in this answer seems to be outdated (the new AWS SDK works a little bit differently). So here it is:

{% gist 5625123 deleteall.rb %}

> Don't forget to replace things like `YOUR_ACCESS_KEY_ID` with your own data.

[StackOverflowAnswer]: http://stackoverflow.com/a/1179190
[AWSS3BucketDelete]: http://docs.aws.amazon.com/AWSRubySDK/latest/AWS/S3/Bucket.html#delete!-instance_method