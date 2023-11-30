# Pipe Operator (`|>`) for JavaScript

* **Stage**: 2
* **Champions**: J. S. Choi, James DiGioia, Ron Buckton, Tab Atkins-Bittner, \[list incomplete] <!-- Alpha order by first name, plz -->
* **Former champions**: Daniel Ehrenberg
* **[Specification][]**
* **[Contributing guidelines][]**
* **[Proposal history][]**
* **Babel plugin**: [Implemented in v7.15][Babel 7.15]. See [Babel documentation][].

(This document uses `%`
as the placeholder token for the topic reference.
This will ***almost certainly not be the final choice***;
see [the token bikeshedding discussion][token bikeshedding] for details.)

[specification]: http://tc39.github.io/proposal-pipeline-operator/
[Babel 7.15]: https://babeljs.io/blog/2021/07/26/7.15.0#hack-style-pipeline-operator-support-13191httpsgithubcombabelbabelpull13191-13416httpsgithubcombabelbabelpull13416
[Babel documentation]: https://babeljs.io/docs/en/babel-plugin-proposal-pipeline-operator
[token bikeshedding]: https://github.com/tc39/proposal-pipeline-operator/issues/91
[contributing guidelines]: https://github.com/tc39/proposal-pipeline-operator/blob/main/CONTRIBUTING.md
[proposal history]: https://github.com/tc39/proposal-pipeline-operator/blob/main/HISTORY.md

