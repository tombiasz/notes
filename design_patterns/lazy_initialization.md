<!-- TOC -->

- [1. Lazy initialization](#1-lazy-initialization)
  - [1.1. Code sample](#11-code-sample)
  - [1.2 Refs](#12-refs)

<!-- /TOC -->
# 1. Lazy initialization

Tactic of delaying the creation of an object, the calculation of a value, or some other expensive process until the first time it is needed

## 1.1. Code sample

```
const apiAuth = async ({ key, secret }) => {
  console.log('api auth');
  return { token: 'token '}
};
const apiCall = async data => {
  console.log('api call')
  return data;
};

class ApiService {
  constructor(config) {
    this.config = config;
  }

  token() {
    if (this._token) {
      return this._token;
    }

    this._token = (async () => await apiAuth(this.config))();
    return this._token;
  }

  async getDocument(id) {
    return apiCall({
      url: 'api-url',
      token: this.token(),
      id,
    });
  }
}

const config = { key: 'key', secret: 'secret' };
const api = new ApiService(config);
api.getDocument(1).then(console.log);
api.getDocument(2).then(console.log);
api.getDocument(3).then(console.log);
api.getDocument(4).then(console.log);
```

## 1.2 Refs
- https://en.wikipedia.org/wiki/Lazy_initialization
