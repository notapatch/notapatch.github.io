---
layout: post
title:  "Elasticsearch and Lucene: memory beasts & how to configure them"
date:   2017-01-09 11:11:11 +0000
categories: databases Elasticsearch
draft: true
---

Whenever you come across a problem you poke and tinker - and  when that doesn't work then you, at last, stop and think. One way out of a hole is explaining your problem by articulating it aloud. So, I was reading and then explaining [how Elasticsearch used memory](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/heap-sizing.html) to the [metal squirrel sitting on the desk](https://en.wikipedia.org/wiki/Rubber_duck_debugging) when it occurred to me neither of us had a clue how it worked. So we will open Elasticsearch ... just a [smidge](http://www.urbandictionary.com/define.php?term=smidge).  

Below, I summarise how Elasticsearch uses memory. It will cover the operating system, Java and Lucene. You will need to understand [virtual memory](https://en.wikipedia.org/wiki/Virtual_memory).


## Elasticsearch and the operating system
Operating systems manage a partially filled and limited space - the physical addresses. While it lies to applications that they have an empty block of memory - the [virtual address space](https://msdn.microsoft.com/en-us/library/aa366912(VS.85).aspx). 

Operating systems can take advantages of virtual memory in two ways:   

First, by reading in files from disk and keeping them in memory - called memory-mapped files. We will see is important in making faster queries when Lucene uses segmented files.  

Second, when an operating system is running out of physical memory it can move parts of that memory that aren’t being used and saving them temporarily on disk - we call this [swapping memory out on Linux and paging memory on Windows](https://en.wikipedia.org/wiki/Paging). Operating systems cannot swap and service Elasticsearch's queries leading to slower Elasticsearch responses. Elasticsearch wants to be configured to avoid any swapping.

#### Table  comparing Memory-mapped Files and Swapping / Paging

| Operation           | [Memory-mapped Files](https://en.wikipedia.org/wiki/Memory-mapped_file) | [Swapping / Paging](https://en.wikipedia.org/wiki/Paging)    |
|:--------------------|:------------------------------------------------------------------------|:-------------------------------------------------------------|
| Physical memory use | Decreases                                                               | Increases                                                    |
| When is it used?    | Any time                                                                | Running out of physical memory                               |
| Elasticsearch Speed | Faster                                                                  | Slower
| Why?                | Queries run faster                                                      | Frees physical memory when the system has run out of options |
{:.table .u-margin-bottom-large}



## Elasticsearch and Java
Elasticsearch wants to work with as little interference from the other systems on the computer. [For servers running on Java this means setting the minimum and maximum heap size the same to increase predictability of the sizing of the virtual machine.](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html#sthref22)   

Java has [managed memory, it garbage collects heap allocations.](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/garbage_collect.html). Java 8.0 frequently pauses while garbage collecting [slowing Elasticsearch's response by ten times](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/garbage_collect.html) which [can cause the entire cluster to wait and even re-elect a paused master](https://www.elastic.co/blog/a-heap-of-trouble).   

[Java 9's concurrent garbage collector, once working with Elasticsearch, could avoid the pauses.](https://www.elastic.co/blog/a-heap-of-trouble)  

## Elasticsearch and Lucene

Elasticsearch uses [Apache Lucene](http://lucene.apache.org/core/), a Java library, for text search.  

Lucene puts search structures into [immutable](https://www.elastic.co/guide/en/elasticsearch/guide/current/making-text-searchable.html#_immutability) files, called segments. Since the files won't change an operating systems can cache them as [memory-mapped files](https://en.wikipedia.org/wiki/Memory-mapped_file) so the next index query can complete in memory without reading from the slower hard drive. Memory mapped caching is at the operating system level and outside the Java managed heap. 

Cached segment structures are the Inverted Index and Doc Values. [Inverted Indexes](https://www.elastic.co/guide/en/elasticsearch/guide/current/making-text-searchable.html#making-text-searchable) answer  “what documents has this query text”. While [Doc Values](https://www.elastic.co/guide/en/elasticsearch/guide/current/docvalues.html) answer “what values are in this document” - since this is the opposite of Inverted - Doc Values are also called uninverted indexes.

### Lucene’s Segmented Files

| Segment                                                                                                         | Cached        | Mapping                | 
| ----------------------------------------------------------------------------------------------------------------|---------------|----------------------- |
| [Inverted Index](https://en.wikipedia.org/wiki/Inverted_index)                                                                | Memory-mapped | Words to documents     |
| [Doc Values / uninverted index / Column-store](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/docvalues-intro.html) | Memory-mapped | Documents to values    |
{:.table .u-margin-bottom}

#### Field data / Field cache

Doc Values are [not efficient for aggregating text of analyzed fields](https://www.elastic.co/guide/en/elasticsearch/guide/current/aggregations-and-analysis.html#_analyzed_strings_and_fielddata) and you should use heap structures called field data or field cache in Elasticsearch and Lucene respectively. 

Elasticsearch generates field data structures on the fly to be saved into the heap, the default usage is unbounded and [monitoring is advisable](https://www.elastic.co/guide/en/elasticsearch/guide/current/_limiting_memory_usage.html) - as it can make a substantial amount of Elasticsearch’s heap.

#### Norms

Norms are numbers that weight the importance of a match on that field. Norms are one byte each and are caculated per document, per field which is seen as an increase in heap use. 

