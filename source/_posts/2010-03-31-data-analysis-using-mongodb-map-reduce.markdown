---
layout: post
title: Attempts at Analyzing 19 million documents using MongoDB map/reduce
permalink: /2010/03/31/data-analysis-using-mongodb-map-reduce.html
published: true
comments: yes
---

Over the course of the last couple weeks we've been developing a system to help analyze the 19+ million documents in the pubmed database.  In my [previous post](/2010/03/18/large-scale-data-processing-with-mongodb.html) I shared details about the process that we've been using to bring down the ~617 zipped XML documents that contain the articles and import them into MongoDB.  Today I'm going to share a few more details about our attempts at analyzing the pubmed database using the Map/Reduce capabilities MongoDB offers.

After completing the download, unzip, parse, and load steps required to get the pubmed articles into our MongoDB instance we set out to use the map/reduce capabilities in MongoDB to do analysis and aggregation.  Our initial work has focused on the keywords and MESH headings within pubmed articles, as well as on the relationships between authors within pubmed.  Our end goal is to have a profile for every author who has published an article in pubmed with details about what keywords and MESH headings appear most within the articles they publish, as well as who they commonly co-author articles with.

In order to build this profile we set out to write a map/reduce job to count the number of articles written by each author by keyword.  Our job writes the results of the map/reduce job to a named collection.

``` ruby
connection = Connection.new config["mongodb"]["host"]
db = connection.db(config["mongodb"]["db"])
collection = db.collection("articles")

map = ...
reduce = ...

result = collection.map_reduce map, reduce, 
                               :verbose => true, 
                               :out => "keywordstats"
```

The "keywordstats" map/reduce job resulted in over a half million documents being inserted into the keywordstats collection. 

``` ruby
#keywordstatus example document
{ 
  _id: author_name, 
  keywords: { 
    keyword1: 310, 
    keyword2: 21, 
    keyword3: 22 
  }
}
```

The running of the keyword map/reduce analysis took approximately 30 minutes and didn't cause us to think twice about our use of MongoDB map/reduce for our analysis.  Next we moved onto doing analysis on MESH headings.  Since MESH headings are pubmed's official way of categorizing articles there are a lot more articles with MESH headings, and thus a lot more crunching for MongoDB to do.  The map/reduce jobs for the MESH headings were almost exactly the same as those for keywords, however, the processing took much longer due to the larger number of articles with MESH headings assigned.  When all was said and done MongoDB was able to process our map/reduce jobs for MESH headings, however, it took over 15 hours to complete (Note: we didn't do any optimization work so its likely this could be trimmed).

The large increase in time required to analyze the MESH headings made us start to think about what other options we might consider.  However, we pressed onto our final analysis: author/co-author relationships.  Our goal with the author/co-author analysis is to be able to see who authors are co-authoring with most.  Additionally, we want to be able to create a network graph of all the authors within pubmed to do social network analysis on the graph.  In order to create the network we need to be able to figure out who has written with one another so we can create an edge between the relevant author nodes.

Since every article within pubmed has an author, and often multiple authors, we expected this bit of analysis  to be the most taxing on MongoDB.  Pretty soon after kicking off our author/co-author jobs we ran into problems.  Due to the large number of author/co-author relationships and the fact that a single author may co-author papers with many other authors we were unable to get our job to run without running into the memory size limitations of documents within MongoDB.

We evaluated other map/reduce strategies that would reduce the document size, however, the limitations that MongoDB places on the mappers and reducers prevented us from implementing those alternate strategies.  To be more specific, MongoDB requires the mapper and reducer to emit the same structure.  From the map phase we were emitting:

``` ruby
  author, {coauthor1: 1} #emit for each author/co-author "pair"
```

And in our reduce phase we were consolidating all the co-author counts into a single hash to end up with:

``` ruby
{ 
  _id: author_name, 
  value: { 
    coauthor1: 31, 
    coauthor2: 211, 
    coauthor3_: 122
  }
}
```

We found that some authors had so many papers and thus so many coauthors that we were blowing past the size limitations MongoDB places on documents. An alternate strategy that we considered was changing our reduce stage to output a single author coauthor relationship with a count rather then our initial approach which reduced to an author with a hash containing all the coauthors with the counts.  However, since we can only reduce to a single output we would need to change our mapper to emit the author/co-author as the key.  Our initial attempts with this approach weren't working well which prompted us to taken another step back to consider alternate approaches.

Given our needs and the amount of custom analysis we want to do against this (and other largish datasets) we decided to spend some time investigating Hadoop and Amazon Elastic Map Reduce.  Our initial experiences have been very positive, and have us feeling much more confident that the technology choice (Hadoop) won't prevent us from exploring different types of analysis.  

We still feel that Mongo will be a great place to persist the output of all of our Map/Reduce steps, however, we don't feel that it's well suited to the type of analysis that we want to do.  With Hadoop we can scale our processing quite easily, we have tremendous flexibility in what we do in both the map and reduce stages, and most importantly to us we're using a tool that is designed specifically for the problem we're trying to "solve".  Mongo is a nice schema free document database with some map/reduce capabilities, however, what we need for our analysis stage is a complete map/reduce framework.  We'll still be using Mongo, we'll just be using it for what it's good at and Hadoop for the rest.