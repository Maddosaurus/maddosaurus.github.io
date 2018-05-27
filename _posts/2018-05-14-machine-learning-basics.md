---
layout: post
title:  "Machine Learning Basics"
date:   2018-05-14 22:29 +0100
tags: definitions ml master
---
Until now, we have successfully defined what an Intrusion Detection System is and how it can be categorized.  
Now I have to say, I am a really lazy person, as many Computer Science people are.  
I like to leave the heavy lifting and processing to machines that are really good and efficient at this task.  
That's where Machine Learning comes into play.  

### Traditional Programming vs Machine Learning
[[Chollet2018](https://www.manning.com/books/deep-learning-with-python)][^1] defines classical programming as something that takes rules and data and transforms this into answers.  
In contrast, machine learning takes answers and data and transforms this into rules.  
Let that sink for a moment.  
Machine Learning takes answers (the desired state or category, i.e. attack category) and data (annotated training data) and produces rules.  
These rules can be applied to previously unseen data to categorise it.  

### Supervised and Unsupervised Learning
At this point you probably encountered various kinds of categories to sort the plethora of different ML algorithms into.  
One of the ways to sort your algorithms could be the distinction of supervised and unsupervised algos.  

Supervised algorithms need fully labeled training datasets to function correctly [[Hodo et al. 2017](https://arxiv.org/abs/1701.02145)].  
What does that mean?  
You always need traning data to kickstart your algorithms. In the case of supervised learning algorithms, this data has to be labeled.  
So every entry in the dataset has to have a label that identifies it as one of the target categories (or... "answers") [[Hodo et al. 2017](https://arxiv.org/abs/1701.02145)].  

Unsupervised algorithms do not need these labels, they are fine when presented with unstructured data.  
While this might sound great at first, this comes at the cost of worse performance as well as potentially worse precision.  

To conquer these problems, a kind-of-third category has been introduced: semi-supervised learning[[Zhou et al. 2014](https://doi.org/10.1016/B978-0-12-396502-8.00022-X)].  
In semi-supervised learning, the algorithm at hand is usually presented with a small ammount of labled data combined with large ammounts of unlabeled data.  
This serves in kickstarting it with precise data while at the same time saving the authors the tedious task of annotating large quantities of data.

### Shallow and Deep Learning
Finally, there it is: The buzzword named "Deep Learning".  
While it has been in the media in recent times, bullshit-free definitions are kind of hard to come by.  
So let's try and change that:  
Deep Learning is a different way of building a machine learning application. While traditional (shallow) takes usually consist of a single processing layer, deep learning differs.  
Deep learning techniques usually consist of networks of interconnected layers, of which at least one layer can be defined as "hidden layer" [[Cylance 2017](https://www.amazon.de/Introduction-Artificial-Intelligence-Security-Professionals-ebook/dp/B07654CFFQ)].[^2]  
A hidden layers output is only used as input for another layer, therefore the name.  

There are many more ways of categorizing and grouping the vast number of algorithms out there, but for now, this should give you a rough idea of the general terminology.

-------

[^1]: Highly recommended read! - at least for building Deep Learning based on Keras / Python
[^2]: This book is an interesting read as well!