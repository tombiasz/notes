<!-- TOC -->

- [1. Refactoring](#1-refactoring)
    - [1.1. General example](#11-general-example)
    - [1.2. Extract function](#12-extract-function)
    - [1.3. Inline function](#13-inline-function)
    - [1.4. Extract Variable (Introduce Explaining Variable)](#14-extract-variable-introduce-explaining-variable)
    - [1.5. Inline variable](#15-inline-variable)
    - [1.6. Introduce Parameter Object](#16-introduce-parameter-object)
    - [1.7. Combine Functions Into Class](#17-combine-functions-into-class)
    - [1.8. Combine Functions in Transform](#18-combine-functions-in-transform)
    - [1.9. Split phase](#19-split-phase)
    - [1.10. Encapsulate Record (Replace Record with Data Class)](#110-encapsulate-record-replace-record-with-data-class)
    - [1.11. Encapsulate Collection](#111-encapsulate-collection)
    - [1.12. Replace Primitive With Object](#112-replace-primitive-with-object)
    - [1.13. Decompose Conditional](#113-decompose-conditional)
    - [1.14. Replace Conditional With Polymorphism](#114-replace-conditional-with-polymorphism)
    - [1.15. Introduce Special Case (Introduce Null Object)](#115-introduce-special-case-introduce-null-object)
    - [1.16. Separate Query From Modifier](#116-separate-query-from-modifier)
    - [1.17. Parameterize Function](#117-parameterize-function)
    - [1.18. Replace Constructor With Factory Function](#118-replace-constructor-with-factory-function)
    - [1.19. Replace Function With Command](#119-replace-function-with-command)
    - [1.20. Replace Type Code With Subclasses](#120-replace-type-code-with-subclasses)
    - [1.21. Replace Subclass With Delegate](#121-replace-subclass-with-delegate)
    - [1.22. Replace Inheritance with Delegation](#122-replace-inheritance-with-delegation)
- [2. Refs](#2-refs)

<!-- /TOC -->

# 1. Refactoring

## 1.1. General example

```json
[
  {
    "customer": "BigCo",
    "performances": [
      {
        "playID": "hamlet",
        "audience": 55
      },
      {
        "playID": "as­like",
        "audience": 35
      },
      {
        "playID": "othello",
        "audience": 40
      }
    ]
  }
]
```

BEFORE
```javascript
function statement(invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `Statement for ${invoice.customer}\n`;

  const format = new Intl.NumberFormat("en­US",
    {
      style: "currency", currency: "USD",
      minimumFractionDigits: 2
    }).format;

  for (let perf of invoice.performances) {
    const play = plays[perf.playID];
    let thisAmount = 0;

    switch (play.type) {
      case "tragedy":
        thisAmount = 40000;
        if (perf.audience > 30) {
          thisAmount += 1000 * (perf.audience ­ 30);
        }
        break;
      case "comedy":
        thisAmount = 30000;
        if (perf.audience > 20) {
          thisAmount += 10000 + 500 * (perf.audience ­ 20);
        }
        thisAmount += 300 * perf.audience;
        break;
      default:
        throw new Error(`unknown type: ${play.type}`);
    }

    // add volume credits
    volumeCredits += Math.max(perf.audience ­ 30, 0);

    // add extra credit for every ten comedy attendees
    if ("comedy" === play.type) volumeCredits += Math.floor(perf.audience / 5);

    // print line for this order
    result += ` ${play.name}: ${format(thisAmount/100)} (${perf.audience} seats)`;

    totalAmount += thisAmount;
  }

  result += `Amount owed is ${format(totalAmount / 100)}\n`;
  result += `You earned ${volumeCredits} credits\n`;
  return result;
}


```

AFTER
```javascript
export default function createStatementData(invoice, plays) {
  const result = {};
  result.customer = invoice.customer;
  result.performances = invoice.performances.map(enrichPerformance);
  result.totalAmount = totalAmount(result);
  result.totalVolumeCredits = totalVolumeCredits(result);
  return result;

  function enrichPerformance(aPerformance) {
    const calculator = createPerformanceCalculator(aPerformance, playFor(aPerformace));
    const result = Object.assign({}, aPerformance);
    result.play = calculator.play;
    result.amount = calculator.amount;
    result.volumeCredits = calculator.volumeCredits;
    return result;
  }

  function playFor(aPerformance) {
    return plays[aPerformance.playID]
  }

  function totalAmount(data) {
    return data.performances
      .reduce((total, p) => total + p.amount, 0);
  }

  function totalVolumeCredits(data) {
    return data.performances

      .reduce((total, p) => total + p.volumeCredits, 0);
  }

}

function createPerformanceCalculator(aPerformance, aPlay) {
  switch (aPlay.type) {
    case "tragedy": return new TragedyCalculator(aPerformance, aPlay);
    case "comedy": return new ComedyCalculator(aPerformance, aPlay);
    default:
      throw new Error(`unknown type: ${aPlay.type}`);
  }
}

class PerformanceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }

  get amount() {
    throw new Error('subclass responsibility');
  }

  get volumeCredits() {
    return Math.max(this.performance.audience ­ 30, 0);
  }
}

class TragedyCalculator extends PerformanceCalculator {
  get amount() {
    let result = 40000;
    if (this.performance.audience > 30) {
      result += 1000 * (this.performance.audience ­ 30);
    }
    return result;
  }
}

class ComedyCalculator extends PerformanceCalculator {
  get amount() {
    let result = 30000;
    if (this.performance.audience > 20) {
      result += 10000 + 500 * (this.performance.audience ­ 20);
    }
    result += 300 * this.performance.audience;
    return result;
  }
  get volumeCredits() {
    return super.volumeCredits + Math.floor(this.performance.audience / 5);
  }
}
```

## 1.2. Extract function
To separate between intention and implementation. If you have to spend effort
looking at a fragment of code and figuring out what it’s doing, then you
should extract it into a function and name the function after the “what.”

BEFORE
```javascript
function printOwing(invoice) {
  let outstanding = 0;
  console.log("***********************");
  console.log("**** Customer Owes ****");
  console.log("***********************");

  // calculate outstanding
  for (const o of invoice.orders) {
    outstanding += o.amount;
  }

  // record due date
  const today = Clock.today; // a wrapper around Date
  invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDate());

  //print details
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
  console.log(`due: ${invoice.dueDate.toLocaleDateString()}`);
}
```

AFTER
```javascript
function printOwing(invoice) {
  printBanner();
  const outstanding = calculateOutstanding(invoices);
  recordDueDate(invoice);
  printDetails(invoice, outstanding);
}

function recordDueDate(invoice) {
  const today = Clock.today; // a wrapper around Date
  invoice.dueDate = new Date(today.getFullYear(), today.getMonth(), today.getDat
}

function calculateOutstanding(invoice) {
  let result = 0; // common naming pattern for variable that will be returned
  for (const o of invoice.orders) {
    result += o.amount;
  }
  return result;
}

function printDetails(invoice, outstanding) {
  console.log(`name: ${invoice.customer}`);
  console.log(`amount: ${outstanding}`);
  console.log(`due: ${invoice.dueDate.toLocaleDateString()}`);
}

function printBanner() {
  console.log("***********************");
  console.log("**** Customer Owes ****");
  console.log("***********************");
}
```

## 1.3. Inline function

When body of a function is as clear as its name

BEFORE I
```javascript
function rating(aDriver) {
  return moreThanFiveLateDeliveries(aDriver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(aDriver) {
  return aDriver.numberOfLateDeliveries > 5;
}
```

AFTER I
```javascript
function rating(aDriver) {
  return aDriver.numberOfLateDeliveries > 5 ? 2 : 1;
}
```

BEFORE II
```javascript
function reportLines(aCustomer) {
  const lines = [];
  gatherCustomerData(lines, aCustomer);
  return lines;
}
function gatherCustomerData(out, aCustomer) {
  out.push(["name", aCustomer.name]);
  out.push(["location", aCustomer.location]);
}
```

AFTER II
```javascript
function reportLines(aCustomer) {
  const lines = [];
  lines.push(["name", aCustomer.name]);
  lines.push(["location", aCustomer.location]);
  return lines;
}
```

## 1.4. Extract Variable (Introduce Explaining Variable)

To add a name to an expression in code

BEFORE I
```javascript
function price(order) {
  //price is base price ­ quantity discount + shipping
  return order.quantity * order.itemPrice -
    Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
    Math.min(order.quantity * order.itemPrice * 0.1, 100);
}
```

AFTER I
```javascript
function price(order) {
  const basePrice = order.quantity * order.itemPrice;
  const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
  const shipping = Math.min(basePrice * 0.1, 100);
  return basePrice - quantityDiscount + shipping;
}
```

AFTER II
```javascript
class Order {
  constructor(aRecord) {
    this._data = aRecord;
  }

  get quantity() { return this._data.quantity; }

  get itemPrice() { return this._data.itemPrice; }

  get price() { return this.basePrice * this.quantityDiscount + this.shipping; }

  get basePrice() { return this.quantity * this.itemPrice; }

  get quantityDiscount() {
    return Math.max(0, this.quantity - 500) * this.itemPrice * 0.05;
  }

  get shipping() { return Math.min(this.basePrice * 0.1, 100); }
}
```

## 1.5. Inline variable

When expression is as clear as its variable name


BEFORE
```javascript
const basePrice = anOrder.basePrice;
return basePrice > 1000;
```

AFTER
```javascript
return anOrder.basePrice > 1000;
```

## 1.6. Introduce Parameter Object

When group of data is often used together


BEFORE
```javascript
function readingsOutsideRange(station, min, max) {
  return station.readings
    .filter(r => r.temp < min || r.temp > max);
}
```

AFTER I
```javascript
class NumberRange {
  constructor(min, max) {
    this._data = { min: min, max: max };
  }
  get min() { return this._data.min; }
  get max() { return this._data.max; }
}

function readingsOutsideRange(station, min, range) {
  return station.readings
    .filter(r => r.temp < range.min || r.temp > range.max);
}
```

AFTER II with additional logic
```javascript
class NumberRange {
  constructor(min, max) {
    this._data = { min: min, max: max };
  }
  get min() { return this._data.min; }
  get max() { return this._data.max; }
  contains(arg) { return (arg >= this.min && arg <= this.max); }
}

function readingsOutsideRange(station, range) {
  return station.readings
    .filter(r => !range.contains(r.temp));
}
```

## 1.7. Combine Functions Into Class

When group of functions operate closely together

page 164

BEFORE
```javascript
const reading = { customer: "ivan", quantity: 10, month: 5, year: 2017 };
const acquireReading = () => reading;

const taxThreshold = year => year < 2010 ? 1.6 : 1.3;

// client 1
const aReading = acquireReading();
const baseCharge = baseRate(aReading.month, aReading.year) * aReading.quantity;

// client 2
const aReading = acquireReading();
const base = (baseRate(aReading.month, aReading.year) * aReading.quantity);
const taxableCharge = Math.max(0, base.taxThreshold(aReading.year));
```

AFTER
```javascript
const reading = { customer: "ivan", quantity: 10, month: 5, year: 2017 };
const acquireReading = () => reading;

class Reading {
  constructor(data) {
    this._customer = data.customer;
    this._quantity = data.quantity;
    this._month = data.month;
    this._year = data.year;
  }

  get customer() { return this._customer; }
  get quantity() { return this._quantity; }
  get month() { return this._month; }
  get year() { return this._year; }

  get baseCharge() {
    return baseRate(this.month, this.year) * this.quantity;
  }

  get taxableCharge() {
    return Math.max(0, this.baseCharge * taxThreshold(this.year));
  }

}

const rawReading = acquireReading();
const aReading = new Reading(rawReading);
const taxableCharge = aReading.taxableCharge;
```

## 1.8. Combine Functions in Transform

BEFORE
```javascript
const reading = { customer: "ivan", quantity: 10, month: 5, year: 2017 };
const acquireReading = () => reading;

const taxThreshold = year => year < 2010 ? 1.6 : 1.3;

// client 1
const aReading = acquireReading();
const baseCharge = baseRate(aReading.month, aReading.year) * aReading.quantity;

// client 2
const aReading = acquireReading();
const base = (baseRate(aReading.month, aReading.year) * aReading.quantity);
const taxableCharge = Math.max(0, base.taxThreshold(aReading.year));
```

AFTER
```javascript
const _ = require('lodash');

const reading = { customer: "ivan", quantity: 10, month: 5, year: 2017 };
const acquireReading = () => reading;

function enrichReading(original) {
  const result = _.cloneDeep(original);
  result.baseCharge = calculateBaseCharge(result);
  result.taxableCharge = Math.max(0, result.baseCharge * taxThreshold(result.year));
  return result;
}

const rawReading = acquireReading();
const aReading = enrichReading(rawReading);
const taxableCharge = aReading.taxableCharge;
```

## 1.9. Split phase

When code is dealing with two different things, look for a way to split it
into separate modules.

BEFORE
```javascript
function priceOrder(product, quantity, shippingMethod) {
  const basePrice = product.basePrice * quantity;
  const discount = Math.max(quantity - product.discountThreshold, 0)
    * product.basePrice * product.discountRate;
  const shippingPerCase = (basePrice > shippingMethod.discountThreshold)
    ? shippingMethod.discountedFee : shippingMethod.feePerCase;
  const shippingCost = quantity * shippingPerCase;
  const price = basePrice - discount + shippingCost;
  return price;
}

```

AFTER
```javascript
function priceOrder(product, quantity, shippingMethod) {
  const priceData = calculatePricingData(product, quantity);
  return applyShipping(priceData, shippingMethod);
}

function calculatePricingData(product, quantity) {
  const basePrice = product.basePrice * quantity;
  const discount = Math.max(quantity - product.discountThreshold, 0)
    * product.basePrice * product.discountRate;
  return { basePrice: basePrice, quantity: quantity, discount: discount };
}

function applyShipping(priceData, shippingMethod) {
  const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshol
    ? shippingMethod.discountedFee : shippingMethod.feePerCase;
  const shippingCost = priceData.quantity * shippingPerCase;
  return priceData.basePrice ­ priceData.discount + shippingCost;
}
```

## 1.10. Encapsulate Record (Replace Record with Data Class)

BEFORE I
```javascript
const organization = {name: "Acme Gooseberries", country: "GB"};

```

AFTER I
```javascript
class Organization {
  constructor(data) {
    this._name = data.name;
    this._country = data.country;
  }
  get name() { return this._name; }
  set name(aString) { this._name = aString; }
  get country() { return this._country; }
  set country(aCountryCode) { this._country = aCountryCode; }
}

```

BEFORE II
```javascript
const customer = {
  "1920": {
    name: "martin",
    id: "1920",
    usages: {
      "2016": {
        "1": 50,
        "2": 55,
        // remaining months of the year
      },
      "2015": {
        "1": 70,
        "2": 63,
        // remaining months of the year
      }
    }
  },
  // more customers
};


customerData[customerID].usages[year][month] = amount;

function compareUsage(customerID, laterYear, month) {
  const later = customerData[customerID].usages[laterYear][month];
  const earlier = customerData[customerID].usages[laterYear - 1][month];
  return { laterAmount: later, change: later - earlier };
}
```

AFTER II
```javascript
class CustomerData {
  constructor(data) {
    this._data = data;
  }

  setUsage(customerID, year, month, amount) {
    this._data[customerID].usages[year][month] = amount;
  }

  usage(customerID, year, month) {
    return this._data[customerID].usages[year][month];
  }
}

const getCustomerData = () => new CustomerData(customer);

getCustomerData().setUsage(customerID, year, month, amount);

function compareUsage(customerID, laterYear, month) {
  const later = getCustomerData().usage(customerID, laterYear, month);
  const earlier = getCustomerData().usage(customerID, laterYear - 1, month)
  return { laterAmount: later, change: later - earlier };
}
```

## 1.11. Encapsulate Collection


BEFORE
```javascript
class Person {
  constructor(name) {
    this._name = name;
    this._courses = [];
  }
  get name() { return this._name; }
  get courses() { return this._courses; }
  set courses(aList) { this._courses = aList; }
}

class Course {
  constructor(name, isAdvanced) {
    this._name = name;
    this._isAdvanced = isAdvanced;
  }
  get name() { return this._name; }
  get isAdvanced() { return this._isAdvanced; }
}


numAdvancedCourses = aPerson.courses
  .filter(c => c.isAdvanced)
  .length;


// unwanted mutation
for (const name of readBasicCourseNames(filename)) {
  aPerson.courses.push(new Course(name, false));
}

```

AFTER
```javascript
class Person {
  constructor(name) {
    this._name = name;
    this._courses = [];
  }
  get name() { return this._name; }

  // return copy
  get courses() { return this._courses.slice(); }

  // copy on set
  set courses(aList) { this._courses = aList.slice(); }

  addCourse(aCourse) {
    this._courses.push(aCourse);
  }

  removeCourse(aCourse, fnIfAbsent = () => { throw new RangeError(); }) {
    const index = this._courses.indexOf(aCourse);
    if (index === 1) {
      fnIfAbsent();
    } else {
      this._courses.splice(index, 1);
    }
  }
}

class Course {
  // stays the same
}

// proper encapsulation
for(const name of readBasicCourseNames(filename)) {
  aPerson.addCourse(new Course(name, false));
}

```

## 1.12. Replace Primitive With Object

BEFORE
```javascript
class Order {
  constructor(data) {
    this.priority = data.priority;
  }
}

const highPriorityCount = orders
  .filter(o => "high" === o.priority || "rush" === o.priority)
  .length;

```

AFTER
```javascript
class Order {
  constructor(data) {
    this._priority = data.priority;
  }

  get priority() { return this._priority; }

  // additional accessor
  get priorityString() { return this._priority.toString(); }

  set priority(aString) { this._priority = new Priority(aString); }
}


class Priority {
  constructor(value) {
    if (value instanceof Priority) return value;

    if (Priority.legalValues().includes(value))
      this._value = value;
    else
      throw new Error(`<${value}> is invalid for Priority`);
  }

  toString() { return this._value; }

  get _index() { return Priority.legalValues().findIndex(s => s === this._value); }

  static legalValues() { return ['low', 'normal', 'high', 'rush']; }

  equals(other) { return this._index === other._index; }

  higherThan(other) { return this._index > other._index; }

  lowerThan(other) { return this._index < other._index; }
}

const highPriorityCount = orders
  .filter(o => o.priority.higherThan(new Priority("normal")))
  .length;

```

## 1.13. Decompose Conditional


BEFORE
```javascript
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
```

AFTER
```javascript
charge = summer() ? summerCharge() : regularCharge();

function summer() {
  return !aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd);
}

function summerCharge() {
  return quantity * plan.summerRate;
}

function regularCharge() {
  return quantity * plan.regularRate + plan.regularServiceCharge;
}
```

## 1.14. Replace Conditional With Polymorphism

BEFORE
```javascript

function plumages(birds) {
  return new Map(birds.map(b => [b.name, plumage(b)]));
}

function speeds(birds) {
  return new Map(birds.map(b => [b.name, airSpeedVelocity(b)]));
}

function plumage(bird) {
  switch (bird.type) {
    case 'EuropeanSwallow':
      return "average";
    case 'AfricanSwallow':
      return (bird.numberOfCoconuts > 2) ? "tired" : "average";
    case 'NorwegianBlueParrot':
      return (bird.voltage > 100) ? "scorched" : "beautiful";
    default:
      return "unknown";
  }
}

function airSpeedVelocity(bird) {
  switch (bird.type) {
    case 'EuropeanSwallow':
      return 35;
    case 'AfricanSwallow':
      return 40 - 2 * bird.numberOfCoconuts;
    case 'NorwegianBlueParrot':
      return (bird.isNailed) ? 0 : 10 + bird.voltage / 10;
    default:
      return null;
  }
}
```

AFTER
```javascript
function plumages(birds) {
  return new Map(birds
    .map(b => createBird(b))
    .map(bird => [bird.name, bird.plumage]));
}

function speeds(birds) {
  return new Map(birds
    .map(b => createBird(b))
    .map(bird => [bird.name, bird.airSpeedVelocity]));
}

function createBird(bird) {
  switch (bird.type) {
    case 'EuropeanSwallow':
      return new EuropeanSwallow(bird);
    case 'AfricanSwallow':
      return new AfricanSwallow(bird);
    case 'NorwegianBlueParrot':
      return new NorwegianBlueParrot(bird);
    default:
      return new Bird(bird);
  }

}

class Bird {
  constructor(birdObject) {
    Object.assign(this, birdObject);
  }

  get plumage() {
    return "unknown";
  }

  get airSpeedVelocity() {
    return null;
  }

}

class EuropeanSwallow extends Bird {
  get plumage() {
    return "average";
  }

  get airSpeedVelocity() {
    return 35;
  }

}

class AfricanSwallow extends Bird {
  get plumage() {
    return (this.numberOfCoconuts > 2) ? "tired" : "average";
  }
  get airSpeedVelocity() {
    return 40 - 2 * this.numberOfCoconuts;
  }
}
class NorwegianBlueParrot extends Bird {
  get plumage() {
    return (this.voltage > 100) ? "scorched" : "beautiful";
  }
  get airSpeedVelocity() {
    return (this.isNailed) ? 0 : 10 + this.voltage / 10;
  }
}
```

BEFORE 2
```javascript
function rating(voyage, history) {
  const vpf = voyageProfitFactor(voyage, history);
  const vr = voyageRisk(voyage);
  const chr = captainHistoryRisk(voyage, history);
  if (vpf * 3 > (vr + chr * 2)) return "A";
  else return "B";
}

function voyageRisk(voyage) {
  let result = 1;
  if (voyage.length > 4) result += 2;
  if (voyage.length > 8) result += voyage.length - 8;
  if (["china", "east­indies"].includes(voyage.zone)) result += 4;
  return Math.max(result, 0);
}

function captainHistoryRisk(voyage, history) {
  let result = 1;
  if (history.length < 5) result += 4;
  result += history.filter(v => v.profit < 0).length;
  if (voyage.zone === "china" && hasChina(history)) result -= 2;
  return Math.max(result, 0);
}

function hasChina(history) {
  return history.some(v => "china" === v.zone);
}

function voyageProfitFactor(voyage, history) {
  let result = 2;
  if (voyage.zone === "china") result += 1;
  if (voyage.zone === "east­indies") result += 1;
  if (voyage.zone === "china" && hasChina(history)) {
    result += 3;
    if (history.length > 10) result += 1;
    if (voyage.length > 12) result += 1;
    if (voyage.length > 18) result -= 1;
  }
  else {
    if (history.length > 8) result += 1;
    if (voyage.length > 14) result -= 1;
  }
  return result;
}

```

AFTER 2
```javascript
function rating(voyage, history) {
  return createRating(voyage, history).value;
}

function createRating(voyage, history) {
  if (voyage.zone === "china" && history.some(v => "china" === v.zone))
    return new ExperiencedChinaRating(voyage, history);
  else return new Rating(voyage, history);
}

class Rating {
  constructor(voyage, history) {
    this.voyage = voyage;
    this.history = history;
  }

  get value() {
    const vpf = this.voyageProfitFactor;
    const vr = this.voyageRisk;
    const chr = this.captainHistoryRisk;
    if (vpf * 3 > (vr + chr * 2)) return "A";
    else return "B";
  }

  get voyageRisk() {
    let result = 1;
    if (this.voyage.length > 4) result += 2;
    if (this.voyage.length > 8) result += this.voyage.length - 8;
    if (["china", "east­indies"].includes(this.voyage.zone)) result += 4;
    return Math.max(result, 0);
  }

  get captainHistoryRisk() {
    let result = 1;
    if (this.history.length < 5) result += 4;
    result += this.history.filter(v => v.profit < 0).length;
    return Math.max(result, 0);
  }

  get voyageProfitFactor() {
    let result = 2;
    if (this.voyage.zone === "china") result += 1;
    if (this.voyage.zone === "east­indies") result += 1;
    result += this.historyLengthFactor;
    result += this.voyageLengthFactor;
    return result;
  }

  get voyageLengthFactor() {
    return (this.voyage.length > 14) ? 1 : 0;
  }

  get historyLengthFactor() {
    return (this.history.length > 8) ? 1 : 0;
  }
}

class ExperiencedChinaRating extends Rating {
  get captainHistoryRisk() {
    const result = super.captainHistoryRisk ­ 2;
    return Math.max(result, 0);
  }

  get voyageLengthFactor() {
    let result = 0;
    if (this.voyage.length > 12) result += 1;
    if (this.voyage.length > 18) result -= 1;
    return result;
  }

  get historyLengthFactor() {
    return (this.history.length > 10) ? 1 : 0;
  }

  get voyageProfitFactor() {
    return super.voyageProfitFactor + 3;
  }
}
```

## 1.15. Introduce Special Case (Introduce Null Object)

Create a special­case element that captures all the common behavior

BEFORE
```javascript
// client code
const aCustomer = site.customer;
let customerName;
if (aCustomer === "unknown") customerName = "occupant";
else customerName = aCustomer.name;

// or
const plan = (aCustomer === "unknown")
  ? registry.billingPlans.basic
  : aCustomer.billingPlan;

// or
if (aCustomer !== "unknown") aCustomer.billingPlan = newPlan;
```

AFTER
```javascript

class UnknownCustomer {
  isUnknown() {
    return true;
  }

  get name() {
    return "occupant";
  }

  get billingPlan() {
    return registry.billingPlans.basic;
  }

  set billingPlan(arg) {
    /* ignore */
  }

  get paymentHistory() {
    return new NullPaymentHistory();
  }
}

class Customer {
  isUnknown() {
    return false;
  }

  // other methods
}

class NullPaymentHistory {
  get weeksDelinquentInLastYear() {
    return 0;
  }
}

// client code
const aCustomer = site.customer;
const customerName = aCustomer.name;

// or
const plan = aCustomer.billingPlan

// or
if (!aCustomer.isUnknown()) aCustomer.billingPlan = newPlan;
```

AFTER II
```javascript
const deepFreeze => obj => obj;

// if no update is required then use simple object
function createUnknownCustomer() {
  return deepFreeze({
    isUnknown: true,
    name: "occupant",
    billingPlan: registry.billingPlans.basic,
    paymentHistory: {
      weeksDelinquentInLastYear: 0,
    },
  });
}
```

## 1.16. Separate Query From Modifier

To avoid functions with side-effects

BEFORE
```javascript
function alertForMiscreant(people) {
  for (const p of people) {
    if (p === "Don") {
      setOffAlarms();
      return "Don";
    }
    if (p === "John") {
      setOffAlarms();
      return "John";
    }
  }
  return "";
}
```

AFTER
```javascript
function findMiscreant(people) {
  for (const p of people) {
    if (p === "Don") {
      return "Don";
    }
    if (p === "John") {
      return "John";
    }
  }
  return "";
}

function alertForMiscreant(people) {
  if (findMiscreant(people) !== "") setOffAlarms();
}

const found = findMiscreant(people);
alertForMiscreant(people);
```

## 1.17. Parameterize Function


BEFORE
```javascript
function baseCharge(usage) {
  if (usage < 0) return usd(0);
  const amount =
    bottomBand(usage) * 0.03
    + middleBand(usage) * 0.05
    + topBand(usage) * 0.07;
  return usd(amount);
}

function bottomBand(usage) {
  return Math.min(usage, 100);
}

function middleBand(usage) {
  return usage > 100 ? Math.min(usage, 200) - 100 : 0;
}

function topBand(usage) {
  return usage > 200 ? usage - 200 : 0;
}

```

AFTER
```javascript
function withinBand(usage, bottom, top) {
  return usage > bottom ? Math.min(usage, top) - bottom: 0;
}

function baseCharge(usage) {
  if (usage < 0) return usd(0);
  const amount =
    withinBand(usage, 0, 100) * 0.03
    + withinBand(usage, 100, 200) * 0.05
    + withinBand(usage, 200, Infinity) * 0.07;
  return usd(amount);
}
```

## 1.18. Replace Constructor With Factory Function


BEFORE
```javascript
class Employee {
  constructor(name, typeCode) {
    this._name = name;
    this._typeCode = typeCode;
  }

  get name() { return this._name; }

  get type() {
    return Employee.legalTypeCodes[this._typeCode];
  }

  static get legalTypeCodes() {
    return { "E": "Engineer", "M": "Manager", "S": "Salesman" };
  }
}

const employee = new Employee('foo', 'E')
```

AFTER
```javascript
function createEngineer(name) {
  return new Employee(name, 'E');
}
```

## 1.19. Replace Function With Command


BEFORE
```javascript
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  let highMedicalRiskFlag = false;
  if (medicalExam.isSmoker) {
    healthLevel += 10;
    highMedicalRiskFlag = true;
  }
  let certificationGrade = "regular";
  if (scoringGuide.stateWithLowCertification(candidate.originState)) {
    certificationGrade = "low";
    result -= 5;
  }
  // lots more code like this
  result -= Math.max(healthLevel - 5, 0);
  return result;
}
```

AFTER
```javascript
function score(candidate, medicalExam, scoringGuide) {
  return new Scorer(candidate).execute(medicalExam, scoringGuide);
}

class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this._candidate = candidate;
    this._medicalExam = medicalExam;
    this._scoringGuide = scoringGuide;
  }
  execute() {
    this._result = 0;
    this._healthLevel = 0;
    this._highMedicalRiskFlag = false;
    this.scoreSmoking();
    this._certificationGrade = "regular";
    if (this._scoringGuide.stateWithLowCertification(this._candidate.originState)) {
      this._certificationGrade = "low";
      this._result -= 5;
    }
    // lots more code like this
    this._result -= Math.max(this._healthLevel - 5, 0);
    return this._result;
  }

  scoreSmoking() {
    if (this._medicalExam.isSmoker) {
      this._healthLevel += 10;
      this._highMedicalRiskFlag = true;
    }
  }
}
```

## 1.20. Replace Type Code With Subclasses

BEFORE I
```javascript
class Employee {
  constructor(name, type) {
    this.validateType(type);
    this._name = name;
    this._type = type;
  }

  validateType(arg) {
    if (!["engineer", "manager", "salesman"].includes(arg))
      throw new Error(`Employee cannot be of type ${arg}`);
  }

  toString() { return `${this._name} (${this._type})`; }
}

function createEmployee(name, type) {
  return new Employee(name, type);
}
```

AFTER I
```javascript
class Employee {
  constructor(name, type) {
    this.validateType(type);
    this._name = name;
    this._type = type;
  }

  get type() { return this._type }

  toString() { return `${this._name} (${this._type})`; }
}

class Engineer extends Employee {
  get type() { return 'engineer' };
}

class Salesman extends Employee {
  get type() { return 'salesman' };
}

class Manager extends Employee {
  get type() { return 'manager' };
}

function createEmployee(name, type) {
  switch (type) {
    case "engineer": return new Engineer(name, type);
    case "salesman": return new Salesman(name, type);
    case "manager": return new Manager(name, type);
    default: throw new Error(`Employee cannot be of type ${type}`);
  }
}
```

BEFORE II
```javascript
class Employee {
  constructor(name, type) {
    this.validateType(type);
    this._name = name;
    this.type = type;
  }

  validateType(arg) {
    if (!["engineer", "manager", "salesman"].includes(arg))
      throw new Error(`Employee cannot be of type ${arg}`);
  }

  get typeString() { return this._type.toString(); }

  get type() { return this._type; }

  set type(arg) { this._type = new EmployeeType(arg); }

  get capitalizedType() {
    return this.typeString.charAt(0).toUpperCase()
      + this.typeString.substr(1).toLowerCase();
  }
}
```

AFTER II
```javascript
class Employee {
  constructor(name, type) {
    this.validateType(type);
    this._name = name;
    this.type = type;
  }

  validateType(arg) {
    if (!["engineer", "manager", "salesman"].includes(arg))
      throw new Error(`Employee cannot be of type ${arg}`);
  }

  get typeString() { return this._type.toString(); }

  get type() { return this._type; }

  set type(arg) { this._type = Employee.createEmployeeType(arg); }

  toString() { return `${this._name} (${this.type.capitalizedName})`; }

  static createEmployeeType(aString) {
    switch (aString) {
      case "engineer": return new Engineer();
      case "manager": return new Manager();
      case "salesman": return new Salesman();
      default: throw new Error(`Employee cannot be of type ${aString}`);
    }
  }
}

class EmployeeType {
  constructor(aString) {
    this._value = aString;
  }

  toString() { return this._value; }

  get capitalizedName() {
    return this.toString().charAt(0).toUpperCase()
      + this.toString().substr(1).toLowerCase();
  }
}

class Engineer extends EmployeeType {
  toString() { return "engineer"; }
}

class Manager extends EmployeeType {
  toString() { return "manager"; }
}

class Salesman extends EmployeeType {
  toString() { return "salesman"; }
}

```

## 1.21. Replace Subclass With Delegate

BEFORE I
```javascript
class Booking {
  constructor(show, date) {
    this._show = show;
    this._date = date;
  }

  get hasTalkback() {
    return this._show.hasOwnProperty('talkback') && !this.isPeakDay;
  }

  get basePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return result;
  }

}

class PremiumBooking extends Booking {
  constructor(show, date, extras) {
    super(show, date);
    this._extras = extras;
  }

  get hasTalkback() {
    return this._show.hasOwnProperty('talkback');
  }

  get basePrice() {
    return Math.round(super.basePrice + this._extras.premiumFee);
  }

  get hasDinner() {
    return this._extras.hasOwnProperty('dinner') && !this.isPeakDay;
  }
}

function createBooking(show, date) {
  return new Booking(show, date);
}

function createPremiumBooking(show, date, extras) {
  return new PremiumBooking(show, date, extras);
}
```

AFTER I
```javascript
class Booking {
  constructor(show, date) {
    this._show = show;
    this._date = date;
  }

  get hasTalkback() {
    return (this._premiumDelegate)
      ? this._premiumDelegate.hasTalkback
      : this._show.hasOwnProperty('talkback') && !this.isPeakDay;
  }

  get basePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return result;
  }

  get basePrice() {
    let result = this._show.price;
    if (this.isPeakDay) result += Math.round(result * 0.15);
    return (this._premiumDelegate)
      ? this._premiumDelegate.extendBasePrice(result)
      : result;
  }

  get hasDinner() {
    return (this._premiumDelegate)
      ? this._premiumDelegate.hasDinner
      : undefined;
  }
}

class PremiumBookingDelegate {
  constructor(hostBooking, extras) {
    this._host = hostBooking;
    this._extras = extras;
  }

  _bePremium(extras) {
    this._premiumDelegate = new PremiumBookingDelegate(this, extras);
  }

  get hasTalkback() {
    return this._host._show.hasOwnProperty('talkback');
  }

  extendBasePrice(base) {
    return Math.round(base + this._extras.premiumFee);
  }

  get hasDinner() {
    return this._extras.hasOwnProperty('dinner') && !this._host.isPeakDay;
  }
}

function createBooking(show, date) {
  return new Booking(show, date);
}

function createPremiumBooking(show, date, extras) {
  const result = new Booking(show, date, extras);
  result._bePremium(extras);
  return result;
}
```


BEFORE II
```javascript
function createBird(data) {
  switch (data.type) {
    case 'EuropeanSwallow':
      return new EuropeanSwallow(data);
    case 'AfricanSwallow':
      return new AfricanSwallow(data);
    case 'NorweigianBlueParrot':
      return new NorwegianBlueParrot(data);
    default:
      return new Bird(data);
  }
}

class Bird {
  constructor(data) {
    this._name = data.name;
    this._plumage = data.plumage;
  }

  get name() { return this._name; }

  get plumage() {
    return this._plumage || "average";
  }

  get airSpeedVelocity() { return null; }
}

class EuropeanSwallow extends Bird {
  get airSpeedVelocity() { return 35; }
}

class AfricanSwallow extends Bird {
  constructor(data) {
    super(data);
    this._numberOfCoconuts = data.numberOfCoconuts;
  }

  get airSpeedVelocity() {
    return 40 - 2 * this._numberOfCoconuts;
  }
}

class NorwegianBlueParrot extends Bird {
  constructor(data) {
    super(data);
    this._voltage = data.voltage;
    this._isNailed = data.isNailed;
  }

  get plumage() {
    if (this._voltage > 100) return "scorched";
    else return this._plumage || "beautiful";
  }

  get airSpeedVelocity() {
    return (this._isNailed) ? 0 : 10 + this._voltage / 10;
  }
}
```

AFTER II
```javascript
function createBird(data) {
  return new Bird(data);
}

class Bird {
  constructor(data) {
    this._name = data.name;
    this._plumage = data.plumage;
    this._speciesDelegate = this.selectSpeciesDelegate(data);
  }

  get name() { return this._name; }

  get plumage() { return this._speciesDelegate.plumage; }

  get airSpeedVelocity() { return this._speciesDelegate.airSpeedVelocity; }

  selectSpeciesDelegate(data) {
    switch (data.type) {
      case 'EuropeanSwallow':
        return new EuropeanSwallowDelegate(data, this);
      case 'AfricanSwallow':
        return new AfricanSwallowDelegate(data, this);
      case 'NorweigianBlueParrot':
        return new NorwegianBlueParrotDelegate(data, this);
      default: return new SpeciesDelegate(data, this);
    }
  }
}

class SpeciesDelegate {
  constructor(data, bird) {
    this._bird = bird;
  }

  get plumage() {
    return this._bird._plumage || "average";
  }

  get airSpeedVelocity() { return null; }
}

class EuropeanSwallowDelegate extends SpeciesDelegate {
  get airSpeedVelocity() { return 35; }
}

class AfricanSwallowDelegate extends SpeciesDelegate {
  constructor(data, bird) {
    super(data, bird);
    this._numberOfCoconuts = data.numberOfCoconuts;
  }

  get airSpeedVelocity() {
    return 40 - 2 * this._numberOfCoconuts;
  }
}

class NorwegianBlueParrotDelegate extends SpeciesDelegate {
  constructor(data, bird) {
    super(data, bird);
    this._voltage = data.voltage;
    this._isNailed = data.isNailed;
  }

  get airSpeedVelocity() {
    return (this._isNailed) ? 0 : 10 + this._voltage / 10;
  }

  get plumage() {
    if (this._voltage > 100) return "scorched";
    else return this._bird._plumage || "beautiful";
  }
}

```

## 1.22. Replace Inheritance with Delegation

BEFORE
```javascript
class CatalogItem {
  constructor(id, title, tags) {
    this._id = id;
    this._title = title;
    this._tags = tags;
  }

  get id() { return this._id; }

  get title() { return this._title; }

  hasTag(arg) { return this._tags.includes(arg); }
}

class Scroll extends CatalogItem {
  constructor(id, title, tags, dateLastCleaned) {
    super(id, title, tags);
    this._lastCleaned = dateLastCleaned;
  }

  needsCleaning(targetDate) {
    const threshold = this.hasTag("revered") ? 700 : 1500;
    return this.daysSinceLastCleaning(targetDate) > threshold;
  }

  daysSinceLastCleaning(targetDate) {
    return this._lastCleaned.until(targetDate, ChronoUnit.DAYS);
  }
}
```

AFTER
```javascript
class Scroll {
  constructor(id, dateLastCleaned, catalogID, catalog) {
    this._id = id;
    this._catalogItem = catalog.get(catalogID);
    this._lastCleaned = dateLastCleaned;
  }

  get id() { return this.id; }

  get title() { return this._catalogItem.title; }

  hasTag(aString) { return this._catalogItem.hasTag(aString); }

  needsCleaning(targetDate) {
    const threshold = this.hasTag("revered") ? 700 : 1500;
    return this.daysSinceLastCleaning(targetDate) > threshold;
  }

  daysSinceLastCleaning(targetDate) {
    return this._lastCleaned.until(targetDate, ChronoUnit.DAYS);
  }
}

// client
const scrolls = aDocument
  .map(record => new Scroll(
    record.id,
    LocalDate.parse(record.lastCleaned),
    record.catalogData.id,
    catalog
  ));

```

# 2. Refs
All code samples comes from "Refactoring: Improving the Design of Existing Code" M. Fowler 2nd Edition
