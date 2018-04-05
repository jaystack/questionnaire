# Bootcamp

## Purpose

The purpose of this bootcamp note is to gain practice in fullstack JavaScript developing included:

- Basic javascript language skills and functional patterns
- React application development
- Node development
- Rest service development in node
- Unit testing
- Docker basics
- Mongo basics
- AWS basics
- Continous integration

This way we will create a simple webapplication. After then we will create a node rest api with MongoDB, which could be used by the application. When the app and the api are ready, they will move in docker containers, and they well be able to deploy to AWS ECS.

## Javascript

Javascript is a single threaded asynchronous script langugage runned by interpreters, such as V8 or Chakra. The popularity of javascript based on that it's running on many platforms: in every browsers, on server side by NodeJS, in mobile applications by React Native, in IOT devices, etc. The other important benefit of javascript is its functional nature. The functional features, such as high order functions, allow us to implement asynchronous behaviours easy, eg.: `onClick` callbacks, HTTP request handlers, etc.

**Check your javascript skills by our [questionaire](https://community.risingstack.com/repatch-the-simplified-redux/)!**

## Functional programming

This topic is too wide to explain in a brief paragraph. As I mentioned above javascript is a very functional-powered language. Although this is not a [pure functional language](https://en.wikipedia.org/wiki/Purely_functional_programming), it has a lot of features that allow us to follow functional patterns. There are many articles and blog posts about functional patterns in javascript, I can also suggest one.

**Check our suggested [book](https://github.com/getify/Functional-Light-JS) about functional patterns!**

### Determinism

Functional programming's primary trait is the determinism. What it means? If I have a function, which provides always the same output on the same input independently of its environment, we call this function as pure function, and pure functions are deterministic.

The biggest part of a code is data transformation, and data transformations could be easily deterministic. I have some input data, and I want to create some output data. It is a simple projection between two data manifestation. The most simple computer program is a data transformation, and we can describe it as an data transformation or data projection. And the functions do this exactly:

![](stateless_projection.png)

Of course we are thinking in short functions, so a program cannot be a big giant functions. This is much more a function composition:

![](stateless_projections.png)

### Immutability

The base requirement of the deterministic behaviour is immutability. That means **we just create things in memory, but never change them!** If I create a variable

```js
let a = 1
```

and I modify this

```js
a = 2
```

then I changed its value in the memory. If any part of our code relies on this variable I influenced its running, and I can cause some unexpected behaviour of the program. And unexpected behaviour does not match with definition of determinism. In a strongly asynchronous environment I can never be sure which kind of tasks are running in what order. **Undeterminism and unexpected behaviours is the devil himself and causes the most of bugs!**

If I create an object

```js
const user = { name: 'John', age: 28 }
```

and I change a property

```js
user.age = 30
```

or I remove a property

```js
delete user.age
```

then I did the same. I changed something in the memory and I may caused unexpected crashes.

If I create an array

```js
const numbers = [10, 20, 30]
```

and I push something within it

```js
numbers.push(40)
```

or I replace an item

```js
numbers[2] = 40
```

or I delete an item

```js
delete numbers[2]
```

then I changed this array, and somebody might hate me.

I do the same when I create a `Date` object

```js
const date = new Date()
```

and I change it

```js
date.setHours(14)
```

There are some practice to handle data immutable:

#### Variables against constants

We never use `var` or `let`. We use only `const`. So we can say, we **do not deal with variables**. We **work only with constants**. Only in very-very sparse cases, when we use `let`. For example when we create some state in a closure. Notice, that redux's store is this case, where the `state` is defined by `let`.

#### Deal with objects

We do not change object properties. And we never delete any of them. Rather than it, we assign a new one:

```js
const user = { name: 'John', age: 28 }
const modifiedUser = { ...user, age: 30, gender: 'male' }
```

If I want to delete an item, I reduce a new one without this:

```js
Object.keys(object).reduce(
  (acc, key) => key !== unnecessaryKey { ...acc, [key]: object[key] } : acc)
  , {}
)
```

#### Deal with arrays

We do not push anything into arrays, and we never replace items or delete any kind of them. Rather than it, we assign a new one:

```js
const numbers = [10, 20, 30]
const extendedArray = [...numbers, 40]
```

For replacing an item we can use the `.map()` function:

```js
const numbers = [10, 20, 30, 40]
const replacedArray = numbers.map((number, i) => i === 2 ? 42 : number)
```

or

```js
users.map(user => user.id === 'abc' ? { ...user, age: 32 } : user)
```

For deleting an item we can use the `.filter()` function:

```js
users.filter(user => user.id !== 'abc')
```

For slicing arrays do not use `.splice()`. Rather than it just use `.slice()`:

```js
const firstTwoItems = [10, 20, 30, 40].slice(0, 2)
```

or use the rest operator:

```js
const [first, second, ...rest] = [10, 20, 30, 40]
```

#### Other useful array methods

##### `.some()`

The `.some()` implements a simple OR behavior on items with a predicate function:

```js
const numbers = [null, null, 10, null, 30]
const isThereValidNumbers = numbers.some(number => Number.isFinite(number)) // true
```

##### `.every()`

The `.every()` implements a simple AND behavior on items with a predicate function:

```js
const numbers = [null, null, 10, null, 30]
const everyNumberisValid = numbers.every(number => Number.isFinite(number)) // false
```

### The state

Why can we say, this is true only on the simple programs. Because in these cases the program has no state. The state brings the compilcation, and every application has some state. A webapplication obviously has state. A REST data service has definitly not, but only in an optimistic world. The real life always works with states.

Then how can we adapt our functional projecting pattern including the state. We can curve our previous figure to this:

![](stateful_program.png)

We separate our **STATE** from the program logic (**REDUCER**). We has an initial state: this is the initial input of the reducer, and in every turn the reducers output will be the next state of the application.

### Reducers

Of course this closed system does nothing on its own. It needs some triggering action from the outside world, such as a user click or a HTTP request, etc. So a reducer's signature is:

```
(state, action) -> nextState
```

>All of us know well the [redux](https://redux.js.org/)'s [reducers](https://redux.js.org/basics/reducers). Its pattern is the same. We have a global application state (this is the [store](https://redux.js.org/basics/store)), and we have a reducer. This is just one reducer. Every other reducer is combined within this one.

Generally true, that a reducer makes from the current state a next state by an action. So a reducer always has two input argument:

- the currently accumulated state
- and an action

#### Reducing arrays

Javascript arrays has a `.reduce()` method, which accepts two argument. The first one is a required reducer, and the second one is an optional initial state. If there is no initial state as the second argument, the initial state will be the first item of the array, and the reducing starts at the second item. The `.reduce()` method can transform an array to anything we want against the rest of array methods, such as `.map()` or `.filter()`. Using `.reduce()` method can replace any kind of `for` loops. So we can say, we can avoid using `for` loops. Think in reducer!

Check this example of summing array of numbers:

```js
const sum = [10, 20, 30].reduce((sum, num) => sum + num, 0)
```

The initial state is `0`. The `.reduce()` starts the reducing at the first item, if we have initial state. The rounds are:

| # | state | item | nextState |
|:-:|:-----:|:----:|:---------:|
| 1 |   0   |  10  |     10    |
| 2 |   10  |  20  |     30    |
| 3 |   30  |  30  |     60    |

The first state will be the initial state, and the last next state will be the result of the `.reduce()` method, so the result is `60`.

I can do the same without initial state:

```js
const sum = [10, 20, 30].reduce((sum, num) => sum + num)
```

Then the initial state will be the first item of the array and the reducing starts at the second item of the array:

| # | state | item | nextState |
|:-:|:-----:|:----:|:---------:|
| 1 |   10  |  20  |     30    |
| 2 |   30  |  30  |     60    |

There are some commonly useful reduce implementation, such as:

```js
const max = [42, 7, 13, 56].reduce((max, num) => num > max ? num : max);
```

```js
const min = [42, 7, 13, 56].reduce((min, num) => num < min ? num : min);
```

### Curry

In many cases we need to bind one or more argument from a function before we call it. Look at this example. We have a callback to handle various items:

```js
handleClick = (id, evt) => { ... }

...

users.map(({id, name}) => (
  <input value={name} onClick={(e) => this.handleClick(id, e)} />
))
```

But if we define handleClick with this signature:

```js
handleClick = id => evt => { ... }
```

We have a bit simpler callbacks in jsx:

```js
users.map(({id, name}) => (
  <input value={name} onClick={this.handleClick(id)} />
))
```

We can also use the `.bind()` method on functions:

```js
handleClick = (id, evt) => { ... }

...

users.map(({id, name}) => (
  <input value={name} onClick={this.handleClick.bind(id)} />
))
```

### Closures