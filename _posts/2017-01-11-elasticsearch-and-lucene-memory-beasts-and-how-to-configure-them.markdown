---
layout: post
title:  "Elasticsearch and Lucene: memory beasts & how to configure them"
date:   2017-01-09 11:11:11 +0000
categories: databases elasticsearch
---

Whenever you come across a problem you poke and tinker - and  when that doesn't work then you, at last, stop and think. One way out of a hole is explaining your problem by articulating it aloud. So, I was reading and then explaining [how Elasticsearch used memory](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/heap-sizing.html) to the [metal squirrel sitting on the desk](https://en.wikipedia.org/wiki/Rubber_duck_debugging) when it occurred to me neither of us had a clue how it worked. So we will open Elasticsearch ... just a [smidge](http://www.urbandictionary.com/define.php?term=smidge).  

Below, I summarise how Elasticsearch uses memory. It will cover the operating system, Java and Lucene.


## Elasticsearch and the operating system
Physical memory is always in short supply. The operating system gives a new application an untouched block of virgin memory but it’s a big phoney lie - this is the [virtual address space](https://msdn.microsoft.com/en-us/library/aa366912(VS.85).aspx) and the addresses you work with are virtual addresses and you leave it up to the operating system to deal with the real, physical addresses.

Operating systems can take advantages of this in two ways:   

First, by reading in files from disk and keeping them in memory - called memory-mapped files. This we will see is important in making faster queries.  

Second, when an operating system is running out of physical memory it can move parts of that memory that aren’t being used and saving them temporarily on disk, until we need them later - we call this [swapping memory out on Linux and paging memory on Windows](https://en.wikipedia.org/wiki/Paging) - because we, err, want to keep people on their toes.   

#### Table  comparing Memory-mapped Files and Swapping / Paging

| Operation           | [Memory-mapped Files](https://en.wikipedia.org/wiki/Memory-mapped_file) | [Swapping / Paging](https://en.wikipedia.org/wiki/Paging)    |
|:--------------------|:------------------------------------------------------------------------|:-------------------------------------------------------------|
| Physical memory use | Decreases                                                               | Increases                                                    |
| When is it used?    | Any time                                                                | Running out of physical memory                               |
| Elasticsearch Speed | Faster                                                                  | Slower
| Why?                | Queries run faster                                                      | Frees physical memory when the system has run out of options |
{:.table .u-margin-bottom-large}



## Elasticsearch and Java
Elasticsearch wants to go about it’s business with as little interference from the other systems on the computer. [For servers running on Java this means setting the minimum and maximum heap size the same to increase predictability of the sizing of the virtual machine.](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html#sthref22)   

Java has [managed memory](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/garbage_collect.html). As now implemented, Java makes frequent pauses while the ‘garbage’ memory is being removed. Elasticsearch wants to avoid the Garbage collector’s pauses. [When the garbage collector is running, it can delay the response to a request by an order of magnitude](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/garbage_collect.html) and [can cause the cluster waiting for the paused server or even requiring the cluster to re-elect the master](https://www.elastic.co/blog/a-heap-of-trouble).   

[Java 9's concurrent garbage collector could ease these problems but bugs are preventing it being recommended yet.](https://www.elastic.co/blog/a-heap-of-trouble)  

## Elasticsearch and Lucene

 [Apache Lucene](http://lucene.apache.org/core/) is a Java library which Elasticsearch has wrapped.  

Where Lucene can it puts search structures into files, called segments, that can be cached by the operating system and presented, as mentioned earlier, as memory-mapped files. Caching is at the operating system level and outside the Java managed heap.  Elasticsearch writes out structures to files called ’Segments’  - files are [immutable](https://www.elastic.co/guide/en/elasticsearch/guide/current/making-text-searchable.html#_immutability), once written out they will not change again. The immutability means that operating systems can cache these files as memory-mapped files so the next index query can avoids the slower hard drive. These structures are the Inverted Index and Doc Values.  

### Lucene’s Segmented Files

| Segment                                                                                                         | Cached        | Mapping                | 
| ----------------------------------------------------------------------------------------------------------------|---------------|----------------------- |
| [Inverted Index](https://en.wikipedia.org/wiki/Inverted_index)                                                                | Memory-mapped | Words to documents     |
| [Doc Values / uninverted index / Column-store](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/docvalues-intro.html) | Memory-mapped | Documents to values    |
{:.table .u-margin-bottom}

#### Field data / Field cache

Doc Values are [not efficient for aggregating text of analyzed fields](https://www.elastic.co/guide/en/elasticsearch/guide/current/aggregations-and-analysis.html#_analyzed_strings_and_fielddata). In this case you should use heap structures called field data in Elasticsearch and field cache in Lucene. 

Field data structures are generated on the fly and saved into the heap, the default is unbounded, it will never evict data from the cache, and [monitoring the memory usage is advisable](https://www.elastic.co/guide/en/elasticsearch/guide/current/_limiting_memory_usage.html) - as it can make a substantial amount of Elasticsearch’s heap. [Field data size can be limited](Whenever you come across a problem you ‘try a few things’ when they don’t work then you have to stop and think. One way out of a hole is explaining your problem. So, I was reading and then explaining [how Elasticsearch used memory](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/heap-sizing.html) to the [metal squirrel sitting on the desk](https://en.wikipedia.org/wiki/Rubber_duck_debugging) when it occurred to me neither of us had a clue how it worked. So we will open Elasticsearch ... just a [smidge](http://www.urbandictionary.com/define.php?term=smidge).  

 Below,  I summarise how Elasticsearch uses memory. It will cover the operating system, Java and Lucene. 

## Elasticsearch and the operating system
Physical memory is always in short supply. The operating system gives a new application an untouched block of virgin memory but it’s a big phoney lie - this is the [virtual address space](https://msdn.microsoft.com/en-us/library/aa366912(VS.85).aspx) and the addresses you work with are virtual addresses and you leave it up to the operating system to deal with the real, physical addresses.

Operating systems can take advantages of this in two ways:   

First, by reading in files from disk and keeping them in memory - called memory-mapped files. This we will see is important in making faster queries.  

Second, when an operating system is running out of physical memory it can move parts of that memory that aren’t being used and saving them temporarily on disk, until we need them later - we call this [swapping memory out on Linux and paging memory on Windows](https://en.wikipedia.org/wiki/Paging) - because we, err, want to keep people on their toes.   

#### Table  comparing Memory-mapped Files and Swapping / Paging

| Operation           | [Memory-mapped Files](https://en.wikipedia.org/wiki/Memory-mapped_file) | [Swapping / Paging](https://en.wikipedia.org/wiki/Paging)    |
|:--------------------|:------------------------------------------------------------------------|:-------------------------------------------------------------|
| Physical memory use | Decreases                                                               | Increases                                                    |
| When is it used?    | Any time                                                                | Running out of physical memory                               |
| Elasticsearch Speed | Faster                                                                  | Slower
| Why?                | Queries run faster                                                      | Frees physical memory when the system has run out of options |
{:.table .u-margin-bottom-large}



## Elasticsearch and Java
Elasticsearch wants to go about it’s business with as little interference from the other systems on the computer. [For servers running on Java this means setting the minimum and maximum heap size the same to increase predictability of the sizing of the virtual machine.](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html#sthref22)   

Java has [managed memory](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/garbage_collect.html). As now implemented, Java makes frequent pauses while the ‘garbage’ memory is being removed. Elasticsearch wants to avoid the Garbage collector’s pauses. [When the garbage collector is running, it can delay the response to a request by an order of magnitude](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/garbage_collect.html) and [can cause the cluster waiting for the paused server or even requiring the cluster to re-elect the master](https://www.elastic.co/blog/a-heap-of-trouble).   

[Java 9's concurrent garbage collector could ease these problems but bugs are preventing it being recommended yet.](https://www.elastic.co/blog/a-heap-of-trouble)  

## Elasticsearch and Lucene

 [Apache Lucene](http://lucene.apache.org/core/) is a Java library which Elasticsearch has wrapped.  

Lucene puts search structures into files, called segments, files are [immutable](https://www.elastic.co/guide/en/elasticsearch/guide/current/making-text-searchable.html#_immutability), once written out they will not change again. The immutability means that operating systems can cache these files as memory-mapped files so the next index query can avoids the slower hard drive. Caching is at the operating system level and outside the Java managed heap. These structures are the Inverted Index and Doc Values.  

[Inverted Indexes](https://www.elastic.co/guide/en/elasticsearch/guide/current/making-text-searchable.html#making-text-searchable) answer  “what documents has this query text”. While [Doc Values](https://www.elastic.co/guide/en/elasticsearch/guide/current/docvalues.html) answer “what values are in this document” - since this is the opposite of Inverted - Doc Values are also called uninverted indexes.

### Lucene’s Segmented Files

| Segment                                                                                                         | Cached        | Mapping                | 
| ----------------------------------------------------------------------------------------------------------------|---------------|----------------------- |
| [Inverted Index](https://en.wikipedia.org/wiki/Inverted_index)                                                                | Memory-mapped | Words to documents     |
| [Doc Values / uninverted index / Column-store](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/docvalues-intro.html) | Memory-mapped | Documents to values    |
{:.table .u-margin-bottom}

#### Field data / Field cache

Doc Values are [not efficient for aggregating text of analyzed fields](https://www.elastic.co/guide/en/elasticsearch/guide/current/aggregations-and-analysis.html#_analyzed_strings_and_fielddata). In this case you should use heap structures called field data in Elasticsearch and field cache in Lucene. 

Field data structures are generated on the fly and saved into the heap, the default is not to bounded and [monitoring the memory usage is advisable](https://www.elastic.co/guide/en/elasticsearch/guide/current/_limiting_memory_usage.html) - as it can make a substantial amount of Elasticsearch’s heap.

#### Norms

Norms are 

