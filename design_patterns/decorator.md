<!-- TOC -->

- [1. Decorator](#1-decorator)
    - [1.1. Code sample](#11-code-sample)
    - [1.2. Notes](#12-notes)
    - [1.3. Refs](#13-refs)

<!-- /TOC -->

# 1. Decorator

Use when you need to extend the behavior of the class on runtime or the
inheritance does not feel right

## 1.1. Code sample

classic approach
```javascript
const ERROR = 'error';
const WARNING = 'warning';
const INFO = 'info';

// domain object
class Alert {
  constructor(level, message) {
    this.level = level;
    this.message= message;
  }

  isError() {
    return this.level === ERROR;
  }

  isWarning() {
    return this.level === WARNING || this.level === ERROR;
  }
}

// component interface
class BaseAlertService {
  register(alert) {
    throw new Error('not implemented')
  }
}

// component implementation
class AlertService extends BaseAlertService {
  constructor(alertRepository) {
    super();
    this.alertRepository = alertRepository;
  }

  register(alert) {
    return this.alertRepository.save(alert);
  }
}

// decorator interface
class AlertServiceDecorator {
  register(alert) {
    throw new Error('not implemented');
  }
}

// decorator implementation
class EmailAlertServiceDecorator extends AlertServiceDecorator {
  constructor(alertService, emailService) {
    super();
    this.alertService = alertService;
    this.emailService = emailService;
  }

  register(alert) {
    const result = this.alertService.register(alert);
    this._registerEmail(result);
    return result;
  }

  _formatEmail(alert) {
    return {
      subject: `[${alert.level}]: ${alert.message}`.slice(0, 20),
      body: alert.message,
    };
  }

  _registerEmail(alert) {
    this.emailService.register(this._formatEmail(alert));
  }
}

// decorator implementation
class SMSAlertServiceDecorator extends AlertServiceDecorator {
  constructor(alertService, smsService) {
    super();
    this.alertService = alertService;
    this.smsService = smsService;
  }

  get messageMaxSize() {
    return 160;
  }

  register(alert) {
    const result = this.alertService.register(alert);
    this._registerSms(result);
    return result;
  }

  _formatMessage(alert) {
    return `${alert.level} ${alert.message}`.slice(0, this.messageMaxSize);
  }

  _registerSms(alert) {
    this.smsService.register({ message: this._formatMessage(alert) });
  }
}

const alertRepository = {
  save(alert) {
    console.log('Storing alert in DB', alert);
    return alert;
  }
}

const emailService = {
  register({ subject, body }) {
    console.log('Sending alert via email', subject, body);
  }
}

const smsService = {
  register({ message }) {
    console.log('Sending alert via sms', message);
  }
}

// you can extend object on runtime
const alertHandler = (alert) => {
  let alertService = new AlertService(alertRepository);

  if (alert.isWarning()) {
    alertService = new EmailAlertServiceDecorator(alertService, emailService);
  }

  if (alert.isError()) {
    alertService = new SMSAlertServiceDecorator(alertService, smsService);
  }

  return alertService.register(alert);
}

const error = new Alert(ERROR, 'foo bar');
const warning = new Alert(WARNING, 'fizz buzz');
const info = new Alert(INFO, 'tim tom');

console.log(alertHandler(info));
console.log(alertHandler(warning));
console.log(alertHandler(error));
```

decorator for command-like object
```javascript
// simple service object
class ActivateUser {
  constructor(user) {
    this.user = user;
  }

  call() {
    // call repository to update user record
    return { ...this.user, active: true };
  }

  static call({ user }) {
    return new this(user).call();
  }
}

class CreateUser {
  constructor(username) {
    this.username = username;
  }

  call() {
    // call repository to insert user record
    return { id: 1, username: this.username };
  }

  static call({ username }) {
    return new this(username).call();
  }
}

// decorator
class PreloadUserCompany {
  constructor(userServiceObject) {
    this.userServiceObject = userServiceObject;
    this.methodToDecorate = 'call';
    this.decorate();
    return userServiceObject;
  }

  decorate() {
    const originalFn = this.userServiceObject[this.methodToDecorate];

    function decoratedFn(...args) {
      const user = originalFn.call(this.userServiceObject, ...args);
      return this.preloadUserCompany(user);
    }

    this.userServiceObject[this.methodToDecorate] = decoratedFn.bind(this);
    return this;
  }

  preloadUserCompany(user) {
    return {
      ...user,
      company: { id: 1, name: 'fizzbuzz llc'}
    };
  }
}

const username = 'foobar';
const userData = { id: 1, username };

// decorate via instance method
console.log(new PreloadUserCompany(new CreateUser(username)).call());
console.log(new PreloadUserCompany(new ActivateUser(userData)).call());

// decorate via static function
console.log(new PreloadUserCompany(CreateUser).call({ username }));
console.log(new PreloadUserCompany(ActivateUser).call({ user: userData }));

```

## 1.2. Notes
- adds responsibility to existing object
- returns original object with altered property/method
- you can chain as many decorators as you want

## 1.3. Refs
- https://crushlovely.com/journal/7-patterns-to-refactor-javascript-applications-decorators
- https://codeclimate.com/blog/7-ways-to-decompose-fat-activerecord-models/#decorators
