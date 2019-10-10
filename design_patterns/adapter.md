<!-- TOC -->

- [1. Adapter](#1-adapter)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Refs](#12-refs)

<!-- /TOC -->
# 1. Adapter

Convert the interface of a class into another interface clients expect. It is
used to make existing classes work with others without modifying their source
code.

## 1.1. Code sample

```javascript
// Initially client code depends on target class
// this code is repeated many times in the code base
function printUsers(usersDataProvider) {
  usersDataProvider
    .getUsers(100)
    .then(console.log)
}


// initial implementation using database
class QueryBuilder {
  constructor(connectionString) {
    this.connectionString = connectionString;
  }

  // builder methods implementation
  table() { return this }
  select() { return this }
  limit() { return this }
  offset() { return this }

  async build() {
    // 1. build sql query
    // 2. connect to db using this.connectionString
    // 3. execute query and return promise with result
    return 'a list of users from repository';
  }
}

// Target
// this interface that will be preserved
class UserRepository {
  constructor(queryBuilder) {
    this.qb = queryBuilder;
  }

  getUsers(limit = 10, offset = 0) {
    return this.qb
      .table('users')
      .select('*')
      .limit(limit)
      .offset(offset)
      .build();
  }
}

// client code
// 1. repo instantiation
const queryBuilder = new QueryBuilder(process.env.DB_CONNECTION_STRING)
const users = new UserRepository(queryBuilder);
// 2. repo usage later in the code
printUsers(users);


// new requirements - use api instead of db repositories, but you can't change
// the api interface to match repo, because it's a 3rd party code
// Solution: use adapter pattern
class ApiClient {
  constructor(clientId, clientSecret) {
    this.clientId = clientId;
    this.clientSecret = clientSecret;
  }

  async getToken() {
    // authorize and return promise with token
    return 'token'
  }

  fetch() { return { data: {} }; }

  async get(url) {
    const token = await this.getToken();

    return this.fetch(
      'GET',
      url,
      { headers: { 'Authorization': `Bearer: ${token}` }},
    );
  }
}

// Adaptee
// this class will be adapted to our interface
class UsersApi {
  constructor(client) {
    this.client = client;
    this.baseUrl = 'https://users.micro/users';
  }

  async listUsers(limit = 10, offset = 0) {
    const { data } = await this.client
      .get(`${this.baseUrl}?limit=${limit}&offset=${offset}`)

    return 'a list of users from api';
  }
}

// Adapter
// same interface as target (UsersRepository)
// delegates work to delegate (UsersApi)
class UsersApiToUsersRepositoryAdapter {
  constructor(usersApi) {
    this.usersApi = usersApi;
  }

  getUsers(limit, offset) {
    return this.usersApi.listUsers(limit, offset)
  }
}

// now you can replace old code with new one
// 1. api instantiation
const apiClient = new ApiClient(
  process.env.API_CLIENT_ID,
  process.env.API_CLIENT_SECRET,
);
const usersApi = new UsersApi(apiClient);
// 2. api adaptation
const usersAdapted = new UsersApiToUsersRepositoryAdapter(usersApi);
// 3. same usage as before but different implementations
printUsers(usersAdapted);
```

## 1.2. Refs
- https://en.wikipedia.org/wiki/Adapter_pattern
