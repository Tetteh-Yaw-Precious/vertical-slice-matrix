---
name: vertical-slice-matrix
description: Use BEFORE writing any helper, formatter, hook, utility or component in an existing codebase, because the repo likely already has one, and this counts the copies before you add another. Also use BEFORE adding or removing a value in any closed set (an enum, union, status list, permission list, payment or delivery method, event type, role, feature flag, error or currency code), because that list is probably declared in several files that will not all get updated, and the one that silently switches off the compiler's exhaustiveness check is the one you will miss. Triggers on "add a new status", "add a payment method", "add a role", "support another provider", "add a type to the enum". Invoked from feature-cycle steps 3 and 6, or on its own. Also use BEFORE proposing any sequence of work (a numbered plan, a set of steps, an answer to "what next" or "what do you prescribe"), because a sequence IS a build order and every item needs a checkable Done when. Also when planning build order for a new project or milestone from a feature list, deciding how to cut epics, or when a roadmap has come out shaped like layers instead of outcomes. Triggers on proposing steps or phases, on "what should we do next", on "what do you prescribe", on adding a util or component to an existing repo, on "where should this live", "we've built this three times", "plan the build order", "what should we build first", "how should we cut this", and on any feature needing a capability other features also need. Portable across projects, stacks and languages.
---

# Vertical Slice Matrix

A feature list read **down** gives you features. Read **across**, it gives you the
handful of capabilities several features share. Those are the slices, and the
same capability built three times is the most expensive thing in a codebase.

**Pick your mode before reading further:**

| You are | Go to |
|---|---|
| Holding a wish list, planning what to build and in what order | Mode A |
| About to write a helper, hook, formatter or component | Mode B |
| About to add a value to an enum, status set, permission list or option list | Mode B, and read its first section before anything else |

---

## Mode A: planning a new project or milestone

Finding the slices is the easy half; agents do it unprompted. What gets skipped
is producing something durable. **A plan in prose evaporates. Build the board.**

### Finding a slice

Do not skip this because slices feel obvious. This is the step where a board
comes out shaped like layers, and a layer-shaped board is worse than no board.

1. **Write the wants as rows, in the user's words.** No solution nouns yet.
2. **Under each want, name the capability it needs as a verb phrase.** "Decide
   who is allowed to do this", not "AuthService".
3. **Where the same capability sits under two or more wants, that is a candidate
   slice.** One want only is a feature, not a slice.
4. **Name the slice by its outcome.** `Slice 3 · See the week`, not
   `Calendar module`. The test: someone who has never read the codebase can tell
   you what changed when it ships.

**The layer test: apply it to every candidate.** If the name is a layer or a
phase, it is not a slice. `Schema Hardening`, `Safety Infrastructure`,
`Operational Hardening`, `Structural Cleanup`, `Phase 2` are all horizontal cuts
wearing slice clothing. They can be true, useful and worth doing, and they are
still not slices: nothing ships to anyone when one completes.

A slice that fails the layer test usually wants to be split across the slices
that actually consume it, or demoted to a checklist inside one.

### The board: three columns

**1. Read through**: the product decisions everything is read through. Five or
six lines. *Decisions, not implementation.* If you cannot write them, stop: you
do not know enough yet to tell a real slice from a coincidence.

**2. What people want**: one row per want, in the user's words. *"Stop losing
shifts to callouts"*, never *"Shift reassignment module"*.

Include rows you are **not** building, marked `FUTURE`. They are free and they
shape the slices.

**3. Build order**: the slices, each one cutting across the rows it serves.

### Slices carry a count

```
Slice 3 · Produce a signed, versioned record     used by 10
```

`used by N` marks what must not drift: the thing worth a guardrail. It is a
judgement, not a measurement.

**It is not a sort key.** Do not build most-depended-on first. Build a slice when
its *second* consumer arrives, so the shape is settled by two real cases instead
of one guess. Finding the engine early stops you writing it three times; it does
not follow that you write it on day one.

### Every slice has a kind

Tag each piece of work inside a slice. Four kinds, and the tag is a claim you
can be wrong about:

| Kind | What it is | Test |
|---|---|---|
| **Engine** | the mechanism the slice adds | lives in one place, called from many |
| **Fold** | what regenerates from the engine | delete it, regenerate, output is identical |
| **Guardrail** | the ratcheted check | fails the build when the number gets worse |
| **Surface** | what the user touches | a screenshot shows the change |

Folds are the kind people forget, and they are where the leverage is: read
models, generated registries, one parametrised test per table row, an exported
state diagram. Anything you would otherwise hand-maintain in parallel with the
engine and watch drift.

- A slice that is **all Surface** is a feature with a big name.
- A slice with **no Guardrail** will drift back, and you will rebuild it.
- A slice with **no Engine** is a chore list.

### Done when

The count tells you what matters. The kinds tell you what it is made of. This
tells you when to stop:

```
Done when: engine exists · folds regenerate · guardrail ratcheted · surface ships
```

