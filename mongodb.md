<!-- TOC -->

- [1. MongoDB](#1-mongodb)
    - [1.1. Relationships](#11-relationships)
        - [1.1.1. Embedding](#111-embedding)
        - [1.1.2. one to N](#112-one-to-n)
        - [1.1.3. two-way referencing](#113-two-way-referencing)
    - [1.2. Schema modeling patterns](#12-schema-modeling-patterns)
        - [1.2.1. Duplication](#121-duplication)
        - [1.2.2. Attribute pattern](#122-attribute-pattern)
        - [1.2.3. Extended reference pattern](#123-extended-reference-pattern)
        - [1.2.4. Subset pattern](#124-subset-pattern)
        - [1.2.5. Computed pattern](#125-computed-pattern)
        - [1.2.6. Bucket pattern](#126-bucket-pattern)
        - [1.2.7. Schema versioning pattern](#127-schema-versioning-pattern)
        - [1.2.8. Tree pattern](#128-tree-pattern)
        - [1.2.9. Polymorphic pattern](#129-polymorphic-pattern)
        - [1.2.10. Approximation pattern](#1210-approximation-pattern)
        - [1.2.11. Outlier pattern](#1211-outlier-pattern)
    - [1.3. Refs:](#13-refs)

<!-- /TOC -->

# 1. MongoDB

## 1.1. Relationships

### 1.1.1. Embedding

```
// person
{
  name: 'foo',
  ssn: '1234',
  addresses : [
     { street: '123 Sesame St', city: 'Anytown', cc: 'USA' },
     { street: '123 Avenue Q', city: 'New York', cc: 'USA' }
  ]
}
```

### 1.1.2. one to N

```
// parts
{
    id : 'AAAA',
    part_no : '123-aff-456',
    name : 'engine',
    price: 3.99
}

// products
{
    name : 'product 1',
    manufacturer : 'Acme Corp',
    catalog_number: 1234,
    parts : ['AAAA', 'AACD']
}
```

### 1.1.3. two-way referencing

```
// user
{
    _id: 'AAF1',
    name: 'foo bar',
    tasks [
        'ADF9',
        'AE02',
        'AE73'
    ]
}

// task
{
    _id: 'ADF9',
    description: 'fix me',
    due_date:  '2014-04-01',
    owner: 'AAF1'
}
```


## 1.2. Schema modeling patterns

### 1.2.1. Duplication

**advantages**:
- faster data access

**disadvantages**:
- may lead to correctness issues
- may lead to consistency issues

example:
shipping address in order
- when there is no duplication then updating address may lead to change the
shipping addresses in past orders
- embedding shipping address in order document resolves above issue

considerations:
1. probably no reason to use duplication if data after write wont change. In
such case its ok to use references
2. duplication can also occurs as a computed value (eg. the sum of values
stored in subdocuments or referenced documents). Carefully consider if
keeping calculated value is worth additional cost of maintenance (update sum
when subdocument is being changed)

### 1.2.2. Attribute pattern
use when you have objects with some shared attributes and some unique attributes to each of them

```
// BEFORE
// key-value pairs
{
    ...shared_attributes,
    key1: value1,
    key2: value2,
}

// AFTER
// array of objects with similar structure
// this way you can add db index to allow for searching
{
    ...shared_attributes,
    additional_attributes: [
        { key: key1, value:  value1},
        { key: key2, value: value2, some_special_attribute: foo },
    ]
}
```

**use cases:**
- characteristic of a product (many different attributes, also unique for
each product type)
- sets of fields with same value type eg. list of dates

**advantages**:
- easy to index and search
- you can add new attributes to document without worrying about search patterns
- you add additional attributes to each key-value pair
- reduces number of indexes (only one on key and value attribute names)

### 1.2.3. Extended reference pattern
when you have too many repetitive joins eg. you have one to many relationship
and whenever you access the object from many side you have to get the object
from one side. Eg. many orders, one customer - when you get order you also
need shipping address. instead of doing constant join, duplicate shipping
data from customer to order and pull customer object only when you need other
information

```
// BEFORE

// customer
{
    id: 123,
    first_name: foo,
    ...
    address1: qwe,
    address2: zxc
}

// order
// keeps reference to user and loads his data every time it's being accessed
{
    total: 1000
    customer_id: 123
}

// AFTER

// customer
{
    id: 123,
    first_name: foo,
    ...
    address1: qwe,
    address2: zxc
}

// order
// duplicates data to avoid additional join
// but still keeps the reference
{
    total: 1000
    customer_id: 123
    shipping_address: {
        address1: qwe,
        address2: zxc
    }
}

```

use case:
- catalogs
- mobile applications
- real-time analytics

**advantages**:
- reduced latency for reads (less joins)

**disadvantages**:
- duplication which have to be managed by the app code

### 1.2.4. Subset pattern
when working with data that is too big and rarely needed eg. most of the time
you need only top 10 reviews instead of all of them. Resolve the issue by
putting all of the reviews in separate collection and additionally store top 10
as a subdocument in main document

```
// BEFORE

// movie
// stores all reviews as subdocument
{
    id: 123,
    title: rambo
    ...
    reviews: [
        { author: tim, body: foo },
        ...
    ]
}

// AFTER

// movie
// stores only top 10 reviews
{
    id: 123,
    title: rambo
    ...
    reviews: [
        { author: tim, body: foo },
        ...
    ]
}

// movie_extra
// stores all reviews
{
    movie_id: 123,
    ...
    reviews: [
        { author: tim, body: foo },
        ...
    ]
}
```

**use cases:**
- list of reviews of a product
- list of comments
- list of actors in movies

**advantages**:
- smaller working set
- faster reads

**disadvantages**:
- more disk space required
- more round trip to server if more data than top 10 is needed

### 1.2.5. Computed pattern
when you need to do repeated calculations (mathematical: sum, avg, median;
fan out; roll up) producing the same result. In other words, cache results of
expensive operations

**use cases:**
- IoT
- event sourcing
- time series data
- aggregation queries

**advantages**:
- faster read queries
- saving cpu

**disadvantages**:
- may be difficult to identify
- overusing can add complexity to app

### 1.2.6. Bucket pattern
when there is so much data it's better to split them into smaller, more
manageable buckets. Middle ground between fully embedding and fully
referencing data

```
// BEFORE 1

// sensor_reads
// one document to hold all data
// so much data that document exceeds 16MB threshold
{
    id: 123,
    ...
    data: [
        { value: 123, date: some_date },
    ]
}

// BEFORE 2

// sensor_read
// document for each read stored separately
// so many documents that it's hard to manage
{
    id: 123,
    value: 123,
    date: some_date,
}

// AFTER

// sensor_read_per_day
// single document to store all data for one day
// multi-access key (id + date)
{
    id: 123,
    sensor_id: 1234, // constant value
    date: some_date,
    data: [12, 11, 12, 65, ...],
}
```

**use cases:**
- IoT
- data warehouse

**advantages**:
- makes data more manageable
- balance between data access and size of data

**disadvantages**:
- difficult to choose proper partitioning method
- adds complexity to app code

### 1.2.7. Schema versioning pattern
when you need to update schema (schema migration) but you don't want to
update all documents at once (eg you have zillions of documents or you cannot
afford downtime). Put `schema_version` field in document, make sure the app can
handle all version in its code and handle migration in app logic (choose the
most appropriate time eg when saving a document)

**advantages**:
- no downtime like in classic migration schema flow
- full control of migration process

**disadvantages**:
- might be complex
- there can be documents that won't ever migrate (legacy, abandoned accounts)

### 1.2.8. Tree pattern
when tree structures are necessary (eg hierarchies, categories)

**variants**:
- `parent reference` - the document contains parent attribute with the id of
its parent
- `child reference` - the document contains the list of all its immediate
children
- `array of ancestors` - the document contains the ancestor attribute with
ids of all nodes from root to this document
- `materialized paths` - like array of ancestors but instead of array we
store all the ancestors as a string (eg
`.Root.SubDocument1.SubSubDocument2`); look ups are achieved using regexps

you can always use a combination of patterns to achieve desired result eg ancestor array + parent reference

**use cases**:
- orgs charts
- product categories

### 1.2.9. Polymorphic pattern
when objects of different types are more similar than different and you want
to keep them in the same collections (eg different products with different
attributes are kept together in a single collection). There is a special
attribute within document that specifies concrete type. Different types are
handled properly by the application code

```
// products in a single collection

{
    id: 123,
    type: book,
    cover: hard,
}
{
    id: 124,
    type: shirt
    size: L
}
```

**use cases:**
- products catalogs
- content management systems
- single views

**advantages**:
- easy to implement
- allow to query single collection

### 1.2.10. Approximation pattern
Use approximation for cumulative data if absolute precision is not needed (eg
page views - increment counter every 10 page views instead of every time user
visit the page)

**use cases:**
- page counters
- any counters with tolerance to imprecision
- metric statistics

**advantages**:
- less writes

**disadvantages**:
- code must be handled in app
- less precision

### 1.2.11. Outlier pattern
When there are some abnormal documents (eg super large document) that are
standing out from the average. Resolve problem by implementing solution to
handle few super big documents will negatively impact majority of documents.
In other words, handle outliers differently or treat them like exceptions

eg super popular article can have million of comments where most of the
article have only a handful of them. Such popular article can be connected to
additional documents with partitioned/paged comments (comments are stored in
batches of 1000 in additional documents)

**use cases:**
- social networks (few users with zillions of followers)
- popularity apps

**advantages**:
- optimized solution for most of the documents

**disadvantages**:
- outliers are handled differently in app code
- difficult for queries


## 1.3. Refs:
- https://university.mongodb.com/courses/M320/about
- https://www.mongodb.com/blog/post/building-with-patterns-a-summary
