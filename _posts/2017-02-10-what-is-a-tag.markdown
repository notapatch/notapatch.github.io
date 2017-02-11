---
layout: post
title: 'What is a tag?'
date: 2017-02-10 11:11:11 +0000
draft: true
---

{% include helpers/image.html name="tags.jpeg" caption='Ever tried Googling for "street art with the word tag in"? Don\'t bother do it yourself.' %}

### Why am I writing this?

I will be [reading the code]({% post_url 2017-02-09-reading-code %}) of a tag library. One of the first code reading steps is to understand what is involved before getting into the code. So, I am getting a quick overview of tags.

### Definition

Tags are commonly used and easily understood.

*reads the start of wikipedia entry*

> [A tag is non-hierarchical keyword or term assigned to a piece of information.  - Wikipedia][1]


*reads through the entire wikipedia entry slowly moving lips*

*Ahem*,  really? I thought this would be easy. I will look at hierarchical before non-hierarchical.

#### Hierarchical Data

Hierarchical data is something with parents and children - a tree structure. The classic example of hierarchical data structure is storing employee management data. But, we will use a blog hierarchy.


| ID | Parent ID | Name                                 |
|---:|----------:| -------------------------------------|
| 1  |      nil  | All Blogs                            |
| 2  |        1  | Chef                                 |
| 3  |        1  | Ruby                                 |
| 4  |        2  | Ruby tips for Chef                   |
| 5  |        3  | Ruby 2.5 Rumours                     |
| 6  |        3  | Why I stopped using Elixir for Ruby  |
{:.table .u-margin-bottom-large}

Where the parent_id can create the following hierarchy.

{% include helpers/image.html name="hierarchical.png" caption='Figure 2: Blog in Folders' %}

If you are using hierarchical data to classify items you can only put the item in one place. Storing blog posts in a hierarchy means placing our Ruby posts and our Chef/Ruby posts in different folders. It is easier to find Ruby posts if they are kept together.


#### Non-hierarchical Data

Non-hierarchical data means that the data does not have parent and child relationships. You can label anything with any label. Examples of non-hierarchical data are blog post tags and Gmail labels - without a limit on how we label Ruby posts. 


{% include helpers/image.html name="non-hierarchical.png" caption='Figure 3: Blog with Tags' %}

#### Implementation

The common way to implement tags is with three tables. If we wanted to tag posts then one posts table, one tags table and one table for the relationship between them - a posts to tags table. See figure 4. [Wordpress ues the three table system.][2]

{% include helpers/image.html name="three-table.png" caption='Figure 4: Three table tag solution' %}

[There are also two and a single table solution to support tagging.][2]


#### Summary

Tags offer a way to classify data without molding it into a hierarchy. It is a popular solution for marking blogs and emails where labelling shouldn't depend on where it is kept. The most common way of implementing tags is using a three table solutions.

#### References

1. [Wikipedia - Tag (metadata)][1]

2. [Tags: Database schemas][2]

[1]:  https://en.wikipedia.org/wiki/Tag_(metadata)
[2]:  http://tagging.pui.ch/post/37027745720/tags-database-schemas
