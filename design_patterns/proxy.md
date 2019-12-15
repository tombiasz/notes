<!-- TOC -->

- [1. Proxy](#1-proxy)
    - [1.1. Code sample](#11-code-sample)
    - [1.3. Refs](#13-refs)

<!-- /TOC -->

# 1. Proxy

Provide a substitute or placeholder for another object to control access to it

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

// proxy
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

## 1.3. Refs
- https://en.wikipedia.org/wiki/Proxy_pattern
