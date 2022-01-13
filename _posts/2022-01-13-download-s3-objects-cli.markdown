---
layout: single
title:  "How to download objects from S3 in da CLI"
categories: s3, aws, cli
---
AWS CLI has been an intimidating entity for me at first, even though i knew it was much better then always going to the console for basic operations.

That is why i decided to write a series of how tos, so that maybe i will memorize commands better and will have a reference instead of roaming the web searching for answers.

I will start with downloading objects from S3, which is a common thing to do. 

## cp
Downloading an entire folder is as simple as 

{% highlight bash %}
    aws s3 cp s3://{bucketname}/{foldername} {local_folder_destination} --recursive
{% endhighlight %}

A cool thing is that this works also the other way around, so uploading objects just means switching source and destination:

{% highlight bash %}
    aws s3 cp {local_folder_destination} s3://{bucketname}/{foldername} --recursive
{% endhighlight %}

If this isn't enough, source and destination can also be both buckets, if copying objects between buckets is needed.

Going deeper into the subjects, to be more selective with what to download the cp command has two options: `--include` and `--exclude`, followed by a quoted string or regex, allow to only select some objects.

## sync
Another more advanced command is `sync`, which allows to download or upload only new or updated objects.

{% highlight bash %}
    aws s3 sync s3://{bucketname}/{foldername} {local_folder_destination}
{% endhighlight %}

Like `cp`, `sync` also has the `--include` and `--exclude` options.

It also has a `--delete` option, which will delete objects if they are present in the destination but not in the source.

`sync` also have a lot more options, that more specific to encryption, permissions and metadata and the details can always be foud with the `help` command

{% highlight bash %}
    aws s3 sync help
{% endhighlight %}