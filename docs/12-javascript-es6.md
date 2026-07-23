# 12. JavaScript ES6+ - Complete Guide

## 🚀 Interview Frequency: 95% | Expected Questions: 20

---

## 1️⃣ Arrow Functions

### Syntax:

```javascript
// Traditional function
function add(a, b) {
  return a + b;
}

// Arrow function equivalent
const add = (a, b) => a + b;
const add = (a, b) => { return a + b; };  // With braces

// Single parameter (parentheses optional)
const square = x => x * x;
const square = (x) => x * x;

// No parameters
const greet = () => 'Hello';

// Returning object (need extra parentheses)
const createUser = (name, age) => ({ name, age });
// Without parentheses would be interpreted as code block
```

### Key Differences:

```javascript
// 1. Arrow functions don't have their own 'this'
const obj = {
  name: 'John',
  greet: function() {
    console.log(this.name);  // 'John' - function has own this
  },
  greetArrow: () => {
    console.log(this.name);  // undefined - arrow inherits outer this
  }
};

// 2. Arrow functions can't be used as constructors
const obj = { id: 1 };
const User1 = function(name) { this.name = name; };
new User1('John');  // Works

const User2 = (name) => { this.name = name; };
new User2('Jane');  // TypeError: User2 is not a constructor

// 3. Arrow functions don't have 'arguments' object
const fn1 = function() {
  console.log(arguments);  // [1, 2, 3]
};
fn1(1, 2, 3);

const fn2 = (...args) => {
  console.log(args);  // [1, 2, 3]
};
fn2(1, 2, 3);
```

---

## 2️⃣ Destructuring

### Array Destructuring:

```javascript
// Basic
const [a, b] = [1, 2];
console.log(a, b);  // 1, 2

// Skip elements
const [first, , third] = [1, 2, 3];
console.log(first, third);  // 1, 3

// Rest operator
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head, tail);  // 1, [2, 3, 4, 5]

// Default values
const [a = 10, b = 20] = [1];
console.log(a, b);  // 1, 20

// Swapping
let x = 1, y = 2;
[x, y] = [y, x];
console.log(x, y);  // 2, 1
```

### Object Destructuring:

```javascript
// Basic
const { name, age } = { name: 'John', age: 30 };
console.log(name, age);  // 'John', 30

// Rename
const { name: fullName, age: years } = { name: 'John', age: 30 };
console.log(fullName, years);  // 'John', 30

// Default values
const { name = 'Guest', city = 'Unknown' } = { name: 'John' };
console.log(name, city);  // 'John', 'Unknown'

// Nested
const { user: { name, profile: { age } } } = {
  user: { name: 'John', profile: { age: 30 } }
};
console.log(name, age);  // 'John', 30

// Rest
const { name, ...rest } = { name: 'John', age: 30, city: 'NYC' };
console.log(rest);  // { age: 30, city: 'NYC' }

// Function parameters
function greet({ name, age = 18 }) {
  console.log(`${name} is ${age}`);
}
greet({ name: 'John', age: 30 });  // 'John is 30'
greet({ name: 'Jane' });  // 'Jane is 18'
```

---

## 3️⃣ Template Literals

```javascript
const name = 'John';
const age = 30;

// Traditional concatenation
const msg1 = 'Hello ' + name + ', you are ' + age + ' years old';

// Template literal
const msg2 = `Hello ${name}, you are ${age} years old`;

// Multiline
const html = `
  <div>
    <h1>${name}</h1>
    <p>Age: ${age}</p>
  </div>
`;

// Expressions
const result = `2 + 2 = ${2 + 2}`;
console.log(result);  // '2 + 2 = 4'

// Tagged templates
function tag(strings, ...values) {
  console.log(strings);  // ['Hello ', ', you are ', ' years old']
  console.log(values);   // ['John', 30]
}
tag`Hello ${name}, you are ${age} years old`;
```

---

## 4️⃣ Spread Operator

```javascript
// Array spreading
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2];
console.log(combined);  // [1, 2, 3, 4, 5, 6]

// Copy array
const original = [1, 2, 3];
const copy = [...original];

// Object spreading
const obj1 = { name: 'John', age: 30 };
const obj2 = { city: 'NYC', country: 'USA' };
const merged = { ...obj1, ...obj2 };
console.log(merged);
// { name: 'John', age: 30, city: 'NYC', country: 'USA' }

// Override properties
const user = { ...obj1, age: 31 };  // age becomes 31

// Function arguments
const numbers = [1, 2, 3];
Math.max(...numbers);  // Same as Math.max(1, 2, 3)
```

---

## 5️⃣ Default Parameters

```javascript
// Traditional way
function greet(name) {
  name = name || 'Guest';
  console.log('Hello ' + name);
}

// ES6 default parameters
function greet(name = 'Guest') {
  console.log(`Hello ${name}`);
}

greet();  // 'Hello Guest'
greet('John');  // 'Hello John'

// Multiple defaults
function createUser(name = 'Unknown', age = 18, city = 'Unknown') {
  return { name, age, city };
}

// Defaults with expressions
function createId(prefix = 'ID', timestamp = Date.now()) {
  return `${prefix}-${timestamp}`;
}

// Defaults with destructuring
function displayUser({ name = 'Guest', age = 18 } = {}) {
  console.log(`${name} is ${age}`);
}
displayUser();  // 'Guest is 18'
displayUser({ name: 'John' });  // 'John is 18'
```

