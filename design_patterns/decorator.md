<!-- TOC -->

- [1. Decorator](#1-decorator)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->

# 1. Decorator

Use when you need to extend the behavior of the class and the inheritance does
not feel right

## 1.1. Code sample

classic approach
```javascript
const userDb = {
  1: {
    firstname: 'foo',
    lastname: 'bar',
  },
  2: {
    firstname: 'fizz',
    lastname: 'buzz,'
  }
};

class UserRepository {
  constructor(db, logger) {
    this._db = db;
    this._logger = logger;
  }

  getUser(id) {
    this._logger.info(`loading user ${id} from db`);
    const record = this._db[id];

    if (!record) {
      this._logger.error(`user ${id} not found`);
      throw new Error('user not found')
    }

    return record;
  }

  updateUser(id, { firstname, lastname }) {
    this.getUser(id);

    this._logger.info(`updating user ${id}`);
    const newRecord = { firstname, lastname };
    this._db[id] = newRecord;

    return true;
  }
}

// decorator
class CachedUserRepository {
  constructor(userRepository, logger) {
    this._userRepository = userRepository;
    this._logger = logger;
    this._cache = {};
  }

  getUser(id) {
    this._logger.info(`loading user ${id} from cache`);
    const cached = this._cache[id];

    if (cached) {
      this._logger.info(`cache hit: user ${id}`);
      return cached;
    }

    this._logger.info(`cache miss: user ${id}`);
    const record = this._userRepository.getUser(id);
    this._cache[id] = record;
    return record;
  }

  updateUser(id, { firstname, lastname }) {
    this._logger.info(`user ${id} marked as dirty`);
    delete this._cache[id];
    return this._userRepository.updateUser(id, { firstname, lastname });
  }
}

const userRepo = new UserRepository(userDb, console);
console.log(userRepo.getUser(1));

const decoratedRepo = new CachedUserRepository(userRepo, console);
console.log(decoratedRepo.getUser(2));
console.log(decoratedRepo.getUser(2));
console.log(decoratedRepo.getUser(2));

decoratedRepo.updateUser(2, { firstname: 'tim', lastname: 'tom' });
console.log(decoratedRepo.getUser(2));
console.log(decoratedRepo.getUser(2));
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
