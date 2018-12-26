<!-- TOC -->

- [1. Service object](#1-service-object)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->
# 1. Service object

## 1.1. Code sample
```
class RegisterUser {
  constructor(username, type) {
    this.username = username;
    this.type = type;
    this.success = undefined;
    this.errors = [];
  }

  call() {
    if (!this.username) {
      return false;
    }

    try {
      this.user = this.createUser();
      this.sendRegistrationEmail();
    } catch (err) {
      this.errors.push(err);
    }
    this.success = this.errors.length === 0;
    return this;
  }

  createUser() {
    const { username, type } = this;
    // throw new Error('no user');
    return { id: 1, username, type };
  }

  sendRegistrationEmail() {
    console.log('user-registered', this.user);
  }

  static call({ username, type }) {
    return new this(username, type).call();
  }
}

console.log(RegisterUser.call({ username: 'foo', type: 'bar' }));

```

## 1.2. Notes
- encapsulates single domain operation/business process (one responsibility)
- single `call`/`process`/`handle`/`run` method
- avoid direct instantiation; use static call instead
- use simple constructor (just setters; no method calling)
- `call` can return
  - `true/false` if it's enough
  - state reader - object itself if there is a need to check internal state as shown above
  - can `throw` exception directly
- easy testing

## 1.3. Refs
- https://medium.com/selleo/essential-rubyonrails-patterns-part-1-service-objects-1af9f9573ca1
- https://crushlovely.com/journal/7-patterns-to-refactor-javascript-applications-service-objects
- https://codeclimate.com/blog/7-ways-to-decompose-fat-activerecord-models/#service-objects
