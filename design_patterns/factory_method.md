<!-- TOC -->

- [1. Factory method](#1-factory-method)
  - [1.1 Code sample](#11-code-sample)
  - [1.2 Refs](#12-refs)

<!-- /TOC -->
# 1. Factory method

Defines an interface for creating a single object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.

## 1.1 Code sample
```
class BaseShipping {
  constructor(order) {
    this.order = order;
  }

  price() {
    throw new Error('not implemented');
  }
}

class StandardShipping extends BaseShipping {
  basePriceThreshold() {
    return 10;
  }

  elevatedPriceThreshold() {
    return 20;
  }

  basePrice() {
    return 20;
  }

  elevatedPrice() {
    return 30;
  }

  price() {
    const count = this.order.totalItems();

    if(count <= this.basePriceThreshold()) {
      return this.basePrice();
    }

    if(count <= this.elevatedPriceThreshold()) {
      return this.elevatedPrice();
    }

    throw new Error('too many items for standard shipping');
  }
}

class ExpressShipping extends BaseShipping {
  basePriceThreshold() {
    return 15;
  }

  basePrice() {
    return 30;
  }

  elevatedPrice(itemCount) {
    return this.basePrice() + (itemCount - this.basePriceThreshold() * 2);
  }

  price() {
    const count = this.order.totalItems();

    if(count <= this.basePriceThreshold()) {
      return this.basePrice;
    }

    return this.elevatedPrice(count);
  }
}

class Order {
  constructor(data) {
    this.data = data;
  }

  // factory method
  shipping() {
    throw new Error('not implemented');
  }

  totalItems() {
    return this.data.reduce((total, { qty }) => total + qty, 0);
  }

  orderPrice() {
    return this.data.reduce((total, { price, qty }) => total + (price * qty), 0);
  }

  totalPrice() {
    return this.orderPrice() + this.shipping().price();
  }
}

class StandardOrder extends Order {
  shipping() {
    return new StandardShipping(this);
  }
}

class PremiumOrder extends Order {
  shipping() {
    return new ExpressShipping(this);
  }
}


const orderData = [
  { item: 'item 1', qty: 1, price: 10 },
  { item: 'item 2', qty: 2, price: 5 },
  { item: 'item 3', qty: 4, price: 5 },
  { item: 'item 4', qty: 8, price: 10 },
  { item: 'item 5', qty: 12, price: 10 },
];

const standard = new StandardOrder(orderData);
const premium = new PremiumOrder(orderData);

console.log(standard.totalPrice()); // throws error
console.log(premium.totalPrice());
```

## 1.2 Refs
- https://en.wikipedia.org/wiki/Factory_method_pattern
