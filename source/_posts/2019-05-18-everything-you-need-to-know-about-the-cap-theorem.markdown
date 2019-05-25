---
layout: post
title: "Everything you need to know about the CAP Theorem"
date: 2019-05-18 15:38:06 -0700
comments: true
categories: Distributed Systems
---

CAP? You thoght, what is it all about? Everytime you heard this word from some experienced guys discussing, you told yourself, this got be very complicated. Well, not really... It's simplier more than what you expected, I tried to figure out what is CAP recently and put everything I went through here for a record (for fun really). Read on...

<!--more-->

CAP == Consistence + Availability + Partition Tolerent. That's it! So simple! But when you want to understand what it is really talking about under the context of Distributed Systems, it suddenly becomes a mystery. Why is that? In my words, it as a design principle itself is too obvious to give us real practical design guidelines. Brought by Dr Eric Brewer from UC Berkeley in 1998 and offically represented in PODC 2000, I guess it's more like learnings from the systems they built at that time.
Actually the context from the very beginning is distributed shared data, i.e. distributed storage. Why is that? Because computing is easy, stateless. Network is always unreliable (by natural), you can't do much about it. Storage is stateful (obviously) and can be reliable using algorithms despite of unreliable hardware.

Let' see how to understand CAP fairly. You may have seen some triagles and be bold, hey, take 2 out of 3. Well, not really. When we go from single-node system to distributed computing, partition tolerant is a must-have property. While CAP can't be achived at the same time, you have to trade off between C and A. You want to have a real high availability system, fine, if partition happens, you have to sacrifice Consistency in some degree (and revover Consistency after partition healed). If you
want Consistency all the time, partition will force you sacrifice Availability, you don't want to response with inconsistent data after all.

Note that, Consistency here means **Strong Consistency**, this is very important when you try to understand CAP. Why? eventual consistency can ensure high availability. The consistency models employed highly depends on business scenarios, for serious data, strong consistency has to be enfored. Production systems may have deploy different consistency models to different sub-systems (components).

Gilbert and Lynch from MIT gave a formal proof in 2002 for Brewer's conjecture. Read that paper, you will find, hmm, this is naive. However, I think the contribution here is to model such systems, give the community a common tongue on pushing the area forward and lay a good foundation since then. I can't help to think, for distributed systems, practice seems a lot complicated than theory. Model can be simple while production systems are always complicated and changing.

In 2012, Brewer wrote a new article to talk about CAP and give it some new meaning refelecting 12 years later after further introduced. This article is much more helpful on discussing how to interpret it correctly (and also point how can it be misleading for the 2 out of 3 saying), the new context of CAP as new systems emerge and evolve, how to keep availabilty in the case of partition and recovery strategies when partition heals. It's worhy a reading, the discussions in it are
practical engough (with very solid and convicing examples).

A paper criticizing CAP appeared in 2015, saying CAP is too high-level to give anything useful when one tries to design such systems, partition in real-life is more like latency. The author came up with a new framework, which is more practical-oriented. I didn't read the paper in details, planning to, it's a very good paper on reviewing what is CAP, how it's proved, what are the short comings and how we can step further. Perhaps, this paper alone will just give you enough context of
CAP, don't bother for other readings.

I assemble a slides and put it [here](https://drive.google.com/file/d/1c6_bZFSRXILkcYWaNSo7ICNsrKH_o4Mf/view?usp=sharing).

Moving forward, I will take a look at Lamport Clocks, Paxos, Raft, etc. So much fun:-)

