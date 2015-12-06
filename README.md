# ES7 Proposal: The Pipeline Operator

*Note: Although the JS community is excited about this proposal, its syntax is somewhat controversial. See [Issue #4](https://github.com/mindeavor/es-pipeline-operator/issues/4) for alternative syntaxes and discussion.*

This proposal introduces a new operator `|>` similar to
  [F#](https://en.wikibooks.org/wiki/F_Sharp_Programming/Higher_Order_Functions#The_.7C.3E_Operator),
  [Elixir](https://www.safaribooksonline.com/library/view/programming-elixir/9781680500530/f_0057.html),
  and [Elm](https://edmz.org/design/2015/07/29/elm-lang-notes.html),
  as well as UNIX pipes. It's a backwards-compatible way of streamlining chained function calls in a readable, functional manner, and provides a practical alternative to extending built-in prototypes.

## Introduction

The pipeline operator is essentially a useful syntactic sugar on a function call with a single argument. In other words, `sqrt(64)` is equivalent to `64 |> sqrt`.

This allows for greater readability when chaining several functions together. For example, given the following functions:

```js
function doubleSay (str) {
  return str + ", " + str;
}
function capitalize (str) {
  return str[0].toUpperCase() + str.substring(1);
}
function exclaim (str) {
  return str + '!';
}
```

...the following invocations are equivalent:

```js
var result = exclaim(capitalize(doubleSay("hello")));
result //=> "Hello, hello!"

var result = "hello"
  |> doubleSay
  |> capitalize
  |> exclaim;

result //=> "Hello, hello!"
```

The pipeline operator does not just provide better readability – it also opens [new possibilities](#sample-usage-with-promises) for API design.

### Functions with Multiple Arguments

The pipeline operator does not need any special rules for functions with multiple arguments; JavaScript already has ways to handle such cases.

For example, given the following functions:

```js
function double (x) { return x + x; }
function add (x, y) { return x + y; }

function validateScore (score) {
  return Math.min(0, Math.max(100, score))
}
```

...you can use an arrow function to handle multi-argument functions (such as `add`):

```js
var person = { score: 75 };

var newScore = person.score
  |> double
  |> score => add(7, score)
  |> validateScore

newScore //=> 107

// As opposed to: var newScore = validateScore( add(7, double(person.score)) )
```

As you can see, because the pipe operator always pipes a single result value, it plays very nicely with the single-argument arrow function syntax. Also, because the pipe operator's semantics are pure and simple, it could be possible for JavaScript engines to optimize away the arrow function.

## Motivating Examples

### Object Decorators

Mixins via `Object.assign` are great, but sometimes you need something more advanced. A **decorator function** is a function that receives an existing object, adds to it (mutative or not), and then returns the result.

Decorator functions are useful when you want to share behavior across multiple kinds of objects. For example, given the following decorators:

```js
function greets (person) {
  person.greet = () => `${person.name} says hi!`;
  return person;
}
function ages (age) {
  return function (person) {
    person.age = age;
    person.birthday = function () { person.age += 1; };
    return person;
  }
}
function programs (favLang) {
  return function (person) {
    person.favLang = favLang;
    person.program = () => `${person.name} starts to write ${person.favLang}!`;
    return person
  }
}
```

...you can create multiple "classes" that share one or more behaviors:

```js
function Person (name, age) {
  return { name: name } |> greets |> ages(age)
}
function Programmer (name, age) {
  return { name: name }
    |> greets
    |> ages(age)
    |> programs('javascript')
}
```

### Validation

Validation is a great use case for pipelining functions. For example, given the following validators:

```js
function bounded (prop, min, max) {
  return function (obj) {
    if ( obj[prop] < min || obj[prop] > max ) throw Error('out of bounds');
    return obj;
  };
}
function format (prop, regex) {
  return function (obj) {
    if ( ! regex.test(obj[prop]) ) throw Error('invalid format')
    return obj;
  };
}
```

...we can use the pipeline operator to validate objects quite pleasantly:

```js
function createPerson (attrs) {
  attrs
    |> bounded('age', 1, 100)
    |> format('name', /^[a-z]$/i)
    |> Person.insertIntoDatabase
}
```

### Usage with Prototypes

Although the pipe operator operates well with functions that don't use `this`, it can still integrate nicely into current workflows:

```js

fetchPlayers()
  .then(function (players) {
    return players
      .filter( p => p.score > 100 )
      .map( p => fetchGames(p) )
      |> Promise.all
  })
  .then(function (playerGames) {
    // ...
  })

```

### Sample Usage with Promises

Although flipping the order of function & argument may seem small, it opens significant possibilities for API design:

```js
fetchPlayers()
  .then(players => players.filter( p => p.score > 100 ).map( fetchGames ))
  |> Promise.all
  |> then(processGames)
  |> catchError( ProcessError, err => [] )
  |> then( forEach(g => console.log(g)) )

function catchError (ErrorClass, handler) {
  return promise => promise.catch(catcher);

  function catcher (error) {
    if (error typeof ErrorClass) return handler(error);
    else throw error;
  }
}

function then (handler) {  return promise => promise.then(handler)  }
function forEach (fn) {  return array => array.map(fn)  }
```

This example is significant because we have added useful Promise functionality (`catchError` catches specific rejection errors) **without extending the Promise prototype itself**. If we wanted to add catchError-like functionality using ES6 and stay fluent, we would have to either *extend* Promise.prototype, or *re-implement* the Promise interface (as [bluebird](https://github.com/petkaantonov/bluebird) and others have done).
