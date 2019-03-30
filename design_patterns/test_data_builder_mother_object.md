## Object mother/Test Data Factory

A factory to create similar objects during tests

```
// common domain object
class User {
  constructor(firstName, lastName, role) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.role = role;
  }
}
```

```
class UsersFactory {
  static regularUser() {
    return new User('tim', 'tom', 'regular');
  }

  static adminUser() {
    return new User('jhon', 'jon', 'admin');
  }
}

console.log(TestFactory.regularUser());
console.log(TestFactory.adminUser());
```

## Test data builder

Allow for fluent approach to test object creation

```
class UserBuilder {
  static aUser() {
    return new UserBuilder();
  }

  withFirstName(firstName) {
    this.firstName = firstName;
    return this;
  }

  withLastName(lastName) {
    this.lastName = lastName;
    return this;
  }

  withRole(role) {
    this.role = role;
    return this;
  }

  withAdminRole() {
    this.role = 'admin';
    return this;
  }

  build() {
    return new User(this.firstName, this.lastName, this.role);
  }
}

console.log(UserBuilder
  .aUser()
  .withFirstName('tim')
  .withLastName('tom')
  .withAdminRole()
  .build()
);
```


## Test data builder + Object mother

Both methods combined for maximum flexibility

```
class TestUsers {
  static aRegularUser() {
    return UserBuilder
      .aUser()
      .withFirstName('tim')
      .withLastName('tom')
      .withRole('regular')
  }

  static anAdminUser() {
    return UserBuilder
      .aUser()
      .withFirstName('john')
      .withLastName('jon')
      .withAdminRole()
  }
}

console.log(
  TestUsers
    .aRegularUser()
    .build()
);
console.log(
  TestUsers
    .anAdminUser()
    .withFirstName('jerry')
    .build()
)
```

you can easily extend this approach to create nested objects
```
// assuming user has address
console.log(
  TestUsers
    .anAdminUser()
    .withFirstName('jerry')
    .withAddress(
      TestAddress
        .aNewYorkAddress()
        build()
    )
    .build()
)

```

## Refs
- https://martinfowler.com/bliki/ObjectMother.html
