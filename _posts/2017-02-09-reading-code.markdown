---
layout: post
title: "Reading Code"
date: 2017-02-09 14:02:00 +0000
categories: learning
draft: true
---


To be great at anything, you run the extra mile, bench press the extra lb, and yes run the stairs before punching the air. So, as you crush your unrecyclable coffee cup, ignoring the fresh coffee stains on your sweatshirt and the environmental cost, you stare wildly across the bay know there's nothing you wouldn't do.

So, how to become a great programmer? Read other programmers code. "That's easy" you rise on the caffeine surge and ready the next step.  What should I read? This is where it gets tricky. [With the notable exceptions of Knuth and Fitzpatrick most could recommend nothing](http://www.gigamonkeys.com/code-reading/).

{% include helpers/image.html name="donald-e-balboa.png" caption="No there isn't Donald. No there isn't." %}

### Why am I writing this?
I am reviewing the available information and writing for a beginner reading open source. The best overview pieces are [How to quickly and effectively read other people’s code](https://selftaughtcoders.com/how-to-quickly-and-effectively-read-other-peoples-code/) and [One sure fire way to improve your editing](https://changelog.com/posts/one-sure-fire-way-to-improve-your-coding). This is a summary of all the content I've reviwed.

### What to choose?

[The general advice is to choose that motivates you to understand it][3]:

1. Code you rely on
  - You will get a long term benefit 
2. Code that impresses you
3. Code by people you like
4. Choose something you understand

### Read code actively

The general advice is to have an 'active reading style' - take notes, explain problems, and treat it like a puzzle which will need visual representations of the problem with charts, tables and diagrams.

1. Work through the `README.md`
  - Look at the examples
  - Keep and work through a list if there is over two items of interest
2. Run the tests
3. Choose a feature you are interested in
  - If lost [start at the end and trace the action backward][2]
4. Test you understanding by changing the code and running the tests

  - Accept that any code other than something you have just written is 'crap code'

### Watching this in action

Videos of this in action brought this together for me:

[James Edward Grey II](https://twitter.com/JEG2) produced two videos on code reading called Codalyzed. He chose small libraries, [Mote](https://github.com/soveran/mote) and [Micromachine](https://github.com/soveran/micromachine), and followed the 'active reading style'. I recommend the videos ... but you cannot buy them. [There are unedited code readings on dotenv on youtube](https://www.youtube.com/watch?v=lKmY_0uY86s) but the paid videos are better.

 
### My Next Step

It's surprising code reading isn't a popular. Am I missing something? While disappointing, it will not stop me grabbing a library I can ‘grok’ and working through it in an active style. 


### References

I have found little on code-reading - are there more references?


[1. Code is not literature](http://www.gigamonkeys.com/code-reading/)

[2. How to quickly and effectively read other people’s code](https://selftaughtcoders.com/how-to-quickly-and-effectively-read-other-peoples-code/)

[3. One sure fire way to improve your editing](https://changelog.com/posts/one-sure-fire-way-to-improve-your-coding)

[4. LittleBIGRuby by James Edward Gray II](https://www.youtube.com/watch?v=_xz7gPHKGxs)

[5. Good grief - while googling Knuth - I find xkcd were first - I will have to tell myself I'm a special comedy snowflake](https://xkcd.com/163/)

[6. 5 tips to quickly understand a new code base - FunFunFunction](https://www.youtube.com/watch?v=OnCeaJdd_sY)

[1]: http://www.gigamonkeys.com/code-reading/
[2]: https://selftaughtcoders.com/how-to-quickly-and-effectively-read-other-peoples-code/, How to quickly and effectively read other people’s code
[3]: https://changelog.com/posts/one-sure-fire-way-to-improve-your-coding
[4]: https://www.youtube.com/watch?v=_xz7gPHKGxs, LittleBIGRuby by James Edward Gray II

