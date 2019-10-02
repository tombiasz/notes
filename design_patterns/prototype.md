<!-- TOC -->

- [1. Prototype](#1-prototype)
  - [1.1. Code sample](#11-code-sample)
  - [1.2 Refs](#12-refs)

<!-- /TOC -->
# 1. Prototype

Specify the kinds of objects to create using a prototypical instance, and create new objects from the 'skeleton' of an existing object, thus boosting performance and keeping memory footprints to a minimum

## 1.1. Code sample

```
class TestUser {
  constructor({
    firstname,
    lastname,
    age,
    email,
    status,
    // lots of other fields
  }) {
    this.firstname = firstname;
    this.lastname = lastname;
    this.age = age;
    this.email = email;
    this.status = status;
  }

  // creates new object from existing one (prototype)
  clone() {
    return new this.constructor(this);
  }
}


const user1 = new TestUser({
  firstname: 'tim',
  lastname: 'tom',
  age: 19,
  email: 'tim@tom.com',
  status: 'active',
});

const user2 = user1.clone();
user2.status = 'inactive';

console.log(user1.status);
console.log(user2.status);
```

## 1.2 Refs
- https://en.wikipedia.org/wiki/Prototype_pattern
