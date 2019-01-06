<!-- TOC -->

- [1. Data mapper](#1-data-mapper)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->

# 1. Data mapper

Use when you need more control over entities/models/business logic

## 1.1. Code sample
```
class Entity {}

class Message extends Entity {
  constructor(dataReceived, message) {
    super();
    this.dataReceived = dataReceived;
    this.message = message;
  }
}

class Staffer extends Entity {
  constructor(email, categories) {
    super();
    this.email = email;
    this.categories = categories;
  }

  isAvailableForCategory(category) {
    return this.categories.indexOf(category) !== -1;
  }
}

const STATUS_OPEN = 'open';
const STATUS_CLOSED = 'closed';
const STATUSES = [STATUS_OPEN, STATUS_CLOSED];

class Ticket extends Entity {
  constructor(subject, status = STATUS_OPEN, category, messages = []) {
    super();
    this.subject = subject;
    this.updateStatus(status);
    this.category = category;
    this.messages = [];
    messages.forEach(msg => this.addMessage(msg));
  }

  updateStatus(status) {
    status = status.toLowerCase();
    if (!STATUSES.includes(status)) {
      throw new TypeError(`invalid status ${status}`);
    }
    this.status = status;
  }

  addMessage(message) {
    if (this.status === STATUS_CLOSED) {
      this.status = STATUS_OPEN;
    }
    this.messages.push(message);
  }

  assignStaffer(staffer) {
    if (!staffer.isAvailableForCategory(this.category)) {
      throw new Error(`staffer cannot be assigned to ticket of category ${this.category}`);
    }
    this.staffer = staffer;
  }
}

class Repository {}

class TicketRepository extends Repository {
  save(ticket) {
    this.db.execute('INSERT INTO ...');
  }
}

const message1 = new Message(new Date(), 'msg 1');
const message2 = new Message(new Date(), 'msg 2');
const ticket = new Ticket('ticket', STATUS_CLOSED, 'fizz', [message1, message2]);
const staffer = new Staffer('foo@bar', ['fizz', 'buzz']);
ticket.assignStaffer(staffer);
console.log(ticket);

const ticketRepo = new TicketRepository();
ticketRepo->save(ticket);

```

## 1.2. Notes
- alternative to ActiveRecord
- use plain object to model business logic
- persistence logic is delegated to the repository
- makes interactions with models clearer
- entities make sure our data is correct

## 1.3. Refs
- https://fideloper.com/how-we-code
