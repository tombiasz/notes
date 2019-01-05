<!-- TOC -->

- [1. Client](#1-client)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->

# 1. Client

Use when you need a tiny wrapper around external API

## 1.1. Code sample
```
const fetch = require('fetch');

class SomeApi {
  constructor(token) {
    this.token = token
    this.baseUrl = this.constructor.baseUrl;
  }

  static get baseUrl() {
    return 'https://some.api/v1'
  }

  buildUrl(endpoint, params = '') {
    const { baseUrl } = this;
    return `${baseUrl}/${endpoint}?${params}`;
  }

  buildHeaders() {
    const { token } = this.token;
    return {
      authorization: `Bearer ${token}`,
    }
  }

  buildParams({ limit = 10, offset = 0 }) {
    return `limit=${limit}&offset=${offset}`;
  }

  parseJson(s) {
    return JSON.parse(s);
  }

  async getSomeData(params) {
    const params = this.buildParams(params);
    const url = this.buildUrl('some/data', params)
    const headers = this.buildHeaders();
    const response = await fetch(url, { method: 'GET', headers });
    return this.parseJson(response);
  }
}
```

## 1.2. Notes
- one to one mapping between methods and API endpoints
- returns exactly what came from endpoint (excluding JSON parsing or
lightweight transformations)
- client should be isolated; minimum dependencies on other parts of the system
- provide instance to allow sharing basic config between subsequent calls (eg auth)

## 1.3. Refs
- https://medium.com/selleo/essential-rubyonrails-patterns-clients-and-wrappers-c19320bcda0
