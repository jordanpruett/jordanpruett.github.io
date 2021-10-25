---
title: "Dash App for a Genre Classifier"
date: 2021-10-21
categories:
  - academic
---

About two years ago, I designed a simple scikit-learn genre classifier for the Textual Optics Lab. You can read about it at [this blog post](https://textual-optics-lab.blogspot.com/2020/07/machine-learning-for-genre.html). 

The classifier itself was strictly internal; we ran the final version once, then added the metadata to the searchable version of the [US Novel Corpus](https://textual-optics-lab.uchicago.edu/us_novel_corpus). But in testing, I found the results incredibly fascinating, so I wanted to release some interactive way of exploring them. I've had a half-finished Dash app up and running for like a year now, but other projects (read: dissertation) kept preventing me from finishing it.

Rather than let it just sit around unseen, I decided to add some touch-ups and deploy a working version via Heroku. You can play around with it [at this link](https://classifier-viz-app.herokuapp.com/). It's live as of today.

The app itself has four components.

#### Search By Author
This lets you search by author names to explore which authors and titles are contained in the corpus. For each author, it lists the title of each of their books in the corpus, that book's publication date, and the predicted genre of that book. Note that the search function is very bare-bones; you'll need to search by exact match.

#### Genre Confidence
For each title in the corpus, this component shows the confidence score associated with each of that title's genre predictions. A value higher than 0.5 resulted in a positive prediction for that genre, but it can also be interesting to see just how confident the classifier was in the prediction. 

#### Top Words for Each Genre
This component lists the top model features for each genre. These are the words that have the highest and lowest assigned coefficients in the model. Out of the 10,000 words in the prediction vocabulary, they have the highest or lowest prediction weights. Most of them are quite intuitive: the top word for detective and mystery ficiton is, unsurprisingly, "detective." The most amusing are the words that are *negatively* associated with religious fiction: damn, shit, hell, bastard, brandy, ass. Sounds like a boring genre!

#### 2-D Projection of the Feature Space
Lastly, the app lets you explore a 2-D [t-SNE projection](https://en.wikipedia.org/wiki/T-distributed_stochastic_neighbor_embedding) of the ~9,000 novels, colored by genre, in the feature space. The underlying features of the data are tf-idf scores for the top 10,000 most-used words in the whole corpus. Treating each word-count as a dimension, this projection basically shows you which novels are close together in that 10,000-dimensional space. The projection largely agrees with the classifier, as we can see at first glance. But be wary: t-SNE preserves the local (but not global) structure of your data. So only the clusters are meaningful, not the relative distance of several clusters from each other. You can mouse over a point to see which novel it is.

Also, I should note that the whole project is pretty rough around the edges. Now that I'm finally revisiting it, it's kind of remarkable how much I've learned in the meantime. This was my first "real" project with scikit-learn and there are a lot of ways that my inexperience shows. For one, I probably wouldn't use a feature set of 10,000 tf-idf scores if I started over. Fewer features likely would have been plenty and there are alternatives to tf-idf. Additionally, the whole project could be redesigned to be much more flexible and modular. For example, it would be great if you could feed the Dash app the classifier directly and have it automatically generate a report. As it stands, the app is just pulling from saved .tsv files.

But hey, despite the rough edges, I figure it's better to have something actually finished out there in the world.

