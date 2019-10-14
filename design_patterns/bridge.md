<!-- TOC -->

- [1. Bridge](#1-bridge)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Refs](#12-refs)

<!-- /TOC -->
# 1. Bridge

Decouples an abstraction from its implementation allowing the two to vary independently

## 1.1. Code sample

```javascript
class EmailMessage {
  constructor(sender) {
    this.sender = sender;
  }

  send({ from, to, subject, body }) {
    return this.sender.send(from, to , subject, body);
  }
}

class RegistrationEmailMessage extends EmailMessage {
  get subject() {
    return 'Registration email';
  }

  get body() {
    return 'Please, confirm your email';
  }

  send({ from, to }) {
    return this.sender.send(from, to , this.subject, this.body);
  }
}


class BaseMessageSender {
  send(from, to, title, body) {
    throw new Error('Implement in subclass')
  }
}

class EmailSender extends BaseMessageSender {
  constructor(credentials) {
    super();
    this.credentials = credentials;
  }

  send(from, to, title, body) {
    // send a real email using credentials
    console.log('sending email:', { from, to, title, body })
    return `email message sent`;
  }
}

class EmailSenderMock extends BaseMessageSender {
  send(from, to, title, body) {
    return {
      from,
      to,
      title,
      body,
    }
  }
}


const data = {
  from: 'foo@bar',
  to: 'fizz@buzz',
  subject: 'Title',
  body: 'Body',
}

// in app
const sender = new EmailSender({ addr: 'email-hostname.io'});
const emailMsg = new EmailMessage(sender);
console.log(emailMsg.send(data));

const regEmailMsg = new RegistrationEmailMessage(sender);
console.log(regEmailMsg.send(data));

// in tests
const senderMock = new EmailSenderMock();
const testEmailMsg = new RegistrationEmailMessage(senderMock);
const testResult = testEmailMsg.send(data);
console.log(testResult)

assert(testResult.from === 'foo@bar');
assert(testResult.to === 'fizz@buzz');
assert(testResult.title === 'Registration email');
assert(testResult.body === 'Please, confirm your email');
```

## 1.2. Refs
- https://en.wikipedia.org/wiki/Bridge_pattern
