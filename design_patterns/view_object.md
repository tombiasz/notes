<!-- TOC -->

- [1. View object (Presenter)](#1-view-object-presenter)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->
# 1. View object (Presenter)

Use when there is a need for logic that exists purely for displaying
 (formatting) purpose

## 1.1. Code sample

```
class UserCardPresenter {
  constructor(user) {
    this.user = user;
  }

  process() {
    return {
      fullName: this.getFullName(),
      balance: this.getBalance(),
      joined: this.getJoinDate(),
    };
  }

  getFullName() {
    const { firstName, lastName } = this.user;
    return `${firstName} ${lastName}`;
  }

  getBalance() {
    const { balance, currency } = this.user;
    const money = (balance / 100.0).toFixed(2);
    return `${money} ${currency}`;
  }

  getJoinDate() {
    const { joined } = this.user;
    return new Date(joined).toISOString();
  }

  static process({ user }) {
    return new this(user).process();
  }
}

const userData = {
  id: 5432,
  firstName: 'Foo',
  lastName: 'Bar',
  balance: 12345,
  currency: 'EUR',
  joined: 1546111384687,
};

console.log(UserCardPresenter.process({ user: userData }));
```

## 1.2. Notes
- use for an API response, for passing data into a third-party service etc
- common formatting functions should be extracted into utilities
- easy testing

## 1.3. Refs
- https://crushlovely.com/journal/7-patterns-to-refactor-javascript-applications-view-objects
- https://codeclimate.com/blog/7-ways-to-decompose-fat-activerecord-models/#view-object