Drop the clauses that do not apply. **Every clause that stays must be checkable
by someone who was not in the conversation.** Not "implement the command table",
but "the router refuses at boot to mount a command the table does not declare".

If you cannot write `Done when` for a slice, the slice is not scoped yet. That
is the finding, not a blocker to work around.

### Build order comes from cost of deferral

Column 3 is called build order, and `used by N` is explicitly not the sort key.
This is:

1. **Dependency.** You cannot build B without A. Rare, and usually less binding
   than it first looks.
2. **Cost of deferral.** One line per slice: what gets worse if this arrives
   later? Most slices answer "nothing much, it is the same work". Those wait for
   their second consumer.

The slice whose answer is *"retrofitting this across everything already built is
a different and much worse project"* goes first, even with no visible change.
Tenancy isolation, audit trails, money representation, tenant-scoped IDs. Call
it **Slice 0 · The floor**. There is usually exactly one, occasionally none, and
if you think you have three you have not cut them finely enough.

**This is the one exception to waiting for the second consumer.** Everywhere
else, two real cases beat one guess. On the floor, there is no later.

### FUTURE rows are the discipline

Where a slice crosses a `FUTURE` row, mark it:

```
prepared for later · not built
```

Two opposite things at once. **Prepared**: the slice was designed knowing that
want is coming, so it will fit. **Not built**: zero lines of it were written.

- ✅ *"This slice must not make per-tenant branding impossible"*, a constraint on
  something you are building anyway. Free.
- ❌ *"Let's add a theming system, we'll probably need it"*, speculative. Wrong.

**FUTURE rows never become tickets.** The moment one does, someone works it.

---

## Mode B: an existing codebase

The trigger is **feature work, not refactor time.** Refactor time never arrives;
nobody funds a sprint that ships no user-visible change. But every feature makes
you look for where a capability lives anyway. This changes what you do with what
you find.

> Before writing it, look for it. If it is already there several times, your
> feature ships on one consolidated version, and one copy is gone.

### First: behaviour, or fact?

**Answer this before you count anything.** Two different things look identical in
a grep and have opposite correct answers, and getting the routing wrong is worse
than not looking at all.

**Behaviour** is something that can have several implementations without any of
them being wrong: a formatter, a hook, a validator, a component. Duplication
costs maintenance and nothing else. The threshold applies, you consolidate
gradually, and two copies really is cheaper than the wrong abstraction.

**A fact** is one truth that many places must agree on: a set of payment
methods, order statuses, permissions, event types, route names, feature flags,
error codes. Nobody chose to duplicate a fact. It spread because each new value
got typed into whichever files the author had open at the time.

The test is one question. **If someone adds a member tomorrow, how many files
have to change for the codebase to still be correct?** One means the fact has a
home. More than one means it has leaked, and every site that misses the update
is wrong without saying so.

For a fact, stop reading the threshold. It does not apply, and nor does
migrating one call site at a time. Use Mode A's vocabulary instead: one
**engine** that holds the fact, and every other site a **fold** that derives
from it and stores nothing of its own. Delete a fold, regenerate it, and the
output is identical. A copy you update by hand is not a fold, however tidy it
looks.

### Facts: find the leak

**1. Grep a member, never the name.** The copies of a fact rarely share a name.
Three sites holding the same list of payment methods were called
`formatPaymentMethod`, `getPaymentMethodLabel`, and an unnamed object literal
inline in a component, so any search by name returns one hit and reads as
conclusive. Search for a value instead:

```bash
grep -rn "mobile_money" .          # a member, not the set
grep -rn "awaiting_approval" .
```

Members are the one thing every copy must contain, in every language, whatever
anybody named the wrapper. This is the single highest-yield search in this
skill.

**2. The threshold is one.** Two copies of a fact is already broken; it is
simply waiting for the next value to prove it. There is no free pass at one or
two.

**3. Ask what the compiler was supposed to catch.** Most languages give you an
exhaustiveness check for free: a total map keyed by a union, a switch with no
default, a sealed type. Find whether one exists here, then check whether it is
telling the truth. A stale declaration does not just add one more wrong copy, it
silently switches off every check downstream of it, which is how a list can be
wrong in a dozen places and still compile green.

**4. Make the copies derive.** Pick the declaration closest to the wire format or
the storage layer, and generate or import the rest from it. The job is finished
when adding a member means editing one file and watching everything that has not
caught up refuse to build.

**5. Name the residue.** Some declarations genuinely cannot derive: a database
CHECK constraint, a framework's struct tag, a copy in a service written in
another language. Count them, say so out loud, and put one narrow test on each.
That is the only place a checker earns its keep, and it is usually one or two
sites rather than a dozen.

### Facts: the deliverable names the source of truth

For a fact, a count is noise. `14 → 13` reads as progress while describing a
codebase still broken in thirteen places. Say this instead:

```
payment methods: one declaration at api/invoices/model.go
derived: client types (generated), both binding validators
residue: payments_payment_method_check, covered by a test
```

### Behaviour: the threshold

