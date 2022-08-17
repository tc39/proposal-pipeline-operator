# Brief history of the JavaScript pipe operator
The pipe operator in JavaScript has a long and twisty history.
Understanding that history can give context behind
what’s been happening to the proposal since its creation in 2015.

For information on what “Stage 1” and “Stage 2” mean,
read about the [TC39 Process][].

More information about contributing is also available in [CONTRIBUTING.md][].

## 2015–2017
People are debating about a [proposed bind operator `::`][bind]
by [@zenparsing][] (Brave).
A binary `::` operator that binds its left-hand side to its right-hand side (a function)
would serve as a “pipe” operator in JavaScript
(e.g., `a::f(b, c)::g(d, e)` would be a “pipeline” equivalent to
`g.call(f.call(a, b, c), d, e)`).
However, parts of TC39 object to using `this`,
saying that using `this` is strange outside of methods.
Debate online in the `::` repository (and also offline) is bogged down in circles.

[@littledan][] (Igalia) and [@gilbert][] create a proposal
for an alternative pipe operator that does not use `this`.
They start out [comparing F# pipes with Elixir pipes][first pipe preso].
They also [explore other syntaxes such as “placeholder” pipes (i.e., Hack pipes)][I4].

Meanwhile, [@rbuckton][] (Microsoft) discusses pipelining with the F# team
and plans to simultaneously propose F# pipes and syntax
for [partial function application][].
Upon discovering [@littledan][] (Igalia)’s proposal for F#-or-Elixir pipes,
[@rbuckton][] (Microsoft) finds that it already
aligns with the F# pipe that he had been planning to present,
and he switches to focusing on syntax for [partial function application][].

## 2017-07
On [2017-09-26][S1], [@littledan][] makes a first
[presentation for F#-or-Elixir pipes to TC39, successfully advancing to Stage 1][S1].

However, parts of TC39 object to redundancy with a bind operator.
A condition of the advancement to Stage 1 is that it would not be redundant
with a bind operator; this will become relevant later.

In addition, parts of TC39 object to aspects of F# pipes,
such as how they handle `await` and `yield`.

[@rbuckton][] (Microsoft) presents syntax for [partial function application][]
[at the same meeting and succeeds in advancing to Stage 1][PFA S1].
However, parts of TC39 push back against its syntax.

## 2017–2018
[@littledan][] and collaborators [attempt to make a stopgap with `|> await`][I66]
but encounter several conceptual and syntactic problems,
including a problem related to unexpected automatic semicolon insertion.
[@littledan][] decides to try to defer handling `await` at all to a later proposal;
he also drops Elixir pipes in favor of F# pipes.

On [2017-11-29][S2 2017], [@littledan][] makes
[another presentation for F# pipes and does not succeed in advancing to Stage 2][S2 2017].
During the presentation, he proposes to TC39 that `|> await` be deferred,
but there is pushback from several other representatives, and presentation time overflows.

Advancement of F# pipes is therefore stymied.

[@gilbert][] suggests resurrecting Hack pipes,
which were [previously explored in 2015][I4],
as a solution to TC39’s blocking concerns.
He also suggests two possible compromises that mix F# pipes and Hack pipes:
[“split-mix pipes” and “smart-mix pipes”][I89].
[@littledan][] (Igalia) agrees to investigate all three styles.
[@mAAdhaTTah][] (pro-F#-pipes) and [@js-choi][] (slightly pro-Hack-pipes)
volunteer to collaborate on competing specs:
[@mAAdhaTTah][] writes a [spec for F# pipes][F# spec]
and [@js-choi][] writes a [spec for smart-mix pipes][smart-mix-pipes spec].

## 2018–2020
On [2018-03-22][S2 2018], [@littledan][] presents F# pipes again—alongside
smart-mix pipes—in an
[update presentation, trying to gain more consensus within TC39 for Stage 2][S2 2018].
However, neither proposal is able to achieve much consensus among TC39 representatives
due to syntactic concerns.
Some TC39 representatives state that no pipe operator
may be worth standardizing at all.

Advancement of F# pipes therefore continues to be stymied;
advancement of smart-mix pipes is also stymied.

[@rbuckton][] (Microsoft) also presents [partial function application][]
again [on 2018-07, attempting to advance it to Stage 2][PFA 2018-07].
However, several TC39 representatives continue to push back against its syntax.
[@syg][] (Google V8) also expresses “strong reservations” about PFA syntax
increasing the “ease with which [developers] can allocate many many closures”,
with regards to memory use.
Partial function application is also unable to advance to Stage 2,
and its advancement is therefore also stymied.

[@codehag][] (Mozilla SpiderMonkey) meanwhile leads a
[Mozilla user study about pipes][Mozilla study].
Its results suggest that developers like and prefer smart-mix pipes slightly more,
but they make slightly less errors with F# pipes.
Her conclusions from the study were that no difference between smart-mix pipes or F# pipes
was significant enough to justify any decision one way or the other.
(There is also another Mozilla user study about partial function application,
and its results suggests that the JavaScript community
is much more interested in partial function application than any pipe operator.)
The Mozilla SpiderMonkey team becomes weakly against any pipe operator.

Work on F# pipes, smart-mix pipes, and PFA syntax stalls.

For the three years, GitHub debate online and debate offline
continues back and forth in circles:
about the best way to increase the odds that smart-mix pipes vs. F# pipes
will reach consensus in TC39.

## 2021-01
The State of JS 2020 is published,
and it reports that one of the [language’s top requested features][SoJS 20]
is some kind of pipe operator.
This galvanizes [@littledan][] (Igalia)’s further work on pipe,
and he prepares a presentation for 2021-03’s TC39 meeting.

## 2021-03
[@littledan][] (Igalia) is managing other projects
and is now unable to devote time to the pipe operator.
He presents about it again to TC39 and asks there for a new lead champion.
[@tabatkins][] (Google), who also does much work in Web Platform standards
such as HTML5 DOM and CSS,
is personally enthusiastic about helping with any pipe operator
and agrees to take on being co-champion.
[@rbuckton][] (Microsoft) also agrees to co-champion the proposal.

[@tabatkins][] (Google) publishes a [Gist comparing F#, Hack, and smart-mix pipes][Gist], concluding that they are all functionally the same,
with only small differences in actual usage,
and that all three would benefit everyone.
Discussion is sparked in the Gist’s comments.

Galvanized by the Gist,
[@js-choi][] switches from writing the [smart-mix-pipes spec][]
to writing a [Hack-pipes spec][].

## 2021-07
[@tabatkins][] (Google) schedules a
[TC39 incubator meeting devoted to pipes][2021-07 incubator]
(see [general information on incubator calls][]).
Attendees of special note in this meeting are [@syg][] (Google V8),
[@littledan][] (Igalia), [@rbuckton][] (Microsoft), and [@ljharb][]. [@codehag][] (Mozilla SpiderMonkey) is unable to attend.

[@tabatkins][] (Google) presents three choices to the attendees again:
F# pipes, Hack pipes, and Elixir pipes.

[@rbuckton][] (Microsoft) is still in favor of F# pipes with his proposed syntax for
[partial function application][] (see [§ 2016–2017](#20162017)).
[@rbuckton][] (Microsoft) debates with [@tabatkins][] (Google)
about whether F# pipes or Hack pipes are more intuitive.
(He also mentions that [@codehag][] (Mozilla SpiderMonkey)
might be interested in co-championing partial function application without pipes,
based on her user studies’ findings; see [§ 2018–2020](#20182020).)

[@syg][] (Google V8) voices concerns again about engine performance
of partial function application and F# pipes (see [§ 2018–2020](#20182020)).

[@tabatkins][] (Google) and [@ljharb][] are supportive
of either Hack pipes or F# pipes:
“90%” F# pipes and “100%” Hack pipes.
However, their 90% support of F# pipes would drop a lot if [@syg][]
(Google V8)’s performance concerns about F# pipes are borne out.

Nobody in the meeting seems to think Elixir pipes are a good fit for JavaScript.

Most everyone in the meeting seems to be in favor of picking some style for pipes
after three years of indecision.

## 2021-08
[@tabatkins][] (Google) plans to present *some* pipe-operator style for Stage 2.
Based on the results of the the preceding meeting, they pick Hack pipes.
This has support from [@ljharb][] and some other TC39 representatives.
[@js-choi][] (Indiana University) joins as co-champion.

Through this month, [@tabatkins][] (Google) continues to debate offline
with [@mAAdhaTTah][] regarding Hack pipes vs. F# pipes.
As a result, [@mAAdhaTTah][] changes his mind from being in favor of F# pipes
to being in favor of Hack pipes,
deciding that [Hack pipes would be better for bridging functional programming][JDG essay]
with the rest of the JavaScript ecosystem.

[@rbuckton][] (Microsoft) joins in debating with [@tabatkins][] (Google) in late August.
[@rbuckton][] (Microsoft) notes the groundswell of support within TC39 about Hack pipes
due to “some of the limitations of F# pipes”.
Therefore feeling that F# pipes would continue to be indefinitely stuck at an impasse,
[@rbuckton][] (Microsoft) thus decides to give “tentative agreement” to Hack pipes.
(See [@rbuckton’s narrative][].)

***

On [2021-08-31][S2 2021], a formal Committee plenary occurs,
and [@tabatkins][] (Google) therefore
[presents Hack pipes as the tentative consensus among the champions,
proposing that TC39 advance them to Stage 2][S2 2021].
There are several responses from other representatives:

[@ljharb][] voices concern that advancing pipe
would kill any future bind operator (see [§ 2017-09](#2017-09)).
Other representatives respond that Hack pipes are now orthogonal to any bind operator
and would not kill it. [@ljharb][] decides not to block Stage 2.

[@codehag][] (Mozilla SpiderMonkey) voices some concerns:
the Mozilla SpiderMonkey team is still somewhat against any pipe operator,
whatever the style.
However, she decides that these concerns are not strong enough for her to block Stage 2.

[@syg][] (Google V8), having previously expressed concerns about memory allocation
encouraged by F# pipes and partial function application
(see [§ 2021-07](#2021-07)), does not give any objection.

[@rbuckton][] (Microsoft) continues to give tentative agreement to Hack pipes.

No other representatives give objections to Stage 2.
Hack pipes therefore succeed in advancing to Stage 2.
[@tabatkins][] (Google) resolves to continue discussing concerns
with [@rbuckton][] (Microsoft), [@codehag][] (Mozilla SpiderMonkey), [@mAAdhaTTah][],
and others offline.
They also discuss concerns with the community on GitHub,
both in the [2021-03 comparison Gist][Gist]’s comments (see [§ 2021-03](#2021-03))
and in the [pipe proposal’s issues][issues].

## 2021-09
In order to explain this proposal’s history and process to other community members,
[@js-choi][] (Indiana University) creates this document.

Inspired by a [defense of unary functions][I233], [@js-choi][] (Indiana University)
also creates a new proposal, [proposal-function-helpers][], that would add several Function
helper methods. These include Function.pipe, pipeAsync, flow, and flowAsync.

## 2021-10
Starting on 2021-10-25, another formal Commitee plenary occurs.
The pipe operator is not presented at this meeting, although
an [incubator meeting on 2021-11 is chartered][incubator charter 2021-11]
for bikeshedding the pipe operator’s topic token.

[On 2021-10-25, PFA syntax is presented again to the Committee
plenary][2021-10 PFA] by [@rbuckton][] (Microsoft) for Stage 2.
The Committee rejects this proposal; several representatives,
including those from Mozilla SpiderMonkey and Google V8, state
that there were insufficiently specific and compelling use cases presented,
with high syntax cost and novelty in comparison to arrow functions.

[On 2021-10-28, proposal-function-helpers is also presented to the
Committee plenary][2021-10 Function] for Stage 1 by [@js-choi][].
The Committee also rejects this proposal due to its being overly broad,
and it requests that it be split up into multiple proposals.
These split proposals would include a proposal specifically
about [Function.pipe and flow][].

## 2021-12: Holistic-dataflow articles
Since December, TC39 has continued to discuss the pipe operator in the greater
context of “dataflow”.

The “dataflow” proposals include the following:

* The [pipe operator][Hack pipes] `… |> …` (aka “Hack pipes”)
* [Function.pipe][] (a function version of the F# pipe operator)
* The [bind-this][] operator `…::…` (and its variant [call-this][] `…@(…)`)
* The [Extensions][] syntaxes `…::…`, `…::…:…`, and `const ::{ … } = …;`
* [Partial function application][PFA syntax] `…~(…)` (aka “PFA syntax”)

These dataflow proposals overlap in various complicated ways. Multiple TC39
representatives expressed concerns about redundancies between these
proposals—that the space as a whole needs to be holistically considered, that
goals need to be more specifically articulated, and that there is not enough
“syntax budget” in the language to approve all of these proposals. This applies
to the pipe operator, as well as all the others in that list.

![][diagram]

* In late December, [@js-choi][] wrote [an article detailing how these
  proposals overlap][2022-12 jschoi dataflow article].
* [@tabatkins then wrote a response article](https://www.xanthir.com/b5Gd0) on
  their own blog.
* Later, [@hax would also write another response
  article](https://hackmd.io/yDDJCsS-Sv2AJwo8arAn3w?view). (@hax is a TC39
  champion of the [Extensions][] syntaxes.)

## 2022-01: Plenary meeting
On [January 26, 2022, a plenary meeting was held][2022-01 plenary] to discuss these overlapping proposals holistically. This discussion overflowed into an [ad-hoc meeting on the next day][2022-01 overflow].

In these two meetings, TC39 representatives debated over such topics as:
* Creating a unified language versus accommodating multiple programming paradigms (e.g., object oriented versus functional).
* [TMTOWTDI][] versus [TOOWTDI][].
* Whether generalized  language features (like the Hack-style pipe operator) or specialized features (like Function.pipe, the F#-style pipe operator, and the bind-this operator) were more desirable.
* Whether it is better for language features to be universal or prescriptive in their usage.
* The merits of specific dataflow proposals, including the pipe operator.

Support among TC39 representatives for the pipe operator as it is now (with a Hack-style topic reference) appears to range from strongly in favor to weakly against. Several representatives reaffirmed that they are moderately or strongly against F#-style syntax. Support for Function.pipe appears to be tepid: neither strongly positive or negative. For more details, see the [conclusions of the ad-hoc overflow meeting][2022-01 overflow conclusions].

## 2022-03 and 2022-04
At the [2022-03 plenary, several candidate tokens for the topic reference were presented][2022-03 plenary pipe]. The Committee mildly preferred `@` as the topic reference. (However, a [delegate subsequently raised serious concerns about `@`][2022-04 serious @ concerns], and thus `@` was excluded again as a topic reference.)

Additionally, an [update about call-this was also presented at the plenary][2022-03 plenary call-this]. The call-this proposal continues to polarize the Committee due to ecosystem-schism concerns.

## 2022-07
[In the plenary on July 21, proposal-function-pipe-flow was formally presented to the Committee, and it was rejected for Stage 1][2022-07 plenary]. The Committee generally found its use cases not compelling enough compared to the pipe operator. Its champion subsequently withdrew it from consideration. (Eventually, after the pipe operator gains users, pain points with the pipe operator may be enough motivation to revive proposal-function-pipe-flow, but that would not occur for a long time.)

There is another [incubator call chartered for more pipe-operator bikeshedding](https://github.com/tc39/incubator-agendas/issues/26), which might or might not occur before the [September plenary](https://github.com/tc39/agendas/blob/main/2022/09.md).

[issues]: https://github.com/tc39/proposal-pipeline-operator/issues?q=is%3Aissue+
[CONTRIBUTING.md]: https://github.com/tc39/proposal-pipeline-operator/blob/main/CONTRIBUTING.md
[general information on incubator calls]: https://github.com/tc39/how-we-work/blob/master/incubator-calls.md

[bind]: https://github.com/tc39/proposal-bind-operator
[partial function application]: https://github.com/tc39/proposal-partial-application
[proposal-function-helpers]: https://github.com/js-choi/proposal-function-helpers
[Function.pipe and flow]: https://github.com/js-choi/proposal-function-pipe-flow

[F# spec]: https://github.com/valtech-nyc/proposal-fsharp-pipelines/
[smart-mix-pipes spec]: https://github.com/js-choi/proposal-smart-pipelines
[Hack-pipes spec]: https://github.com/tc39/proposal-hack-pipes
[Function.pipe]: https://github.com/js-choi/proposal-function-pipe-flow
[bind-this]: https://github.com/tc39/proposal-bind-this
[call-this]: https://github.com/tabatkins/proposal-call-this-operator
[Extensions]: https://github.com/tc39/proposal-extensions
[PFA syntax]: https://github.com/tc39/proposal-partial-application
[diagram]: https://jschoi.org/21/es-dataflow/map/
[Hack pipes]: https://github.com/tc39/proposal-pipeline-operator
[TMTOWTDI]: https://en.wikipedia.org/wiki/There%27s_more_than_one_way_to_do_it
[TOOWTDI]: https://wiki.python.org/moin/TOOWTDI

[first pipe preso]: https://docs.google.com/presentation/d/1qiWFzi5dkjuUVGcFXwypuQbEbZk-BV7unX0bYurcQsA/edit#slide=id.g1fa08b5c5c_0_93

[I4]: https://github.com/tc39/proposal-pipeline-operator/issues/4
[I75]: https://github.com/tc39/proposal-pipeline-operator/issues/75
[I66]: https://github.com/tc39/proposal-pipeline-operator/pull/66
[I89]: https://github.com/tc39/proposal-pipeline-operator/issues/89
[I233]: https://github.com/tc39/proposal-pipeline-operator/issues/233

[S1]: https://github.com/tc39/notes/blob/master/meetings/2017-09/sept-26.md#11iia-pipeline-operator
[S2 2017]: https://github.com/tc39/notes/blob/master/meetings/2017-11/nov-29.md#9iii-pipeline-operator-for-stage-2
[PFA S1]: https://github.com/tc39/notes/blob/master/meetings/2017-09/sept-28.md#13i-partial-application
[S2 2018]: https://github.com/tc39/notes/blob/master/meetings/2018-03/mar-22.md#10ive-pipeline-operator
[PFA 2018-07]: https://github.com/tc39/notes/blob/master/meetings/2018-07/july-25.md#partial-application
[2021-07 incubator]: https://github.com/tc39/incubator-agendas/blob/master/notes/2021/06-17.md#pipeline
[Mozilla study]: https://github.com/tc39/notes/blob/master/meetings/2019-06/june-6.md#javascript-and-syntax-research-methods
[S2 2021]: https://github.com/tc39/notes/blob/master/meetings/2021-08/aug-31.md#pipeline-operator-for-stage-2
[incubator charter 2021-11]: https://github.com/tc39/incubator-agendas/issues/21
[2021-10 PFA]: https://github.com/tc39/notes/blob/master/meetings/2021-10/oct-25.md#partial-function-application-for-stage-2
[2021-10 Function]: https://github.com/tc39/notes/blob/master/meetings/2021-10/oct-28.md#function-helpers
[2022-01 plenary]: https://github.com/tc39/notes/blob/main/meetings/2022-01/jan-26.md#holistic-discussion-of-tc39-dataflow-proposals
[2022-01 overflow]: https://github.com/tc39/incubator-agendas/blob/main/notes/2022/01-27.md
[2022-01 overflow conclusions]: https://github.com/tc39/incubator-agendas/blob/main/notes/2022/01-27.md#conclusions
[2022-12 jschoi dataflow article]: https://jschoi.org/21/es-dataflow/
[2022-03 plenary pipe]: https://github.com/tc39/notes/blob/main/meetings/2022-03/mar-29.md#bikeshedding-the-pipe-operator-topic-token
[2022-03 plenary call-this]: https://github.com/tc39/notes/blob/main/meetings/2022-03/mar-29.md#bikeshedding-call-this-syntax
[2022-04 serious @ concerns]: https://github.com/tc39/proposal-pipeline-operator/issues/91#issuecomment-1084946624
[2022-07 plenary]: https://github.com/tc39/notes/blob/main/meetings/2022-07/jul-21.md#functionpipe--flow-for-stage-1

[TC39 process]: https://tc39.es/process-document/
[Gist]: https://gist.github.com/tabatkins/1261b108b9e6cdab5ad5df4b8021bcb5
[@rbuckton’s narrative]: https://github.com/tc39/proposal-pipeline-operator/issues/91#issuecomment-917645179
[SoJS 20]: https://2020.stateofjs.com/en-US/opinions/?missing_from_js
[JDG essay]: https://jamesdigioia.com/hack-pipe-for-functional-programmers-how-i-learned-to-stop-worrying-and-love-the-placeholder/

[@littledan]: https://github.com/littledan/
[@gilbert]: https://github.com/gilbert/
[@tabatkins]: https://github.com/tabatkins/
[@codehag]: https://github.com/codehag/
[@mAAdhaTTah]: https://github.com/mAAdhaTTah/
[@js-choi]: https://github.com/js-choi/
[@syg]: https://github.com/syg/
[@ljharb]: https://github.com/ljharb/
[@rbuckton]: https://github.com/rbuckton/
[@zenparsing]: https://github.com/zenparsing
