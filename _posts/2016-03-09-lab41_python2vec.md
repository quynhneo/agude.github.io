---
layout: post
title: "Python2Vec: Word Embeddings for Source Code"
gab41: http://www.lab41.org/py2vec/
description: >
  Parsing source code is easy; just let the interpreter do it! But what if you
  want to recommend code snippets? Then you need word embeddings, like my
  Python2Vec!
image: /files/python2vec/header.jpg
image_alt: >
  A generic picture of code on a screen. It's HTML, not Python, but whatever.
categories: 
  - lab41
  - hermes
---

{% capture file_dir %}/files/python2vec/{% endcapture %}

The reason that Lab41 embarked on the [Hermes recommender systems
challenge][anna] was to move towards assisting data scientists and software
engineers in finding data, tools, and code. Without knowing anything about
such items (except for how users interacted with them) we are limited to using
simple collaborative filtering algorithms. However, if we learned more about
the properties of the various items, we could use more sophisticated
algorithms.

[anna]: https://gab41.lab41.org/recommending-recommendation-systems-cc39ace3f5c1#.li0lnmqi8

For some items---for example, movies---it is relatively easy to imagine what
sort of properties are relevant in order to make recommendations: genre,
actors, director, story elements. For other items it is more difficult; in a
code database, what characteristics of the array function would we track to
make sure it is recommended to the right users?

Fortunately, we do not have to answer this question to make use of
content-based recommender systems for code (although if we were able to, we
would likely produce better predictions). Instead, we just need to determine
how to extract the properties of the code into a vector representation, taking
both syntactic and semantic meaning into account. If the final vector is
inscrutable, so be it, so long as it works for our application. Of course,
this sounds like an even more intractable problem, until we remember... word
embeddings!

## Word Embeddings

