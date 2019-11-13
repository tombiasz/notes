<!-- TOC -->

- [1. Singleton](#1-singleton)
    - [1.1. Code sample](#11-code-sample)
    - [1.2. Refs](#12-refs)

<!-- /TOC -->
# 1. Singleton

Ensure a class has only one instance, and provide a global point of access to it


## 1.1. Code sample

```javascript
class Logger {
  static getInstance() {
    if (this._instance) {
      return this._instance;
    }

    const instance = new this();
    this._instance = instance;
    return instance;
  }

  error({ message, context }) {
    this._logger.error(message);
  }

  info({ message, context }) {
    this._logger.info(message);
  }
}

class ConsoleLogger extends Logger {
  constructor() {
    super();
    this._logger = console;
  }
}

class NullLogger extends Logger {
  error({ message, context }) {}

  info({ message, context }) {}
}


const logger = ConsoleLogger.getInstance();
logger.error({ message: 'error' });

const logger2 = ConsoleLogger.getInstance();
logger2.info({ message: 'info' });


const logger3 = NullLogger.getInstance();
logger3.error({ message: 'will be ignored' });

const logger4 = NullLogger.getInstance();
logger3.error({ message: 'will be ignored' });


console.log(logger === logger2); // true
console.log(logger3 === logger4); // true
console.log(logger === logger3); // false
```

## 1.2. Refs
- https://en.wikipedia.org/wiki/Singleton_pattern
