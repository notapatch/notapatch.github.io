---
layout: post
title:  "Elasticsearch and Lucene: memory beasts & how to configure them"
date:   2017-01-09 11:11:11 +0000
categories: databases elasticsearch
---

Whenever you come across a problem you ‘try a few things’ when they don’t work then you have to stop and think. One way out of a hole is explaining your problem. So, I was reading and then explaining [how Elasticsearch used memory](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/heap-sizing.html) to the [metal squirrel sitting on the desk](https://en.wikipedia.org/wiki/Rubber_duck_debugging) when it occurred to me neither of us had a clue how it worked. So we will open Elasticsearch ... just a [smudge](http://www.urbandictionary.com/define.php?term=smidge).  

This is an overview and will layout a framework of the memory usage. Further detail will require more study.  

## Elasticsearch and the Penguin
Physical memory is always in short supply. The operating system gives a new application an untouched block of virgin memory but it’s a big phoney lie - this is the [virtual address space](https://msdn.microsoft.com/en-us/library/aa366912(VS.85).aspx) and the addresses you work with are virtual addresses and you leave it up to the operating system to deal with the, real, physical addresses.

Operating systems can take advantages of this in two ways:  
First, by reading in files from disk we can keep them in memory if they are needed again - called memory mapped files. This we will see is important in making Elasticsearch indexes quickly searchable.  
Second, when an operating system is running out of physical memory it can move parts of that memory that aren’t being used and saving them temporarily on disk, until we need them later - we call this [swapping memory out on Linux and paging memory on Windows](https://en.wikipedia.org/wiki/Paging) - because we, err, want to keep people on their toes.   

#### Table  comparing Memory Mapped Files and Swapping / Paging

| Operation                                                               | Physical memory use  | When is it used?               |
|:----------------------------------------------------------------------- |:---------------------|:-------------------------------|
| [Memory Mapped Files](https://en.wikipedia.org/wiki/Memory-mapped_file) | Increase             | Any time                       |
| [Swapping / Paging](https://en.wikipedia.org/wiki/Paging)               | Decreases            | Running out of physical memory |
{:.table .u-margin-bottom}



## Elasticsearch and the Coffee Cup
Elasticsearch wants to go about it’s business with as little interference from the other systems on the computer. [For servers running on Java this means setting the minimum and maximum heap size the same to increase predictability of the sizing of the virtual machine.](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/sizing.html#sthref22)   

Java has [managed memory](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/garbage_collect.html). As it is currently implemented, Java will pause frequently while the ‘garbage’ memory is being removed. Elasticsearch wants to also avoid the Garbage collector’s pauses. [When the garbage collector is running it can delay the response to a request by an order of magnitude](https://docs.oracle.com/cd/E13150_01/jrockit_jvm/jrockit/geninfo/diagnos/garbage_collect.html) and [can result in the cluster waiting for the paused server or even, in certain cases, requiring the cluster requiring a new election](https://www.elastic.co/blog/a-heap-of-trouble).   

[A lot of these problems could be alleviated with a concurrent garbage collector but it is not recommended yet.](https://www.elastic.co/blog/a-heap-of-trouble)  

## Elasticsearch and Lucene

 [Apache Lucene](http://lucene.apache.org/core/) is a Java library which Elasticsearch has wrapped.  

Lucene uses up heap space with field data. ??? more 

Where Lucene can it puts search structures into files, called segments, that can be cached by the operating system and presented, as mentioned earlier, as memory mapped files. Caching is at the operating system level and outside the Java managed heap.  Elasticsearch writes out structures to files called ’Segments’  - files are [immutable](https://www.elastic.co/guide/en/elasticsearch/guide/current/making-text-searchable.html#_immutability), once they are written out they will not change again. The immutability means that operating systems can cache these files as memory mapped files so that when the next time the index is queried, there is no need to go back to the glacially slower hard drive. These structures are the Inverted Index and Doc Values.  

### Lucene’s Segmented Files

| Segment                                                                                                         | Cached        | Mapping                | 
| ----------------------------------------------------------------------------------------------------------------|---------------|----------------------- |
| [Inverted Index](https://en.wikipedia.org/wiki/Inverted_index)                                                                | Memory Mapped | Words to documents     |
| [Doc Values / uninverted index / Column- Store](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/docvalues-intro.html) | Memory Mapped | Documents to values    |
{:.table}

