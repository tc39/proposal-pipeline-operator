As of 2021-11, there are five Stage-1-or-2 TC39 proposals that overlap in
various complex ways:

* A [pipe operator `|>`][pipe] (this repository).
* A [partial function application (PFA) operator][PFA] `~(` `)`.
* A [bind-this operator][bind-this] `::`.
* An [extensions syntax][extensions] that includes two operators `::` and
  `::` `:` and an extraction statement `::{` `}` `from`.
* [Function.pipe, pipeAsync, flow, and flowAsync][Function.pipe] functions.

[pipe]: https://github.com/tc39/proposal-pipeline-operator
[PFA]: https://github.com/tc39/proposal-partial-application
[bind-this]: https://github.com/tc39/proposal-bind-this
[extensions]: https://github.com/tc39/proposal-extensions
[Function.pipe]: https://github.com/js-choi/proposal-function-pipe-flow

Multiple TC39 representatives have expressed concerns about redundancies between
these proposals – that the space as a whole needs to be holistically considered,
that goals need to be more specifically articulated, and that there is not
enough “syntax budget” in the language to approve all of these proposals.

All five proposals touch the general space of **linear dataflow** and data
transformation – quick local modification and data processing, without having to
assign each transformation step to a temporary variable. The general principles
of fluent dataflow are described in the pipe-operator explainer’s “[why a pipe
operator?][]” section. The principles can be summarized as so:

1. Natural word order and linear reading direction are important for code
   readability.
2. Deeply nested expressions scramble word order, make the reading direction
   switch between left to right and right to left, and compromise their code’s
   readability.
3. It is important for developers to be able to unnest their deeply nested
   expressions and improve the readability of their code.
4. Right now, these “[fluent interfaces][]” are possible only with method
   chaining, where each method already belongs to the prototype chain of its
   receiver. We should make the benefits of fluent interfaces applicable to
   more code.

[why a pipe operator?]: https://github.com/tc39/proposal-pipeline-operator/blob/main/README.md#why-a-pipe-operator
[fluent interfaces]: https://en.wikipedia.org/wiki/Fluent_interface

However, these principles leave several questions unanswered:

1. Which is more useful for dataflow: Universal applicability or prescriptivism?

   The pipe operator tries to apply linear dataflows to as many kinds of
   existing expressions as possible.
   
   Bind-this and extensions are prescriptive in what expressions
   can be used for dataflow (only function calls that use the `this` binding).
   
   Function.pipe etc. are also prescriptive (only unary function calls).

2. Which dataflow expressions are most useful to optimize for conciseness?

   The pipe operator does not attempt to optimize for conciseness, beyond
   allowing developers to avoid unnecessary temporary variables. It is sometimes
   wordy with its topic reference `^^`. Its benefit is flattening nested
   expressions and rearranging their word order, which may outweigh conciseness
   concerns (while still being more concise than temporary variables for each
   step).
   
   Bind-this optimizes for conciseness of two function calls, `.bind()` and
   `.call()`, which may be [two of the most frequently used calls in all
   JavaScript][bind/call frequency].
   
   Extensions also optimizes for conciseness of `.call()` (but not `.bind()`).
   It also optimizes for several other cases:
   
   * Conciseness of using unary function calls in certain circumstances (with
     its polymorphic ternary syntax `arg0::ns:fn`, where `ns` is an object that
     is not a constructor and which contains `fn`).
   * Conciseness of property-descriptor extraction (with its `const ::{ prop }
     from obj;` syntax, equivalent to `prop =
     Object.getOwnPropertyDescriptor(obj, 'prop')`).
   * Conciseness of getting with property descriptors (`receiver::prop` is
     equivalent to `receiver::prop.get()`).
   * Conciseness of setting with property descriptors (`receiver::prop = value`
     is equivalent to `receiver::prop.set(value)`).

   Function.pipe etc. optimize for conciseness of unary function calls.

3. Which expressions are *not* worth optimizing for conciseness, due to
   syntactic complexity? Each syntax adds additional complexity to the language;
   see also [The Tragedy of the Common Lisp][tragedy]. `.bind()`, `.call()`,
   property-descriptor extraction/getting/setting, and unary function calls.
   Which of these are not worth optimizing?

4. Are proposals that optimize for maximum applicability (like pipe) compatible
   with proposals that are only narrowly applicable (like bind-this and
   Function.pipe)? What about proposals that have more intermediate breadth
   (like extensions)?

5. What is the relationship between the dataflow proposals and the PFA operator?
   Does the PFA operator count as a dataflow proposal?

[bind/call frequency]: https://github.com/tc39/proposal-bind-this/blob/main/README.md#bind-and-call-are-very-common
[tragedy]: https://erights.medium.com/the-tragedy-of-the-common-lisp-why-large-languages-explode-4e83096239b9

Each of these proposals answers these questions in a different way. This table
compares each proposal’s answer in a variety of use cases. Ordinary method
chaining is also included as a standard of comparison.

* `method` is a function that uses the `this` binding.
* `receiver` is an object that will receiver a function via its `this` binding.
* `ns.fn` is a function that does not use the `this` binding.
* `ns` is a non-constructor object. The extensions syntax’s ternary `::` `:`
  operator is polymorphic depending on whether its middle operand is a
  constructor or not. For more information, see the [2020-11 extensions
  presentation][] and the [extensions–bind-this comparison document][].

[2020-11 extensions presentation]: https://johnhax.net/2020/tc39-nov-ext/slide
[extensions–bind-this comparison document]: https://github.com/tc39/proposal-bind-this/blob/main/extensions-comparison.md

