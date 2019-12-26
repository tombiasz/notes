<!-- TOC -->

- [1. Command](#1-command)
    - [1.1. Code sample](#11-code-sample)
    - [1.2. Refs](#12-refs)

<!-- /TOC -->
# 1. Command


## 1.1. Code sample

```javascript
class Command {
  execute() {
    throw new Error('not implemented');
  }
}

class ValidateUsername extends Command {
  execute({ username }) {
    if (username.trim() !== '') {
      return true;
    }
    return false;
  }
}

class SendMessage extends Command {
  constructor({ messageService }) {
    super();
    this._messageService = messageService;
  }

  execute({ username }) {
    console.log(`sending welcome message to ${username} using message service`);
  }
}

class CreateUser extends Command {
  constructor({ userRepository }) {
    super();
    this._userRepository = userRepository;
  }

  execute({ username }) {
    console.log(`storing user ${username} using repository`);
  }
}

class RegisterUser extends Command {
  constructor({ createUser, sendMessage, validateUsername }) {
    super();
    this._createUser = createUser;
    this._sendMessage = sendMessage;
    this._validateUsername = validateUsername;
  }

  _ensureUsername({ username }) {
    const isValid = this._validateUsername.execute({ username });

    if (!isValid) {
      throw new Error('username is not valid');
    }

    return username;
  }

  execute({ username: _username }) {
    const username = this._ensureUsername({ username: _username });
    this._createUser.execute({ username });
    this._sendMessage.execute({ username });
  }
}

const fakeUserRepository = {};
const fakeNotificationService = {};

const validateUsername = new ValidateUsername();
const sendMessage = new SendMessage({ messageService: fakeNotificationService });
const createUser = new CreateUser({ userRepository: fakeUserRepository });
const registerUser = new RegisterUser({ validateUsername, createUser, sendMessage });

registerUser.execute({ username: 'foobar '});

```

## 1.2. Refs
- https://en.wikipedia.org/wiki/Command_pattern
