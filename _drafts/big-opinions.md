---
layout: post
title: 'Big Opinions'
date: 2018-03-05 23:07:02 +0530
comments: false
excerpt: 'Making architectural decisions by accident.'
categories:
---

> We're gonna make some big decisions in our little world.

_Bob Ross_

Once upon a time, a teammate and I were tasked with building a backend API using
[Node.js](https://nodejs.org/). My knowledge of Node.js was limited at best: it was
JavaScript on the server, right? And asynchronous?

The situation called for a "batteries included" framework, something akin to Rails
that could get us up and running quickly. Given its popularity, [Express](https://expressjs.com/)
was one of the first frameworks I looked into.

Here are some quotations from the [Express FAQ](https://expressjs.com/en/starter/faq.html):

> _How should I structure my application?_
>
> There is no definitive answer to this question. The answer depends on the scale of your
> application and the team that is involved. To be as flexible as possible, Express makes no
> assumptions in terms of structure.

Oh.

> _How do I define models?_
>
> Express has no notion of a database.

Oh.

> _How can I authenticate users?_
>
> Authentication is another opinionated area that Express does not venture into.

Oh no.

Lacking a basic understanding of how one would structure a product using the specified
technology, I needed opinions more so than anything. Suffice to say the product was
not built with Express.

## from hello world to hell no

Here's the standard Express "Hello World" example:

```javascript
var express = require('express');
var app = express();

app.get('/', function(req, res) {
  res.send('Hello World!');
});

app.listen(3000, function() {
  console.log('Example app listening on port 3000!');
});
```

Here, we:

1. Pull in Express
2. Assign an instance of the express object to `app`
3. Add a `GET` route
4. Start the application on port 3000

Let's move on to a more functional example of a route in Express.
Here's a `POST` route for a notional notekeeping application:

```javascript
app.post('/note', function(req, res) {
  // involves pulling userId from req, validating note
  // content, persisting to data store, returning res
});
```

Here, the anonymous callback function does quite a bit.

This is a test.