---

## 6️⃣ Promises & Async/Await

### Promises:

```javascript
// Create promise
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('Success!');
    // or reject(new Error('Failed!'));
  }, 1000);
});

// Consume promise
myPromise
  .then(result => console.log(result))  // 'Success!'
  .catch(error => console.error(error))
  .finally(() => console.log('Done'));

// Promise.all
Promise.all([promise1, promise2, promise3])
  .then(results => console.log(results))  // All succeeded
  .catch(error => console.error(error));   // Any failed

// Promise.race
Promise.race([promise1, promise2])
  .then(result => console.log(result));  // First to complete
```

### Async/Await:

```javascript
// Async function returns a promise
async function fetchUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    return user;
  } catch (error) {
    console.error('Error:', error);
    return null;
  }
}

// Usage
fetchUser(1).then(user => console.log(user));

// Or with await
async function main() {
  const user = await fetchUser(1);
  console.log(user);
}
main();

// Multiple awaits
async function getFullUserData(id) {
  const user = await fetch(`/api/users/${id}`).then(r => r.json());
  const posts = await fetch(`/api/users/${id}/posts`).then(r => r.json());
  const comments = await fetch(`/api/users/${id}/comments`).then(r => r.json());
  return { user, posts, comments };
}

// Parallel awaits
async function getFullUserDataParallel(id) {
  const [user, posts, comments] = await Promise.all([
    fetch(`/api/users/${id}`).then(r => r.json()),
    fetch(`/api/users/${id}/posts`).then(r => r.json()),
    fetch(`/api/users/${id}/comments`).then(r => r.json())
  ]);
  return { user, posts, comments };
}
```

---

## 7️⃣ Modules (Import/Export)

### Export:

```javascript
// math.js

// Named export
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

export function multiply(a, b) {
  return a * b;
}

// Default export
export default class Calculator {
  static add(a, b) { return a + b; }
}

// Re-export
export { add as addition, subtract } from './math';
```

### Import:

```javascript
// Import named exports
import { add, subtract } from './math';
const result = add(5, 3);  // 8

// Import default
import Calculator from './math';
Calculator.add(5, 3);

// Import both
import Calculator, { add, subtract } from './math';

// Rename import
import { add as addition } from './math';
addition(5, 3);

// Import all as namespace
import * as math from './math';
math.add(5, 3);

// Dynamic import
const module = await import('./math');
module.add(5, 3);
```

---

## 8️⃣ Classes

```javascript
// Class definition
class User {
  // Constructor
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  // Method
  greet() {
    return `Hello, I'm ${this.name}`;
  }
  
  // Getter
  get info() {
    return `${this.name} (${this.age})`;
  }
  
  // Setter
  set info(value) {
    [this.name, this.age] = value.split(',');
  }
  
  // Static method
  static create(name, age) {
    return new User(name, age);
  }
}

// Inheritance
class Admin extends User {
  constructor(name, age, permissions) {
    super(name, age);  // Call parent constructor
    this.permissions = permissions;
  }
  
  // Override method
  greet() {
    return super.greet() + ' [Admin]';
  }
}

const user = new User('John', 30);
console.log(user.greet());  // "Hello, I'm John"

const admin = new Admin('Jane', 28, ['read', 'write']);
console.log(admin.greet());  // "Hello, I'm Jane [Admin]"
```

---

## 9️⃣ Array Methods

```javascript
const numbers = [1, 2, 3, 4, 5];

// map: transform each element
const doubled = numbers.map(n => n * 2);  // [2, 4, 6, 8, 10]

// filter: keep elements that pass test
const evens = numbers.filter(n => n % 2 === 0);  // [2, 4]

// reduce: combine into single value
const sum = numbers.reduce((acc, n) => acc + n, 0);  // 15
const product = numbers.reduce((acc, n) => acc * n, 1);  // 120

// find: get first match
const found = numbers.find(n => n > 3);  // 4

// findIndex: get index of first match
const index = numbers.findIndex(n => n > 3);  // 3

// some: test if any element passes
const hasEven = numbers.some(n => n % 2 === 0);  // true

// every: test if all elements pass
const allPositive = numbers.every(n => n > 0);  // true

// includes: check if value exists
const hasThree = numbers.includes(3);  // true

// sort
const sorted = [3, 1, 4, 1, 5].sort((a, b) => a - b);  // [1, 1, 3, 4, 5]

// Complex example
const users = [
  { id: 1, name: 'John', age: 30 },
  { id: 2, name: 'Jane', age: 25 },
  { id: 3, name: 'Bob', age: 35 }
];

// Get names of users over 25
const names = users
  .filter(u => u.age > 25)
  .map(u => u.name);
// ['John', 'Bob']

// Calculate total age
const totalAge = users.reduce((sum, u) => sum + u.age, 0);  // 90
```

---

## Interview Tips:

✅ **Know arrow function differences from regular functions**

✅ **Understand destructuring for arrays and objects**

✅ **Know spread operator use cases**

✅ **Master async/await over promises**

✅ **Know array methods (map, filter, reduce)**

✅ **Understand classes and inheritance**

✅ **Know module import/export**

---

📖 **Next Topic:** [Git & Version Control](./13-git.md)
