---
title: "Authorless Topic Models"
date: 2021-08-12
categories:
  - dissertation
---

For the first chapter of my dissertation (and, hopefully soon, a published article) I've been working with a topic model trained on a corpus of 2,348 _New York Times_ bestsellers held by HathiTrust, matched against Ted Underwood's [NovelTM dataset.](http://view.data.post45.org/index) Lots of people use MALLET for topic modeling, but I've been using the [scikit-learn implementation of Latent Dirichlet Allocation](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.LatentDirichletAllocation.html), since I'm familiar with the API. Early on, I noticed a bit of a problem with my modeled topics: a few of them seemed to be almost entirely dominated by just a single author. For example, consider the following topic derived from an early version of the experiment (this version was based on only nouns). 

```
Topic 68: children kids dinner mother week months work house school bed 
```

At first glance, the topic seems perfectly servicable; these are all words that one is likely to find in novels about marriage, family, and domestic life. The problem is that almost all of the top novels by topic proportion for this topic were written by Danielle Steel. Strictly speaking, this is a correct result; the model is simply picking up on the fact Steel has an identifiable style, which manifests in words that tend to appear together in the novels she wrote. But it's trivial. I already know that they are many novels by Danielle Steel in this corpus. The model isn't telling me anything new, which is not ideal for an unsupervised model used for exploratory analysis. Worse still, it trivializes any directional or historical claims you might make with the model. If Topic 68 is becoming more prevalent closer to the end of the century, for example, this may simply be due to the fact that Danielle Steel is hitting the list more frequently.

This is a known limitation of topic modeling and researchers have experimented with different ways of reducing its impact. Mathew Jockers, for instance, expands his stoplist to include all personal names. Since character names in fiction tend to be specific to a single book or series - "Harry Potter" only appears in _Harry Potter_ - they end up being so unevenly distributed that they can break topic models trained on fiction. It's a nice solution, but a little simplistic. What if I don't know in advance the particular linguistic quirks that are causing this behavior? 

Luckily, Laure Thompson and David Mimno have a [paper](http://www.cs.cornell.edu/~laurejt/papers/authorless-tms-2018.pdf) that provides a more elegant solution, which they call "authorless topic models." I encourage you to read the paper if you're interested, but the gist is that by subsampling words that are highly-correlated with known source metadata - in this case, the author of a given document - you can reduce the prevalence of source-specific topics.

The Github repo for the project is available [here](https://github.com/laurejt/authorless-tms). A side note: their downsampling script requires MALLET-formatted documents, but unfortunately I'm working with HathiTrust extracted features, which come in the form of word-counts rather than raw text. So I had to "expand" each dictionary of word-counts into one long string of space-separated tokens. Like many text-preprocessing tasks, this is thankfully easy to do in Python:

```python
expanded_tokens = []
for token, count in token_counts.items():
        expanded_tokens.extend([token] * count)
mallet_text = ' '.join(expanded_tokens)
```

Fitting my topic model on a downsampled version of the corpus has produced much more informative topics that are less likely to be totally dominated by one author. They also seem to generally be more "thematic" than the topics uncovered by my previous model. Consider the following topic:

```
Topic 5: car street road driver front drove drive cars truck seat window 
```

This is clearly a topic derived from scenes set in cars or on the road. But best of all, the texts that exhibit the greatest topic proportion for this topic come from a variety of authors: Stephen King's _Christine_ (1983), about an apparently-possessed 1958 Plymouth Fury; or Clive Cussler's _The Chase_ (2007), which is notable because the villain escapes on a 1905 Harley.

Even more impressive are the topics that feel bound by style rather than purely theme but still manage to cut across source authors. This topic, for example, seems to be about investigation or fact-finding:

```
Topic 36: might most also fact found knew being since course few case possible information
```

The words in this set seem to reflect the style in which scenes of investigation are narrated: words reflecting uncertainty ("might," "possible"), the process of deduction ("since"), or the specification of facts ("fact," "most," "few"). My first thought was that it's a topic common to police procedurals, but in fact the top novels by topic proportion are mostly medical thrillers. It includes books like Michael Crichton's _The Andromeda Strain_, Arthur Hailey's _Strong Medicine_, and Robin Cook's _Contagion_. This is much more useful information about the corpus than what I had received previously. I hadn't realized just how prevalent the seemingly-specific genre of medical thriller was in the corpus, nor did I know that this kind of language was its characteristic feature.

But most importantly, this extra pre-processing step gave me much more meaningful results when I plotted the rise and fall of different topics over the historical window covered by the corpus. I'll talk about those results in a future post.
