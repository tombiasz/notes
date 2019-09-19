<!-- TOC -->

- [1. Abstract factory](#1-abstract-factory)
  - [1.1 Code sample](#11-code-sample)
  - [1.2 Refs](#12-refs)

<!-- /TOC -->
# 1. Abstract factory

Provides an interface for creating families of related or dependent objects without specifying their concrete classes

## 1.1 Code sample
```
// products interfaces
class UsersRepository {
  getUser(id) {
    throw new Error('not implemented');
  }
}
class OrdersRepository {
  getOrders(userId) {
    throw new Error('not implemented');
  }
}

// products implementations
class PgUsersRepository extends UsersRepository {
  constructor(connection) {
    super();
    this.connection = connection;
  }

  getUser(id) {
    console.log(`executing: select * from users where id=${id}`);
    return { userId: 123 }
  }
}

class PgOrdersRepository extends OrdersRepository {
  constructor(connection) {
    super();
    this.connection = connection;
  }

  getOrders(userId) {
    console.log(`executing: select * from orders where userId=${userId}`);
    return [{ orderId: 867, userId }];
  }
}

class InMemoryUsersRepository extends UsersRepository {
  getUser(id) {
    console.log(`loading user ${id} from memory`);
    return { userId: 123 }
  }
}
class InMemoryOrdersRepository extends OrdersRepository {
  getOrders(userId) {
    console.log(`loading orders for user ${userId} from memory`);
    return [{ orderId: 867, userId }];
  }
}

// abstract factory - encapsulates a group of individual factories
class RepositoryFactory {
  users() {
    throw new Error('not implemented');
  }
  orders() {
    throw new Error('not implemented');
  }
}

// abstract factory implementations - concrete factories
class PgRepositoryFactory extends RepositoryFactory {
  constructor(config) {
    super();
    this.connection = `create db connection based on config`;
  }

  users() {
    return new PgUsersRepository(this.connection);
  }
  orders() {
    return new PgOrdersRepository(this.connection);
  }
}

class InMemoryRepositoryFactory extends RepositoryFactory {
  users() {
    return new InMemoryUsersRepository();
  }

  orders() {
    return new InMemoryOrdersRepository();
  }
}

// client code
class App {
  constructor(config = {}) {
    this.config = config;
  }

  makeRepositories() {
    return new PgRepositoryFactory(this.config);
  }
}

class TestApp {
  makeRepositories() {
    return new InMemoryRepositoryFactory();
  }
}

// client code cares only about the generic interface of the product
console.log('Pg repository')
const repos = new App().makeRepositories();
console.log(repos.users().getUser(123));
console.log(repos.orders().getOrders(123));

console.log('In memory repository')
const testRepos = new TestApp().makeRepositories();
console.log(testRepos.users().getUser(123));
console.log(testRepos.orders().getOrders(123));
```

## 1.2 Refs
- https://en.wikipedia.org/wiki/Abstract_factory_pattern
