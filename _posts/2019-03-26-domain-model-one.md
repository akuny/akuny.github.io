---
layout: post
title: 'Domain Logic in Express: Part 3'
date: 2019-03-26 20:07:02 +0530
comments: false 
excerpt: 'Toward a somewhat coherent Domain Model'
category: software
tags: [express, architecture, design]
---

Where we left off in the [previous post in this series](https://www.andykuny.com/software/2019/03/12/transaction-script-two.html),
our route for handling the creation of job postings had grown out of control.
In this post, we'll refactor using the [Domain Model](https://martinfowler.com/eaaCatalog/domainModel.html) architectural pattern.

## Domain Model at a Glance
Here's Martin Fowler's definition of a Domain Model (emphasis mine):

> At its worst business logic can be very complex. Rules and logic describe
> many different cases and slants of behavior, and it's this complexity that
> objects were designed to work with. A Domain Model creates __a web of
> interconnected objects, where each object represents some meaningful
> individual,__ whether as large as a corporation or as small as a single
> line on an order form.

To implement a Domain Model (or anything, really) effectively, we need to step
back and consider the underlying data schema of JobFair, our example application.
Although the JobFair implementation uses a silly in-memory database, let's assume
that in practice it would interface with a relational database structured around
Users, Organizations, and Job Postings as such:[^1]

| Table        | Field         | Type                 | Notes              |
| ------------ | ------------- | -------------------- | ------------------ |
| user         | id            | autoincrementing int | primary key        |
| user         | first_name    | string               |                    |
| user         | last_name     | string               |                    |
| user         | org_id        | int                  | forgeign key       |
| organization | id            | autoincrementing int | primary key        |
| organization | name          | text                 |                    |
| organization | tier          | enum                 | 'gold' or 'normal' |
| organization | featured_left | int                  |                    |
| job_posting  | id            | autoincrementing int | primary key        |
| job_posting  | title         | text                 |                    |
| job_posting  | description   | text                 |                    |
| job_posting  | contact       | text                 |                    |
| job_posting  | is_featured   | boolean              |                    |
| job_posting  | user_id       | int                  | forgeign key       |
| job_posting  | org_id        | int                  | forgeign key       |
  
[^1]: I've omitted fields like `created_at` or `updated_at` that might be included automatically depending on one's RDBMS and configuration.

These three data entities relate to one another in the following ways:

1. A User __belongs to one__ Organization and __has many__ Job Postings
2. An Organization __has many__ Users and __has many__ Job Postings
3. A Job Posting __belongs to one__ Organization and __belongs to one__ User

To create our Domain Model, we'll need objects that represent Users, Organizations,
and Job Postings.[^2] Classes defining what state these objects will store and how they
will behave are in the `app-domain-model/model` directory of the
[JobFair repository](https://github.com/akuny/express-domain-logic/tree/master/app-domain-model/model).
Here are some changes highlighting how we've moved on from Transaction Script:

[^2]: Were I a bit more courageous, I could include old-fashioned UML class and sequence diagrams here.

## Database Access
Due to the simplicity of JobFair's Domain Model, I chose a simple, limited
implementation of the [Active Record](https://www.martinfowler.com/eaaCatalog/activeRecord.html) pattern.
Active Record objects know how to communicate with the database: for instance, the `Organization`
model knows how to retrieve a specific instance of an Organization from disk by calling `read(orgId)`.
In a more sophisticated context, I'd have implemented database access methods for
these Active Record classes using the [Layer Supertype](https://martinfowler.com/eaaCatalog/layerSupertype.html)
pattern instead of writing these methods for each model redundantly. In a real-world context,
I'd use an object-relational mapper (ORM) like [Sequelize](http://docs.sequelizejs.com/).

In any case, now Users, Organizations, and Job Postings are responsible for accessing the database,
not a script.

## Business Logic
These models are now responsible for validating themselves and for checking whether or not their instances
conform to certain business rules. For example, an Organization is responsible for determining whether it has
any featured posts remaining:

```javascript
class Organization {
  /// ...
  hasFeatured() {
    if (this.featuredRemaining > 0 && this.tier === 'gold') {
      return true;
    }
    return false;
  }
}
```

As with database access, this logic is no longer the concern of a script. If we needed to incorporate
business logic concerning a "silver" or a "platinum" tier, we'd be able to do so in the `Organization` class and not
in a disparate group of scripts.

## Tidier Route
After refactoring to a Domain Model, here's the `POST /job-posting` endpoint:

```javascript
app.post('/job-posting', (req, res) => {
  const jobPosting = new JobPosting(req.body.jobPosting);
  jobPosting.create((err, savedJobPosting) => {
    if (err) {
      return res.status(501).send({
        error: 'There was an error saving the job posting',
        message: err
      });
    }
    return res.status(201).send(savedJobPosting);
  });
});
```

Even with logical checks against whether or not a Job Posting may or may not be featured
having been incorporated into the application, our route is cleaner than it was before:
create a new job posting and send it back to me.

## Potential Improvements

There's no shortage of issues with this implementation of a Domain Model for JobFair:

1. `JobPosting` instantiation is hard-coded into the `POST /job-posting` endpoint. What if another interface with the application needed to create a `JobPosting`? Including a service layer would mitigate the risk of redundancy were a requirement like this to crop up.
2. The belongs-to-one and has-many relationships described above are not enforced. Using a robust ORM in a non-academic setting is a must.
3. The fact that `JobPosting` knows when to decrement its `Organization`'s number of featured posts remaining may be not-so-good.
4. I'm taking for granted that these fusty OOP concepts have some bearing on an Express application; others may disagree strongly.

Despite these shortcomings, I think JobFair is in better shape than it was before. The straightforward nature of Transaction
Script remains appealing for many uses cases, but if there's any risk of complicated domain logic down the road, working
with a Domain Model from the ground up might not be a bad idea.

***