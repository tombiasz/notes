<!-- TOC -->

- [1. Repository design pattern](#1-repository-design-pattern)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->

# 1. Repository design pattern

## 1.1. Code sample
```
class BaseRepository { // interface
  getAll() {
    throw new Error('not implemented')
  }

  save(obj) {
    throw new Error('not implemented')
  }

  getById(id) {
    throw new Error('not implemented')
  }
}

class ArrayRepository extends BaseRepository {
  constructor() {
    super()
    this.storage = [];
  }

  getAll() {
    return this.storage;
  }

  save(obj) {
    this.storage.push(obj);
    return obj.id;
  }

  getById(id) {
    return this.storage.find(obj => obj.id === id);
  }
}

class MapRepository extends BaseRepository {
  constructor() {
    super()
    this.storage = new Map();
  }

  getAll() {
    return Array.from(this.storage.values());
  }

  save(obj) {
    this.storage.set(obj.id, obj);
    return obj.id;
  }

  getById(id) {
    return this.storage.get(id);
  }
}

const obj1 = { id: 1, name: 'foo'};
const obj2 = { id: 2, name: 'bar'};
const obj3 = { id: 3, name: 'fizz'};
const obj4 = { id: 4, name: 'buzz'};

const arrayRepository = new ArrayRepository();
arrayRepository.save(obj1)
arrayRepository.save(obj2)
arrayRepository.save(obj3)
arrayRepository.save(obj4)
console.log(arrayRepository.getById(2));
console.log(arrayRepository.getAll());

const mapRepository = new MapRepository();
mapRepository.save(obj1)
mapRepository.save(obj2)
mapRepository.save(obj3)
mapRepository.save(obj4)
console.log(mapRepository.getById(1));
console.log(mapRepository.getAll());
```

## 1.2. Notes
 - create(attrs) does not belong to repository. Repository is a collection not
 a factory
 - implementation can be easily swap if using common interface which can be
 beneficial in testing (swap DbRepository with ArrayRepository)
 - repository interfaces (here BaseRepository) belong to the domain-layer
 - the implementation of repositories belong to the application-service layer

## 1.3. Refs
- http://shawnmc.cool/the-repository-pattern
