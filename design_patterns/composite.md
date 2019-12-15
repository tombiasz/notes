<!-- TOC -->

- [1. Composite](#1-composite)
    - [1.1. Code sample](#11-code-sample)
    - [1.2 Refs](#12-refs)

<!-- /TOC -->
# 1. Composite

Compose objects into tree structures to represent part-whole hierarchies.
Composite lets clients treat individual objects and compositions of objects
uniformly.

## 1.1. Code sample

```javascript
class DomainObject {}

class Product extends DomainObject {
  constructor(name, price, tags) {
    super();
    this.name = name;
    this.price = price;
    this.tags = tags;
  }
}

class Tag extends DomainObject {
  constructor(name) {
    super();
    this.name = name;
  }
}

// component interface
class Serializer {
  toJSON() {
    throw new Error('not implemented')
  }
}

// leaf
class TagSerializer extends Serializer {
  constructor(tag) {
    super();
    this.tag = tag;
  }

  toJSON() {
    return {
      name: this.tag.name,
    }
  }
}

// composite
class TagListSerializer extends Serializer {
  constructor() {
    super();
    this.tags = [];
  }

  add(tag) {
    this.tags.push(tag);
  }

  total() {
    return this.tags.length;
  }

  toJSON() {
    return {
      meta: {
        total: this.total(),
      },
      data: this.tags.map(t => t.toJSON()),
    }
  }
}

// composite
class ProductSerializer extends Serializer {
  constructor(product) {
    super();
    this.product = product;
    this.tags = [];
  }

  addTag(tag) {
    this.tags.push(tag);
  }

  toJSON() {
    return {
      name: this.product.name,
      price: this.product.price,
      tags: this.tags.map(t => t.toJSON()),
    }
  }
}

// composite
class ProductListSerializer extends Serializer {
  constructor() {
    super();
    this.products = [];
  }

  add(product) {
    this.products.push(product);
  }

  total() {
    return this.products.length;
  }


  toJSON() {
    return {
      meta: {
        total: this.total(),
      },
      data: this.products.map(p => p.toJSON()),
    }
  }
}

const tag1 = new Tag('foo');
const tag2 = new Tag('bar');
const tag3 = new Tag('fizz');
const tag4 = new Tag('buzz');

const product1 = new Product('fib', 11235, [tag1, tag2]);
const product2 = new Product('pi', 3.1415, [tag3, tag4]);

const productList = [product1, product2];

const productListSerializer = new ProductListSerializer();

productList.forEach(product => {
  const tagsSerializer = new TagListSerializer();
  product.tags.map(t => tagsSerializer.add(new TagSerializer(t)));

  const productSerializer = new ProductSerializer(product);
  productSerializer.addTag(tagsSerializer);

  productListSerializer.add(productSerializer)
})

console.log(JSON.stringify(productListSerializer.toJSON()));

```

## 1.2 Refs
- https://en.wikipedia.org/wiki/Composite_pattern
