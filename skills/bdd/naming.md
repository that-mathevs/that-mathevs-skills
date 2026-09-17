# Naming the specification

## The grammar

```
describe('<the thing, as a noun phrase>')
  it('given <context>, <verb phrase in the present indicative>')
```

The `describe` names the subject as the domain says it: `a schedule`, `a programmer filling a broadcast window`, `the VJ booth`. The `it` completes the sentence. Read them joined and they form the line that belongs in a spec document:

> *a schedule — given an instant before the broadcast begins, reports nothing is on*

Drop `given` when the context is the subject's default state: `reports when it runs out, so a client knows when to fetch more`.

## Present indicative, third person

The subject *does* something now. `reports`, `refuses`, `keeps`, `moves on`, `joins`, `drops`.

| Instead of | Write |
|---|---|
| `should return null when empty` | `given no items, reports nothing is on` |
| `test empty schedule` | `given no items, is empty and nothing is ever on` |
| `handles the overlap case` | `given items that overlap in time, refuses to exist` |
| `works correctly` | name the capability the caller relies on |

`refuses` is worth reaching for by name: it says the object rejects the input to protect an invariant, which is a behaviour, where `throws` names a mechanism.

## Names that carry their reason

A name has room for the *why* when the why is short, joined by a colon or a comma:

```js
it('given an empty pool, refuses rather than publishing an empty broadcast')
it('given items that overlap, refuses to exist: a channel broadcasts one thing at a time')
it('never plays the same program twice in a row')
it('gives the same video the same card every time, so a card can be cached forever')
```

Each names the behaviour *and* the constraint it protects. When the reason needs more than a clause, put it in a comment above the test and keep the name clean.

## One assertion of one idea

Several `assert` calls are fine when they check one outcome from different angles:

```js
it('given an instant inside an item, reports how far into it we have arrived', () => {
  const nowPlaying = broadcast().nowAt(1200)
  assert.equal(nowPlaying.offset, 20)
  assert.equal(nowPlaying.remaining, 100)
})
```

Offset and remaining are one idea — where we are in the item. Two ideas need two scenarios, so a failure names which one broke.

## Naming the fixtures

Builders are part of the language. Name them for the role they play in the scenario, and let each scenario override only what it is about:

```js
const program = (id, duration = 60) => Program.of({ id, title: id, artist: id, duration, play: { youtube: id } })

const broadcast = () => Schedule.of('channel:genre:shoegaze', [
  new ScheduleItem({ segment: program('first', 180), startsAt: 1000 }),
  new ScheduleItem({ segment: program('second', 120), startsAt: 1180 })
])
```

`broadcast()` reads as the domain; `makeTestScheduleWithTwoItems()` reads as the test framework. Prefer the former.

## Naming a regression test

A bug fix begins with a failing test that reproduces it, and that test is named for the behaviour that was missing — never for the ticket:

| Instead of | Write |
|---|---|
| `regression: issue #412` | `given the clock still points at what just finished, moves on to the next item` |
| `fixes duplicate playback bug` | `never plays the same program twice in a row` |

The ticket number belongs in the commit message, where it stays findable. The test name has to still make sense to someone who never reads that ticket.
