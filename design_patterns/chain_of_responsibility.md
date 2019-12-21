<!-- TOC -->

- [1. Chain of responsibility](#1-chain-of-responsibility)
    - [1.1. Code sample](#11-code-sample)
    - [1.2 Refs](#12-refs)

<!-- /TOC -->
# 1. Chain of responsibility

Chain the receiving objects and pass the request along the chain until an
object handles it

## 1.1. Code sample

```javascript
const ADMIN = 'admin';
const USER = 'user';

// common handler class
class Handler {
  constructor() {
    this._next = null;
  }

  next(request) {
    if (this._next) {
      return this._next.handle(request);
    }

    throw new Error('next is not defined');
  }

  setNext(handler) {
    let last = this;

    while (last._next !== null ) {
      last = last._next
    }

    last._next = handler;
    return this;
  }

  handle(request) {
    return this._next.handle(request);
  }
}

// handler 1
class AuthHandler extends Handler {
  constructor(jwtService, userRepository) {
    super();
    this._jwtService = jwtService;
    this._userRepository = userRepository;
  }

  handle(request) {
    const token = request.header.authorization;

    if (!token) {
      throw new Error('unauthorized');
    }

    const { id: userId } = this._jwtService.decodeToken(token);
    const user = this._userRepository.getUserById(userId);
    request.loggedInUser = user;

    return this.next(request);
  }
}

// handler 2
class RoleHandler extends Handler {
  constructor(role) {
    super();
    this._role = role;
  }

  handle(request) {
    const { loggedInUser: user } = request;

    if (!user.roles.has(this._role)) {
      throw new Error('forbidden')
    }

    return this.next(request);
  }
}

// handler 3
class AdminActionHandler extends Handler {
  handle(request) {
    return `admin action handled`;
  }
}


// fake dependencies
const userRepository = {
  users: {
    1: { id: 1, username: 'foo', roles: new Set([ADMIN, USER])},
    2: { id: 2, username: 'bar', roles: new Set([USER])},
  },
  getUserById(id) {
    return this.users[id];
  }
}

const jwtService = {
  decodeToken(token) {
    return token;
  }
}

// set the chain
const requestHandler = new AuthHandler(jwtService, userRepository)
  .setNext(new RoleHandler(ADMIN))
  .setNext(new AdminActionHandler());


// client code
const request1 = { header: { authorization: { id: 1 } } };
console.log(requestHandler.handle(request1));

const request2 = { header: { authorization: { id: 2 } } };
try {
  console.log(requestHandler.handle(request2));
} catch (error) {
  console.error(error);
}

```

## 1.2 Refs
- https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern
