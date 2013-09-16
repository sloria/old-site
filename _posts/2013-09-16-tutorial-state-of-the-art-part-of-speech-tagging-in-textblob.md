---
layout: post
title: "Tutorial: State-of-the-art Part-of-Speech Tagging in TextBlob"
description: ""
category: programming
tags: [programming, python, textblob, nlp]
---
{% include JB/setup %}

Following the tradition of writing a short tutorial with each new [TextBlob][] release (0.6.3, [changelog](https://textblob.readthedocs.org/en/latest/changelog.html)), here's an introduction to TextBlob's first outside code contribution from Matthew Honnibal, a.k.a. [syllog1sm](https://github.com/syllog1sm): a part-of-speech tagger based on the [Averaged Perceptron][Perceptron] algorithm which is faster and more accurate than [NLTK](http://nltk.org/)'s and [pattern](http://www.clips.ua.ac.be/pages/pattern-en)'s default implementations.

Matthew Honnibal wrote a clear and detailed blog post about the Averaged Perception and his implementation [here][PyAP]. For this reason, this post will focus on how to get and use the tagger without providing implementation details.

## Getting the `PerceptronTagger`

First, upgrade to the latest version of `TextBlob`.

<pre><code class="bash">$ pip install -U textblob</code></pre>

The `PerceptronTagger` requires a `trontagger.pickle` file that is not included in the TextBlob distribution (in order to keep the distribution lightweight).

The file can be downloaded from [TextBlob's Releases page](https://github.com/sloria/TextBlob/releases) on Github [^1].

<a href="https://github.com/sloria/TextBlob/releases"><img src="https://dl.dropboxusercontent.com/u/1693233/blog/releases.png" alt="TextBlob releases page"></a>

After downloading, the file, unzip it. On MacOSX, this can be done by double-clicking the file. Or you can use the shell:

<pre><code class="bash">$ gunzip trontagger.pickle.gz</code></pre>

You should now have `trontagger.pickle`. You need to put this in your `TextBlob` installation directory. To find where this is, run

<pre><code class="bash">$ python -c "import text; print(text.__path__[0])"</code></pre>

This will output the TextBlob directory. Place `trontagger.pickle` in this directory.

You're all set to use the tagger!

## A Short Intro to the `Blobber` Class

Let's start tagging some text. To do this, you pass an instance of the tagger into the `TextBlob` constructor.

<pre><code class="python">from text.blob import TextBlob as tb
from text.taggers import PerceptronTagger

ap_tagger = PerceptronTagger()
# This is verbose; we'll see a DRYer version later
b1 = tb("Beautiful is better than ugly.", pos_tagger=ap_tagger)
b2 = tb("Simple is better than complex.", pos_tagger=ap_tagger)
print(b1.tags)
# [('Beautiful', u'NNP'), ('is', u'VBZ'), ('better', u'JJR'), ('than', u'IN'), ('ugly', u'RB')]
print(b2.tags)
# [('Simple', u'NN'), ('is', u'VBZ'), ('better', u'JJR'), ('than', u'IN'), ('complex', u'JJ')]
</code></pre>

However, passing the tagger can get repetitive when making many `TextBlobs`. To avoid this, we can use the ``Blobber`` class, which is a "factory" that creates `TextBlobs` that share the same models. Let's rewrite the above code using a `Blobber`.

<pre><code class="python">from text.blob import Blobber
from text.taggers import PerceptronTagger

tb = Blobber(pos_tagger=PerceptronTagger())
b1 = tb("Beautiful is better than ugly.")
b2 = tb("Simple is better than complex")
print(b1.pos_tagger is b2.pos_tagger)  # True
print(b1.tags)
print(b2.tags)
</code></pre>

## Evaluating the Taggers

Now let's do a quick-and-dirty accuracy comparison of the Perceptron tagger with NLTK's and pattern's implementations.

The test data will be three tagged sentences (81 total words), stored as a list of lists.

<pre><code class="python">test = [[(u'Pierre', u'NNP'), (u'Vinken', u'NNP'), (u',', u','), (u'61', u'CD'), 
            (u'years', u'NNS'), (u'old', u'JJ'), (u',', u','), (u'will', u'MD'), 
            (u'join', u'VB'), (u'the', u'DT'), (u'board', u'NN'), (u'as', u'IN'), 
            (u'a', u'DT'), (u'nonexecutive', u'JJ'), (u'director', u'NN'), 
            (u'Nov.', u'NNP'), (u'29', u'CD'), (u'.', u'.')], 
        [(u'Mr.', u'NNP'), (u'Vinken', u'NNP'), (u'is', u'VBZ'), (u'chairman', u'NN'), 
            (u'of', u'IN'), (u'Elsevier', u'NNP'), (u'N.V.', u'NNP'), (u',', u','), 
            (u'the', u'DT'), (u'Dutch', u'NNP'), (u'publishing', u'VBG'), 
            (u'group', u'NN'), (u'.', u'.'), (u'Rudolph', u'NNP'), (u'Agnew', u'NNP'), 
            (u',', u','), (u'55', u'CD'), (u'years', u'NNS'), (u'old', u'JJ'), 
            (u'and', u'CC'), (u'former', u'JJ'), (u'chairman', u'NN'), (u'of', u'IN'), 
            (u'Consolidated', u'NNP'), (u'Gold', u'NNP'), (u'Fields', u'NNP'), 
            (u'PLC', u'NNP'), (u',', u','), (u'was', u'VBD'), (u'named', u'VBN'), 
            (u'a', u'DT'), (u'nonexecutive', u'JJ'), (u'director', u'NN'), (u'of', u'IN'), 
            (u'this', u'DT'), (u'British', u'JJ'), (u'industrial', u'JJ'), 
            (u'conglomerate', u'NN'), (u'.', u'.')], 
        [(u'A', u'DT'), (u'form', u'NN'), 
            (u'of', u'IN'), (u'asbestos', u'NN'), (u'once', u'RB'), (u'used', u'VBN'), 
            (u'to', u'TO'), (u'make', u'VB'), (u'Kent', u'NNP'), (u'cigarette', u'NN'), 
            (u'filters', u'NNS'), (u'has', u'VBZ'), (u'caused', u'VBN'), (u'a', u'DT'), 
            (u'high', u'JJ'), (u'percentage', u'NN'), (u'of', u'IN'), 
            (u'cancer', u'NN'), (u'deaths', u'NNS'),
            (u'among', u'IN'), (u'a', u'DT'), (u'group', u'NN'), (u'of', u'IN'), 
            (u'workers', u'NNS'), (u'exposed', u'VBN'), (u'to', u'TO'), (u'it', u'PRP'), 
            (u'more', u'RBR'), (u'than', u'IN'), (u'30', u'CD'), (u'years', u'NNS'), 
            (u'ago', u'IN'), (u',', u','), (u'researchers', u'NNS'), 
            (u'reported', u'VBD'), (u'.', u'.')]]
</code></pre>

We then define an `accuracy()` method that is passed our test dataset and an instance of a tagger.

<pre><code class="python">import string
from text.blob import Blobber
from text.taggers import PerceptronTagger, PatternTagger, NLTKTagger

def accuracy(test_set, tagger):
    n_correct = 0
    total = 0
    tb = Blobber(pos_tagger=tagger)
    for tagged_sentence in test_set:
        # Get the untagged sentence string
        # e.g. "Pierre Vinken , 61 years old , will join the board ..."
        raw_sentence = ' '.join([word for word, tag in tagged_sentence])
        blob = tb(raw_sentence)  # Create a blob that uses the specified tagger
        # tagger excludes punctuation by default
        tags = [tag for word, tag in blob.tags]
        # exclude punctuation in test data
        target_tags = [tag for word, tag in tagged_sentence 
                       if tag not in string.punctuation]
        total += len(tags)
        # Add the number of correct tags
        n_correct += sum(1 for i in range(len(tags)) if tags[i] == target_tags[i])
    return float(n_correct) / total  # The accuracy
</code></pre>

We can then get the accuracy of each tagger.

<pre><code class="python">print(accuracy(test, PerceptronTagger()))
print(accuracy(test, NLTKTagger()))
print(accuracy(test, PatternTagger()))
</code></pre>

Full script is [here](https://gist.github.com/sloria/6576933).

### Results

| Tagger             | Accuracy   |
| :---------         | :--------- |
| `PerceptronTagger` | **98.8%**  |
| `NLTKTagger`       | 94.0%      |
| `PatternTagger`    | 91.6%      |

You can find more extensive evaluations of these three taggers at [Matthew's blog post][PyAP]. The numbers are quite impressive. Thank you Matthew for all your hard work on this!

## Further Reading

* [A Good POS Tagger in About 200 Lines of Python - Matthew Honnibal](http://honnibal.wordpress.com/2013/09/11/a-good-part-of-speechpos-tagger-in-about-200-lines-of-python/)
* [A Course On Machine Learning, Ch. 3. - The Perceptron][Perceptron]
* [TextBlob Docs - Overriding Models and the Blobber Class](https://textblob.readthedocs.org/en/latest/advanced_usage.html)

[^1]: Hosting supplemental models and data on the Releases page is not an ideal solution for the long-term because Github imposes a 5MB limit on file uploads, but it will work for now. If you have a suggestion for a better solution, please join the discussion [here](https://github.com/sloria/TextBlob/issues/20).

[TextBlob]: https://textblob.readthedocs.org/
[PyAP]: http://honnibal.wordpress.com/2013/09/11/a-good-part-of-speechpos-tagger-in-about-200-lines-of-python/
[Perceptron]: http://ciml.info/dl/v0_8/ciml-v0_8-ch03.pdf
