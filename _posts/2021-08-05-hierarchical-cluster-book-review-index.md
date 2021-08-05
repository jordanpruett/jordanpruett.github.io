---
title: "A Hierarchical Cluster Analysis of the _Book Review Index_"
date: 2021-08-05
categories:
  - dissertation
---

I recently produced a hierarchical cluster analysis of the _Book Review Index_ data that I've been working with, though unfortunately I didn't have enough time to include it in my ACH presentation last month. That's a shame, since the resulting dendrogram is so interesting. You can view the visualization [here.](/assets/images/08-21/dendrogram_8-5-21.png) It's too gigantic to comfortably embed on this page, but most browsers will let you open the image itself in a new tab.

The diagram itself was produced by first generating a similarity matrix comparing each of the 285 journals to each other journal. The underlying data is simply a collection of binary vectors indicating which journals reviewed each book. As such, I used the Jaccard distance as my similarity metric: the similarity of two journals equals the intersection of the books they both reviewed divided by the union of all books reviewed by either journal.
