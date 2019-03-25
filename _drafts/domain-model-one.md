---
layout: post
title: 'Domain Logic in Express: Part 3'
date: 2019-03-26 20:07:02 +0530
comments: false
excerpt: 'Toward a Somewhat Coherent Domain Model'
category: software
tags: [express, architecture, design]
---

Where we left off in the [previous post in this series](https://www.andykuny.com/software/2019/03/12/transaction-script-two.html),
our route for handling the creation of job postings had grown out of control. In this post, we'll refactor using the
[Domain Model](https://martinfowler.com/eaaCatalog/domainModel.html) architectural pattern.

## Domain Model at a Glance
Here's Martin Fowler's definition of a Domain Model:

> At its worst business logic can be very complex. Rules and logic describe many different cases and slants of behavior, and it's this complexity that objects were designed to work with. A Domain Model creates a web of interconnected objects, where each object represents some meaningful individual, whether as large as a corporation or as small as a single line on an order form.

To implement a Domain Model (or anything, really) effectively, we need to step back and consider the underlying data schema of JobFair, our example application. Although the JobFair implementation uses a silly in-memory database, let's assume that in practice it would interface with a relational database structured around Users, Organizations, and Job Postings as such:

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
  
These three data entities relate to one another in the following ways:

1. A User __belongs to one__ Organization and __has many__ Job Postings
2. An Organization __has many__ Users and __has many__ Job Postings
3. A Job Posting __belongs to one__ Organization and __belongs to one__ User

