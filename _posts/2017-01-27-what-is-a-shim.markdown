---
layout: post
title: 'What is a shim?'
date:   2017-01-27 11:11:11 +0000
draft: false
---

You're stuck. 

You visit Stackoverflow.  

You double check Wikipedia.  

Nine times out of ten, that's you sorted.

However, this was that tenth time.

"What is a shim?" - I could now write my own wiki entry, but remained clueless as to how to apply my new-found knowledge.

 So, here goes - go eleven - the blog post.  

![Get Shimmying](http://i.imgur.com/IwdBJKD.gif)
{% include helpers/image.html name="catshim.gif" caption="Get shimmying" %}


### What is a shim?

> "We can solve any problem by introducing an extra [level of indirection ][2]." 
> - [David Wheeler][1]

A shim is a small library that intercepts and changes calls to another library, mainly to aid compatibility. 

{% include helpers/image.html name="not-a-shim.png" caption="This is not a shim" %}

### Examples of using a shim
Summarising examples I have found:


- [Compatibility between versions](#compatibility-between-versions)
- [Compatibility between runtime environments](#compatibility-between-runtime-environments)  
- [Compatibility between Operating Systems](#compatibility-between-operating-systems)  
- [Multiple Rubies](#multiple-rubies)  

<br/>

### <a name='compatibility-between-versions'>Compatibility between versions</a>

Maintaining multiple versions of a library is necessary to support your clients. Shim libraries translate old to new library calls before forwarding on to the new library. Fewer libraries now need supporting.


In our example we are making version 7 of our famous quotes library Recite.  We want to use an update of the Random Quote Library, V 1.1, but do not wish to upgrade to the breaking change of the Quote Library V2.0.


| | Recite V 6.0  | Recite V 7.0 |
|:--------------|:------------------|:-------------------------------------------|
| Random Quote Library | V 1.0 | V 1.1 |
| API call | quote() | quote("author":) |
{:.table .u-margin-bottom-large}

Unknown to us, the random quote library developers have stopped working on version 1 of the library and are only releasing version 2. Version 2 breaks with Version 1 of the API; to get a random quote, we must pass an empty string to the "author" attribute. The developers are maintaining compatibility by supplying a shim that sets the author to an empty string. 

{% include helpers/image.html name="version-compatibility.png" caption="Both applications think they are using V6.0 of the Random quote library" %}
Developers will only have to support Version 2 of the quote library and a small shim library for compatibility.
<br/>
<br/>




### <a name="compatibility-between-runtime-environments">Compatibility between runtime environments</a>

Shim allow 64-bit applications to call 32-bit libraries. 64-bit applications cannot load 32-bit libraries. Shims solve incompatibility issues such as size of the data, pointers and dependencies. The shim library creates a child-process for completing the application's requests. The shim acting as a go between for forwarded requests to the child-process and returning results back to the application.

{% include helpers/image.html name="runtime-environments.png" caption="Shim to allow a 64-bit application to use a 32-bit library" %}

#### Source
[A guide to using shims to deal with incompatible runtime environments][3]

<br/>

### <a name='compatibility-between-operating-systems'>Compatibility between Operating Systems</a>

Microsoft uses shims to fake an application's Windows calls. When an application makes a system call it goes through the 'Import Address Table'. 

{% include helpers/image.html name="windows.png" caption="Application calling into Windows from IAT" %}

You can change the table and replace the Windows call with a call to a shim. The shim in the example is a 'version-lie' shim. The application thinks it's on a Windows 7 machine.
{% include helpers/image.html name="windows-shimmed.png" caption="Application call redirected to shim from IAT" %}

#### Source
[Understanding Shims](<https://technet.microsoft.com/en-gb/library/dd837644(v=ws.10).aspx>)
[Demystifying Shims - or - Using the App Compat Toolkit to make your old stuff work with your new stuff](https://blogs.technet.microsoft.com/askperf/2011/06/17/demystifying-shims-or-using-the-app-compat-toolkit-to-make-your-old-stuff-work-with-your-new-stuff/)  
<br/>
<br/>

### <a name='multiple-rubies'>Multiple Rubies</a>

Linux installations normally come with a single Ruby version. Ruby developers need to have different Ruby versions on their many active projects. Rbenv uses shims to solve this problem.

Running a Ruby command in Linux, means checking for the executable in the path - which it searches from left to right. So, running the Ruby command `rails server`, means Linux finds and runs Rails, a Ruby executable, in the first directory.

{% include helpers/image.html name="single-ruby.png" caption="Running a Ruby application, rails, normally" %}

Rbenv adds a `shims` directory and loads it with shim scripts before prepending the Path. There is a shim script for every Ruby application and running a Ruby application now means running the matching shim script. So, running rails means executing the Rails script in the `shims` directory and not the Rails application in `usr/local/bin`. The script works out the required Ruby version and then runs its matching application, in this case Rails, under that expected Ruby version.

{% include helpers/image.html name="multiple-rubies-with-rbenv.png" caption="Running a Ruby application, rails, with RBenv" %}

#### Source
[Understanding shims](https://github.com/rbenv/rbenv#understanding-shims)  
[Rbenv - How it works](https://medium.com/@Sudhagar/rbenv-how-it-works-e5a0e4fa6e76#.5a0h5ydx9)
<br/>
<br/>

### Similar Terms

Shim's versatility makes it confusing to understand all the associated terms. Here I list similar terms and their relationship to shims.

| term          | Definition                                                                        |
|:--------------|:----------------------------------------------------------------------------------|
| [Adapter][4]  | a design patterns by the [Gang of four][5] that is a shim.                        |
| [Facade][6]   | see Adapter                                                                       |
| [Hooking][7]  | covers a range of techniques to alter the running of software including shims.    |
| [Polyfill][8] | is a shim associated with [providing fallback functionality to older browsers][9] |
| [Proxy][10]   | see Adapter                                                                       |
{:.table .u-margin-bottom-large}

### Summary

Use shims for [bodging](https://en.wiktionary.org/wiki/bodge) by supporting older APIs, lying to applications or calling across runtime environment. Or use shims as a design choice with supporting multiple Rubies.

We can solve any problem by introducing an extra level of indirection - adding an extra hop remains a versatile tool.


[1]: <https://en.wikipedia.org/wiki/David_Wheeler_(British_computer_scientist)>

[2]: https://en.wikipedia.org/wiki/Fundamental_theorem_of_software_engineering

[3]: https://www.ibm.com/developerworks/rational/library/shims-incompatible-runtime-environments/ 'A guide to using shims to deal with incompatible runtime environments'

[4]: https://en.wikipedia.org/wiki/Adapter_pattern

[5]: https://en.wikipedia.org/wiki/Design_Patterns

[6]: https://en.wikipedia.org/wiki/Facade_pattern

[7]: https://en.wikipedia.org/wiki/Hooking

[8]: https://en.wikipedia.org/wiki/Polyfill

[9]: https://www.paulirish.com/i/7570.png

[10]: https://en.wikipedia.org/wiki/Proxy_pattern

