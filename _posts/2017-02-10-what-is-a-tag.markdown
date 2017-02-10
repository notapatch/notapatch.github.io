---
layout: post
title: 'What is a tag?'
date: 2017-02-10 11:11:11 +0000
draft: false
---

{% include helpers/image.html name="tags.jpeg" caption='Ever tried Googling for "street art with the word tag in"? Don\'t bother and do it yourself.' %}

### What are tags?

I intend to code read a tag library and wanted to get an idea about how to implement tagging.


### Definition

Tags are commonly used and easily understandable.

*reads the start of wikipedia entry*

> [A tag is non-hierarchical keyword or term assigned to a piece of information.  - Wikipedia][1]


*reads through entire wikipedia entry slowly*

Tags are commonly used and can be understood by people who are not boolean impared.

Start again. Lets define hierarchical.

#### Hierarchical

Hierarchical is something with parents and children - a tree like structure. Storing things in hierarchies is what we do in computing when we put things into a folder and generally we store things in one folder. If we stored blog posts into this folder hierarchy we could see that our Chef/Ruby posts and our Ruby posts would not be in the same folder. Examples of hierarchical structure is keeping files on your computer system.

{% include helpers/image.html name="hierarchical.png" caption='Figure 1: Blog in Folders' %}

#### Non-hierarchical

Non-hierarchical means that the data does not have parent and child relationships. You can label anything with any label. Examples of non-hierarchical labels are blog post tags and Gmail labels.


{% include helpers/image.html name="non-hierarchical.png" caption='Figure 2: Blog with Tags' %}

Implementation

The common way to implement tags is a many to many relationship. Where the model you are tagging, say Posts, has relationships to a tags and a posts to tags table.



{% include helpers/image.html name="three-table.png" caption='Figure 3: 3 table tag solution' %}



[1]:  https://en.wikipedia.org/wiki/Tag_(metadata)