| Copies found | Do |
|---|---|
| 1–2 | Use it, or leave them. Duplication is cheaper than the wrong abstraction. |
| 3 | Judgement. Consolidate only if they change together. |
| 4+ | Consolidate one. You are paying rent every sprint. |

### Behaviour: the recipe

**1. Count them all before touching anything.**

```bash
grep -rn "function formatBytes" src/ | grep -v '\.test\.'
grep -rn "func formatBytes" --include="*.go" . | grep -v '_test\.go'
```

Stopping at the first hit is the common failure: it finds the exported one and
misses the private copies, which are the ones causing the bug.

**Search the concept before the symbol.** Those greps only find a thing you can
already name, and the case that hurts is the one where you cannot: you are about
to write `formatRelativeTime` and the existing one is called `timeAgo`. If the
project keeps a concept index (a keywords line in file headers, a prior-art
script, a tagged registry), search it first, with the words you would say out
loud rather than the identifier you were about to type. Where no index exists,
try two or three different names for the same idea before concluding it is
absent; a single symbol grep returning nothing is weak evidence.

**2. Diff them.** Copies that disagree are a live bug, and the disagreement tells
you which behaviour is canonical. Copies that agree are cheap to merge.

**3. Pick one home and move the implementation there.**

**4. Migrate the call site your feature needed, and delete its copy in the same
change.** One. Not all of them.

**5. Leave the rest counted.**

### Behaviour: the deliverable states the count

Every consolidating change says this, explicitly, in the PR body or handover:

```
formatBytes: 5 → 4
remaining: ui/FileRow.tsx, admin/UsageTable.tsx, reports/ExportSummary.tsx
```

A change that adds a canonical home and moves no number has not consolidated
anything, it has forked. There are now N+1 implementations and the newest one
claims to be authoritative, so the next reader trusts it and never learns that
four screens bypass it.

**A re-export shim does not count as removal.** It keeps the old path valid, so
nothing forces migration and the number never moves.

### Refuse loudly

Stop and raise it as its own ticket when:

- **Merging would change behaviour a user sees.** Two copies rendering `45m` and
  `45 min` cannot be unified without picking one. That is a product decision.
- **They do not change together.** Check the history. Files that never move in
  the same commit are two things that happen to rhyme; merging them couples
  things that were correctly separate.
- **The scope is growing.** One consolidation per feature. If your feature is
  becoming a refactor, ship the feature and leave the count where it is.

Say which one, and why. A silent refusal is indistinguishable from not looking.

### Make it ratchet

Once a count exists, hold it: a check that fails if the number goes up, baselined
at today. Nobody has to finish the consolidation, it just cannot get worse.

**REQUIRED BACKGROUND for building that check:** use the `architecture-guardrails`
skill. Do not re-derive the ratchet mechanics here.

---

## Quick reference

| Situation | Move |
|---|---|
| Wish list, nothing built | Mode A. Board first, engines named, nothing built yet |
| About to write a helper | `grep` for it first |
| About to add a value to an enum, status set or option list | That is a fact. Route to Facts, not the threshold |
| Looking for copies of a list | Grep a member, never the name |
| Found 2 copies of a fact | Already broken. No free pass |
| Found 4 copies | Consolidate one, delete it, state the count |
| Found 2 copies | Leave them |
| Copies disagree | Separate ticket, it is a product decision |
| A list is wrong everywhere and still compiles | A stale declaration switched the compiler off. Fix that first |
| Adding a canonical home | Not done until a copy is deleted |
| Slice touches 2 rows | Not a slice. A feature with a big name |
| Slice named for a layer | Not a slice. Split it across the slices that consume it |
| Cannot write `Done when` | Not scoped. Say so |
| Deferring it means retrofitting everything | Slice 0. Build it first |

## Common mistakes

**Ordering by `used by N`.** It marks what must not drift, not what is urgent.

**Treating the counts as measurement.** Draw the boxes differently and the numbers
change. They are a signal, not arithmetic.

**Cutting by layer.** Hardening, infrastructure, cleanup and phases are honest
work and terrible slices. Nothing ships when one finishes.

**Skipping the kinds.** A slice with no Guardrail drifts back; a slice that is
all Surface was never a slice.

**`Done when` that only the author can check.** "Implement X" is a restatement of
the title, not a completion condition.

**Building FUTURE rows.** They shape slices. They do not get tickets.

**Consolidating on resemblance.** Two things that look alike and never change
together are two things. Check the history before merging.

**Treating a fact like behaviour.** The threshold, the gradual migration and the
count all assume several implementations may coexist. Apply them to a list of
statuses or payment methods and you will leave most of the copies stale on
purpose, then ship the next value into production wrong. Route first, count
second.

**Managing duplication instead of ending it.** Counting copies, baselining them
and ratcheting the number is maintenance, and maintenance thinking cannot
produce a source of truth. If the answer is a fact, the question is never how
many copies there are; it is which single declaration the rest should derive
from.

**Adding the engine and moving on.** The count is the deliverable. If it did not
go down, nothing was consolidated.

**Drawing the board once.** A board nobody reopens is decoration. The ratchet is
what brings you back to it.
