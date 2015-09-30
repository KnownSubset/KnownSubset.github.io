---
layout: post
title: AWS S3 Errors - EntityTooSmall
---

I was recently reworking an upload process at my job to make use of the Multipart Upload service that is a part of AWS S3. Everything was working fine and I was able to send the 1 MB chunks off to S3.  However, once I went to complete the multipart upload I started to receive the EntityTooSmall error back from the Ruby SDK.  I wasn't entirely too clear about why it would complain since it accepted all of the chunks and only rejected things once I tried to complete the multipart upload.

The [AWS docs](http://docs.aws.amazon.com/AmazonS3/latest/API/ErrorResponses.html) do not do much to tell you what the hell happened if you are receiving this error. You will have to dig down to these [docs](http://docs.aws.amazon.com/AmazonS3/latest/API/mpUploadComplete.html) to finally get some clue.  Apparently AWS considers 5 MB to be the minimum size of a multipart upload that they should have to encounter (excluding the last chunk).  
