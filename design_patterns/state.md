<!-- TOC -->

- [1. State](#1-state)
    - [1.1. Code sample](#11-code-sample)
    - [1.2. Refs](#12-refs)

<!-- /TOC -->
# 1. State
Allows an object to alter its behavior when its internal state changes. The object will appear to change its class.

## 1.1. Code sample

classic approach
```javascript
class State {
  constructor(name) {
    this._name = name;
  }

  toString() {
    return `${this._name}`;;
  }
}

class AccountState extends State {
  login(context, username, password) {
    throw new Error(`login operation not supported in state ${this}`);
  }

  resetPassword(context, newPassword) {
    throw new Error(`resetPassword operation not supported in state ${this}`);
  }

  delete(context) {
    throw new Error(`delete operation not supported in state ${this}`);
  }
}

class ActiveAccountState extends AccountState {
  constructor() {
    super('ACTIVE');
  }

  login(context, username, password) {
    if (context._username === username && context._password === password) {
      return true;
    }

    context._failedLoginAttempt += 1;

    if (context._failedLoginAttempt >= 3) {
      context._blockedAt = Date.now();
      context._state = new BlockedAccountState();
    }

    return false;
  }

  resetPassword(context, resetPassword) {
    context._password = newPassword;
    context._blockedAt = null;
    return true;
  }

  delete(context) {
    context._deletedAt = Date.now();
    context._state = new DeletedAccountState();
  }
}

class BlockedAccountState extends AccountState {
  constructor() {
    super('BLOCKED');
  }

  resetPassword(context, newPassword) {
    context._password = newPassword;
    context._blockedAt = null;
    context._state = new ActiveAccountState();
    return true;
  }
}

class DeletedAccountState extends AccountState {
  constructor() {
    super('DELETED')
  }
}

class Account {
  constructor({
    id,
    username,
    password,
    failedLoginAttempt,
    blockedAt,
    deletedAt,
    state,
  }) {
    this._id = id;
    this._username = username;
    this._password = password;
    this._failedLoginAttempt = failedLoginAttempt;
    this._blockedAt = blockedAt;
    this._deletedAt = deletedAt;
    this._state = state;
  }

  get state() {
    return this._state.toString();
  }

  login(username, password) {
    this._state.login(this, username, password);
  }

  resetPassword(oldPassword, newPassword) {
    this._state.resetPassword(this, oldPassword, newPassword);
  }

  delete() {
    this._state.delete(this);
  }
}

const accountObj = {
  id: 1234,
  username: 'foo',
  password: 'bar',
  failedLoginAttempt: 0,
  blockedAt: null,
  deletedAt: null,
  state: new ActiveAccountState(),
}

const account = new Account(accountObj);
console.log({ state: account.state });

account.login('foo', 'bar');
console.log({ state: account.state });

account.login('foo', 'notbar');
account.login('foo', 'notbar');
account.login('foo', 'notbar');
console.log({ state: account.state });

try {
  account.login('foo', 'bar');
} catch (err) {
  console.error(err.message);
}

account.resetPassword('newbar');
console.log({ state: account.state });

account.login('foo', 'newbar');
console.log({ state: account.state });

account.delete()
console.log({ state: account.state });

try {
  account.login('foo', 'newbar');
} catch (err) {
  console.error(err.message);
}
```

decoupled approach
```javascript
const assert = require('assert');

class Condition {
  constructor(condition) {
    this._condition = condition;
  }

  toString() {
    return `${this._condition}`;
  }

  equal(other) {
    return this === other;
  }
}

class Transition {
  constructor(from, condition, to) {
    assert(from instanceof State, 'from must be an instance of State');
    assert(to instanceof State, 'to must be an instance of State');
    assert(condition instanceof Condition, 'condition must be an instance of Condition');

    this._from = from;
    this._condition = condition;
    this._to = to;
  }

  get to() {
    return this._to;
  }

  get condition() {
    return this._condition;
  }

  get from() {
    return this._from;
  }
}

class StateMachine {
  constructor(start, transitions) {
    assert(start instanceof State, 'start must be an instance of State')
    assert(
      transitions.every(o => o instanceof Transition),
      'transitions must be list of transitions'
    );

    this._current = start;
    this.transitions = transitions;
  }

  get current() {
    return this._current;
  }

  apply(condition) {
    return this.getNextState(condition);
  }

  getNextState(condition) {
    for (const transition of this.transitions) {
      const isCurrent = transition.from.equal(this.current);
      const match = transition.condition.equal(condition);

      if (isCurrent && match) {
        return transition.to;
      }
    }

    return this.current;
  }
}

class State {
  constructor(name) {
    this._name = name;
  }

  toString() {
    return `${this._name}`;;
  }

  equal(other) {
    return this === other;
  }
}

class AccountState extends State {
  login(context, username, password) {
    throw new Error(`login operation not supported in state ${this}`);
  }

  resetPassword(context, newPassword) {
    throw new Error(`resetPassword operation not supported in state ${this}`);
  }

  delete(context) {
    throw new Error(`delete operation not supported in state ${this}`);
  }
}

class ActiveAccountState extends AccountState {
  constructor() {
    super('ACTIVE');
  }

  login(context, username, password) {
    if (context._username === username && context._password === password) {
      return true;
    }

    context._failedLoginAttempt += 1;

    if (context._failedLoginAttempt >= 3) {
      context._blockedAt = Date.now();
      context._state = context._stateMachine.apply(blockAccount);
    }

    return false;
  }

  resetPassword(context, newPassword) {
    context._password = newPassword;
    context._blockedAt = null;
    return true;
  }

  delete(context) {
    context._deletedAt = Date.now();
    context._state = context._stateMachine.apply(deleteAccount);
    return true;
  }
}

class BlockedAccountState extends AccountState {
  constructor() {
    super('BLOCKED');
  }

  resetPassword(context, newPassword) {
    context._password = newPassword;
    context._blockedAt = null;
    context._state = context._stateMachine.apply(activateAccount);
    return true;
  }
}

class DeletedAccountState extends AccountState {
  constructor() {
    super('DELETED')
  }
}

class Account {
  constructor({
    id,
    username,
    password,
    failedLoginAttempt,
    blockedAt,
    deletedAt,
    state,
  }) {
    this._id = id;
    this._username = username;
    this._password = password;
    this._failedLoginAttempt = failedLoginAttempt;
    this._blockedAt = blockedAt;
    this._deletedAt = deletedAt;
    this._state = state;
    this._stateMachine = new StateMachine(activeAccountState, transitions);
  }

  get state() {
    return this._state.toString();
  }

  login(username, password) {
    this._state.login(this, username, password);
  }

  resetPassword(oldPassword, newPassword) {
    this._state.resetPassword(this, oldPassword, newPassword);
  }

  delete() {
    this._state.delete(this);
  }
}

const activateAccount = new Condition('activateAccount');
const blockAccount = new Condition('blockAccount');
const deleteAccount = new Condition('deleteAccount');

const activeAccountState = new ActiveAccountState();
const blockedAccountState = new BlockedAccountState();
const deletedAccountState = new DeletedAccountState();

const transitions = [
  new Transition(activeAccountState, blockAccount, blockedAccountState),
  new Transition(activeAccountState, deleteAccount, deletedAccountState),
  new Transition(blockedAccountState, activateAccount, activeAccountState),
];

const accountObj = {
  id: 1234,
  username: 'foo',
  password: 'bar',
  failedLoginAttempt: 0,
  blockedAt: null,
  deletedAt: null,
  state: new ActiveAccountState(),
}

const account = new Account(accountObj);
console.log({ state: account.state });

account.login('foo', 'bar');
console.log({ state: account.state });

account.login('foo', 'notbar');
account.login('foo', 'notbar');
account.login('foo', 'notbar');
console.log({ state: account.state });

try {
  account.login('foo', 'bar');
} catch (err) {
  console.error(err.message);
}

account.resetPassword('newbar');
console.log({ state: account.state });

account.login('foo', 'newbar');
console.log({ state: account.state });

account.delete()
console.log({ state: account.state });

try {
  account.login('foo', 'newbar');
} catch (err) {
  console.error(err.message);
}
```

## 1.2. Refs
- https://en.wikipedia.org/wiki/State_pattern
