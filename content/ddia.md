+++
title = "Magic Scaling Sauce"
date = 2022-04-02
draft = true
[taxonomies] 
tags = ["Book Review"] 
+++ 

A central thesis of DDIA is that by learning from the tradeoffs made in the data structures that stuctures that power databases, communication protocols, stream processing, etc.., we will be able to make the correct decisions in our LOB applications.

0. No Secret Scaling Sauce
1. Database is still king
2. Thinking hard about forward and backward compatibility
3. The meta-database of everything
4. Small puzzles everywhere
  - databases
  - partitioning
5. Even single-leader replication has issues
6. replication has been tried and it only sort of works
7. event streams / mapReduce
8. read and write strings
9. what if everything were either a database or Unix
10. distributed computing should been avoided whenever possible

These "lessons" are at best murky, because "any application suited for one load unlikely to be suited for 10x that load". Optimizing for load depends on estimating the load parameters, guessing wrong will be at best redundant or worst add uneeded complexity to an application.  

The book includes many well-chosen topics that strike a balance between providing knowledge that generalizes to LOB applications and providing deep knowledge on specific data concept. Here are a few that stood out.

  - How backwards and forwards compatiblitiy works in several binary communication protocol
  - Discussing the design for database by starting with this:
  - ...and then working up to the need for indices and comparing LSM-trees and B-trees
  - Comparing today's debate of NoSQL and SQL with that of SQL and CODASYL in the 1970s
  - Comparing different database partitioning strategies
  - a rather dour chapter on database replication
  - a re-evaluation of transactions and what ACID means, and why they "C" should be dropped
  - isolation levels
  - what mapReduce and unix pipes have in common

Will these lessons help us in our project today, or should we be as sceptical of this as say, the most notorious tests of engineering skill? people of learning algorithms (twitter link)? [link](https://twitter.com/KevinNaughtonJr/status/1514423070345342979)

First, he does this sort of anaylsis himself in chapter 12. Designing systems around dataflow, eschewing distributing locking mechanisms, making systems look like Unix pipelines.
499 - comparing Unix and databases as storage engines

One encouraging data point is how useful a text this was in a bookclub. I read this with Jason Giles, a staff software engineer at SEP, as a part of the SEP Mentors program. The stories of tricky integration problems, database management, faulty hardware and networks, the challenges of working with systems, even if we weren't, say, designing the next distributed database.

I'll say I'm no less amazed at seeing how Amazon s3 provides 99.999999% durability on faulty hardware, or how Aurora performantly manages synchronous data backups. But getting a lay of the land (the book comes with illustrations of a 'map') is a start.

Of course, this book leaves much about the operation side of working with these systems as an excercise for the reader. Very much looking forward to the "Maintaining and Operating Data-Intensive Applications", "Deploying...", "Testing", etc..

"There is no secret sauce", he writes, instead all we get is a detailed map with the forest and tree details of distributed, high-traffic applications. The wide-ranging. Detailed account of each individual piece of technology yet be able to 

====
Sean Kenney builds Lego sculptures.

The centerpiece to his travel show, Animal Super Powers, which has at the Indiana State Fair in ????, is a sculpture of a hummingbird hovering over a trumpet flower.

delicate things / sturdy things ... need to build this big
all the work to transport (the manpower to move something so heavy, having a truck to fit something so large, making some parts removable to fit, custom box.

He estimates he keeps a million legos in his studio in Brooklyn.

Duck head (high-res)

vin-mar cabinets by stanley
boxes on pull out carts
labels on doors
they are impressive, not complex

weather-proofing
used a custom-made lego piece for a mouse nose!
made once, sent Outputs

custom lego harnesses for transfer
keep a bicycle up by gluing to plastic (lazer-cut acryclic sheet)

===========

I intro Sean's work as a way of introduction into Kleppman's work in Designing Data-Intensive Applications. 
1. a taste for building large systems (In the amount or complexity or data), an interest in its asthetics and its engineering challenge (as opposed to any moral weight)
2. a belief in elegant at scale - reproducing the enthusiam of how the tool feels at small scale at large scale, that most work at
Second, like a Lego collection, how certain ideas can cary over across systems, and a steadfast believe that the software built can stay elegant in the face of stagering difficulty.
distribution, concurrency, transactions, 
- not sure Kleppmann's elegant theory works -> there are just a ton to know about any one of these systems, and every case has its details.
====================

It's a gift to programmers to not have to understand everything about a system in order to contribute to it. Best practices suggest you do so. Larger programs are split into functions. Side-effects are eliminated. Outputs are tested against inputs.

The Software Engineering bit google exposes is "Software Engineering is Programming x time". How scaling works. This book displays how these systems work at the database layer. The trade-offs between availability and consistency, between performance and consistency. It goes into great detail about how these trade-offs affect replication, distribution, and encoding.

He really hits his stride in Chapter 2, Encoding and Evolution, where he weaves a story about the importance of backward and forward compatibilty into a technical story about how different binary protocols. It's a good story to remember in order to reason about compatibility in any system. Also, here he starts painting his understanding of apps deployed as 'nodes' where you have several deployed with different versions of both provider and consumer in your system, old and new data, essential as you have a system with rolling updates and zero downtime.

This complexity will play out on a grand scale in talks of database partitioning and replication. Though this section would be more pertinent if I programmed databases. Though my hunch that this section will help me wade through the many configuration options made available through databases and cloud providers. And the clever but fundamentally limited solutions.

To top it all off, it's a fine piece of technical writing, going into fine detail while never leaving its conversation tone througout. He revels in the myriad of problems partial failures can introduce into your system. An eagle eye view without a loss of detail.