## Why a pipe operator
In the State of JS 2020 survey, the **fourth top answer** to
[“What do you feel is currently missing from
JavaScript?”](https://2020.stateofjs.com/en-US/opinions/#missing_from_js)
was a **pipe operator**. Why?

When we perform **consecutive operations** (e.g., function calls)
on a **value** in JavaScript,
there are currently two fundamental styles:
* passing the value as an argument to the operation
  (**nesting** the operations if there are multiple operations),
* or calling the function as a method on the value
  (**chaining** more method calls if there are multiple methods).

That is, `three(two(one(value)))` versus `value.one().two().three()`.
However, these styles differ much in readability, fluency, and applicability.

### Deep nesting is hard to read
The first style, **nesting**, is generally applicable –
it works for any sequence of operations:
function calls, arithmetic, array/object literals, `await` and `yield`, etc.

However, nesting is **difficult to read** when it becomes deep:
the flow of execution moves **right to left**,
rather than the left-to-right reading of normal code.
If there are **multiple arguments** at some levels,
reading even bounces **back and forth**:
our eyes must **jump left** to find a function name,
and then they must **jump right** to find additional arguments.
Additionally, **editing** the code afterwards can be fraught:
we must find the correct **place to insert** new arguments
among **many nested parentheses**.

<details>
<summary><strong>Real-world example</strong></summary>

Consider this [real-world code from React](https://github.com/facebook/react/blob/17.0.2/scripts/jest/jest-cli.js#L295).

```js
console.log(
  chalk.dim(
    `$ ${Object.keys(envars)
      .map(envar =>
        `${envar}=${envars[envar]}`)
      .join(' ')
    }`,
    'node',
    args.join(' ')));
```

This real-world code is made of **deeply nested expressions**.
In order to read its flow of data, a human’s eyes must first:

1. Find the **initial data** (the innermost expression, `envars`).
2. And then scan **back and forth** repeatedly from **inside out**
   for each data transformation,
   each one either an easily missed prefix operator on the left
   or a suffix operators on the right:

   1. `Object.keys()` (left side),
   2. `.map()` (right side),
   3. `.join()` (right side),
   4. A template literal (both sides),
   5. `chalk.dim()` (left side), then
   6. `console.log()` (left side).

As a result of deeply nesting many expressions
(some of which use **prefix** operators,
some of which use **postfix** operators,
and some of which use **circumfix** operators),
we must check **both left and right sides**
to find the **head** of **each expression**.

</details>

### Method chaining is limited
The second style, **method chaining**, is **only** usable
if the value has the functions designated as **methods** for its class.
This **limits** its applicability.
But **when** it applies, thanks to its postfix structure,
it is generally more usable and **easier** to read and write.
Code execution flows **left to right**.
Deeply nested expressions are **untangled**.
All arguments for a function call are **grouped** with the function’s name.
And editing the code later to **insert or delete** more method calls is trivial,
since we would just have to put our cursor in one spot,
then start typing or deleting one **contiguous** run of characters.

Indeed, the benefits of method chaining are **so attractive**
that some **popular libraries contort** their code structure
specifically to allow **more method chaining**.
The most prominent example is **[jQuery][]**, which
still remains the **most popular JS library** in the world.
jQuery’s core design is a single über-object with dozens of methods on it,
all of which return the same object type so that we can **continue chaining**.
There is even a name for this style of programming:
**[fluent interfaces][]**.

[jQuery]: https://jquery.com/
[fluent interfaces]: https://en.wikipedia.org/wiki/Fluent_interface

Unfortunately, for all of its fluency,
**method chaining** alone cannot accommodate JavaScript’s **other syntaxes**:
function calls, arithmetic, array/object literals, `await` and `yield`, etc.
In this way, method chaining remains **limited** in its **applicability**.

### Pipe operators combine both worlds
The pipe operator attempts to marry the **convenience** and ease of **method chaining**
with the wide **applicability** of **expression nesting**.

The general structure of all the pipe operators is
`value |>` <var>e1</var> `|>` <var>e2</var> `|>` <var>e3</var>,
where <var>e1</var>, <var>e2</var>, <var>e3</var>
are all expressions that take consecutive values as their parameters.
The `|>` operator then does some degree of magic to “pipe” `value`
from the lefthand side into the righthand side.

<details>
<summary><strong>Real-world example</strong>, continued</summary>

Continuing this deeply nested [real-world code from React][react/scripts/jest/jest-cli.js]:

```js
console.log(
  chalk.dim(
    `$ ${Object.keys(envars)
      .map(envar =>
        `${envar}=${envars[envar]}`)
      .join(' ')
    }`,
    'node',
    args.join(' ')));
```

…we can **untangle** it as such using a pipe operator
and a placeholder token (`%`) standing in for the previous operation’s value:

```js
Object.keys(envars)
  .map(envar => `${envar}=${envars[envar]}`)
  .join(' ')
  |> `$ ${%}`
  |> chalk.dim(%, 'node', args.join(' '))
  |> console.log(%);
```

Now, the human reader can **rapidly find** the **initial data**
(what had been the most innermost expression, `envars`),
then **linearly** read, from **left to right**,
each transformation on the data.

</details>

### Temporary variables are often tedious
One could argue that using **temporary variables**
should be the only way to untangle deeply nested code.
Explicitly naming every step’s variable
causes something similar to method chaining to happen,
with similar benefits to reading and writing code.

<details>
<summary><strong>Real-world example</strong>, continued</summary>

For example, using our previous modified
[real-world example from React][react/scripts/jest/jest-cli.js]:

```js
Object.keys(envars)
  .map(envar => `${envar}=${envars[envar]}`)
  .join(' ')
  |> `$ ${%}`
  |> chalk.dim(%, 'node', args.join(' '))
  |> console.log(%);
```

…a version using temporary variables would look like this:

```js
const envarString = Object.keys(envars)
  .map(envar => `${envar}=${envars[envar]}`)
  .join(' ');
const consoleText = `$ ${envarString}`;
const coloredConsoleText = chalk.dim(consoleText, 'node', args.join(' '));
console.log(coloredConsoleText);
```

</details>

But there are reasons why we encounter deeply nested expressions
in each other’s code **all the time in the real world**,
**rather than** lines of temporary variables.
And there are reasons why the **method-chain-based [fluent interfaces][]**
of jQuery, Mocha, and so on are still **popular**.

It is often simply too **tedious and wordy** to **write**
code with a long sequence of temporary, single-use variables.
It is arguably even tedious and visually noisy for a human to **read**, too.

If [**naming** is one of the **most difficult tasks** in programming][naming hard],
then programmers will **inevitably avoid naming** variables
when they perceive their benefit to be relatively small.

[naming hard]: https://martinfowler.com/bliki/TwoHardThings.html

### Reusing temporary variables is prone to unexpected mutation
One could argue that using a single **mutable variable** with a short name
would reduce the wordiness of temporary variables, achieving
similar results as with the pipe operator.

<details>
<summary><strong>Real-world example</strong>, continued</summary>

For example, our previous modified
[real-world example from React][react/scripts/jest/jest-cli.js]
could be re-written like this:
```js
let _;
_ = Object.keys(envars)
  .map(envar => `${envar}=${envars[envar]}`)
  .join(' ');
_ = `$ ${_}`;
_ = chalk.dim(_, 'node', args.join(' '));
_ = console.log(_);
```

</details>

But code like this is **not common** in real-world code.
One reason for this is that mutable variables can **change unexpectedly**,
causing silent bugs that are hard to find.
For example, the variable might be accidentally referenced in a closure.
Or it might be mistakenly reassigned within an expression.

<details>
<summary>Example code</summary>

```js
// setup
function one () { return 1; }
function double (x) { return x * 2; }

let _;
_ = one(); // _ is now 1.
_ = double(_); // _ is now 2.
_ = Promise.resolve().then(() =>
  // This does *not* print 2!
  // It prints 1, because `_` is reassigned downstream.
  console.log(_));

// _ becomes 1 before the promise callback.
_ = one(_);
```

This issue would not happen with the pipe operator.
The topic token cannot be reassigned, and
code outside of each step cannot change its binding.

```js
let _;
_ = one()
  |> double(%)
  |> Promise.resolve().then(() =>
    // This prints 2, as intended.
    console.log(%));

_ = one();
```

</details>

For this reason, code with mutable variables is also harder to read.
To determine what the variable represents at any given point,
you must to **search the entire preceding scope** for places where it is **reassigned**.

The topic reference of a pipeline, on the other hand, has a limited lexical scope,
and its binding is immutable within its scope.
It cannot be accidentally reassigned, and it can be safely used in closures.

Although the topic value also changes with each pipeline step,
we only scan the previous step of the pipeline to make sense of it,
leading to code that is easier to read.

### Temporary variables must be declared in statements
Another benefit of the pipe operator over sequences of assignment statements
(whether with mutable or with immutable temporary variables)
is that they are **expressions**.

Pipe expressions are expressions that can be directly returned,
assigned to a variable, or used in contexts such as JSX expressions.

Using temporary variables, on the other hand, requires sequences of statements.

<details>
<summary>Examples</summary>

<table>
<thead>
<th>Pipelines</th>
<th>Temporary Variables</th>
</thead>

<tbody>
<tr>
<td>

```js
const envVarFormat = vars =>
  Object.keys(vars)
    .map(var => `${var}=${vars[var]}`)
    .join(' ')
    |> chalk.dim(%, 'node', args.join(' '));
```

</td>
<td>

```js
const envVarFormat = (vars) => {
  let _ = Object.keys(vars);
  _ = _.map(var => `${var}=${vars[var]}`);
  _ = _.join(' ');
  return chalk.dim(_, 'node', args.join(' '));
}
```

</td>
</tr>
<tr>
<td>

```jsx
// This example uses JSX.
return (
  <ul>
    {
      values
        |> Object.keys(%)
        |> [...Array.from(new Set(%))]
        |> %.map(envar => (
          <li onClick={
            () => doStuff(values)
          }>{envar}</li>
        ))
    }
  </ul>
);
```

</td>
<td>

```js
// This example uses JSX.
let _ = values;
_= Object.keys(_);
_= [...Array.from(new Set(_))];
_= _.map(envar => (
  <li onClick={
    () => doStuff(values)
  }>{envar}</li>
));
return (
  <ul>{_}</ul>
);
```

</td>
</tr>
</tbody>
</table>

</details>

## Why the Hack pipe operator
There were **two competing proposals** for the pipe operator: Hack pipes and F# pipes.
(Before that, there **was** a [third proposal for a “smart mix” of the first two proposals][smart mix],
but it has been withdrawn,
since its syntax is strictly a superset of one of the proposals’.)

[smart mix]: https://github.com/js-choi/proposal-smart-pipelines/

The two pipe proposals just differ **slightly** on what the “magic” is,
when we spell our code when using `|>`.

**Both** proposals **reuse** existing language concepts:
Hack pipes are based on the concept of the **expression**,
while F# pipes are based on the concept of the **unary function**.

Piping **expressions** and piping **unary functions**
correspondingly have **small** and nearly **symmetrical trade-offs**.

### This proposal: Hack pipes
In the **Hack language**’s pipe syntax,
the righthand side of the pipe is an **expression** containing a special **placeholder**,
which is evaluated with the placeholder bound to the result of evaluating the lefthand side's expression.
That is, we write `value |> one(%) |> two(%) |> three(%)`
to pipe `value` through the three functions.

**Pro:** The righthand side can be **any expression**,
and the placeholder can go anywhere any normal variable identifier could go,
so we can pipe to any code we want **without any special rules**:

* `value |> foo(%)` for unary function calls,
* `value |> foo(1, %)` for n-ary function calls,
* `value |> %.foo()` for method calls,
* `value |> % + 1` for arithmetic,
* `value |> [%, 0]` for array literals,
* `value |> {foo: %}` for object literals,
* `` value |> `${%}` `` for template literals,
* `value |> new Foo(%)` for constructing objects,
* `value |> await %` for awaiting promises,
* `value |> (yield %)` for yielding generator values,
* `value |> import(%)` for calling function-like keywords,
* etc.

**Con:** Piping through **unary functions**
is **slightly more verbose** with Hack pipes than with F# pipes.
This includes unary functions
that were created by **[function-currying][] libraries** like [Ramda][],
as well as [unary arrow functions
that perform **complex destructuring** on their arguments][destruct]:
Hack pipes would be slightly more verbose
with an **explicit** function call suffix `(%)`.

(Complex destructuring of the topic value
will be easier when [do expressions][] progress,
as you will then be able to do variable assignment/destructuring
inside of a pipe body.)

[function-currying]: https://en.wikipedia.org/wiki/Currying
[Ramda]: https://ramdajs.com/
[destruct]: https://github.com/js-choi/proposal-hack-pipes/issues/4#issuecomment-817208635

### Alternative proposal: F# pipes
In the [**F# language**’s pipe syntax][F# pipes],
the righthand side of the pipe is an expression
that must **evaluate into a unary function**,
which is then **tacitly called**
with the lefthand side’s value as its **sole argument**.
That is, we write `value |> one |> two |> three` to pipe `value`
through the three functions.
`left |> right` becomes `right(left)`.
This is called [tacit programming or point-free style][tacit].

[F# pipes]: https://github.com/valtech-nyc/proposal-fsharp-pipelines
[tacit]: https://en.wikipedia.org/wiki/Tacit_programming

<details>
<summary><strong>Real-world example</strong>, continued</summary>

For example, using our previous modified
[real-world example from React][react/scripts/jest/jest-cli.js]:

```js
Object.keys(envars)
  .map(envar => `${envar}=${envars[envar]}`)
  .join(' ')
  |> `$ ${%}`
  |> chalk.dim(%, 'node', args.join(' '))
  |> console.log(%);
```

…a version using F# pipes instead of Hack pipes would look like this:

```js
Object.keys(envars)
  .map(envar => `${envar}=${envars[envar]}`)
  .join(' ')
  |> x=> `$ ${x}`
  |> x=> chalk.dim(x, 'node', args.join(' '))
  |> console.log;
```

</details>

**Pro:** The restriction that the righthand side
**must** resolve to a unary function
lets us write very terse pipes
**when** the operation we want to perform
is a **unary function call**:

* `value |> foo` for unary function calls.

This includes unary functions
that were created by **[function-currying][] libraries** like [Ramda][],
as well as [unary arrow functions
that perform **complex destructuring** on their arguments][destruct]:
F# pipes would be **slightly less verbose**
with an **implicit** function call (no `(%)`).

**Con:** The restriction means that **any operations**
that are performed by **other syntax**
must be made **slightly more verbose** by **wrapping** the operation
in a unary **arrow function**:

* `value |> x=> x.foo()` for method calls,
* `value |> x=> x + 1` for arithmetic,
* `value |> x=> [x, 0]` for array literals,
* `value |> x=> ({foo: x})` for object literals,
* `` value |> x=> `${x}` `` for template literals,
* `value |> x=> new Foo(x)` for constructing objects,
* `value |> x=> import(x)` for calling function-like keywords,
* etc.

Even calling **named functions** requires **wrapping**
when we need to pass **more than one argument**:

* `value |> x=> foo(1, x)` for n-ary function calls.

**Con:** The **`await` and `yield`** operations are **scoped**
to their **containing function**,
and thus **cannot be handled by unary functions** alone.
If we want to integrate them into a pipe expression,
[`await` and `yield` must be handled as **special syntax cases**][enhanced F# pipes]:

* `value |> await` for awaiting promises, and
* `value |> yield` for yielding generator values.

[enhanced F# pipes]: https://github.com/valtech-nyc/proposal-fsharp-pipelines/

### Hack pipes favor more common expressions
**Both** Hack pipes and F# pipes respectively impose
a small **syntax tax** on different expressions:\
**Hack pipes** slightly tax only **unary function calls**, and\
**F# pipes** slightly tax **all expressions except** unary function calls.

In **both** proposals, the syntax tax per taxed expression is **small**
(**both** `(%)` and `x=>` are **only three characters**).
However, the tax is **multiplied** by the **prevalence**
of its respectively taxed expressions.
It therefore might make sense
to impose a tax on whichever expressions are **less common**
and to **optimize** in favor of whichever expressions are **more common**.

Unary function calls are in general **less common**
than **all** expressions **except** unary functions.
In particular, **method** calling and **n-ary function** calling
will **always** be **popular**;
in general frequency,
**unary** function calling is equal to or exceeded by
those two cases **alone** –
let alone by other ubiquitous syntaxes
such as **array literals**, **object literals**,
and **arithmetic operations**.
This explainer contains several [real-world examples][]
of this difference in prevalence.

[real-world examples]: #real-world-examples

Furthermore, several other proposed **new syntaxes**,
such as **[extension calling][]**,
**[do expressions][]**,
and **[record/tuple literals][]**,
will also likely become **pervasive** in the **future**.
Likewise, **arithmetic** operations would also become **even more common**
if TC39 standardizes **[operator overloading][]**.
Untangling these future syntaxes’ expressions would be more fluent
with Hack pipes compared to F# pipes.

[extension calling]: https://github.com/tc39/proposal-extensions/
[do expressions]: https://github.com/tc39/proposal-do-expressions/
[record/tuple literals]: https://github.com/tc39/proposal-record-tuple/
[operator overloading]: https://github.com/tc39/proposal-operator-overloading/

### Hack pipes might be simpler to use
The syntax tax of Hack pipes on unary function calls
(i.e., the `(%)` to invoke the righthand side’s unary function)
is **not a special case**:
it simply is **explicitly writing ordinary code**,
in **the way we normally would** without a pipe.

On the other hand, **F# pipes require** us to **distinguish**
between “code that resolves to an unary function”
versus **“any other expression”** –
and to remember to add the arrow-function wrapper around the latter case.

For example, with Hack pipes, `value |> someFunction + 1`
is **invalid syntax** and will **fail early**.
There is no need to recognize that `someFunction + 1`
will not evaluate into a unary function.
But with F# pipes, `value |> someFunction + 1` is **still valid syntax** –
it’ll just **fail late** at **runtime**,
because `someFunction + 1` isn’t callable.

### TC39 has rejected F# pipes multiple times
The pipe champion group has presented F# pipes for Stage 2 to TC39 **twice**.
It was **unsuccessful** in advancing to Stage 2 both times.
Both F# pipes (and [partial function application (PFA)][PFA syntax])
have run into strong pushback from multiple other TC39 representatives
due to various concerns. These have included:

* Memory performance concerns (e.g., [especially from browser-engine implementors][V8 pushback]),
* Syntax concerns about `await`.
* Concerns about encouraging ecosystem bifurcation/forking, etc.

[V8 pushback]: https://github.com/tc39/proposal-pipeline-operator/blob/main/HISTORY.md#2021-07

This pushback has occurred from **outside** the pipe champion group.
See [HISTORY.md][] for more information.

It is the pipe champion group’s belief that any pipe operator is better than none,
in order to [easily linearize deeply nested expressions](#why-a-pipe-operator)
without resorting to named variables.
Many members of the champion group believe that Hack pipes are slightly better than F# pipes,
and some members of the champion group believe that F# pipes are slightly better than Hack pipes.
But everyone in the champion group agrees that F# pipes have met with far too much resistance
to be able to pass TC39 in the foreseeable future.

To emphasize, it is likely that an attempt to switch from Hack pipes back to F# pipes
will result in TC39 never agreeing to any pipes at all.
[PFA syntax][] is similarly facing an uphill battle in TC39 (see [HISTORY.md][]).
Many members of the pipe champion group think this is unfortunate,
and they are willing to fight again **later** for an [F#-pipe split mix][split mix] and [PFA syntax][].
But there are quite a few representatives (including [browser-engine implementers][V8 pushback])
outside of the Pipe Champion Group
who are generally against encouraging [tacit programming][] (and [PFA syntax][]),
regardless of Hack pipes.

[HISTORY.md]: https://github.com/tc39/proposal-pipeline-operator/blob/main/HISTORY.md
[tacit programming]: https://en.wikipedia.org/wiki/Tacit_programming
[PFA syntax]: https://github.com/tc39/proposal-partial-application
[split mix]: #tacit-unary-function-application-syntax

## Description
(A [formal draft specification][specification] is available.)

The **topic reference** `%` is a **nullary operator**.
It acts as a placeholder for a **topic value**,
and it is **lexically scoped** and **immutable**.

<details>
<summary><code>%</code> is not a final choice</summary>

(The precise [**token** for the topic reference is **not final**][token bikeshedding].
`%` could instead be `^`, or many other tokens.
We plan to [**bikeshed** what actual token to use][token bikeshedding]
before advancing to Stage 3.
However, `%` seems to be the [least syntactically problematic][],
and it also resembles the placeholders of **[printf format strings][]**
and [**Clojure**’s `#(%)` **function literals**][Clojure function literals].)

[least syntactically problematic]: https://github.com/js-choi/proposal-hack-pipes/issues/2
[Clojure function literals]: https://clojure.org/reference/reader#_dispatch
[printf format strings]: https://en.wikipedia.org/wiki/Printf_format_string

</details>

The **pipe operator** `|>` is an **infix operator**
that forms a **pipe expression** (also called a **pipeline**).
It evaluates its lefthand side (the **pipe head** or **pipe input**),
immutably **binds** the resulting value (the **topic value**) to the **topic reference**,
then evaluates its righthand side (the **pipe body**) with that binding.
The resulting value of the righthand side
becomes the whole pipe expression’s final value (the **pipe output**).

The pipe operator’s precedence is the **same** as:
* the function arrow `=>`;
* the assignment operators `=`, `+=`, etc.;
* the generator operators `yield` and `yield *`;

It is **tighter** than only the comma operator `,`.\
It is **looser** than **all other** operators.

For example, `v => v |> % == null |> foo(%, 0)`\
would group into `v => (v |> (% == null) |> foo(%, 0))`,\
which in turn is equivalent to `v => foo(v == null, 0)`.

A pipe body **must** use its topic value **at least once**.
For example, `value |> foo + 1` is **invalid syntax**,
because its body does not contain a topic reference.
This design is because **omission** of the topic reference
from a pipe expression’s body
is almost certainly an **accidental** programmer error.

Likewise, a topic reference **must** be contained in a pipe body.
Using a topic reference outside of a pipe body
is also **invalid syntax**.

To prevent confusing grouping,
it is **invalid** syntax to use **other** operators that have **similar precedence**
(i.e., the arrow `=>`, the ternary conditional operator `?` `:`,
the assignment operators, and the `yield` operator)
as a **pipe head or body**.
When using `|>` with these operators, we must use **parentheses**
to explicitly indicate what grouping is correct.
For example, `a |> b ? % : c |> %.d` is invalid syntax;
it should be corrected to either `a |> (b ? % : c) |> %.d`
or `a |> (b ? % : c |> %.d)`.

Lastly, topic bindings **inside dynamically compiled** code
(e.g., with `eval` or `new Function`)
**cannot** be used **outside** of that code.
For example, `v |> eval('% + 1')` will throw a syntax error
when the `eval` expression is evaluated at runtime.

There are **no other special rules**.

A natural result of these rules is that,
if we need to interpose a **side effect**
in the middle of a chain of pipe expressions,
without modifying the data being piped through,
then we could use a **comma expression**,
such as with `value |> (sideEffect(), %)`.
As usual, the comma expression will evaluate to its righthand side `%`,
essentially passing through the topic value without modifying it.
This is especially useful for quick debugging: `value |> (console.log(%), %)`.

## Real-world examples
The only changes to the original examples were dedentation and removal of comments.

From [jquery/build/tasks/sourceMap.js][]:
```js
// Status quo
var minLoc = Object.keys( grunt.config( "uglify.all.files" ) )[ 0 ];

// With pipes
var minLoc = grunt.config('uglify.all.files') |> Object.keys(%)[0];
```

From [node/deps/npm/lib/unpublish.js][]:
```js
// Status quo
const json = await npmFetch.json(npa(pkgs[0]).escapedName, opts);

// With pipes
const json = pkgs[0] |> npa(%).escapedName |> await npmFetch.json(%, opts);
```

From [underscore.js][]:
```js
// Status quo
return filter(obj, negate(cb(predicate)), context);

// With pipes
return cb(predicate) |> _.negate(%) |> _.filter(obj, %, context);
```

From [ramda.js][].
```js
// Status quo
return xf['@@transducer/result'](obj[methodName](bind(xf['@@transducer/step'], xf), acc));

// With pipes
return xf
  |> bind(%['@@transducer/step'], %)
  |> obj[methodName](%, acc)
  |> xf['@@transducer/result'](%);
```

From [ramda.js][].
```js
// Status quo
try {
  return tryer.apply(this, arguments);
} catch (e) {
  return catcher.apply(this, _concat([e], arguments));
}

// With pipes: Note the visual parallelism between the two clauses.
try {
  return arguments
    |> tryer.apply(this, %);
} catch (e) {
  return arguments
    |> _concat([e], %)
    |> catcher.apply(this, %);
}
```

From [express/lib/response.js][].
```js
// Status quo
return this.set('Link', link + Object.keys(links).map(function(rel){
  return '<' + links[rel] + '>; rel="' + rel + '"';
}).join(', '));

// With pipes
return links
  |> Object.keys(%).map(function (rel) {
    return '<' + links[rel] + '>; rel="' + rel + '"';
  })
  |> link + %.join(', ')
  |> this.set('Link', %);
```

From [react/scripts/jest/jest-cli.js][].
```js
// Status quo
console.log(
  chalk.dim(
    `$ ${Object.keys(envars)
      .map(envar => `${envar}=${envars[envar]}`)
      .join(' ')}`,
    'node',
    args.join(' ')
  )
);

// With pipes
Object.keys(envars)
  .map(envar => `${envar}=${envars[envar]}`)
  .join(' ')
  |> `$ ${%}`
  |> chalk.dim(%, 'node', args.join(' '))
  |> console.log(%);
```

From [ramda.js][].
```js
// Status quo
return _reduce(xf(typeof fn === 'function' ? _xwrap(fn) : fn), acc, list);

// With pipes
return fn
  |> (typeof % === 'function' ? _xwrap(%) : %)
  |> xf(%)
  |> _reduce(%, acc, list);
```

From [jquery/src/core/init.js][].
```js
// Status quo
jQuery.merge( this, jQuery.parseHTML(
  match[ 1 ],
  context && context.nodeType ? context.ownerDocument || context : document,
  true
) );

// With pipes
context
  |> (% && %.nodeType ? %.ownerDocument || % : document)
  |> jQuery.parseHTML(match[1], %, true)
  |> jQuery.merge(%);
```

[ramda.js]: https://github.com/ramda/ramda/blob/v0.27.1/dist/ramda.js
[node/deps/npm/lib/unpublish.js]: https://github.com/nodejs/node/blob/v16.x/deps/npm/lib/unpublish.js
[node/deps/v8/test/mjsunit/regress/regress-crbug-158185.js]: https://github.com/nodejs/node/blob/v16.x/deps/v8/test/mjsunit/regress/regress-crbug-158185.js
[express/lib/response.js]: https://github.com/expressjs/express/blob/5.0/lib/response.js
[react/scripts/jest/jest-cli.js]: https://github.com/facebook/react/blob/17.0.2/scripts/jest/jest-cli.js
[jquery/build/tasks/sourceMap.js]: https://github.com/jquery/jquery/blob/2.2-stable/build/tasks/sourcemap.js
[jquery/src/core/init.js]: https://github.com/jquery/jquery/blob/2.2-stable/src/core/init.js
[underscore.js]: https://underscorejs.org/docs/underscore-esm.html

## Relationships with other proposals

### `Function` helpers
Hack pipes can and would coexist with the [`Function` helpers proposal][helpers],
including its `pipe` and `flow` functions.
These simple (and commonly downloaded) convenience functions
manipulate unary functions without extra syntax.

[helpers]: https://github.com/js-choi/proposal-function-helpers

[TC39 has rejected the F# pipe operator twice][rejected].
Given this reality, TC39 is considerably more likely to pass
`pipe` and `flow` helper functions than a similar syntactic operator.

[rejected]: #tc39-has-rejected-f-pipes-multiple-times

Standardized `pipe` and `flow` convenience functions
may also obviate some of the need for a F#-pipe infix operator.
(They would not preclude standardizing an equivalent operator later.
For example, TC39 standardized binary `**` even when `Math.pow` existed.)

### Partial-function-application syntax
Hack pipes can coexist with a syntax for **partial function application** (PFA).
There are two approaches with which they may coexist.

The **first approach** is with an **eagerly** evaluated PFA syntax,
which has [already been proposed in proposal-partial-application][PFA syntax].
This eager PFA syntax would add an `…~(…)` operator.
The operator’s right-hand side would be a list of arguments,
each of which is an ordinary expression or a `?` placeholder.
Each consecutive `?` placeholder would represent another parameter.

Ordinary expressions would be evaluated **before** the function is created.
For example, `f~(g(), ?, h(), ?)` would evaluate `f`, then `g()`, then `h()`,
and *then* it would create a partially applied version of `f` with two arguments.

An optional number after `?` placeholder
would override the parameter’s position.
For example, `f~(?1, ?0)` would have two parameters but would switch them when calling `f`.

The **second approach** is with a **lazily** evaluated syntax.
This could be handled with an **extension to Hack pipes**,
with a syntax further inspired by
[Clojure’s `#(%1 %2)` function literals][Clojure function literals].
It would do so by **combining** the Hack pipe `|>`
with the **arrow function** `=>`
into a **pipe-function** operator `+>`,
which would use the same general rules as `|>`.

`+>` would be a **prefix operator** that **creates a new function**,
which in turn **binds its argument(s)** to topic references.
**Non-unary functions** would be created
by including topic references with **numbers** (`%0`, `%1`, `%2`, etc.) or `...`.
`%0` (equivalent to plain `%`) would be bound to the **zeroth argument**,
`%1` would be bound to the next argument, and so on.
`%...` would be bound to an array of **rest arguments**.
And just as with `|>`, `+>` would require its body
to contain at least one topic reference
in order to be syntactically valid.

| Eager PFA                  | Pipe functions             |
| ---------------------------| -------------------------- |
|`a.map(f~(?, 0))`           |`a.map(+> f(%, 0))`         |
|`a.map(f~(?, ?, 0))`        |`a.map(+> f(%0, %1, 0))`    |
|`a.map(x=> x + 1)`          |`a.map(+> % + 1)`           |
|`a.map(x=> x + x)`          |`a.map(+> % + %)`           |
|`a.map(x=> f(x, x))`        |`a.map(+> f(%, %))`         |

In contrast to the [eagerly evaluated PFA syntax][PFA syntax],
topic functions would **lazily** evaluate its arguments,
just like how an arrow function would.

For example, `+> f(g(), %0, h(), %1)` would evaluate `f`,
and then it would create an arrow function that closes over `g` and `h`.
The created function would **not** evaluate `g()` or `h()`
until the every time the created function is called.

No matter the approach taken, Hack pipes could coexist with PFA.

### Eventual sending / pipelining
Despite sharing the word “pipe” in their name,
the pipe operator and the [eventual-send proposal][]’s remote-object pipelines
are orthogonal and independent.
They can coexist and even work together.

```js
const fileP = E(
  E(target).openDirectory(dirName)
).openFile(fileName);

const fileP = target
|> E(%).openDirectory(dirName)
|> E(%).openFile(fileName);
```

[eventual-send proposal]: https://github.com/tc39/proposal-eventual-send/

## Possible future extensions

### Hack-pipe syntax for `if`, `catch`, and `for`–`of`
Many **`if`, `catch`, and `for` statements** could become pithier
if they gained **“pipe syntax”** that bound the topic reference.

`if () |>` would bind its condition value to `%`,\
`catch |>` would bind its caught error to `%`,\
and `for (of) |>` would consecutively bind each of its iterator’s values to `%`.

| Status quo                  | Hack-pipe statement syntax |
| --------------------------- | -------------------------- |
|`const c = f(); if (c) g(c);`|`if (f()) \|> g(%);`        |
|`catch (e) f(e);`            |`catch \|> f(%);`           |
|`for (const v of f()) g(v);` |`for (f()) \|> g(%);`       |

### Optional Hack pipes
A **short-circuiting** optional-pipe operator `|?>` could also be useful,
much in the way `?.` is useful for optional method calls.

For example, `value |> (% == null ? % : await foo(%) |> (% == null ? % : % + 1))`\
would be equivalent to `value |?> await foo(%) |?> % + 1`.

### Tacit unary function application syntax
**Syntax** for **tacit unary function application** – that is, the F# pipe operator –
has been [rejected twice by TC39][rejected].
However, they could still eventually be added to the language in two ways.

First, it can be added as a convenience function `Function.pipe`.
This is what the [function-helpers proposal][helpers] proposes.
`Function.pipe` may obviate much of the need for an F#-pipe operator,
while still not closing off the possibility of an F#-pipe operator.

Secondly, it can be added as **another pipe operator** `|>>` –
similarly to how [Clojure has multiple pipe macros][Clojure pipes]
`->`, `->>`, and `as->`.\
For example, `value |> % + 1 |>> f |> g(%, 0)`\
would mean `value |> % + 1 |> f(%) |> g(%, 0)`.

[Clojure pipes]: https://clojure.org/guides/threading_macros

There was an [informal proposal for such a **split mix** of two pipe operators][split mix],
which was set aside in favor of single-operator proposals.
This split mix might return as a proposal after Hack pipes.

[split mix]: https://github.com/tc39/proposal-pipeline-operator/wiki#proposal-3-split-mix
