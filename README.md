# ES7 Proposal: The Pipeline Operator

This proposal introduces a new operator `|>` similar to
  [F#](https://en.wikibooks.org/wiki/F_Sharp_Programming/Higher_Order_Functions#The_.7C.3E_Operator),
  [OCaml](http://caml.inria.fr/pub/docs/manual-ocaml/libref/Pervasives.html#VAL%28|%3E%29),
  [Elixir](https://www.safaribooksonline.com/library/view/programming-elixir/9781680500530/f_0057.html),
  [Elm](https://edmz.org/design/2015/07/29/elm-lang-notes.html),
  [Julia](http://docs.julialang.org/en/release-0.4/stdlib/base/?highlight=|%3E#Base.|%3E),
  and [LiveScript](http://livescript.net/#piping),
  as well as UNIX pipes. It's a backwards-compatible way of streamlining chained function calls in a readable, functional manner, and provides a practical alternative to extending built-in prototypes.

## Introduction

The pipeline operator is essentially a useful syntactic sugar on a function call with a single argument. In other words, `sqrt(64)` is equivalent to `64 |> sqrt()`.

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
let result = exclaim(capitalize(doubleSay("hello")));
result //=> "Hello, hello!"

let result = "hello"
  |> doubleSay()
  |> capitalize()
  |> exclaim();

result //=> "Hello, hello!"
```

The pipeline operator does not just provide better readability – it also opens [new possibilities](#sample-usage-with-promises) for API design.

### Functions with Multiple Arguments

The pipeline operator shines in functional-style programming. Because of this, it has a form of **partial application** built in. For example, take the following functions:

```js
function double (x) { return x + x; }
function add (x, y) { return x + y; }

function boundScore (min, max, score) {
  return Math.max(0, Math.min(100, score));
}
```

You can use multi-parameter functions (like `add` and `boundScore`) with no additional effort:

```js
let person = { score: 25 };

let newScore = person.score
  |> double()
  |> add(7)
  |> boundScore(0, 100);

newScore //=> 57

// As opposed to:
// let newScore = boundScore( 0, 100, add(7, double(person.score)) )
```

Let's review what just happened:

- `person.score` got piped into `double()`, resulting in `double(person.score)`
- That got piped into `add(7)` **as the last argument**, resulting in `add(7, double(person.score))`
- In the same manner, *that* got piped into `boundScore(0, 100)` as the last argument, resulting in the commented code you see above.

Piping to the last argument allows easy use of multiple-parameter functions, without the overhead of currying or "actual" partial application. Also, because the pipeline operator pipes to the last argument of an invocation, it plays well with functions that are partial-application-friendly, increasing the reusability of such functions.

## Motivating Examples

### Object Decorators

Mixins via `Object.assign` are great, but sometimes you need something more advanced. A **decorator function** is a function that receives an existing object, adds to it (mutative or not), and then returns the result.

Decorator functions are useful when you want to share behavior across multiple kinds of objects. For example, given the following decorators:

```js
function greets (person) {
  person.greet = () => `${person.name} says hi!`;
  return person;
}
function ages (age, person) {
  person.age = age;
  person.birthday = () => (person.age += 1);
  return person;
}
function programs (favLang, person) {
  person.favLang = favLang;
  person.program = () => `${person.name} starts to write ${person.favLang}!`;
  return person;
}
```

...you can create multiple "classes" that share one or more behaviors:

```js
function Person (name, age) {
  return { name: name } |> greets() |> ages(age);
}
function Programmer (name, age) {
  return { name: name }
    |> greets()
    |> ages(age)
    |> programs('javascript');
}
```

### Validation

Validation is a great use case for pipelining functions. For example, given the following validators:

```js
function bounded (prop, min, max, obj) {
  if ( obj[prop] < min || obj[prop] > max ) throw Error('out of bounds');
  return obj;
}
function format (prop, regex, obj) {
  if ( ! regex.test(obj[prop]) ) throw Error('invalid format')
  return obj;
}
```

...we can use the pipeline operator to validate objects quite pleasantly:

```js
function createPerson (attrs) {
  attrs
    |> bounded('age', 1, 100)
    |> format('name', /^[a-z]$/i)
    |> Person.insertIntoDatabase();
}
```

### Usage with Prototypes

Although the pipe operator operates well with functions that don't use `this`, it can still integrate nicely into current workflows:

```js
import Lazy from 'lazy.js';

getAllPlayers()
  .filter( p => p.score > 100 )
  .sort()
  |> Lazy() // <-- This invocation will be populated!
  .map( p => p.name )
  .take(5)
  |> renderLeaderboard('#my-div');

// Equivalent to:
// renderLeaderboard(
//   '#my-div',
//   Lazy( getAllPlayers().filter( p => p.score > 100).sort() )
//   .map( p => p.name )
//   .take(5)
// }
```

As you can see, the pipeline operator populates the very first invocation it finds with its argument. This is critical for allowing a fluent programming style, interwieving function calls with method calls.

### Sample Usage with Promises

Although flipping the order of function & argument may seem small, it opens significant possibilities for API design:

```js
fetchPlayers()
  .then( players => players.filter( p => p.score > 100 ).map( fetchGames ) )
  .then( games => Promise.all(games) )
  .then( processGames )
  |> catchError( ProcessError, err => [] )
  .then( games => games.forEach(g => console.log(g)) );


function catchError (ErrorClass, handler, promise) {
  return promise.catch( error =>
    (error instanceof ErrorClass) ? handler(error) : (throw error)
  );
}
```

This example is significant because we have added useful Promise functionality (`catchError` catches specific rejection errors) **without extending the Promise prototype itself**. If we wanted to add catchError-like functionality using ES6 and stay fluent, we would have to either *extend* Promise.prototype, or *re-implement* the Promise interface (as [bluebird](https://github.com/petkaantonov/bluebird) and others have done).


## Syntax Behavior

The pipeline operator is quick – it matches the first invocation it sees. If there is no invocation, a Syntax Error is thrown.

```js
//
// Standard behavior is to fill in first available invocation
//
x |> f()     //=> f(x)
x |> f(10)   //=> f(10, x)
x |> f(10)() //=> f(10, x)()
x |> f       //=> Syntax Error

x |> App.f()   //=> App.f(x)
x |> App.f(20) //=> App.f(20, x)
x |> App.f     //=> Syntax Error

x |> App.f().g() //=> App.f(x).g()
x |> App.f().g   //=> App.f(x).g
x |> App.f.g()   //=> App.f.g(x)
x |> App.f.g     //=> Syntax Error


//
// Surrounding parenthesis indicate direct invocation
// (Necessary?)
//
x |> (f)       //=> (f)(x)
x |> (f(10))   //=> (f(10))(x)
x |> (f(10))() //=> (f(10))(x)

//
// Special case: Function literals
//
x |> function (arg) { ... } //=> (function (arg) { ... })(x)

//
// Special case:
// If single-expression arrow functions are
// used in the parenthesis manner,
// they can be optimized out by the compiler.
//
x |> (n => f(n, 30)) //=> f(x, 20)

arg
  |> (x => x + 1)
  |> (y => y + 2)

//=> ((arg + 1) + 2)

x |> arg => arg + 1 //=> Syntax Error
```
