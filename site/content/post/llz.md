---
title: "XRCE is dead. Long live zenoh"
date: 2018-03-23T09:14:43+01:00
draft: false
---

Yesterday was an incredibly sad day. I witnessed the **OMG MARS** task-force
*murder* XRCE. It was murdered by a blend of superficiality, incompetence,
self-interest, and political games. It really was horrible! It was
disappointing!

But wait... Actually something even worse happened. The essence of **XRCE**
was corrupted and violated by accepting a proposal that is inefficient, clumsy,
baroque, anti-innovative, and extremely boring... But, hey, it uses UML and has
a PIM! Isn't that great?

The alternative proposal was recommended by the evaluation team setup
by the task-force in November 2017. Voters followed diligently the infamous recommendation.
A recommendation, which as I will explain below was based on lack of understanding
and, for some of the members, connivence with vendors that did not want a protocol
that compete with DDSI-RTPS.

One of the evaluation team claims, was about our proposal being *less complete*.
This was an unqualified claim, thus I asked to provide some examples. The answer
was that our proposal did not address how to run on TCP/IP and that we did not
specify resources.

Both of these statements are false as you can find out for yourself by looking
up section 2.1.1 and section 2.2.1 in the [zenoh](http://zenoh.io/download/pdf/2018.04.23-zenoh.pdf)
protocol specification. Specifically Section 2.2.1 explains how to do **zenoh**
framing on non-boundary preserving transports -- such as TCP/IP. Section 2.1.1 defines
resources.

Thus at this point the question is: did the evaluation team ever read our
proposal? I think the skimmed through it, but fast enough to miss even the vary
basics. Beside, they never asked a single technical clarification on any of the
protocol mechanisms, and I can safely assume that this is not because they understood
it all!

The other argument was complexity... Which is fallacious argument as **zenoh**
did far more than the competing submission more efficiently and I'd argue elegantly.
But again some people prefer Lipton tea to Li Shan High Mountain Oolong, as they
cannot appreciate the fine and subtle.

But wait, there is more, one of my favourite arguments raised by the  evaluation
team is about our use of Variable Length Encoding (VLE). The evaluation team finds it complicated and favour
CDR... (used by the other proposal) Yes, CDR!!! You know the Common Data
Representation used by CORBA and inherited by DDS. The alignment requirements,
the re-ordering caused by endianess mismatch, etc., did not bother them at
all.

**zenoh** uses VLE to (1) ensure that space is not wasted when sending small
integers and (2) to have the ability to represent large integers when required.
Thus you pay for what you use. No fixed size and bigger upper bound.

Now let's get to the politics. The submitters of the alternative proposal did
not really wanted a protocol that is way more efficient than DDSI-RTPS, in time
and space -- and supports peer-to-peer, client-to-broker and broker-to-broker
communication. This was clear since the very beginning, again those of you
that followed the creation of the RFP will recall that they opposed to calling
the RFP DDSI-XRCE. Which technically was the correct name and correct scope.
A new protocol for addressing constrained environments, that can also be used
as a replacement for RTPS.

These folks are completely missing the point. The power of DDS is in its abstractions,
in its support for data-centricity, temporal and spatial decoupling.
Equipping DDS with a better wire-protocol would have been beneficial to the users
and it would have helped the technology adoption.

Thus this is the epilogue of XRCE. XRCE has been deprived of its dignity, it has
been corrupted, violated. In short XRCE is dead!

At this point I feel like saying  **Long Live zenoh**, **Long Live zenoh**,
**Long Live zenoh**!

We will continue to work to make **zenoh** the most efficient protocol available
around. The community  will eventually decide which one was the right protocol.


-- Angelo
