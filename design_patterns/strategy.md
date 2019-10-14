<!-- TOC -->

- [1. Strategy](#1-strategy)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Refs](#12-refs)

<!-- /TOC -->
# 1. Strategy

Defines a family of algorithms, encapsulate each one, and make them
interchangeable at runtime. Strategy lets the algorithm vary independently
from clients that use it.

## 1.1. Code sample

```javascript
class Fetcher {
  async get (url, options) {
    console.log('GET', url, options)
  }
}

class JwtStrategy {
  constructor(clientId, clientSecret) {
    this.clientId = clientId;
    this.clientSecret = clientSecret;
    this.transporter = transporter;
  }

  async getAccessToken() {
    const { accessToken } = await this.generateJwtToken();
    return accessToken;
  }

  async generateJwtToken() {
    // generate tokens based on clientId and secret
    return {
      accessToken: 'jwt-generated-access-token',
    }
  }
}

class UsernamePasswordStrategy {
  constructor(username, password, transporter) {
    this.username = username;
    this.password = password;
    this.transporter = transporter;
    this.url = 'some-api.cc/auth'
  }

  get authPayload() {
    return {
      body: {
        username: this.username,
        password: this.password,
      },
    };
  }

  async getAccessToken() {
    const { accessToken } = await this.auth();
    return accessToken;
  }

  auth() {
    const { username, password } = this;
    // authorize using username and password
    return this.transporter
      .get(this.url, this.authPayload)
      .then(() => ({
        accessToken: 'username-password-generated-access-token',
      }));
  }
}

class ApiClient {
  constructor(authorizer, transporter) {
    this.authorizer = authorizer;
    this.transporter = transporter;
  }

  async get(url) {
    return this.transporter.get(
      url,
      {
        headers: {
          ...await this.authHeader()
      }
    })
  }

  async authHeader() {
    const accessToken = await this.authorizer.getAccessToken();
    return {
      'Authorization': `Bearer: ${accessToken}`,
    }
 }
}

const transporter = new Fetcher();

// select desired strategy at runtime
const jwtAuth = new JwtStrategy('clientId', 'clientSecret');
const jwtApi = new ApiClient(jwtAuth, transporter);
jwtApi.get('some-url')

const passAuth = new UsernamePasswordStrategy('username', 'password', transporter);
const passApi = new ApiClient(passAuth, transporter);
passApi.get('some-url')
```

## 1.2. Refs
- https://en.wikipedia.org/wiki/Strategy_pattern
