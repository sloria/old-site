---
layout: post
title: "Tutorial: What is WordNet? A Conceptual Introduction Using Python"
description: ""
category: programming
tags: [programming, python, textblob, nlp]
---
{% include JB/setup %}

In short, WordNet is a database of English words that are linked together by their semantic relationships. It is like a supercharged dictionary/thesaurus with a graph structure.

[TextBlob](http://textblob.readthedocs.org/) 0.7
([changelog](http://textblob.readthedocs.org/en/latest/changelog.html)) now integrates NLTK's WordNet interface, making it very simple to interact with WordNet.

This tutorial is a gentle introduction to WordNet concepts, using TextBlob for the examples. To follow along with the examples, make sure you have the latest version of TextBlob.

<pre><code class="bash">$ pip install -U textblob</code></pre>

## Synsets

As you know, synonyms are words that have similar meanings. A synonym set, or **synset**, is a group of synonyms. A synset, therefore, corresponds to an abstract concept.

In TextBlob, you can access the synsets that a word belongs to by accessing the `synsets` property of a `Word` object.

<pre><code>from text.blob import Word
word = Word("plant")
word.synsets[:5]
# [Synset('plant.n.01'),
#  Synset('plant.n.02'),
#  Synset('plant.n.03'),
#  Synset('plant.n.04'),
#  Synset('plant.v.01')]
</code></pre>


It would be helpful to know the definitions of these synsets. You can access these via the `definitions` property.

<pre><code>word.definitions[:5]
# ['buildings for carrying on industrial labor',
#  '(botany) a living organism lacking the power of locomotion',
#  'an actor situated in the audience whose acting is rehearsed but seems spontaneous to the audience',
#  'something planted secretly for discovery by another',
#  'put or set (seeds, seedlings, or plants) into the ground']
</code></pre>

For this tutorial, let's study "plant" as a living organism. We can see from our
`definitions` list that this is the second synset (index 1).


<pre><code>plant = word.synsets[1]
</code></pre>

The synonyms contained within a synset are called **lemmas**. You can access the string versions of these synonyms via a `Synset`'s `lemma_names` property.

<pre><code>plant.lemma_names
# ['plant', 'flora', 'plant_life']
</code></pre>

## The Wordnet Hierarchy

Synsets form relations with other synsets to form a hierarchy of concepts,
ranging from very general ("entity", "state") to moderately abstract ("animal")
to very specific ("plankton").

Some terminology: For a given synset, its. . .

- **hypernyms** are the synsets that are more *general*
- **hyponyms** are the synsets that are more *specific*

Hyponyms have an **"is-a"** relationship to their hypernyms.

Let's use our plant synset as an example.

<pre><code class="python">plant.hypernyms()
# [Synset('organism.n.01')]
plant.hyponyms()[:3]
# [Synset('phytoplankton.n.01'), Synset('aquatic.n.01'), Synset('perennial.n.01')]
</code></pre>

We can therefore see the following relationships:

![Hypernyms and hyponyms graph](https://dl.dropboxusercontent.com/u/1693233/blog/hypernyms-hyponyms.png)

Along with "is-a" relationships, we can explore "is-made-of" and "comprises"
relationships.

For a given synset, its. . .

* **holonyms** are things that the item is contained in
* **meronyms** are components or substances that make up the item

Let's look at our plant example again.

<pre><code>plant.member_holonyms()
# [Synset('plantae.n.01')]
plant.part_meronyms()
# [Synset('plant_part.n.01'), Synset('hood.n.02')]
</code></pre>

This shows the following relationships:

![Holonym and meronym graph](https://dl.dropboxusercontent.com/u/1693233/blog/holonyms-meronyms.png)

## Semantic similarity

Given that synsets can be organized as a graph, as shown above, we can measure
the similarity of synsets based on the shortest path between them. This is
called the **path similarity**, and it is equal to `1 /
(shortest_path_distance(synset1, synset2) + 1)`. It ranges from 0.0 (least
similar) to 1.0 (identical).

Let's compare the path similarities between "octopus" and "nautilus" (another
cephalapod), "shrimp" (a non-cephalopod), and "pearl" (a mineral). We'll create the synsets directly.

<pre><code>from text.wordnet import Synset
octopus = Synset("octopus.n.02")
nautilus = Synset('paper_nautilus.n.01')
shrimp = Synset('shrimp.n.03')
pearl = Synset('pearl.n.01')
</code></pre>

The results are as expected, with octopus more similar to another cephalopod
than a non-cephalapod and most dissimilar to a non-living thing.

<pre><code>octopus.path_similarity(octopus)  # 1.0
octopus.path_similarity(nautilus)  # 0.33
octopus.path_similarity(shrimp)  # 0.11
octopus.path_similarity(pearl)  # 0.07
</code></pre>

There are other WordNet-based measures of similarity to explore, which can be found at the [NLTK Wordnet Docs](http://nltk.googlecode.com/svn/trunk/doc/howto/wordnet.html). All these methods are  accessible in TextBlob.

### Further reading

- [WordNet on Wikipedia](https://en.wikipedia.org/wiki/Wordnet)
- [NLTK Book Ch. 2](http://nltk.org/book/ch02.html)
- [NLTK Wordnet
Docs](http://nltk.googlecode.com/svn/trunk/doc/howto/wordnet.html)


### Other Tutorials in this Series

- [State-of-the-art Part-of-speech Tagging in
TextBlob](http://www.stevenloria.com/tutorial-state-of-the-art-part-of-speech-
tagging-in-textblob/)
- [Finding Important Words in Text Using TF-IDF](http://www.stevenloria.com
/finding-important-words-in-a-document-using-tf-idf/)
- [Simple Text Classification in Python and TextBlob](http://www.stevenloria.com
/how-to-build-a-text-classification-system-with-python-and-textblob/)
