# Writing scenarios

## One behaviour, three runners

The same specification, expressed in whatever the project already uses. The words stay identical; only the syntax moves.

**`node:test`** (zero dependency, Node ≥ 20):

```js
import { describe, it } from 'node:test'
import assert from 'node:assert/strict'

describe('a schedule', () => {
  it('given the exact instant an item ends, the next item is already on', () => {
    assert.equal(broadcast().nowAt(1300).segment.id, 'third')
  })
})
```

**Vitest / Jest**: identical shape, `expect(...).toBe(...)` in place of `assert.equal`.

**Gherkin** (`.feature` file, Cucumber and friends):

```gherkin
Feature: What is on now

  Scenario: an item ends and the next begins
    Given a schedule whose second item ends at 1300
    When a viewer tunes in at 1300
    Then the third item is on
```

Gherkin earns its cost when non-engineers read or write the scenarios, because the step definitions are the price of that audience. When the readers are all engineers, a `describe`/`it` whose names read as sentences gives the same specification without the indirection. Choose by audience, then keep the wording the same either way.

## Example tables

When one rule has many cases, the rule is the scenario and the cases are data. The table *is* the specification:

```js
it('reads a YouTube duration', () => {
  const examples = [
    ['PT4M13S', 253],
    ['PT1H2M3S', 3723],
    ['PT45S', 45],
    ['nonsense', null]
  ]

  for (const [given, expected] of examples) {
    assert.equal(secondsFromIso8601(given), expected, `for ${given}`)
  }
})
```

Two rules apply. Every expected value is an independent literal, worked out by hand from the spec rather than recomputed the way the code computes it. And the failure message names the row, so a red run says *which* example broke.

Split a row out into its own named scenario when it carries a distinct reason: a rejected input that protects an invariant deserves a name, where the fifth happy-path case does not.

## One level of abstraction

Every line of a scenario sits at the same altitude. When setup drops two levels to fiddle with fields the behaviour never reads, hide it behind a builder named for its role.

```js
// The scenario shows only what it is about: three items, back to back, from t=1000.
const broadcast = () => Schedule.of('channel:genre:shoegaze', [
  new ScheduleItem({ segment: program('first', 180), startsAt: 1000 }),
  new ScheduleItem({ segment: program('second', 120), startsAt: 1180 }),
  new ScheduleItem({ segment: program('third', 240), startsAt: 1300 })
])
```

A reader sees the shape of the given at a glance. The ids, titles and play maps that `program()` fills in are real but incidental, so they live in the builder.

## Fakes that state the behaviour

At a boundary, a hand-written fake keeps the scenario readable and says what the collaborator does. Name it for its behaviour, and let the test assert through the interface rather than on call counts:

```js
// A selector that walks the candidates in order: deterministic, so the specs describe
// scheduling behaviour rather than randomness.
const inOrder = { select: (candidates) => candidates[0] }

class FakeFiles {
  written = new Map()
  async write (path, body) { this.written.set(path, body) }
  read (path) { return JSON.parse(this.written.get(path)) }
}
```

`FakeFiles` lets the scenario assert on *what was published*, which is the behaviour, instead of *that write was called*, which is the mechanism. Randomness and clocks arrive the same way — as a port the scenario supplies (`draw: () => 0.7`, `now: () => new Date('2026-09-16T14:00:00Z')`), so the example stays deterministic and the domain stays free of I/O.

Reach for a fake at a boundary the system does not own. Inside the domain, use the real object.

## Turning acceptance criteria into scenarios

Given a ticket's criteria, each bullet becomes one named scenario, in the ticket's own words. When a bullet resists — too vague to name an observable outcome — that is the finding to raise before writing code, because a criterion you cannot name is a criterion nobody can verify.

Work one scenario at a time: name it, watch it fail for the reason you expect, make it pass. A red run that fails for the wrong reason means the scenario is testing something other than what its name claims.
