<!-- TOC -->

- [1. Facade](#1-facade)
    - [1.1. Code sample](#11-code-sample)
    - [1.2 Refs](#12-refs)

<!-- /TOC -->
# 1. Facade

Defines a higher-level interface that makes the subsystem easier to use.

## 1.1. Code sample

```javascript
const express = require('express');

const logger = () => console;

const invalidJsonErrorHandler = () => (err, req, res, next) => {
  return err instanceof SyntaxError
    ? res.status(400).send('invalid json')
    : next(err);
}

const generalErrorHandler = () => (err, req, res, next) => {
  return res.status(500).send('oops... something went wrong ;(');
}

const errorHandlers = () => [
  invalidJsonErrorHandler(),
  generalErrorHandler(),
];

// facade
class App {
  constructor() {
    this._server = express();
    this._logger = logger();
    this._errorHandlers = errorHandlers();
  }

  _attachLogger() {
    const { _logger } = this;
    this._server.use((req) => req.logger = _logger)
  }

  _attachErrorHandlers() {
    const { _errorHandlers, _server } = this;
    _errorHandlers.forEach(h => _server.use(h));
  }

  start(port) {
    this._attachLogger();
    this._attachErrorHandlers();

    return new Promise((resolve, reject) => this._server
      .listen(8000, (err) => (err ? reject(err) : resolve(port)))
    );
  }
}

new App()
  .start(8000)
  .then((port) => console.log(`server running on ${port}`))
  .catch((err) => console.error('ohh no', err))
```

## 1.2 Refs
- https://en.wikipedia.org/wiki/Facade_pattern