Word embeddings have proved to be a useful way of representing text for
machine learning. We used them in [our work on Chinese censorship][chinese],
where we found them effective even on small, minimally-processed foreign
language datasets. In this post, I will give a brief overview of how Word2Vec
works; for a more complete summary see [Karl's post on Anything2Vec][a2v] and
[this paper by Goldberg and Levy][paper].

[chinese]: https://gab41.lab41.org/can-word-vectors-help-predict-whether-your-chinese-tweet-gets-censored-711e7682d12f#.jckj1z8q1
[a2v]: https://gab41.lab41.org/anything2vec-e99ec0dc186#.cdtxo834y
[paper]: https://arxiv.org/abs/1402.3722

Word2Vec embeds words into an n-dimensional vector space such that words that
appear close in the source text (code in our case) are close in the final
vector space. It does this by creating two types of vectors: word vectors
(used after the training) and context vectors (used in training and mostly
forgotten afterwards). Let's consider an example using Python source code:

![Taking two Python inport statements and converting them to word arrays.]({{
site.url }}/files/python2vec/01_import.svg)

In this case `import math` and `import numpy as np` are two lines from a
Python script. We are interested in the target words "math" and "numpy". These
are the words we are trying to embed in our word vector space, which will
eventually help us decide when to recommend the math and numpy libraries to
users. The words surrounding the targets are the context. I have highlighted
our target words in red and the context words in blue. Word2Vec tries to
optimize the position of both sets of vectors such that the dot product
between a target word vector and its context vectors is large (making the
angle between them small). This might result in vectors (in 2D for clarity,
although in practice anything between 50 and 300 dimensions is common) that
look like this:

![The words "numpy" and "math" as vectors with context vectors.]({{ site.url
}}/files/python2vec/02_context_space.svg)

Notice that the red target word vectors have been selected such that they end
up close to their blue context vectors. Because `math` and `numpy` share
similar contexts (they both appear near import in the code, for example) they
will appear near each other in the final vector space. As a final step, the
context vectors are discarded and only the red word vectors remain in the
model:

![The words "numpy" and "math" as vectors, now embedded in a Word2Vec space.
]({{ site.url }}/files/python2vec/03_word_space.svg)

There are, of course, some complications. The algorithm could maximize the dot
product by making all the word vectors the same. This is handled by selecting
random pairs of words and contexts (for example, selecting the context
`import ... as np` and the word `for`) and performing the same vector procedure,
but this time insisting that the dot product be small (driving the vectors
apart). The assumption here is that such random selection (otherwise known as
negative sampling) selects words that do not belong in the context, and hence
should not be near those context vectors. This might result in a vector like
the following:

![Negative sampling the word "for" to push it apart from the word "import" and
"math".]({{ site.url }}/files/python2vec/04_negative_word_space.svg)

Once you have completed this process, you have a Word2Vec model!

## Training Python2Vec

In order to build a Python2Vec model we need lots of Python data. In [my last
post][datasets_post] I introduced some of the datasets we worked with,
including a single Python repository. Since then we have added many
repositories to the dataset. These are:

[datasets_post]: {% post_url 2016-02-08-lab41_recommender_systems_datasets %}

| Library                         | Description                       |
|:--------------------------------|:----------------------------------|
| [**ansible**][ansible]          | An IT automation platform.        |
| [**beets**][beets]              | A music library manager.          |
| [**django**][django]            | A web framework.                  |
| [**flask**][flask]              | A micro web framework.            |
| [**Mailpile**][mailpile]        | An email client.                  |
| [**matplotlib**][mpl]           | A plotting library.               |
| [**NewsBlur**][newsblur]        | A newsreader.                     |
| [**numpy**][numpy]              | A scientific computing library.   |
| [**pandas**][pandas]            | A columnar data analysis library. |
| [**requests**][requests]        | A HTTP requests library.          |
| [**salt**][salt]                | An IT automation platform.        |
| [**scikit-learn**][skl]         | A machine learning library.       |
| [**scipy**][scipy]              | A scientific computing library.   |
| [**scrapy**][scrapy]            | A web scraper.                    |
| [**sentry**][sentry]            | A crash reporting utility.        |
| [**sshuttle**][sshuttle]        | A proxy server.                   |
| [**SublimeCodeIntel**][sublime] | An autocomplete engine.           |

[mpl]: https://github.com/matplotlib/matplotlib
[skl]: https://github.com/scikit-learn/scikit-learn
[numpy]: https://github.com/numpy/numpy
[pandas]: https://github.com/pandas-dev/pandas
[django]: https://github.com/django/django
[scipy]: https://github.com/scipy/scipy
[flask]: https://github.com/pallets/flask
[requests]: https://github.com/kennethreitz/requests
[ansible]: https://github.com/ansible/ansible
[sentry]: https://github.com/getsentry/sentry
[scrapy]: https://github.com/scrapy/scrapy
[mailpile]: https://github.com/mailpile/Mailpile
[sshuttle]: https://github.com/apenwarr/sshuttle
[salt]: https://github.com/saltstack/salt
[newsblur]: https://github.com/samuelclay/NewsBlur
[beets]: https://github.com/beetbox/beets
[sublime]: https://github.com/SublimeCodeIntel/SublimeCodeIntel

Originally we were heavily biased towards data science-related libraries (as
you might expect, us being data scientists), so we have tried to diversify by
selecting some popular non-data science related projects.

In total this gives us 9424 files written by 5307 authors, consisting of
2,555,194 lines. We tokenize the lines by lowercasing everything and splitting
on all whitespace and punctuation characters. This leaves us with 10,325,475
words, of which 254,851 are distinct.

This data were processed with [MLlib's Word2Vec class][mllib_w2v] as follows:

[mllib_w2v]: https://spark.apache.org/docs/latest/mllib-feature-extraction.html#word2vec

```python
from pyspark.mllib.feature import Word2Vec

word2vec = Word2Vec()
word2vec.setMinCount(25)
word2vec.setVectorSize(50)
```

The resulting model and code needed to explore it can be found
[here][p2v_files]. Included is an [Jupyter notebook with an example of using
the model][model_nb]. If you would like to try training your own Python2Vec
model, use the [data processing script][data_sh] in the main Hermes project to
download the raw data and then use the [model training notebook][training_nb]
to train your model.

[p2v_files]: https://github.com/Lab41/Misc/tree/master/blog/python2vec
[model_nb]: https://github.com/Lab41/Misc/blob/master/blog/python2vec/Python2Vec%20Example.ipynb
[data_sh]: https://github.com/Lab41/hermes/blob/master/src/utils/code_etl/download_data.sh
[training_nb]: https://github.com/Lab41/hermes/blob/master/src/data_prep/model/Python2Vec--Save%20Model.ipynb

## Exploring the Model

Now that we have a model, we can explore it and see what comes out! First we
find all the words near `range`, a function often used in for loops:

```python
>>> model.closest_words('range', n=5)

[(1.701, u'i'),
 (2.025, u'zip'),
 (2.128, u'xrange'),
 (2.536, u'03d'),
 (2.586, u'k')]
```

The five words that come up are all things you would expect to appear near
`range` in the code: `i` and `k` are common looping variable names, `zip` is a
function to manipulate lists, and `xrange` performs the same function as `range`
but without instantiating a list. `03d` seems slightly odd at first, but it is
used in number formatting expressions and pairs well with `range`, which
generates lists of numbers. In a recommender sense, suggesting `xrange` and
`zip` would be useful, as someone using `range` would almost certainly benefit
from knowing about them if they did not.

Let's try our original example from the introduction:

```python
>>> model.closest_words('array', n=5)

[(1.563, u'masked_equal'),
 (1.579, u'zeros'),
 (1.616, u'newshape'),
 (1.618, u'rational'),
 (1.696, u'unravel_index')]
```

Again we find many similarities to the `array` function: `zeros` creates an
all-zero array, `masked_equal` operates on an array, `newshape` is an argument
used when reshaping an array, `rational` is a data type that can be stored in
an array, and `unravel_index` converts 1D indices into multiple dimensions.
_Note_: I had never heard of `unravel_index` until looking at this model! So
Python2Vec is already exposing me to new information.

One of the striking features of Word2Vec models trained on English is that
they learn "analogy" vectors. For example, the infamous `king - man + woman =
queen` relation (and other similar examples) show that there exists some
"gender" vector even though it does not correspond directly to a vector for
any English word. Looking for similar features in Python2Vec is tough: we
would first have to determine a relationship and then see if it exists in the
model. So far I have not found any.

## Can You Recommend It?

The Hermes challenge, though, is about recommending, so are these vectors
useful for that purpose? To test this, we looked at every line of code in the
dataset and collected its most recent author from the git information. We then
assembled vectors mapping each user to a function, method, or library that
they had used in a line of code. We split this data into a training and a test
set, trained several recommender algorithms on the training set, and tested
them on the test set. We compared a collaborative filtering algorithm (which
uses only the interaction data) to content-based algorithms which use the
Python2Vec vectors. A recommendation is considered successful if the item
appears in the training set.

In terms of metrics (see [Tiffany's post][tps] and [Anna's post][metrics] for
a refresher of what these metrics mean), the RMSE and MAE decrease with
Python2Vec compared to [MLlib's ALS collaborative filter][mllib_collab].
Serendipity is also higher. However, the item coverage is significantly
reduced. This is likely because we only consider words that appear in our
corpus at least 25 times, so we only cover the most common functions, methods,
and libraries. The user coverage decreases, but not as significantly---most
users, it seems, use at least a few of the most common functions.

[tps]: https://gab41.lab41.org/tps-report-for-recommender-systems-yeah-that-would-be-great-3beb26ab9fe0#.o2frg2eat
[metrics]:  https://gab41.lab41.org/recommender-systems-its-not-all-about-the-accuracy-562c7dceeaff#.n291twu23
[mllib_collab]: https://spark.apache.org/docs/latest/mllib-collaborative-filtering.html

The low item coverage is a particularly challenging issue. We set the
inclusion threshold to 25 occurrences in order to remove variable names and
one-off functions, but it also removes functions that are rarely used. These
rarely used functions are precisely the ones that we are most interested in
recommending to the user. Some additional work in this area is needed.

Recommending source code, it turns out, is challenging. Python2Vec offers some
promise in this regard, but it is likely that different methods of extracting
meaning from source code could outperform it. Python2Vec considers matters at
the word level, but a larger unit of code is probably more useful.
Additionally, the distance between objects in the source codes (as used by
Python2Vec) indicates that they are related, but doesn't give us any hint as
to their function. We have tried to incorporate information about the purpose
of these functions in some other models, mainly by trying to parse the
documentation, but we have not yet achieved performance comparable to the
model presented here.
