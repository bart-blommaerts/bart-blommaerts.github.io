---
layout: post
authors: [bart_blommaerts]
title: 'Innovation in Riga.'
image: /img/riga/riga.jpg
tags: [innovation, legacy, microservices]
category: conference
comments: false
---

Last week, I was invited to talk about innovation at [RigaDevDays](https://rigadevdays.lv/).
It was the second time I got the opportunity to speak at the biggest tech conference in Baltic States.
The conference offers presentations on Java, .NET, DevOps, Cloud, Software architecture and emerging technologies.

When it comes to innovation in technology, there is no shortage of topics to choose from and many (most) conferences focus on these specifically.
[Serverless](https://bbconsulting.be/tech/2016/11/12/TheServerlessCloud.html), Blockchain, reactive programming and deep learning are just a few of the buzzwords currently surrounding software conferences.
Ask any passionate software engineer and he or she will be aware of the constant change in tools, libraries and methodologies.
As software engineers, we are always eager to learn how to do something better or faster.
That makes the technology industry so interesting: it stays exciting. 
We always want to improve.
We never settle.

On the other hand, conferences are not always a good representation of the state of technology in the enterprise.
Most people work on systems that have been around for a while.
And that's obviously a good thing, because it means these systems get used and have proven their value.
Conferences are aspirational and want to inspire its attendees.

I wanted to talk about the combination of new technology and the fear of change in existing applications: combining new ideas and new technology on a system that has been around for a longer time.

As it turned out, this topic also appealed to other people:

> [Robert Smallshire](https://twitter.com/robsmallshire?lang=en)
>
> Observation: There's a colossal gap between a lot of solutions presented at conferences and the kinds of problems many folks are actually trying to solve.


# Approach

Assuming, the startpoint is an existing, legacy application, I started with three approaches to move from a large monolith to smaller services introducing several integration patterns to compose data repositories. These patterns allow the new services to use a domain model, tailored to their specific needs.

Next, I provided an example of bounded contexts in a domain and how these can be identified in an iterative manner.

The service decomposition and bounded context identification were demonstrated in a demo, using an event-driven approach to increase decoupling.

When there are multiple moving parts, it's important to define API Guidelines and have all edge services documented.

Finally, I added some best-practices and explained how microservices can support continuous experimenation by embracing data-driven decision making.

If you are interested in this topic for your conference or company, feel free to <a href="mailto: bart@bbconsulting.be">contact me</a>.

# Networking

Conferences facilitate knowledge sharing that allow us to meet the expectations of our customers.
In a changing landscape it's necessary to stay up-to-date and to know how innovative technology can deliver business value.

But conferences also provide an opportunity to meet interesting people or to reconnect with old friends and former colleagues.
Where else can you get tips for the logging stack you use, from [the actual evangelists](https://twitter.com/DaggieBe/status/1002111639389396992) of that stack? 

<p style="text-align: center;">  
  <img class="image fit" alt="What" src="/img/riga/bart.jpg">
</p>

