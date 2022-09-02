---
layout: single
title:  "How to set up AWS SES to receive emails"
categories: AWS
tags: ses route53 sns email
toc: true
excerpt: "AWS SES can receive email, but its set up is not so straightforward."
---
# The Task
AWS Simple Email System is a very useful tool to send emails at scale in a programatic way, but sometimes you need to also take advantage of incoming mails, especially when you want to programmatically react to one.

But, if setting SES up for sending emails is fairly straightforward, in my experience the same cannot be told for receiving, and in this post I will explain why and how to get it done quickly.

# Why is it difficult
Well, it is not per se, but as it is often the case, the devil hides in the details and elsewhere it is never or badly explained that, while SES can send emails from every AWS region, **it can only receive them in three**.

So if you, like me, have all your resources in basically one region, `eu-central-1` in my case, make sure yours is one of the three (spoiler alert, mine was not).

The regions allowed to receive emails are:
- Ireland (`eu-west-1`)
- N. Virginia (`us-east-1`)
- Oregon (`us-west-2`)

Ha! Hours of reviewing the documentation, and it all came down to this!

# Set up
Well from now on it is nothing but downhill!

Once you've placed yourself in one of the above locations, you will see that a new voice has appeared in the SES menu: **Email receiving**. Great! Now what?

First of all you need to verify a new domain in this region, so go to Verified Identities, select Create Identity, chose Domain and follow the instructions. I'd give it a meaningful subdomain name, like *mail*, so for instance *mail.example.com*.

To verify it, you will need to change your DNS configuration and if you are using Route53 you can select to add the verification records automatically.

Now that you have your mail domain, you need to add an MX record in your provider's settings (e.g. Route53) so create a new `mail.example.com` record, select record type MX and as value write `10 inbound-smtp.<your-region-x>.amazonaws.com`. 10 is the record priority, so if your provider has a specific field for it, put it there separately.

Cool! SES can now receive emails sent to any address *@mail.example.com*! Now we can move on and do something with them.

# Income Rules
Right now if you try to send an email to your domain, nothing will happen. That's because we still need to define some rules so SES knows what to do of the message it receives.

To do so, go to *Email Receiving* (finally!), select `create Rule set` and then `create rule`. Here you can chose whether to apply this rule to specific addresses or to all of them leaving the field empty, and then what to do with the message. The options are:

- Add header
- Bounce
- Invoke lambda
- Deliver to S3
- Publish to SNS topic
- Stop rule set
- Integrate with workmail

To just test that things work, we'll select to publish our message to an SNS topic, so go ahead and create one, give it a name and an access policy that grants SES permissions to publish in it. Finally, create a new subscription of type email or email json and select your preferred endpoint to receive the notification.

Save everything, go back to SES and finish creating the rule. 

**Aaaand you're done!**

To test it, go to Verified identities, selct your new domain, click send test email, write a FROM address and select a Custom scenario to send the message to a custom receipient, which will be anything like *test@mail.example.com*. 

Type in a subject and a body message and hit send. 

You should receive a notification at the address you subscribed to the SNS topic. If so, everything worked and you can move on to doing more interesting things, like fireing up lambdas and all.

Now, pat yourself on the shoulder.