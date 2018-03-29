---
title: "XRCE is dead. Long live zenoh"
date: 2018-03-23T09:14:43+01:00
draft: false
---

*Simplicity is a great virtue but it requires hard work to achieve it and education to appreciate it. And to make matters worse: complexity sells better.*


**― Edsger W. Dijkstra**


Yesterday was an incredibly sad day. Of the two proposals that had been submitted in response to the DDS-XRCE RFP -- one of those being [zenoh](http://zenoh.io) -- the alternative proposal was recommended by the evaluation team setup by the task-force in November 2017. Voters followed diligently the recommendation. A recommendation that, as explained below, in several instances appeared as lacking objective and measurable justifications.

Whilst the vote is over and nothing can be done to change it, and I think it is important to share with those interested what happened and why we will go ahead with [zenoh](http://zenoh.io).

One of the the main motivations raised by the evaluation for favouring the alternative submission was completeness. When we asked to qualify this generic statement, the answer pointed that: (1) our proposal did not address how to run on TCP/IP, and (2) our proposal did not specify resources.

As you can find out for yourself by looking up section 2.1.1 and section 2.2.1 in the [zenoh](http://zenoh.io/download/pdf/2018.04.23-zenoh.pdf) protocol specification, Section 2.2.1 explains how to do [zenoh](http://zenoh.io) framing on non-boundary preserving transports – such as TCP/IP, Section 2.1.1 defines resources.

The other argument was complexity, which is hard for me to comprehend as [zenoh](http://zenoh.io) does far more than the alternative proposal more efficiently and I’d argue elegantly.

But wait, there is more, one of my favourite arguments raised by the evaluation team is about our use of Variable Length Encoding (VLE). The evaluation team finds it complicated and favours CDR… (used by the other proposal) Yes, CDR, the Common Data Representation used by CORBA and inherited by DDS. CDR's alignment requirements, the re-ordering caused by endianness mismatch, etc., did not bother them at all.

[zenoh](http://zenoh.io) uses VLE to (1) ensure that space is not wasted when sending small integers and (2) to have the ability to represent large integers when required. Thus you pay for what you use. No fixed size and bigger upper bound.

The fact that [zenoh](http://zenoh.io) (1) has only 4 bytes overhead for data messages, and (2) is far more efficient than the alternative proposal, was not really taken into account.

But perhaps the real reason for so much resistance to [zenoh](http://zenoh.io) was that too many parties did not really wanted a protocol that is way more efficient than DDSI-RTPS, in time and space – and supports peer-to-peer, client-to-broker and broker-to-broker communication. This was clear since the very beginning, again those of you that followed the creation of the RFP will recall the resistance to calling the RFP DDSI-XRCE -- which technically was the correct name and correct scope. A new protocol for addressing constrained environments, that can also be used as a replacement for RTPS.

I don't understand this resistance as the power of DDS is in its abstraction, in the temporal and spatial decoupling. Equipping DDS with a better wire-protocol would have been beneficial for the users and to further the technology adoption.

Thus for what concerns me this is the end of XRCE.  **Long Live zenoh! Long Live zenoh! ... Long Live zenoh!**

We will continue to work to make [zenoh](http://zenoh.io) the most efficient protocol available around. We will make [zenoh](http://zenoh.io) available as a stand-alone protocol for constrained devices as well as one of the wire-protocol for our DDS implementation. The community will eventually decide which one was the right protocol!

– Angelo
