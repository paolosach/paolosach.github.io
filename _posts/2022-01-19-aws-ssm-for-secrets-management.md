---
layout: single
title:  "Using secrets in aws ssm"
categories: update awscli ssm aws
toc: true
excerpt: "secrets management is a hot topic for cloud applications, and aws ssm can be very helpful, if you know how"
---
I have been using aws elastic beanstalk a lot in the last few months and - forcefully - I have  become acquainted with the `.ebextensions` folder and the `.config` files that live in it.

At first I didn't feel comfortable at all with this kind of architecture, as all the commands and files in the ec2 instances managed by beanstalk are handled through a number of yaml files, but I got used to it in time and now I am at peace with it.

Of course, having to run commands that rely on secrets, and being in need of deploying code to github, I found myself confronting the awkward situation of where to put those secrets.
## Strategies
### Environment variables
There are of course a number of alternative strategies. My first and worst was to deploy environment variables separately at each app deployment to beanstalk, but this process is obviously less then suboptimal and not very secure.

### Secrets manager
The second attempt, which I'm still using, but I find less practical, is aws secrets manager. I will write about this service sooner or later, as I will have to di into secrets rotation soon, but for now let's just say that secrets manager is a little bit more complex to handle than ssm and, more importantly, it far more expensive.

### SSM parameters
Finally, I decided to try **ssm parameters** and must say that for my simple usecase it works far better than the previous two.

First of all, it has a simple (at least when you have hit your head against it a couple of times) command line api. In fact, creating a new secret is as simple as 
{% highlight bash %}
    aws ssm put-parameter --name "a_very_secret_secret" --type "SecureString" --value "the_very_secret_secret"
{% endhighlight %}

The secret will be encrypted with the default kms key and stored, but it is possible to provide a custom kms key passing its id after a `--key-id` option, add tags with the `--tags` option and more: `aws ssm put-parameter help` is your friend if you want to discover even more options.

Now, retrieving a secret is a little bit more cumbersome. Using
{% highlight bash %}
    aws ssm get-parameter --name "a_very_secret_secret"
{% endhighlight %}

will in fact return a dictionary like this
{% highlight bash %}
    {
        "Parameter": {
            "Name": "a_very_secret_secret",
            "Type": "SecureString",
            "Value": "the_encrypted_very_secret_secret",
            "Version": 1,
            "LastModifiedDate": "2022-01-19T15:20:22.231000+01:00",
            "ARN": "arn:aws:ssm:aws-region-x:xxxxxxxxx:parameter/a_very_secret_secret",
            "DataType": "text"
        }
    }
{% endhighlight %}

which I believe is rarely what one is looking for, espetially because the value is return encrypted.

To get a more useful result, the option we want is `--with-decryption`
{% highlight bash %}
    aws ssm get-parameter --name "a_very_secret_secret --with-decryption"
{% endhighlight %}

 which will still get us a dictionary, but at least will expose the actual secret value.
{% highlight bash %}
    {
        "Parameter": {
            "Name": "a_very_secret_secret",
            "Type": "SecureString",
            "Value": "the_very_secret_secret",
            "Version": 1,
            "LastModifiedDate": "2022-01-19T15:20:22.231000+01:00",
            "ARN": "arn:aws:ssm:aws-region-x:xxxxxxxxx:parameter/a_very_secret_secret",
            "DataType": "text"
        }
    }
{% endhighlight %}

To access the nodes in the dictionay, we must use the cli command option `--query`, which you won't find in the help because it is common to all of the aws cli commands, but will allow you to access the value of your secret.
{% highlight bash %}
    aws ssm get-parameter --name a_very_secret_secret --with-decryption --query Parameter.Value
{% endhighlight %}

will return
{% highlight bash %}
    "the_very_secret_secret"
{% endhighlight %}

Depending on how you have configured your aws cli, the output can be of different type. The default is **json**, so your value will appear between quotes.
If you need it to be in plain text like I did, because maybe you want to use it in a bash script, you can set the `--output` option to `text`. So
{% highlight bash %}
    aws ssm get-parameter --name a_very_secret_secret --with-decryption --query Parameter.Value --output text
{% endhighlight %}

will return
{% highlight bash %}
    the_very_secret_secret
{% endhighlight %}

Nice. Be aware though that there is also another command, `get-parameters` (plural), that will instead retrieve a list of parameters given a list of names. I write about this because I found an policy that granted an instance permission to use `GetParameters` but not `GetParameter` and this wasn't that straight forward to figure.

`get-parameters` is very similar to his singular version, except that options are plural as well, and it retrieves a list of dictionaries, so we need to take this into account when running it.
{% highlight bash %}
    aws ssm get-parameters --names a_very_secret_secret --with-decryption
{% endhighlight %}

will return
{% highlight bash %}
    {
        "Parameters": [
            {
                "Name": "a_very_secret_secret",
                "Type": "SecureString",
                "Value": "the_very_secret_secret",
                "Version": 1,
                "LastModifiedDate": "2022-01-19T15:20:22.231000+01:00",
                "ARN": "arn:aws:ssm:aws-region-x:xxxxxxxxx:parameter/a_very_secret_secret",
                "DataType": "text"
            }
        ],
        "InvalidParameters": []
    }
{% endhighlight %}

So to access our secret value in plain text, we will run
{% highlight bash %}
    aws ssm get-parameters --names a_very_secret_secret --with-decryption --query "Parameters[0].Value" --output text
{% endhighlight %}

Mind the quotes around `"Parameters[0].Value"`, as it won't work without them.