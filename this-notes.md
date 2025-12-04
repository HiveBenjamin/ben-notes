# Understanding 'this', call(), bind(), and apply() in JavaScript

A comprehensive guide to context binding in JavaScript

---

## Table of Contents

- [The 'this' Keyword](#the-this-keyword)
  - [Basic Behavior](#basic-behavior)
  - [Arrow Functions](#arrow-functions)
  - [Callbacks](#callbacks)
- [call()](#call)
- [bind()](#bind)
- [apply()](#apply)
- [Summary Comparison](#summary-comparison)
- [Key Takeaways](#key-takeaways)
- [Practical Examples](#practical-examples)

---

## The 'this' Keyword

### Basic Behavior

The `this` keyword is used to identify the context where a piece of code should be run. In objects, `this` refers to the object that the method is attached to.

Its value is very context dependent, changing based on where and how it is used.

**Quick notes:**

- `this` refers to the global object when in non-strict mode
- `undefined` for strict mode
- We use non-strict mode

**Example: Basic 'this' in object methods**

```javascript
const obj = {
  name: 'ben',
  func() {
    return this.name
  },
}

console.log(obj.func()) // "ben"
```

### Arrow Functions

In arrow functions, `this` is the value of its scope - no new binding for `this` is created. Arrow functions are effectively automatically bound, meaning `this` is bound to what the arrow function's context was when created.

**Example: Arrow functions and lexical 'this'**

```javascript
const objArrow = {
  getThisGetter() {
    const getter = () => this
    return getter
  },
}

const fn = objArrow.getThisGetter()
console.log(fn() === objArrow) // true
```

If trying to assign it to a const without calling it, you instead get the globalThis:

```javascript
const fn2 = objArrow.getThisGetter
console.log(fn2()() === globalThis) // true
```

### Callbacks

When passing methods as callbacks, `this` can lose its context.

**Example: Lost context in callbacks**

```javascript
const objCallback = {
  name: 'ben',
  greet() {
    console.log(`Hello, ${this.name}`)
  },
}

setTimeout(objCallback.greet, 1000) // Hello, undefined
```

The method loses its `this` binding because it's called as a standalone function, not as objCallback.greet().

**SOLUTIONS:**

- **Solution 1: Use arrow function wrapper**

  ```javascript
  setTimeout(() => objCallback.greet(), 1000) // Hello, ben
  ```

- **Solution 2: Use bind()**

  ```javascript
  setTimeout(objCallback.greet.bind(objCallback), 1000) // Hello, ben
  ```

- **Solution 3: Use arrow function method (lexical this)**

  ```javascript
  const objArrowMethod = {
    name: 'ben',
    greet: () => {
      console.log(`Hello, ${this.name}`) // Inherits 'this' from outer scope
    },
  }
  ```

---

## call()

### What It Is

`call()` is a method that calls a function with a given `this` value and arguments provided individually. The first argument is the value you want `this` to be, followed by any arguments the function needs.

**Syntax:** `function.call(thisArg, arg1, arg2, ...)`

### Common Alternatives

We typically just pass parameters directly to functions:

```javascript
const greet = (animal: string, sleepDuration: string) => {
  console.log(animal, "typically sleep between", sleepDuration)
}

const animalObj = {
  animal: "cats",
  sleepDuration: "12 and 16 hours"
}

greet(animalObj.animal, animalObj.sleepDuration)
// cats typically sleep between 12 and 16 hours
```

### When We Might Need To Use It

- **Use case 1: Borrowing methods from other objects**

  ```javascript
  function greetWithThis() {
    console.log(this.animal, 'typically sleep between', this.sleepDuration)
  }

  const catObj = {
    animal: 'cats',
    sleepDuration: '12 and 16 hours',
  }

  greetWithThis.call(catObj) // cats typically sleep between 12 and 16 hours
  ```

- **Use case 2: Function chaining/inheritance**

  ```javascript
  function Product(name: string, price: number) {
    this.name = name
    this.price = price
  }

  function Food(name: string, price: number) {
    Product.call(this, name, price) // Inherit from Product
    this.category = "food"
  }

  const cheese = new Food("feta", 5)
  console.log(cheese.name) // feta
  ```

- **Use case 3: Using array methods on array-like objects**

  ```javascript
  function logArguments() {
    // 'arguments' is array-like but not an actual array
    const args = Array.prototype.slice.call(arguments)
    console.log(args)
  }

  logArguments(1, 2, 3) // [1, 2, 3]
  ```

---

## bind()

### What It Is

`bind()` creates a new function with `this` permanently bound to the value you specify. Unlike `call()` and `apply()`, `bind()` doesn't execute the function immediately - it returns a new function that can be called later.

**Syntax:** `function.bind(thisArg, arg1, arg2, ...)`

### Common Alternatives

Arrow functions are the modern alternative for maintaining `this` context:

```javascript
// Old way with bind
class ComponentOld {
  constructor() {
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick() {
    console.log(this)
  }
}

// Modern way with arrow functions
class ComponentModern {
  handleClick = () => {
    console.log(this)
  }
}
```

### When We Might Need To Use It

- **Use case 1: Event handlers (preserving 'this' context)**

  ```javascript
  const button = document.querySelector('button')

  const counterObj = {
    count: 0,
    increment() {
      this.count++
      console.log(this.count)
    },
  }

  // Without bind, 'this' would be the button element
  button?.addEventListener('click', counterObj.increment.bind(counterObj))
  ```

- **Use case 2: Callbacks that lose context**

  ```javascript
  const user = {
    name: 'Ben',
    greet() {
      console.log(`Hello, ${this.name}`)
    },
  }

  setTimeout(user.greet.bind(user), 1000) // Hello, Ben
  ```

- **Use case 3: Partial application (pre-setting arguments)**

  ```javascript
  function multiply(a: number, b: number) {
    return a * b
  }

  const double = multiply.bind(null, 2) // Pre-set first argument to 2
  console.log(double(5)) // 10
  console.log(double(10)) // 20
  ```

- **Use case 4: React class components (before hooks)**

  ```javascript
  class MyComponent extends React.Component {
    constructor(props) {
      super(props)
      this.handleClick = this.handleClick.bind(this)
    }

    handleClick() {
      this.setState({ clicked: true })
    }

    render() {
      return <button onClick={this.handleClick}>Click me</button>
    }
  }
  ```

---

## apply()

### What It Is

`apply()` is almost identical to `call()`, but it takes arguments as an array instead of individually. The first argument is still the `this` value, but the second is an array of arguments.

**Syntax:** `function.apply(thisArg, [argsArray])`

### Common Alternatives

The spread operator (`...`) is the modern alternative:

```javascript
const numbers = [5, 6, 2, 3, 7]

// Old way with apply
const maxOld = Math.max.apply(null, numbers)

// Modern way with spread
const maxModern = Math.max(...numbers)
```

### When We Might Need To Use It

- **Use case 1: Working with variable-length argument lists**

  ```javascript
  function sum() {
    return Array.prototype.reduce.call(arguments, (a, b) => a + b)
  }

  const numberArray = [1, 2, 3, 4, 5]
  console.log(sum.apply(null, numberArray)) // 15
  ```

- **Use case 2: Appending arrays (before spread operator)**

  ```javascript
  const array1 = [1, 2, 3]
  const array2 = [4, 5, 6]

  // Old way
  Array.prototype.push.apply(array1, array2)

  // Modern way
  array1.push(...array2)
  ```

- **Use case 3: Using Math functions with arrays**

  ```javascript
  const numberSet = [5, 6, 2, 3, 7]

  const maxValue = Math.max.apply(null, numberSet) // 7
  const minValue = Math.min.apply(null, numberSet) // 2

  // Modern equivalent
  const maxValueModern = Math.max(...numberSet)
  const minValueModern = Math.min(...numberSet)
  ```

- **Use case 4: Forwarding arguments to another function**

  ```javascript
  function wrapper() {
    console.log('Before')
    originalFunction.apply(this, arguments)
    console.log('After')
  }
  ```

---

## Summary Comparison

| Method  | Executes immediately? | Arguments format                   | Primary use case                  |
| ------- | --------------------- | ---------------------------------- | --------------------------------- |
| call()  | ✅ Yes                | Individual: call(this, arg1, arg2) | Borrow methods, inheritance       |
| apply() | ✅ Yes                | Array: apply(this, [arg1, arg2])   | Variable-length args, Math ops    |
| bind()  | ❌ No (returns fn)    | Individual: bind(this, arg1, arg2) | Event handlers, permanent binding |

---

## Key Takeaways

1.  `this` is context-dependent
    Its value changes based on how and where a function is called

2.  Arrow functions inherit `this`
    They use lexical binding from their enclosing scope

3.  `call()` and `apply()` execute immediately
    Both execute functions with a specified `this` value

4.  `bind()` creates a new function
    It returns a function with a permanently bound `this` value

5.  Modern JavaScript provides cleaner alternatives
    Arrow functions and spread operators handle most use cases

---

## Practical Examples

### Example 1: Event Handling in React (Class Components)

```javascript
class TodoItem extends React.Component {
  constructor(props) {
    super(props)
    this.state = { completed: false }

    // Need to bind to access 'this.setState'
    this.handleToggle = this.handleToggle.bind(this)
  }

  handleToggle() {
    this.setState({ completed: !this.state.completed })
  }

  render() {
    return (
      <div onClick={this.handleToggle}>
        {this.state.completed ? '✓' : '○'} {this.props.text}
      </div>
    )
  }
}
```

### Example 2: Modern React with Arrow Functions (Functional Components)

```javascript
const TodoItemModern = ({ text }) => {
  const [completed, setCompleted] = useState(false)

  // Arrow function automatically binds 'this' (not needed in functional components)
  const handleToggle = () => {
    setCompleted(!completed)
  }

  return (
    <div onClick={handleToggle}>
      {completed ? '✓' : '○'} {text}
    </div>
  )
}
```

### Example 3: Method Borrowing with call()

```javascript
const person1 = {
  firstName: 'John',
  lastName: 'Doe',
  getFullName() {
    return `${this.firstName} ${this.lastName}`
  },
}

const person2 = {
  firstName: 'Jane',
  lastName: 'Smith',
}

// Borrow person1's method for person2
console.log(person1.getFullName.call(person2)) // Jane Smith
```

### Example 4: Partial Application with bind()

```javascript
const baseURL = "https://api.example.com"

function makeRequest(method: string, endpoint: string, data?: any) {
  console.log(`${method} ${baseURL}${endpoint}`, data)
}

// Create specialized functions
const get = makeRequest.bind(null, "GET")
const post = makeRequest.bind(null, "POST")

get("/users") // GET https://api.example.com/users
post("/users", { name: "Ben" }) // POST https://api.example.com/users { name: 'Ben' }
```

### Example 5: Array Methods with apply()

```javascript
const findMax = (numbers: number[]) => {
  // Old way
  const maxOld = Math.max.apply(null, numbers)

  // Modern way
  const maxModern = Math.max(...numbers)

  return { maxOld, maxModern }
}

console.log(findMax([1, 5, 3, 9, 2])) // { maxOld: 9, maxModern: 9 }
```

---
