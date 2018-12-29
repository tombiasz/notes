<!-- TOC -->

- [1. Value object](#1-value-object)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->

# 1. Value object

Use when you need to create a domain specific type

## 1.1. Code sample
```
class PublicationDate {
  constructor(start, end) {
    const now = new Date();
    const today = new Date (now.setHours(0, 0, 0, 0));
    const { isDate } = this.constructor;

    if (!isDate(start)) {
      throw new TypeError('start date is not a date');
    }
    if (!isDate(end)) {
      throw new TypeError('end date is not a date');
    }
    if (start < today) {
      throw new TypeError('start date cannot be less than todays date');
    }
    if (start > end) {
      throw new TypeError('end date cannot be less than start date');
    }

    this._start = start;
    this._end = end;
  }

  get start() {
    return this._start;
  }

  get end() {
    return this._end;
  }

  extendPublicationByDays(days) {
    const { start, end } = this;
    const newEnd = new Date(end);
    newEnd.setDate(newEnd.getDate() + days);
    return new this.constructor(start, newEnd);
  }

  isOngoing() {
    const now = new Date();
    return now < this.end;
  }

  hasEnded() {
    return !this.isOngoing();
  }

  toString() {
    return `${this.start} - ${this.end}`;
  }

  isEqual(another) {
    return this.start === another.start && this.end === another.end;
  }

  toJSON() {
    return {
      start: this.start,
      end: this.end,
    };
  }

  copy() {
    return new this.constructor(this.start, this.end);
  }

  static isDate(o) {
    return o instanceof Date;
  }

  static todayOnly() {
    const start = new Date().setHours(0, 0, 0, 0);
    const end = new Date().setHours(23, 59, 59, 999);
    return new this(new Date(start), new Date(end));
  }

  static fromJSON(json) {
    const { start, end } = JSON.parse(json);
    return new this(new Date(start), new Date(end));
  }
}

const start = new Date('2018-12-28');
const end = new Date('2019-12-11');

const publicationDate = new PublicationDate(start, end);
console.log(`${publicationDate}`);
console.log(publicationDate.isOngoing());
console.log(publicationDate.hasEnded());

const json = JSON.stringify(publicationDate);
console.log(json);
console.log(PublicationDate.fromJSON(json).toString());

const publicationDate2 = publicationDate.extendPublicationByDays(20);
console.log(`${publicationDate2}`);

console.log(publicationDate === publicationDate.copy()); // false
console.log(publicationDate.isEqual(publicationDate.copy())); // true

console.log(PublicationDate.todayOnly());

```

## 1.2. Notes
- immutability:
  - no setters
  - only copies
  - no side effects
- value equality - equal by value not reference
- self validation - you cannot create invalid object
- easy to test

## 1.3. Refs
- https://hackernoon.com/value-objects-like-a-pro-f1bfc1548c72
- https://crushlovely.com/journal/7-patterns-to-refactor-javascript-applications-value-objects
- https://martinfowler.com/bliki/ValueObject.html
