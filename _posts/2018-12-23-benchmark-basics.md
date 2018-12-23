---
layout: post
title:  "Benchmark Basics"
date:   2018-12-23 20:00
tags: definitions ml master
---
Now that we have this fancy lab, a benchmark is needed for qualitative evaluation of different algorithms and products.  
For this, I've built a Python 3-based framework for ease of use.

<!--more-->

The idea is to build a precise but still modular and flexible benchmark that can work with a variety of inputs.  
As Splunk is already running [in the lab]({{ site.baseurl }}{% post_url 2018-06-12-lab-overview%}), I wanted to incorporate it as a data source besides well-known test datasets.  
The main entry point to the application is a `run.py`, that can be configured using command line switches (thanks to Python argpase!).  
It can roughly be divided into three parts: Dataset preparation, Benchmark runs and tools.  
All the introduced bits and pieces can be swapped out and extended with minimal effort, as the system is loosely coupled. The only hard defined thing are the interchange formats between the moving pieces. But enough chit chat, let's have a look at the workflow:

<div class="mermaid">
graph LR
    A["Splunk"] -->|JSON REST API| C["Data Ingest"]
    B["Datasets"]-->|CSV/Pickle| C
    C --> D["Normalization <br/> & Cleanup"]
    D --> E["Testrunner"]
    E --> F["Metrics"]
</div>

## Data Ingest / Dataset preparation
This is the main point of entrance for data into the benchmark. For performance reasons, it has been designed mainly around big CSV files (which seem to be the main distribution channel of academic datasets), because this enables the use of heavy caching and intermediate serialization, thus lowering the average benchmark run sugnificantly.  
The CSV files are loaded, joined together in a Pandas DataFrame and then serialized to disk as a Python Pickle. This Pickle is then loaded, split into training and test partitions, normalized and strings are replaced.  
For the Splunk API, at the time writing I am able to pull arbitrary data out through search queries and organize them in a DataFrame. From here on, they are handled identically to the datasets.  

## Normalization & Cleanup
There is a very important step to ensure your base data leads to meaningful results and that is cleanup!  
Let's take Network traffic as an example. There are features like the TCP flags, which are quite small. Or the number of sent and received packets, which can grow quite a lot. Or even take port numbers, which can get really big. All these could be considered different axis, or scales. Machine Learning algorithms tend to get a problem with badly scaled features, so it might be a good idea to normalize and scale every feature, so that they lie on unifed scales.  
But wait! There's room for error! You could get the idea to simply scale all data at once, but that actually leads to information bleed between your training and test partitions. You really don't want this, so make sure to treat these two partitions separately.

## Testrunner
The testrunners can be considered the real guts of the benchmark. They are responsible for the heavy lifting and trigger the result serialization and metrics generation. Currently, there are two runners implemented: A k-Fold Crossvaltion (CV)[[Kohavi1995](ai.stanford.edu/~ronnyk/accEst.pdf)] runner and a single benchmark runner. While the former performs a customizable Crossvalidation on the training partition, the latter one trains on the full training partition and tests against the test partition.  
The CV runner is intended to be used for model selection or parameter optimization, whilst the single benchmark should be used to run these optimized models for qualitative comparsion.

## Metrics
For every run, every single prediction is saved, as well as a plethora of metrics: Training Time, [ROC & AUC](https://en.wikipedia.org/wiki/Receiver_operating_characteristic), [Accuracy](https://en.wikipedia.org/wiki/Accuracy_and_precision#In_binary_classification), [Precision & Recall](https://en.wikipedia.org/wiki/Precision_and_recall), and the [F1-Score](https://en.wikipedia.org/wiki/F1_score). For every metric, the mean, variance and standard deviation are calculated, if it is a CV run.