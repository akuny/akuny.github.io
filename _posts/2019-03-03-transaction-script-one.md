---
layout: post
title: 'Domain Logic in Express: Part 1'
date: 2019-03-04 23:07:02 +0530
comments: false
excerpt: 'The seductive simplicity of the Transaction Script'
category: software
tags: [express, architecture, design]
---

When creating a new [Express](https://expressjs.com/) application, I tend to jam as much logic as I can
in the anonymous callback that [`app.METHOD()`](https://expressjs.com/en/4x/api.html#app.METHOD)
takes as its second argument. It's easy to reason with: here's
everything that happens in response to a given HTTP request in one location.

Let's say we're working on a widget app: when the client sends
a `POST` request to `/widget/make`, the backend needs to validate the widget and
save it to disk. In the example below, assume that we've assigned validation and
database access modules to `validator` and `database` variables.

```javascript
app.post('/widget/make', (req, res) => {
  const widget = req.body;
  validator.check(widget, (err, validatedWidget) => {
    if (err) {
      return res.status(500).send({ error: 'Validation failed', message: err });
    }
    database.saveWidget(validatedWidget, (err, savedWidget) => {
      if (err) {
        return res
          .status(500)
          .send({ error: 'Database access failed', message: err });
      }
      return res.status(200).send(savedWidget);
    });
  });
});
```

This approach is an instance of Martin Fowler's
[Transaction Script](https://martinfowler.com/eaaCatalog/transactionScript.html)
pattern: it "organizes business logic by procedures where each procedure handles
a single request from the presentation." The client calls a particular endpoint,
and that endpoint runs a routine that handles any and all logic
required in response to the request: validation, calculation, database access,
calls to external services, and so on.

We could refactor the code snippet above by extracting the anonymous callback
to a separate controller function. Assuming we don't mind taking the time to
rework our validation and data access modules, we could also refactor to
[Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
and do away with our burgeoning "pyramid of doom." But despite those
changes, we're still organizing our domain logic in a single procedure
called by the client.

Next time around, we'll consider how this single procedure could become
gnarly as our feature set grows.
