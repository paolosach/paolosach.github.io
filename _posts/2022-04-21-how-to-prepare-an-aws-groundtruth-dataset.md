---
layout: single
title:  "How to prepare an AWS SageMaker GroundTruth text dataset"
categories: update aws python sagemaker json
tags: pandas unicode groundtruth
toc: true
excerpt: "Supervised learning requires data labeling and AWS GroundTruth is the ideal tool for the task, but the set up has not proved to be so obvious"
---
# The Task
I want to train a text classification model and to do this I need a labeled dataset. To label a few thousands documents from uncontrolled sources, I decided to use AWS GroundTruth, as it has a ready to use interface, is connected with Cognito to provide authenticated access and handles sources and outputs in S3. It is also able to automatically import `.csv` or `.txt` data. **Theoretically**.
I use `python` and `pandas` to build my labeling dataset and dump it to a `.csv` file.

# The Problem
Turns out that, after going through the basic configuration, feeding the system a `.csv` file from S3, the creation of the labeling process fails without any clue about the cause. And once this fails, I cannot even delete the failed process.

Going through the documentation, I discover that the automated import process creates a `.manifest` file that contains my documents, formatted as sequence of json documents, one per line. A `.jsonl` format, they say.

So I find this `.manifest` file in S3 and give it a look. What I find is that instead of writing a json line for each line in my `.csv`, it wrote a line every time it found a new line character `\n`.

So I go through my documents and escape every `\n` carachter, as python requires it to print it as two characters instead of a single control character. This way, json can correctly parse it.

This is not enough though. JSON is not very flexible and does not admit control characters in strings, so I have to filter out all of them.

# The Solution
To remove everything I don't want, I use a mix of regex search, match search and the package `unicodedata` which has a method `category` than can determine if a `char` is a control character.

My code to do this looks like this, and it works, but is very unefficient.

{% highlight python %}
    import unicodedata
    label_dataset=raw_dataset.assign(doc_id=raw_dataset.index.to_series())
    label_dataset.loc[:,'doc'] = label_dataset.doc.apply(lambda x: ''.join(char for char in x if (unicodedata.category(char)[0] != 'C' or char == '\n')))
    label_dataset.loc[:,'doc'] = label_dataset.doc.str.replace(r'\\[uU]','u', regex=True)
    label_dataset.loc[:,'doc'] = label_dataset.doc.str.replace('\\','/', regex=False)
    label_dataset.loc[:,'doc'] = label_dataset.doc.str.replace('\n','\\n', regex=False)
    label_dataset.loc[:,'doc'] = label_dataset.doc.str.replace(r' {2,}',' ', regex=True)
    label_dataset.loc[:,'doc'] = label_dataset.doc.str.replace('"',"'", regex=False)
    label_dataset = label_dataset.assign(to_write='{"source": "doc_id=' + label_dataset.doc_id + '\\n' + label_dataset.doc + '"}')
    with open('../data/to_label/to_label.manifest','w') as f:
        [f.writelines(x+'\n') for x in label_dataset.to_write]
{% endhighlight %}

The output of this, as you may guess, is a `.manifest` file ready to be uploaded to S3 and directly read by GroundTruth.

The file format looks like this:

{% highlight json %}
    {"source":"text of first document"}
    {"source":"text of second document\nwith new line"}
{% endhighlight %}

# Conclusions
Text encoding gives me a decent pain, but is fixable. 

In a later article I might give another better solution that I am currently looking into.