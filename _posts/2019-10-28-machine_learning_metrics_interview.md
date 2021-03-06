---
layout: post
title: "Interview Question: What Machine Learning Metric to Use"
description: >
  One of my favorite questions to ask in an interview is "What metric should
  you use to decide if your model works?". Read on to find out what a good
  answer looks like!
image: /files/interviews/Artgate_Fondazione_Cariplo_-_Canova_Antonio,_Allegoria_della_Giustizia.jpg
image_alt: >
  A relief showing a women holding up a set of balance scales.
categories:
  - machine-learning
  - interviewing
  - interview-prep
---

{% capture file_dir %}/files/interviews/{% endcapture %}

As part of our interview cycle, candidates work with some data and build a
simple model. After we talk through the modeling and data work, I ask them to
come up with a business case for the model. Once they have done so, I follow
up with:

> How would you measure the success of this model in production?

I have heard a lot of answers. They generally fall into two categories:
machine learning theory focused, and business focused. The first is good, the
second is better. I will go through each below.

To have something concrete to discuss, we will consider the following problem:
"Train a model to classify customers as _'high value customer'_, and use it to
decide if we are going to show them an up-sell page."

## Machine Learning Theory Focused

A common answer, especially from more junior interviewees, is "Accuracy",
which is the number of things your model classifies correctly divided by the
total number of things.

**Accuracy is rarely a good answer for real world problems**, because the
classes are often imbalanced. If only 1 in 1000 users is a _'high value
customer'_, then a model that only returns 'false', despite having an accuracy
of 99.9%, would be clearly worthless.

When pushed about accuracy's obvious shortcomings, the candidate may fall back
to something like F1 score, which is the harmonic mean of precision and
recall. **F1 is a lazy answer**. It is better than accuracy because it is
relatively less sensitive to imbalanced data, but rarely are precision and
recall equally important (how the customer interacts with the model determines
their relative weighting) and so the harmonic mean is not often applicable.

## Customer Focused

The best candidates consider the problem more deeply. Instead of jumping to
find a metric, they start by thinking about the experience of using a model
from a business or user perspective. A good way to frame this is "What does a
false positive cost my user?" and "How does that compare to the cost of a
false negative?"

Sometimes a **false positive is costly**, as might be the case if the
resulting action is drastic, like shutting down a user's account. To avoid
that, we need to be confident when we take action that we are only targeting
the right users. In such a scenario, [precision][precision] is more important
since we want to make sure that most of the events the model flags are true
positives. This is not the situation for our _'high value customer'_ model,
because showing a user an up-sell page is unlikely to hurt them or cause them
to churn.

[precision]: https://en.wikipedia.org/wiki/Precision_and_recall#Precision

Instead, in our case a **false negative is costly**, because we lose the
chance at a large revenue increase from the up-sell. [Recall][recall] is more
important, as we would rather show a few extra users our up-sell page than
miss the chance to convert a sale.

[recall]: https://en.wikipedia.org/wiki/Precision_and_recall#Recall

There are many other metrics that might be useful. The important part is not
so much the metric itself, but what is motivating it, which should be the
business use case and customer experience.

## A Great Metric: Dollars

A great metric is the formalized version of our customer focused one: dollars.
Assigning a dollar value to each model result (true positive, false positive,
etc.) would allow us to optimize for revenue. This is often doable in simple
models (like our _'high value customer'_ model), but in more complicated ones
(like the fraud models I work on) it can be difficult.[^1] For an example of
using dollars in model optimization, see Airbnb's post on [_Fighting Financial
Fraud with Targeted Friction_][airbnb].

[airbnb]: https://medium.com/airbnb-engineering/fighting-financial-fraud-with-targeted-friction-82d950d8900e

In short, a good metric is one which ties closely to the business or customer
use. Consider their point of view when answering a metrics question.

---

[^1]: One of our largest costs is [reputational risk][rep_risk], which is very hard to assign a number to.

[rep_risk]: https://en.wikipedia.org/wiki/Reputational_risk
