<!-- TOC -->

- [1. Observer](#1-observer)
    - [1.1. Code sample](#11-code-sample)
    - [1.2 Refs](#12-refs)

<!-- /TOC -->
# 1. Observer

Define a one-to-many dependency between objects where a state change in one
object results in all its dependents being notified and updated automatically

## 1.1. Code sample

```javascript
class Event {
  constructor(type, context) {
    this.type = type;
    this.context = context;
  }
}

class UserPasswordChangedEvent extends Event {
  constructor(context) {
    super('UserPasswordChangedEvent', context);
  }
}

class UserPasswordResetEvent extends Event {
  constructor(context) {
    super('UserResetPasswordEvent', context);
  }
}

const isUserPasswordChangedEvent = (event) => event instanceof UserPasswordChangedEvent;
const isUserPasswordResetEvent = (event) => event instanceof UserPasswordResetEvent;

// subject
class EventSource {
  constructor() {
    this._eventHandlers = [];
  }

  attach(eventHandler) {
    this._eventHandlers.push(eventHandler)
  }

  detach(eventHandler) {
    this._eventHandlers = this._eventHandlers
      .filter(handler => handler !== eventHandler)
  }

  dispatch(event) {
    this._eventHandlers
      .forEach(handler => handler.handleEvent(event))
  }
}

// observer
class EventHandler {
  handleEvent(event) {
    console.log({ event })
  }
}

// concrete subject
// we use composition to attach EventSource to domain object
// inheritance is also a viable option here
class User {
  constructor({ username, password }) {
    this._eventSource = new EventSource();
    this.username = username;
    this.password = password;
  }

  changePassword(newPassword) {
    this.password = newPassword;

    const { username } = this;
    this._eventSource.dispatch(new UserPasswordChangedEvent({ username }));
  }

  resetPassword() {
    // do some magic
    const { username } = this;
    this._eventSource.dispatch(new UserPasswordResetEvent({ username }));
  }
}

// concrete observer
class SendEmailOnPasswordReset extends EventHandler {
  handleEvent(event) {
    if (!isUserPasswordResetEvent(event)) {
      return;
    }

    const { username } = event.context;
    console.log(`Sending a message to ${username} saying that the password has been changed`);
  }
}

// concrete observer
class InvalidateUserTokens extends EventHandler {
  handleEvent(event) {
    if (!(isUserPasswordChangedEvent(event) || isUserPasswordResetEvent(event))) {
      return;
    }

    const { username } = event.context;
    console.log(`Delete all ${username} active tokens`);
  }
}

// concrete observer
class EventLogger extends EventHandler {
  constructor({ domain }) {
    super()
    this.domain = domain;
  }
  handleEvent(event) {
    const { type, context } = event;
    const { domain } = this;

    console.log({ domain, type, context });
  }
}

const user = new User({
  username: 'foo',
  password: 'secret',
});

const sendEmailOnPasswordReset = new SendEmailOnPasswordReset();
const invalidateUserTokens = new InvalidateUserTokens();
const userEventLogger = new EventLogger({ domain: 'user' });

user._eventSource.attach(sendEmailOnPasswordReset);
user._eventSource.attach(invalidateUserTokens);
user._eventSource.attach(userEventLogger);

user.resetPassword();
user.changePassword();
```

## 1.2 Refs
- https://en.wikipedia.org/wiki/Observer_pattern
