---
layout: post
title:  "Elasticsearch and Lucene: memory beasts & how to configure them"
date:   2017-01-09 11:11:11 +0000
categories: devops chef
---

Elasticsearch is an open source search engine built on top of [Apache Lucene](http://lucene.apache.org/core/). Together Elasticsearch and Lucene are fantastic at devouring memory and it is a very good idea to know how to control this. It is also not an obvious topic to pick up as it skirts between knowledge of the search engine and how Linux manages memory.  

I read the [Elasticsearch article on heap sizing](https://www.elastic.co/guide/en/elasticsearch/guide/current/heap-sizing.html) but I didn’t understand it, if you had that experience this post is for you.  

###  What is virtual memory?
When a process starts it is presented with a virgin block of memory to use at it wishes. This is, of course, a lie. The operating system is juggling a number of processes at the same time and they all think that they’ve got their own virgin block. However, if they all asked for the space at once the system would go into meltdown. [This is known as an overcommitted system](https://www.etalabs.net/overcommit.html). To square the larger virtual memory to the smaller physical memory operating systems have an intermediate responsible for diverting the request to  
Linux supports virtual memory, by mixing RAM and hard disk space it allows a larger available ‘memory’ than the machine restricted itself to RAM alone.

````
Process 1’s               Page Table           Physical Memory
Virtual Memory  
+------+ ------------> +----------+                +----------+
|  n   |               |     n    |                |    n     |
+------+               +----------+                +----------+
| ...  |               |   ...    |                |    2000  |
+------+               +----------+                +----------+
|   0  |               |   1000   |                |    1999  |
+------+ ------------->+----------+                +----------+
                       |   ...    |                |    1998  |
 Process 2’s           +----------+                +----------+
Virtual Memory         |   ...    |                |    1997  |
+------+ ------------> +----------+                +----------+
|  n   |               |    500   |                 Hard Disk
+------+               +----------+                +----------+
|  ... |               |   ...    |                |  50,000  |
+------+               +----------+                +----------+
|   0  |               |    300   |                |   49,999  |
+------+ ------------> +----------+                +----------+
                       |   ...    |                |  49,998  |
                       +----------+                +----------+
                       |      0   |                |   ...    |
                       +----------+                +----------+
````

#### Reference
[Difference between physical/logical/virtual memory address](http://stackoverflow.com/a/15851473/1511504)  
[Linux Memory Mangement](http://www.thegeekstuff.com/2012/02/linux-memory-management)  
[Virtual memory - Wikipedia](https://en.wikipedia.org/wiki/Virtual_memory)  
[What is virtual memory?](http://www.tldp.org/LDP/sag/html/vm-intro.html)  

Introduction to Lucene
Lucene is a full-text search library in Java. It creates search indexes from supplied source data; the equivalent of storing the index at the back of the book.  

#### Reference
[Lucene Tutorial basic concepts](http://www.lucenetutorial.com/basic-concepts.html)  

### Swap space
Swap space is the part of virtual memory that lives on the hard disk. Virtual memory is only an issue when you are running out of RAM space, when RAM is not an issue neither is swapping.
#### Reference
[SwapFaq](https://help.ubuntu.com/community/SwapFaq)  
[Linux Swap Space](http://www.linuxjournal.com/article/10678)  


### How much you swap depends on what you are doing.
The size of memory you are swapping depends on how much RAM you have and what applications you are running. [Databases can be functioning normally with a RAM usage of 95%](https://blog.engineyard.com/2013/database-memory).
http://unix.stackexchange.com/questions/88693/why-is-swappiness-set-to-60-by-default  

### Elasticsearch’s Best Practice

[Reading Elastic search’s best practices](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/heap-sizing.html) they prefer to lock the system down and ban Linux from doing any swapping as you could be swapping a page over when a query comes in, delaying that query. Their preference is:

````
| Preference | Configuration            | Description                               |
| ---------- | ------------------------ | ----------------------------------------- |
| 1          | sudo swap off -a         | Disable swapping on the entire system     |
| 2          | vm.swappiness = 1        | Swap only in an emergency memory shortage |
| 3          | bootstrap.mlockall: true | The process running    
 
````

`sudo swapoff -a`

If that cannot occur then the next option is

`vm.swappiness = 1`
* depending on versions of the kernel

This means that swapping can only occur in the worst memory constraints.

Finally, the last approach is to lock the memory on th


