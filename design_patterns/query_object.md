<!-- TOC -->

- [1. Query object](#1-query-object)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->
# 1. Query object

Use when it would be better for readability to split DB query into smaller, easier to understand parts

## 1.1. Code sample

Simplest approach
```
const knex = require('knex');

const RECENT_USER_PROJECTS_COUNT = 5;

class RecentUserProjectsQuery {
  constructor(db, userId, orderDirection) {
    this.db = db;
    this.userId = userId;
    this.orderDirection = orderDirection;
  }

  query() {
    return this.db
      .from('projects as p')
      .leftJoin('users as u', 'u.id', '=', 'p.id')
      .where('u.id', this.userId)
      .orderBy('p.created_at', this.orderDirection)
      .limit(RECENT_USER_PROJECTS_COUNT);
  }

  static query({ userId, db = knex, orderDirection = 'desc' } = {}) {
    return new this(db, userId, orderDirection).query();
  }
}

RecentUserProjectsQuery.query({ userId: 1337 });
```

Functional approach
```
const knex = require('knex');

const RECENT_USER_PROJECTS_COUNT = 5;

class RecentUserProjectsQuery {
  constructor(db, userId) {
    this.db = db;
    this.userId = userId;
  }

  baseQuery() {
    return this.db.from('projects as p');
  }

  projectsCreatedByUser(userId) {
    return query => query
      .leftJoin('users as u', 'u.id', '=', 'p.id')
      .where('u.id', userId);
  }

  newestProjectsFirst() {
    return query => query.orderBy('p.created_at', 'desc');
  }

  takeMostRecent(limit) {
    return query => query.limit(limit)
  }

  query() {
    const chain = [
      this.projectsCreatedByUser(this.userId),
      this.newestProjectsFirst(),
      this.takeMostRecent(RECENT_USER_PROJECTS_COUNT),
    ];

    return chain.reduce((curr, next) => next(curr), this.baseQuery());

    // same as
    // const projects = this.baseQuery();
    // const byUser = projectsCreateByThisUser(this.userId);
    // const newestFirst = this.newestProjectsFirst();
    // const recent = this.takeMostRecent(5);
    // return recent(newestFirst(byUser(projects)));
  }

  static query({ userId, db = knex } = {}) {
    return new this(db, userId).query();
  }
}

RecentUserProjectsQuery.query({ userId: 1337 });

```

## 1.2. Notes
- readability is the most important feature here
- inheritance can remove the need of defining the `baseQuery` on each Query object
- Here we pass `db` as a first parameter, but it could be easily changed to accept other queries
- `query` method returns `knex` object which can be chained further
- query objects should be tests against real database
- group query objects in namespaces

## 1.3. Refs
- https://crushlovely.com/journal/7-patterns-to-refactor-javascript-applications-query-objects
- https://codeclimate.com/blog/7-ways-to-decompose-fat-activerecord-models/#service-objects#query-objects
- https://spin.atomicobject.com/2017/07/03/knex-queries/
