---
name: bdd
description: Behaviour-driven development. Use when tests should read as a specification, when naming tests given/when/then, when a repo mandates BDD-style specs, or when turning acceptance criteria into executable examples.
---

# Behaviour-Driven Development

BDD is TDD with the vocabulary settled. The `tdd` skill owns the loop — red before green, one vertical slice at a time, seams agreed up front — and this skill owns what each test *says*: the **specification** the suite becomes when you read its names in order.

Call the Skill tool with "tdd" for the loop, and stay here for the language. Read `CONTEXT.md` and any glossary before naming anything, so the names carry the project's own words.

## The suite is the specification

The completion criterion for every cycle: **print the test names and read them as a document.** A stranger should learn what the system does from that list alone, without opening the code. If a name teaches them nothing, the name is wrong — rewrite it before moving on.

```
a schedule
  ✔ given an instant inside an item, reports that item as what is on now
  ✔ given the exact instant an item ends, the next item is already on
  ✔ given items that overlap in time, refuses to exist: a channel broadcasts one thing at a time
```

That is the deliverable. Passing is the minimum; reading as the spec is the bar.

## The unit is a scenario

One test is one **example** of one behaviour, in three beats:

- **Given** the context the behaviour needs
- **When** the one action under test
- **Then** the single observable outcome

Two `when`s means two scenarios. See [naming.md](naming.md) for the grammar that compresses those beats into one line, and [scenarios.md](scenarios.md) for the same behaviour written in `node:test`, Vitest and Gherkin, plus example tables.

## Ubiquitous language

Every noun and verb in a test name comes from the domain, matching the words in the code and the words the user says out loud. When the test says `schedule.resumeAfter(...)` and the team says "what's on now", one of the two is wrong and the conversation that fixes it is the point of BDD.

Name the **behaviour**, using the domain's words; the vocabulary in the names is the same vocabulary in the interface. When a name has no domain word available, that gap is a finding: raise it, agree the term, then write the test.

## Outside-in

Start at the outermost behaviour someone can observe, and let the examples you cannot yet satisfy pull the next layer into existence. Each inward step is its own red → green cycle at its own seam.

State the rule at the altitude it lives at. A domain rule reads in domain terms ("a channel broadcasts one thing at a time"), never in the vocabulary of the button that triggers it.

## Write the reason into the test

A scenario carries **why**, not only what. When a rule exists for a reason a reader would otherwise question, say it in the name or a one-line comment above it:

```js
it('given the clock still points at what just finished, moves on to the next item', () => {
  // Schedules built from feeds carry assumed durations. A video that runs short ends while
  // the clock still sits inside its slot — reading the clock alone would replay it forever.
```

This is the line that survives the refactor that deletes the code. It is also how a failing test tells a future reader what broke in the world, rather than which assertion moved.

## Anti-patterns

- **Should-itis**: `should return true` names a return value and hedges about it. State the behaviour in the present indicative: `reports nothing is on`.
- **Implementation leakage**: `calls resolveNext() twice` binds the name to the call graph. The tell is a name that changes during a refactor that changed no behaviour.
- **Scenario soup**: several `when`s in one test, so a failure cannot say which behaviour broke. Split per action.
- **Incidental detail**: setup that states values the behaviour never reads. Keep only what the `given` genuinely requires; push the rest into a builder or factory so the scenario shows its own point.
- **Restating the code**: `given duration 240, remaining is 240 minus offset` describes the arithmetic. Name the capability instead: `reports how far into it we have arrived`.
