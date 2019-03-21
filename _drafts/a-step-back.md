---
layout: post
title: 'Taking a Schema for Granted'
date: 2019-03-22 20:07:02 +0530
comments: false
excerpt: 'Abstracting away the icky bits'
category: software
tags: [architecture, design]
---

Whie working on the in-progress series about organizing domain logic in
Express applications, I had to make some make some compromises (or cut some corners)
in order to get anything posted in the first place. The first post in the series
concerned a non-existent "widget" application; the second post shifted to
a notional job board application that in practice had a loosely adhered to (at best)
underlying schema and an in-memory faux-database to enable (or justify)
said loosey goosey-ness regarding how the different data entities in the
application related to each other.

1. high-level view of how the job-posting, org, and user entities fit together
2. The dreaded UML class diagram
3. Segue into preview of implementation
4. Tie back into opening section: would have been far more instructive to have defined this up front.
5. (Counterforce: would such an approach always-already lead to domain model?)
