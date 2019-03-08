---
layout: post
title: 'Domain Logic in Express: Part 2'
date: 2019-03-08 23:07:02 +0530
comments: false
excerpt: 'Transaction Script''s pitfalls'
category: software
tags: [express, architecture, design]
---

While considering how the [Transaction Script](https://martinfowler.com/eaaCatalog/transactionScript.html)
architectural pattern can become difficult to manage as an application's feature set grows,
we'll need a better example than the non-existent widget management app discussed in the previous post.

## Phase One

Wormwood is a job board that lets organizations submit job postings.
Job seekers can read these job postings and click a link to submit an
application on the organization's website.

The client makes asynchronous calls to a REST API
implemented using Express. Here are the endpoints that the client
depends on so that organizations can create, read, update, and
delete job postings:

| Verb   | URL              | Action                          |
| ------ | ---------------- | ------------------------------- |
| GET    | /job-posting     | Get all job postings            |
| POST   | /job-posting     | Create a job posting            |
| GET    | /job-posting/:id | Get a particular job posting    |
| PUT    | /job-posting/:id | Update a particular job posting |
| DELETE | /job-posting/:id | Delete a particular job posting |

So, Wormwood allows users who are associated with an organization
to submit and update job postings; users who are not associated with
an organization can only view these job postings. Organizations pay
$50 a month for their users to post jobs on Wormwood.

Given the modest feature set, the first version of Wormwood's backend
handles job posting use cases with the Transaction Script pattern.
Here's the initial implementation of the `POST /job-posting` endpoint:

```javascript
app.post('/job-posting', (req, res) => {
  validator.checkJobPosting(req.body.jobPosting, (err, validatedJobPosting) => {
    if (err) {
      return res.status(500).send({ error: 'validation error', message: err });
    }
    databaseGateway.saveJobPosting(
      validatedJobPosting,
      (err, savedJobPosting) => {
        if (err) {
          return res
            .status(500)
            .send({ error: 'database error', message: err });
        }
        return res.status(201).send(savedJobPosting);
      }
    );
  });
});
```

As in the throwaway example in the previous post, the anonymous callback that
`app.post()` takes as its second argument handles everything: it calls a
validation module to make sure the job posting is safe to persist, and then
calls a database gateway module to save the job posting. If either of those
processes throws an error, the callback sends that error along to the client;
if the job posting is saved successfully, the callback sends the saved
job posting back to the client.

You can look at the rest of the code for this initial implementation in the
`app/transaction-script-phase-one` file in the [Wormwood repository on GitHub](https://github.com/akuny/express-domain-logic). It's straightforward: the backend doesn't need to do
anything apart from validate `POST` and `PUT` request payloads and perform
simple CRUD operations.

## Phase Two

Wormwood has been up and running for a few months, and leadership has
an exciting idea: what if some organizations could opt into a "Gold"
tier that allowed them to select three featured job postings a month to be
displayed on a splashy UI that users would see upon logging in? For
$75 a month, this tier is theirs.

We can't naively save job postings anymore. Instead, we need to
check to see whether the job posting has been selected as featured,
check what tier an organization is in (Normal or Gold),
check whether the organization has already submitted three or more
featured job postings this given month, and then accept or reject
the submission depending on what those checks turned up.

Looks like we'll need to cram a lot more functionality in our
route's callback. What might that look like? Check out
`app/transaction-script-phase-two` to see. Here's the same
`POST /job-posting` route we saw above with the exciting new
features incorporated.

```javascript

/*
*
* To anyone reading this while it's in draft: yes, I know the routine below
* is monsterous beyond reason, but it's the best I could come up with after
* a long day. Come to think of it, does that make it closer to what some
* poor soul would crank out in a blur?
*
*/
app.post('/job-posting', (req, res) => {
  validator.checkJobPosting(req.body.jobPosting, (err, validatedJobPosting) => {
    if (err) {
      return res.status(500).send({ error: 'validation error', message: err });
    }
    // TODO check if jobPosting has featured property set to true
    if (jobPosting.organization.type === 'gold') {
      validator.countFeaturedPosts(validatedPobPosting, (err, featuredAvailable, validatedJobPosting) => {
        if (featuredAvailable) {
          databaseGateway.saveJobPosting(validatedJobPost, (req, res) => {
            if (err) {
              return res
                .status(500)
                .send({ error: 'database error', message: err });
            }
            return res.status(201).send(savedJobPosting);
          })
        } else {
          return res
            .status(500)
            .send({ error: 'feature error', message: err })
        } else {
          databaseGateway.saveJobPosting(
            validatedJobPosting,
            (err, savedJobPosting) => {
              if (err) {
                return res
                  .status(500)
                  .send({ error: 'database error', message: err });
              }
              return res.status(201).send(savedJobPosting);
            }
          );
        }
      });
    }
  });
});
```

It's hideous, but it works, for now. (NOTE: does not actually work at time of drafting.)

A few months later, another turn of the screw: now leadership wants a Silver
and a Platinum tier. The Silver tier offers participants one featured post a
month, while the Platinum introduces an exciting enhancement: organizations
in the Platinum tier that submit at least two job postings a week get one
featured post for every additional job posting they submit each week.

Consider the nightmare of conditional logic we'd have to reckon with if we
cram this additional logic into that poor callback. It's enough to reconsider
our approach: maybe there's a better way to design our backend, given the
complicated business rules we now have to handle? Next time, we'll reason
through the [Domain Model](https://martinfowler.com/eaaCatalog/domainModel.html)
architectural pattern and how it could help us out of this mess.