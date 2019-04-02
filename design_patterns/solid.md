<!-- TOC -->

- [1. SOLID](#1-solid)
  - [1.1. Single responsibility principle](#11-single-responsibility-principle)
  - [1.2. Open/close principle](#12-openclose-principle)
  - [1.3. Liskov substitution principle](#13-liskov-substitution-principle)
  - [1.4. Interface segregation principle](#14-interface-segregation-principle)
  - [1.5. Dependency inversion principle](#15-dependency-inversion-principle)
  - [1.6. Refs](#16-refs)

<!-- /TOC -->

# 1. SOLID

## 1.1. Single responsibility principle

A class should have only a single responsibility

```
// Bad
// the class is unnecessarily responsible for storing user data and validation
class Person {
  public username: string;
  public email: string;

  constructor(username, email) {
    this.username = username;

    if (this.validateEmail(email)) {
      throw new Error('email is not valid');
    }
    this.email = email;
  }

  validateEmail(email) {
    // silly validation
    return email.indexOf('@') !== -1
  }
}
```

```
// Better
// move validation to separate class eg ValueObject
class Email {
  public email: string;

  constructor(email) {
    if (this.validateEmail(email)) {
      throw new Error('email is not valid');
    }
    this.email = email;
  }

  validateEmail(email) {
    // silly validation
    return email.indexOf('@') !== -1
  }
}

class Person {
  public username: string;
  public email: Email;

  constructor(username, email) {
    this.username = username;
    this.email = email;
  }
}

const username = 'tim';
const email = new Email('tim@tom.com');
const p = new Person(username, email)
```

## 1.2. Open/close principle

Software should be open for extension, but closed for modification.

```
// Bad
// adding a new class requires unexpected changed in getArea function
class Rectangle {
    public width: number;
    public height: number;
}

class Circle {
    public radius: number;
}

function getArea(shapes: (Rectangle|Circle)[]) {
    return shapes.reduce(
        (previous, current) => {
            if (current instanceof Rectangle) {
                return current.width * current.height;
            } else if (current instanceof Circle) {
                return current.radius * current.radius * Math.PI;
            } else {
                throw new Error("Unknown shape!")
            }
        },
        0
    );
}
```

```
// Better
// polymorphism allows extensions without modifications
class Rectangle {
    public width: number;
    public height: number;

    constructor(width, height) {
      this.width = width;
      this.height = height;
    }

    public area() {
        return this.width * this.height;
    }
}

class Circle {
    public radius: number;

    constructor(radius) {
      this.radius = radius;
    }

    public area() {
        return this.radius * this.radius * Math.PI;
    }
}

function getArea(shapes: Array<Rectangle|Circle>) {
    return shapes.reduce(
        (previous, current) => previous + current.area(),
        0
    );
}

const r= new Rectangle(10, 20);
const c = new Circle(3);
getArea([r, c, c])
```

## 1.3. Liskov substitution principle

Objects in a program should be replaceable with instances of their subtypes
without altering the correctness of that program

```
// Same sample us above, but this time we are using a common interface
// and allow getArea to take any object that adheres to common interface

interface Shape {
  area(): number
}

class Rectangle implements Shape {

    public width: number;
    public height: number;
    constructor(width, height) {
      this.width = width;
      this.height = height;
    }

    public area() {
        return this.width * this.height;
    }
}

class Circle implements Shape {

    public radius: number;

    constructor(radius) {
      this.radius = radius;
    }

    public area() {
        return this.radius * this.radius * Math.PI;
    }
}

function getArea(shapes: Array<Shape>) {
    return shapes.reduce(
        (previous, current) => previous + current.area(),
        0
    );
}

const r= new Rectangle(10, 20);
const c = new Circle(3);
getArea([r, c, c])
```

## 1.4. Interface segregation principle

No client should be forced to depend on methods it does not use. Many
client-specific interfaces are better than one general-purpose interface.

```
// Many interfaces to split the behaviors
interface RectangleInterface {
    width: number;
    height: number;
}

interface CircleInterface {
    radius: number;
}

interface Shape {
    area(): number;
}

interface Serializable {
    serialize(): string;
}

class Rectangle implements RectangleInterface, Shape {

    public width: number;
    public height: number;

    public area() {
        return this.width * this.height;
    }
}

class Circle implements CircleInterface, Shape {

    public radius: number;

    public area() {
        return this.radius * this.radius * Math.PI;
    }
}

function getArea(shapes: Shape[]) {
    return shapes.reduce(
        (previous, current) => previous + current.area(),
        0
    );
}

// similar to Shape but different behavior
class RectangleSerializer implements RectangleInterface, Serializable {

    public width: number;
    public height: number;

    public serialize() {
        return JSON.stringify(this);
    }
}

class CircleSerializer implements CircleInterface, Serializable {

    public radius: number;

    public serialize() {
        return JSON.stringify(this);
    }
}
```

## 1.5. Dependency inversion principle

We should always try to have dependencies on interfaces, not classes

```
// doesn't matter what type of object we pass if its implementing proper interface
function getArea(shapes: Shape[]) {
    return shapes.reduce(
        (previous, current) => previous + current.area(),
        0
    );
}
```


## 1.6. Refs
- https://dev.to/remojansen/implementing-the-onion-architecture-in-nodejs-with-typescript-and-inversifyjs-10ad
