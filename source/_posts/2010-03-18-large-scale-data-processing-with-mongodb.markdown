---
layout: post
title: Large Scale Data Processing with MongoDB Map/Reduce (Part 1:Background)
permalink: /2010/03/18/large-scale-data-processing-with-mongodb.html
comments: yes
---

Over the course of the last week I've been working with a member of our team to develop a prototype data processing "engine" for analyzing articles within the [pubmed database](http://www.ncbi.nlm.nih.gov/pubmed).  The pubmed database consists of approximately 19 million articles that can be downloaded as approximately 617 zipped XML documents.

Our initial work has focused on downloading the complete dataset, pulling out the bits that we have interest in, and importing them into MongoDB.  For our initial analysis we're focusing on a subset of the details available for each article.  In the future we'll likely expand our analysis to include more details.

We started by downloading the 617 zipped XML documents from pubmed.  Once downloaded we unzipped each file, parsed out the bits that we're interested in and saved the details in a JSON file optimized for importing into MongoDB. [1]

Once all the XML files were processed and the details were saved out to a JSON file, we used the mongoimport utility to import the JSON files into MongoDB.

The above process was run over the course of a couple days.  The most time consuming part was the parsing of the XML files.  We wrote [Resque](http://github.com/defunkt/resque) workers to handle the above  so that the work could be distributed to multiple nodes running on EC2, however, I ended up running things locally so that I could test the process.  Given the pubmed database doesn't change that often, and that we'll rarely need to re-process the entire dataset having it run on a single machine over the course of a couple days will likely suffice.  

After importing all the articles into MongoDB we had a pretty large MongoDB database consisting of ~18 million "documents".  With the articles loaded into MongoDB, we moved onto the next step...[analyzing all 18 million documents](/2010/03/31/data-analysis-using-mongodb-map-reduce.html).

---
[1] MongoDB likes a single JSON record on each line.
[2] This is my first blog post in ages, I need to get back into it slowly, oh so very slowly! :-)