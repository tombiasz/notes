<!-- TOC -->

- [1. Policy object](#1-policy-object)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->

# 1. Policy object

Use when you need complex read operations that encapsulates one business rule

## 1.1. Code sample
```
class PublishedPostPolicy {
  constructor(post) {
    this.post = post;
  }

  check() {
    return this.isNotEmpty()
      && this.isNotDraft()
      && this.isPublished();
  }

  isNotEmpty() {
    const { title, body } = this.post;
    return title.length > 0 && body.length > 0;
  }

  isNotDraft() {
    const { isDraft } = this.post;
    return !isDraft;
  }

  isPublished() {
    const { publicationDate } = this.post;
    const now = new Date();
    return new Date(publicationDate) <= now;
  }

  static check({ post }) {
    return new this(post).check();
  }
}

const postData = {
  title: 'The title', body: 'This is post body',
  isDraft: false,
  publicationDate: 1546197258680,
};

console.log(PublishedPostPolicy.check({ post: postData }));

```

## 1.2. Notes
- can call other policy object
- performs `read only operations
- return only `true` or `false`

## 1.3. Refs
- https://codeclimate.com/blog/7-ways-to-decompose-fat-activerecord-models/#policy-objects
- https://crushlovely.com/journal/7-patterns-to-refactor-javascript-applications-policy-objects
