<!-- TOC -->

- [1. Functional programming](#1-functional-programming)
  - [1.1. Function composition](#11-function-composition)
  - [1.2. Curry](#12-curry)
  - [1.3. Higher order function](#13-higher-order-function)
  - [1.4. Functors](#14-functors)
  - [1.5. Monads](#15-monads)
  - [1.6 Object composition/mixins/functional inheritance](#16-object-compositionmixinsfunctional-inheritance)
  - [1.6. Refs](#16-refs)

<!-- /TOC -->

# 1. Functional programming

## 1.1. Function composition
```
const g = n => n + 1;
const f = n => n * 2;

let h = x => f(g(x));

h(20); // 42

const compose1 = (f, g) => x => f(g(x));
h = compose1(f, g);
h(20); // 42

const compose2 = (...fns) => x => fns.reduceRight((y, f) => f(y), x);
h = compose2(f, g);

h(20); // 42
```

```
// pipe(...fns: [...Function]) => x => y
const pipe = (...fns) => x => fns.reduce((y, f) => f(y), x);

const g = n => n + 1;
const f = n => n * 2;
const trace = label => value => {
  console.log(`${ label }: ${ value }`);
  return value;
};

const doStuffBetter = pipe(
  g,
  trace('after g'),
  f,
  trace('after f')
);

const value = doStuffBetter(20);
console.log(value) // 42
```

## 1.2. Curry
- A curried function is a function that takes multiple arguments one at a time
- A partial application is a function which has been applied to some, but not yet all of its arguments
- Partial applications can take as many or as few arguments a time as desired
- All curried functions return partial applications, but not all partial applications are the result of curried functions

```
const map = fn => mappable => mappable.map(fn);

const arr = [1, 2, 3, 4];

const isEven = n => n % 2 === 0;
const stripe = n => isEven(n) ? 'dark' : 'light';
const stripeAll = map(stripe);
const striped = stripeAll(arr);
log(striped); // ["light", "dark", "light", "dark"]

// same map different use case
const double = n => n * 2;
const doubleAll = map(double);
const doubled = doubleAll(arr);
log(doubled); // [2, 4, 6, 8]
```

## 1.3. Higher order function
A higher order function is a function that takes a function as an argument, or returns a function.

```
const reduce = (reducer, initial, arr) => {
  // shared stuff
  let acc = initial;
  for (let i = 0, { length } = arr; i < length; i++) {

    // unique stuff in reducer() call
    acc = reducer(acc, arr[i]);

  // more shared stuff
  }
  return acc;
};

const v1 =reduce((acc, curr) => acc + curr, 0, [1,2,3]);
console.log(v1); // 6

const filter = (
  fn, arr
) => reduce((acc, curr) => fn(curr) ?
  acc.concat([curr]) :
  acc, [], arr
);

const v2 = filter(a => a % 2 === 1, [1,2,3,4]);
console.lof(v2); // [1, 3]
```

## 1.4. Functors
 - A functor data type is something you can map over.
 - It’s a container which has an interface which can be used to apply a function to the values inside it.
 - When you see a functor, you should think “mappable”.
 - Functor types are typically represented as an object with a .map() method that maps from inputs to outputs while preserving structure, which means that the return value is the same type of functor (though values inside the container may be a different type).
 - Functors must respect identity and composition (functor laws)

```
const Identity = value => ({
  map: fn => Identity(fn(value)),
  valueOf: () => value,
  toString: () => `Identity(${value})`,
  [Symbol.iterator]: function* () {
    yield value;
  },
  constructor: Identity
});

Object.assign(Identity, {
  toString: () => 'Identity',
  is: x => typeof x.map === 'function'
});

// trace() is a utility to let you easily inspect
// the contents.
const trace = x => {
  console.log(x);
  return x;
};

const u = Identity(2);

// Identity law
u.map(trace);             // 2
u.map(x => x).map(trace); // 2

const f = n => n + 1;
const g = n => n * 2;

// Composition law
const r1 = u.map(x => f(g(x)));
const r2 = u.map(g).map(f);

r1.map(trace); // 5
r2.map(trace); // 5

const ints = (Identity(2) + Identity(4));
trace(ints); // 6

const hi = (Identity('h') + Identity('i'));
trace(hi); // "hi"

// [Symbol.iterator] enables standard JS iterations:
const arr = [6, 7, ...Identity(8)];
trace(arr); // [6, 7, 8]


// Sample abstraction
// Create the predicate
const exists = x => (x.valueOf() !== undefined && x.valueOf() !== null);

const ifExists = x => ({
  map: fn => exists(x) ? x.map(fn) : x
});

const add1 = n => n + 1;
const double = n => n * 2;

// Nothing happens...
ifExists(Identity(undefined)).map(trace);
// Still nothing...
ifExists(Identity(null)).map(trace);

// 42
ifExists(Identity(20))
  .map(add1)
  .map(double)
  .map(trace)
;


// Functor and composition
const curry = (
  f, arr = []
) => (...args) => (
  a => a.length === f.length ?
    f(...a) :
    curry(f, a)
)([...arr, ...args]);
const map = curry((fn, F) => F.map(fn));

const double = n => n * 2;

const mdouble = map(double);
mdouble(Identity(4)).map(trace); // 8
```

## 1.5. Monads
- https://medium.com/javascript-scene/javascript-monads-made-simple-7856be57bfe8o

## 1.6 Object composition/mixins/functional inheritance
- https://medium.com/javascript-scene/functional-mixins-composing-software-ffb66d5e731c
- https://medium.com/humans-create-software/composition-over-inheritance-cb6f88070205
- https://stackoverflow.com/a/39002948

```
const flying = o => {
  let isFlying = false;

  return Object.assign({}, o, {
    fly () {
      isFlying = true;
      return this;
    },

    isFlying: () => isFlying,

    land () {
      isFlying = false;
      return this;
    }
  });
};
const bird = flying({});
console.log( bird.isFlying() ); // false
console.log( bird.fly().isFlying() ); // true

const quacking = quack => o => Object.assign({}, o, {
  quack: () => quack
});
const quacker = quacking('Quack!')({});
console.log( quacker.quack() ); // 'Quack!'


const createDuck = quack => quacking(quack)(flying({}));
const duck = createDuck('Quack!');
console.log(duck.fly().isFlying());
console.log(duck.quack());
```

## 1.6. Refs
- https://medium.com/javascript-scene/composing-software-the-book-f31c77fc3ddc
