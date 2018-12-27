<!-- TOC -->

- [1. Form object](#1-form-object)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->
# 1. Form object

## 1.1. Code sample
```
const USERNAME_REGEX = /^([\w]+)$/;

class PostMessage {
  constructor(username, body) {
    this.username = username;
    this.body = body;
    this.success = undefined;
    this.errors = [];
  }

  process() {
    const steps = [
      this.isValid(),
      this.saveMessage(),
      this.sendNotifications(),
    ];

    this.success = steps.every(Boolean);
    return this;
    // or
    // use dedicated Service Object to process data
    // const { username, body } = this;
    // CreateMessage.call({ username, body });
  }

  isValid() {
    this.validateUsername();
    this.validateBodyMinLength();
    this.validateBodyMaxLength();
    return this.errors.length === 0;
  }

  validateUsername() {
    const valid = this.username.match(USERNAME_REGEX);
    if (!valid) {
      this.errors.push('Username can only contain alphanumeric characters');
    }
  }

  validateBodyMinLength() {
    const valid = this.body.length > 0;
    if (!valid) {
      this.errors.push('Body cannot be empty');
    }
  }

  saveMessage() {
    const { username, body } = this;
    return {
      id: 1,
      username,
      body,
      createdAt: new Date(),
    };
  }

  sendNotifications() {
    console.log('notify-users-about-new-message', this.username, this.body);
    return true;
  }

  validateBodyMaxLength() {
    const valid = this.body.length <= 140;
    if (!valid) {
      this.errors.push('Max body length is 140 characters');
    }
  }

  static process({ username, body }) {
    return new this(username, body).process();
  }
}

console.log(PostMessage.process({ username: 'foo', body: 'fizzbuzz' }));
```

## 1.2. Notes
- specialized Service Object dedicated to perform validation and persistence

## 1.3. Refs
- https://crushlovely.com/journal/7-patterns-to-refactor-javascript-applications-form-objects
