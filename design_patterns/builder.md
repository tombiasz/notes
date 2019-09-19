<!-- TOC -->

- [1. Builder](#1-builder)
  - [1.1. Code sample](#11-code-sample)
  - [1.2 Refs](#12-refs)

<!-- /TOC -->
# 1. Builder

Separates the construction of a complex object from its representation

## 1.1. Code sample

```
const fetchImpl = v => Promise.resolve(v);

class OrdersApiWrapper {
  constructor(params = []) {
    this.url = 'https://some.api/orders';
    this.params = params;
  }

  static builder() {
    return new OrdersApiWrapperBuilder();
  }

  fetch() {
    const { url, params } = this;
    return fetchImpl(`${url}?${params.join('&')}`);
  }
}

class OrdersApiWrapperBuilder {
  constructor() {
    this.params = [];
  }

  _addParam(param) {
    this.params.push(param);
    return this;
  }

  createdByUsers(...ids) {
    this._addParam(`user=${ids.join(',')}`);
    return this;
  }

  createdWithin(min, max) {
    this._addParam(`createdAt=${min},${max}`);
    return this;
  }

  withDetails() {
    this._addParam('details=1');
    return this;
  }

  build() {
    return new OrdersApiWrapper(this.params);
  }
}

const api = OrdersApiWrapper
  .builder()
  .createdByUsers(123, 345, 11231)
  .createdWithin('2019-07-01', '2019-08-01')
  .withDetails()
  .build();

api.fetch().then(console.log);
```

## 1.2 Refs
- https://en.wikipedia.org/wiki/Builder_pattern
