# Brief history of the JavaScript pipe operator
The pipe operator in JavaScript has a long and twisty history.
Understanding that history can give context behind
what’s been happening to the proposal since its creation in 2017.

More information about contributing is also available in [CONTRIBUTING.md][].

## 2015–2017
People are debating about a [proposed bind operator `::`][bind]
by [@zenparsing][] (Brave).
A binary `::` operator that binds its left-hand side to its right-hand side (a function)
would serve as a “pipe” operator in JavaScript
(e.g., `a::f(b, c)::g(d, e)` would be a “pipeline” equivalent to
`g.call(f.call(a, b, c), d, e)`).
However, parts of TC39 object to using `this`,
citing that using `this` is strange outside of methods.
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
[@littledan][] makes a
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

[@littledan][] makes
[another presentation for F# pipes and does not succeed in advancing to Stage 2][S2-2017].
During the presentation, he proposes to TC39 that `|> await` be deferred,
but there is pushback from several other members, and presentation time overflows.

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
For three years, debate happens back and forth in circles
about smart-mix pipes vs. F# pipes.

[@littledan][] presents smart-mix pipes and F# pipes in
[a presentation on 2018-03][S2 2018-03].
Neither proposal is able to achieve much consensus among TC39 members
due to syntactic concerns.
Some TC39 members state that no pipe operator
may be worth standardizing at all.

Advancement of F# pipes therefore continues to be stymied;
advancement of smart-mix pipes is also stymied.

[@rbuckton][] (Microsoft) also presents [partial function application][]
again [on 2018-07, attempting to advance it to Stage 2][PFA 2018-07].
However, several TC39 members continue to push back against its syntax.
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

Work on F# pipes, smart-mix pipes, and PFA syntax stalls,
while GitHub debate and debate offline continues in circles.

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
This has support from [@ljharb][] and some other TC39 members.

Through this month, [@tabatkins][] (Google) continues to debate offline
with [@mAAdhaTTah][] regarding Hack pipes vs. F# pipes.
As a result, [@mAAdhaTTah][] changes his mind from being in favor of F# pipes
to being in favor of Hack pipes,
deciding that Hack pipes would be better for bridging functional programming
with the rest of the JavaScript ecosystem.

[@rbuckton][] (Microsoft) joins in debating with [@tabatkins][] (Google) in late August.
[@rbuckton][] (Microsoft) notes the groundswell of support among TC39 about Hack pipes
due to “some of the limitations of F# pipes’ limitations”.
Therefore feeling that F# pipes would continue to be indefinitely stuck at an impasse,
[@rbuckton][] (Microsoft) thus decides to give “tentative agreement” to Hack pipes.
(See [@rbuckton’s narrative][].)

On 2021-08-31, [@tabatkins][] (Google) therefore
presents Hack pipes as the tentative consensus among the champions,
proposing that TC39 advance them to Stage 2.
There are several responses from other members:

[@ljharb][] voices concern that advancing pipe
would kill any future bind operator (see [§ 2017-09](#2017-09)).
Other members respond that Hack pipes are now orthogonal to any bind operator
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

[issues]: https://github.com/tc39/proposal-pipeline-operator/issues?q=is%3Aissue+
[CONTRIBUTING.md]: https://github.com/tc39/proposal-pipeline-operator/blob/main/CONTRIBUTING.md
[general information on incubator calls]: https://github.com/tc39/how-we-work/blob/master/incubator-calls.md

[bind]: https://github.com/tc39/proposal-bind-operator
[partial function application]: https://github.com/tc39/proposal-partial-application

[F# spec]: https://github.com/valtech-nyc/proposal-fsharp-pipelines/
[smart-mix-pipes spec]: https://github.com/js-choi/proposal-smart-pipelines
[Hack-pipes spec]: https://github.com/tc39/proposal-hack-pipes

[first pipe preso]: https://docs.google.com/presentation/d/1qiWFzi5dkjuUVGcFXwypuQbEbZk-BV7unX0bYurcQsA/edit#slide=id.g1fa08b5c5c_0_93

[I4]: https://github.com/tc39/proposal-pipeline-operator/issues/4
[I75]: https://github.com/tc39/proposal-pipeline-operator/issues/75
[I66]: https://github.com/tc39/proposal-pipeline-operator/pull/66
[I89]: https://github.com/tc39/proposal-pipeline-operator/issues/89

[S1]: https://github.com/tc39/notes/blob/master/meetings/2017-09/sept-26.md#11iia-pipeline-operator
[S2-2017]: https://github.com/tc39/notes/blob/master/meetings/2017-11/nov-29.md#9iii-pipeline-operator-for-stage-2
[PFA S1]: https://github.com/tc39/notes/blob/master/meetings/2017-09/sept-28.md#13i-partial-application
[S2 2018-03]: https://github.com/tc39/notes/blob/master/meetings/2018-03/mar-22.md#10ive-pipeline-operator
[PFA 2018-07]: https://github.com/tc39/notes/blob/master/meetings/2018-07/july-25.md#partial-application
[2021-07 incubator]: https://github.com/tc39/incubator-agendas/blob/master/notes/2021/06-17.md#pipeline
[Mozilla study]: https://github.com/tc39/notes/blob/master/meetings/2019-06/june-6.md#javascript-and-syntax-research-methods

[TC39 process]: https://tc39.es/process-document/
[Gist]: https://gist.github.com/tabatkins/1261b108b9e6cdab5ad5df4b8021bcb5
[@rbuckton’s narrative]: https://github.com/tc39/proposal-pipeline-operator/issues/91#issuecomment-917645179
[SoJS 20]: https://2020.stateofjs.com/en-US/opinions/?missing_from_js

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