<table>
<thead>
<tr>
<th>

Use case

</th>
<th>

Method calling `.` `(` `)`

</th>
<th>

Pipe operator `|>`

</th>
<th>

PFA operator `~(` `)`

</th>
<th>

Bind-this operator `::`

</th>
<th>

Extensions syntax

</th>
<th>

Function.pipe etc.

</th>
</tr>
</thead>

<tbody>
<tr>
<td>

Dataflow through `receiver.method` (when `method` is already in the `receiver`)

</td>
<td>

```js
receiver.method()
```

</td>
<td>

```js
receiver |> ^^.method()
```

</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>

Dataflow through `obj.method` (when `method` is not already in the `receiver`)

</td>
<td></td>
<td>

```js
receiver |> obj.method.call(^^)
```

</td>
<td></td>
<td>

```js
receiver::obj.method()
```

</td>
<td>

```js
receiver::(obj.method)()
```

</td>
<td></td>
</tr>
<tr>
<td>

Dataflow through `arg0` of unary `ns.fn`

</td>
<td></td>
<td>

```js
arg0 |> ns.fn(^^)
```

</td>
<td></td>
<td></td>
<td>

```js
arg0::ns:fn()
```

</td>
<td>

```js
Function.pipe(arg0, ns.fn)
```

</td>
</tr>
<tr>
<td>

Dataflow through `arg0` of n-ary `ns.fn`

</td>
<td></td>
<td>

```js
arg0 |> ns.fn(^^, arg1, arg2)
```

</td>
<td></td>
<td></td>
<td>

```js
arg0::ns:fn(arg1, arg2)
```

</td>
<td>

```js
Function.pipe(arg0,
  arg0 => ns.fn(arg0, arg1, arg2),
)
```

</td>
</tr>
<tr>
<td>

Dataflow through non-zeroth argument of n-ary `ns.fn`

</td>
<td></td>
<td>

```js
arg1 |> ns.fn(arg0, ^^, arg2)
```

</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>

Dataflow through `await arg0` of async unary `ns.fn`

</td>
<td></td>
<td>

```js
await arg0 |> await ns.fn(^^)
```

</td>
<td></td>
<td></td>
<td></td>
<td>

```js
await Function.pipeAsync(arg0, ns.fn)
```

</td>
</tr>
<tr>
<td>

Dataflow through `await arg0` of async n-ary `ns.fn`

</td>
<td></td>
<td>

```js
await arg0 |> await ns.fn(^^, arg1, arg2)
```

</td>
<td></td>
<td></td>
<td></td>
<td>

```js
await Function.pipeAsync(await arg0,
  arg0 => ns.fn(arg0, arg1, arg2),
)
```

</td>
</tr>
<tr>
<td>

Dataflow through array literal

</td>
<td></td>
<td>

```js
element |> [ ^^ ]
```

</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>

Dataflow through object literal

</td>
<td></td>
<td>

```js
element |> { key: ^^ }
```

</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>

Dataflow through tuple literal

</td>
<td></td>
<td>

```js
element |> #[ ^^ ]
```

</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>

Dataflow through record literal

</td>
<td></td>
<td>

```js
element |> #{ key: ^^ }
```

</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>

Dataflow through template literal

</td>
<td></td>
<td>

```js
element |> `${ ^^ }`
```

</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>

Dataflow through arithmetic operation

</td>
<td></td>
<td>

```js
element |> ^^ + 1
```

</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr>
<td>

Extraction of `property` descriptor from `obj`

</td>
<td>

```js
const property = Object
  .getOwnPropertyDescriptor(obj, 'property')
```

</td>
<td>

```js
const property = obj
  |> Object
    .getOwnPropertyDescriptor(^^, 'property')
```

</td>
<td></td>
<td></td>
<td>

```js
const ::{ property } from obj;
```

</td>
<td></td>
</tr>
<tr>
<td>

Dataflow through `receiver` of extracted `property` descriptor’s getter

</td>
<td></td>
<td>

```js
receiver |> property.get.call(^^)
```

</td>
<td></td>
<td>

```js
receiver::property.get()
```

</td>
<td>

```js
receiver::property
```

</td>
<td></td>
</tr>
<tr>
<td>

Setting `value` of extracted `property` descriptor in `receiver`

</td>
<td></td>
<td>

```js
receiver |> property.set.call(^^, value)
```

</td>
<td></td>
<td>

```js
receiver::property.set(value)
```

</td>
<td>

```js
receiver::property = value
```

</td>
<td></td>
</tr>
<tr>
<td>

Creating callback from `obj.method` bound to `receiver` with no arguments

</td>
<td></td>
<td>

```js
receiver |> obj.method.bind(^^)
```

</td>
<td></td>
<td>

```js
receiver::obj.method
```

</td>
<td></td>
<td></td>
</tr>
<tr>
<td>

Extracting `receiver.method` bound to `receiver` with no arguments

</td>
<td>

```js
receiver.method.bind(ns)
```

</td>
<td>

```js
receiver |> ^^.method.bind(^^)
```

</td>
<td>

```js
receiver.method~()
```

</td>
<td>

```js
receiver::receiver.method
```

</td>
<td></td>
<td></td>
</tr>
<tr>
<td>

Extracting n-ary `receiver.method` bound to `receiver` with `arg0`, omitting
`arg1`

</td>
<td>

```js
receiver.method.bind(receiver, arg0)
```

</td>
<td>

```js
receiver.method.bind(receiver, arg0)
```

</td>
<td>

```js
receiver.method~(arg0, ?)
```

</td>
<td></td>
<td></td>
<td></td>
</tr>
</tbody>
</table>
