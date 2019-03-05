---
layout: post
title: 'Domain Logic in Express: Part 2'
date: 2019-03-06 23:07:02 +0530
comments: false
excerpt: 'Transaction Script's pitfalls'
category: software
tags: [express, architecture, design]
---

While considering how the Transaction Script architectural pattern
can become difficult to manage as an application's feature set grows,
we'll need a better example than the non-existent widget management
app in the previous post.

TBD is a single-page application that lets users TBD. It calls a
REST API implemented using Express. Here are the endpoints that
the client depends on:

| Verb      | URL      | Action                  |
| --------- | -------- | ----------------------- |
| GET       | tbd      | Get all TBD             |
| POST      | tbd      | Create a TBD            |
| GET       | tbd/{id} | Get a particular TBD    |
| PUT/PATCH | tbd/{id} | Update a particular TBD |
| DELETE    | tbd/{id} | Delete a particular TBD |

Eventually, the application need to do TBD as well. When a TBD is
created, a corresponding TBD needs to be created as well and associated
with the new TBD.