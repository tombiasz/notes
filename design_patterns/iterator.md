<!-- TOC -->

- [1. Iterator](#1-iterator)
    - [1.1. Code sample](#11-code-sample)
    - [1.2 Refs](#12-refs)

<!-- /TOC -->
# 1. Iterator

Provide a way to access the elements of an aggregate object sequentially
without exposing its underlying representation.

## 1.1. Code sample

```javascript
class StopIteration extends Error {}
class EmailAlreadyExists extends Error {}

class DomainObject {}

class Email extends DomainObject {
  constructor({ emailAddress, isPrimary }) {
    super();
    this.emailAddress = emailAddress;
    this.isPrimary = isPrimary;
  }

  removePrimaryMark() {
    this.isPrimary = false;
  }
}

// iterator with additional domain logic
class EmailsCollection extends DomainObject {
  constructor() {
    super();
    this._emails = [];
    this._current = 0;
  }

  hasNext() {
    return this._current < this.size();
  }

  next() {
    if (this.hasNext()) {
      const current = this._emails[this._current];
      this._current += 1;
      return current;
    }

    throw new StopIteration();
  }

  resetIteration() {
    this._current = 0;
  }

  addEmail(email) {
    const exists = this.findEmailByEmailAddress(email);
    if (exists) {
      throw new EmailAlreadyExists();
    }

    if (email.isPrimary) {
      this.resetPrimaryEmails();
    }

    this._emails.push(email);
    return this;
  }

  removeEmail({ emailAddress }) {
    this._emails = this._emails.filter(e => e.emailAddress !== emailAddress);
    return this;
  }

  resetPrimaryEmails() {
    this._emails.forEach(o => o.removePrimaryMark());
    return this;
  }

  findEmailByEmailAddress({ emailAddress }) {
    return this._emails.find(e => e.emailAddress === emailAddress);
  }

  size() {
    return this._emails.length;
  }
}


const email1 = new Email({ emailAddress: 'foo@bar', isPrimary: true });
const email2 = new Email({ emailAddress: 'fizz@buzz', isPrimary: true });

const iterator = new EmailsCollection();
iterator.addEmail(email1);
iterator.addEmail(email2);

try {
  while (iterator.hasNext()) {
    console.log(iterator.next());
  }
} catch (error) {
  if (!(error instanceof StopIteration)) {
    console.error(error);
  }
}

try {
  iterator.addEmail(email1);
} catch (error) {
  console.log(error);
}

iterator.removeEmail(email1);

iterator.resetIteration();
try {
  while (iterator.hasNext()) {
    console.log(iterator.next());
  }
} catch (error) {
  if (!(error instanceof StopIteration)) {
    console.error(error);
  }
}

```

## 1.2 Refs
- https://en.wikipedia.org/wiki/Iterator_pattern
