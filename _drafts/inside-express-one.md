---
layout: post
title: 'Inside Express: Part 1'
date: 2018-03-03 23:07:02 +0530
comments: false
excerpt: 'Exploring Express''s source code.'
categories:
---

Having used [Express](https://expressjs.com/) to build small applications,
I'd like to better understand how the framework does its thing. I'll use
a little "Hello World" application as a springboard. Here are its first
two lines:

```javascript
// index.js

var express = require('express');
var app = express();
```

First, we declare an `express` variable and assign the Express module
to it using the native Node.js [`require` function](https://nodejs.org/api/modules.html#modules_require_id).
Then, we declare an `app` variable and assign `express()` to it
in order to create our application.

What is `express()`? The `index.js` file in the Express module's root directory
contains one line of code:

```javascript
module.exports = require('./lib/express');
```

The centerpieces of `lib/express.js` is `createApplication()`. Unsurprisingly, this
function returns an Express application. Here's its first statement:

```javascript
var app = function(req, res, next) {
  app.handle(req, res, next);
};
```

Next, the `merge-descriptors` module, which has been assigned to the `mixin` variable,
VERB the native Node.js `EventEmitter` class and the object defined in `lib/application.js`.

```javascript
mixin(app, EventEmitter.prototype, false);
mixin(app, proto, false);
```
