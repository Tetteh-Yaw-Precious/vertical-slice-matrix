# Vertical Slice Matrix

A skill that fires at two moments most work gets wrong: when you are about to
propose a plan, and when you are about to write something the codebase already
has.

A feature list read **down** gives you features. Read **across**, it gives you
the handful of capabilities several features share. Those are the slices. The
same capability built three times is the most expensive thing in a codebase, and
by the time you can see all three it is too late to merge them cheaply.

## Why this exists

Two failures, one skill.

**Planning produces layers.** Ask for a build order and you get `Phase 1:
Schema Hardening`, `Phase 2: Safety Infrastructure`, `Phase 3: Cleanup`. Every
item is honest work. Nothing ships to a user when one completes, nobody outside
the conversation can tell when one is done, and the plan quietly becomes a
to-do list with ambitions.

**Building produces copies.** You need a date formatter. You write
`formatRelativeTime`. The existing one is called `timeAgo`, so your grep found
nothing and now there are two. Repeat for a year. Worse, the thing you
duplicated was never a formatter at all, it was a *list*: the set of order
statuses, or payment methods, or permissions. Nobody chose to duplicate that.
It spread because each new value got typed into whichever files the author had
open at the time.

Those two failures want opposite treatment, which is the part this skill gets
right and most dedup advice gets wrong. See [behaviour or fact](#first-question-behaviour-or-fact).

The sharpest consequence, and the reason Mode B was rewritten: **a stale
declaration of a list does not just add one more wrong copy, it silently
switches off every exhaustiveness check downstream of it.** That is how a set of
order statuses or payment methods can be wrong in a dozen places and still
compile green. See [the incident](#the-incident-that-rewrote-mode-b).

> **This is not vertical slice architecture.** If you came here for the .NET
> pattern (one folder per feature, MediatR handlers, FluentValidation), that is a
> different thing that shares the words. This skill is about which capabilities
> to build in what order, and about finding the one declaration a list should
> derive from. It is stack-agnostic and says nothing about folder layout.

## Install

This is a skill for [Claude Code](https://claude.com/claude-code) and any
harness that reads the same format.

```bash
# personal scope, active in every project
git clone https://github.com/Tetteh-Yaw-Precious/vertical-slice-matrix \
  ~/.claude/skills/vertical-slice-matrix

# or project scope, checked in and shared with the team
git clone https://github.com/Tetteh-Yaw-Precious/vertical-slice-matrix \
  .claude/skills/vertical-slice-matrix
```

The only required file is `SKILL.md`. Nothing to configure, no dependencies, no
language or stack assumptions.

If your project has a `CLAUDE.md`, pointing at the moments is what makes it
actually fire rather than sit there:

```markdown
| Moment                                                    | Skill                        |
| --------------------------------------------------------- | ---------------------------- |
| About to propose a sequence of work, or asked "what next"  | vertical-slice-matrix (A)    |
| About to write any helper, formatter, hook or component    | vertical-slice-matrix (B)    |
| About to add a value to an enum, status set or option list | vertical-slice-matrix (B)    |
```

You can also read it as a document and apply it by hand. It is prose and tables,
not code.

## Pick a mode first

| You are | Mode |
| --- | --- |
| Holding a wish list, planning what to build and in what order | A |
| About to write a helper, hook, formatter or component | B |
| About to add or remove a value in an enum, status set, permission list | B, and its first section before anything else |

---

## Mode A: planning a project or milestone

Finding the slices is the easy half. What gets skipped is producing something
durable, so the rule is blunt: **a plan in prose evaporates, build the board.**

### Finding a slice

1. Write the wants as rows, in the user's words. No solution nouns yet.
2. Under each want, name the capability it needs as a **verb phrase**. "Decide
   who is allowed to do this", not "AuthService".
3. Where the same capability sits under two or more wants, that is a candidate
   slice. One want only is a feature, not a slice.
4. Name the slice by its **outcome**. `Slice 3 · See the week`, not
   `Calendar module`. The test: someone who has never read the codebase can tell
   you what changed when it ships.

### The layer test

Apply it to every candidate. If the name is a layer or a phase, it is not a
slice. `Schema Hardening`, `Safety Infrastructure`, `Operational Hardening`,
`Structural Cleanup`, `Phase 2` are horizontal cuts wearing slice clothing. They
can be true, useful and worth doing, and they are still not slices, because
nothing ships to anyone when one completes.

A candidate that fails usually wants to be split across the slices that consume
it, or demoted to a checklist inside one.

### Every slice has a kind

Tag each piece of work inside a slice. The tag is a claim you can be wrong
about, which is the point.

| Kind | What it is | Test |
| --- | --- | --- |
| **Engine** | the mechanism the slice adds | lives in one place, called from many |
| **Fold** | what regenerates from the engine | delete it, regenerate, output is identical |
| **Guardrail** | the ratcheted check | fails the build when the number gets worse |
| **Surface** | what the user touches | a screenshot shows the change |

A slice that is all Surface is a feature with a big name. A slice with no
Guardrail will drift back and you will rebuild it. A slice with no Engine is a
chore list.

Folds are the kind people forget and where the leverage is: read models,
generated registries, one parametrised test per table row, an exported state
diagram. Anything you would otherwise hand-maintain in parallel with the engine
and watch drift.

### Done when

```
Done when: engine exists · folds regenerate · guardrail ratcheted · surface ships
```

Drop the clauses that do not apply. **Every clause that stays must be checkable
by someone who was not in the conversation.** Not "implement the command table",
but "the router refuses at boot to mount a command the table does not declare".

If you cannot write `Done when` for a slice, the slice is not scoped yet. That
is the finding, not a blocker to work around.

### Order comes from cost of deferral

Slices carry a `used by N` count. It marks what must not drift, the thing worth
a guardrail. **It is not a sort key.** Do not build most-depended-on first.
Build a slice when its *second* consumer arrives, so the shape is settled by two
real cases instead of one guess.

The sort key is one line per slice: what gets worse if this arrives later? Most
answer "nothing much, it is the same work either way". Those wait.

The slice whose answer is *"retrofitting this across everything already built is
a different and much worse project"* goes first, even with no visible change.
Tenancy isolation, audit trails, money representation, tenant-scoped IDs. Call
it `Slice 0 · The floor`. There is usually exactly one, occasionally none, and
if you think you have three you have not cut them finely enough. This is the one
exception to waiting for the second consumer, because on the floor there is no
later.

---

## Mode B: an existing codebase

The trigger is **feature work, not refactor time.** Refactor time never arrives;
nobody funds a sprint that ships no user-visible change. But every feature makes
you go looking for where a capability lives anyway. This changes what you do
with what you find.

> Before writing it, look for it. If it is already there several times, your
> feature ships on one consolidated version, and one copy is gone.

### First question: behaviour or fact?

Answer this before you count anything. Two different things look identical in a
grep and have **opposite** correct answers, and getting the routing wrong is
worse than not looking at all.

**Behaviour** can have several implementations without any of them being wrong:
a formatter, a hook, a validator, a component. Duplication costs maintenance and
nothing else.

**A fact** is one truth that many places must agree on: a set of payment
methods, order statuses, permissions, event types, route names, feature flags,
error codes.

The test is one question. **If someone adds a member tomorrow, how many files
have to change for the codebase to still be correct?** One means the fact has a
home. More than one means it has leaked, and every site that misses the update
is wrong without saying so.

### Facts: find the leak

**Grep a member, never the name.** The copies of a fact rarely share a name.
Three sites holding the same list of payment methods were called
`formatPaymentMethod`, `getPaymentMethodLabel`, and an unnamed object literal
inline in a component, so any search by name returns one hit and reads as
conclusive.

```bash
grep -rn "mobile_money" .          # a member, not the set
grep -rn "awaiting_approval" .
```

Members are the one thing every copy must contain, in every language, whatever
anybody named the wrapper. This is the single highest-yield search in the skill,
and it works whether or not you use an agent.

Then: the threshold is **one** (two copies of a fact is already broken, just
waiting for the next value to prove it). Ask what the compiler was supposed to
catch, and whether it is telling the truth. Make the copies **derive** from the
declaration closest to the wire format or storage layer. Name the residue that
genuinely cannot derive (a database CHECK constraint, a framework struct tag, a
service in another language) and put one narrow test on each.

The deliverable names the source of truth, because for a fact a count is noise.
`14 → 13` reads as progress while describing a codebase still broken in thirteen
places.

```
payment methods: one declaration at api/invoices/model.go
derived: shared-core types (generated), both binding validators
residue: payments_payment_method_check, covered by a test
```

### Behaviour: the threshold

| Copies found | Do |
| --- | --- |
| 1 to 2 | Use it, or leave them. Duplication is cheaper than the wrong abstraction. |
| 3 | Judgement. Consolidate only if they change together. |
| 4+ | Consolidate one. You are paying rent every sprint. |

Count them all before touching anything, and **search the concept before the
symbol**: a grep only finds a thing you can already name, and the case that
hurts is `formatRelativeTime` versus the existing `timeAgo`. Diff the copies,
because copies that disagree are a live bug and the disagreement tells you which
behaviour is canonical. Pick one home, migrate the call site your feature
needed, delete its copy in the same change, leave the rest counted.

Then state the count, explicitly, in the PR body:

```
formatBytes: 5 → 4
remaining: ui/FileRow.tsx, admin/UsageTable.tsx, reports/ExportSummary.tsx
```

A change that adds a canonical home and moves no number has not consolidated
anything, it has forked. There are now N+1 implementations and the newest one
claims to be authoritative, so the next reader trusts it and never learns that
four screens bypass it. A re-export shim does not count as removal: it keeps the
old path valid, so nothing forces migration and the number never moves.

### Refuse loudly

Stop and raise it as its own ticket when merging would change behaviour a user
sees (two copies rendering `45m` and `45 min` cannot be unified without picking
one, and that is a product decision), when the copies do not change together
(check the history, files that never move in the same commit are two things that
happen to rhyme), or when the scope is growing (one consolidation per feature).

Say which one, and why. A silent refusal is indistinguishable from not looking.

---

## The incident that rewrote Mode B

Mode B originally taught the threshold, gradual migration and a stated count for
everything. That is wrong for facts, and here is the bill.

A payment-method list was declared about fourteen times across a Go API and a
TypeScript monorepo. The canonical TypeScript union was stale by one member. The
label maps were written as `Record<Union, string>`, which is exhaustive, so a
five-key map checked against a five-member union compiled perfectly clean.

Gift-card payment shipped. Receipts printed the raw string `gift_card`.

The lesson is not "we missed a file". It is that **a stale declaration does not
just add one more wrong copy, it silently switches off every exhaustiveness
check downstream of it**, which is how a list can be wrong in a dozen places and
still go green in CI.

The deeper diagnosis, which drove the rewrite: the old Mode B taught *managing*
duplication, and maintenance thinking cannot produce a source of truth. If the
answer is a fact, the question is never how many copies there are. It is which
single declaration the rest should derive from.

## Honest limits

- **Mode A pays off on greenfield.** On a live product with real users, build
  order comes from customer feedback and incidents, not from a dependency graph.
  Use Mode A for a new project or a new milestone, not to re-cut a backlog that
  is already in flight.
- **Do not retrofit slices onto an existing issue tracker.** A board nobody
  reopens is decoration.
- **The counts are a signal, not arithmetic.** Draw the boxes differently and
  the numbers change.
- **`FUTURE` rows never become tickets.** They shape the slices, which is free.
  The moment one becomes a ticket, someone works it, and you have built
  speculation.

## Related

The ratchet in Mode B (a check that fails if a count goes up, baselined at
today) is deliberately not specified here. If you want it, pair this with an
`architecture-guardrails` skill that owns the mechanics of enforced checks. The
`Done when` clause `guardrail ratcheted` assumes something like it exists.

## Credits

Written and used in production by [@Tetteh-Yaw-Precious](https://github.com/Tetteh-Yaw-Precious). Mode A's
vocabulary (Engine, Fold, Guardrail, Surface) and the layer test came out of
cutting a real roadmap badly first.

## License

MIT.
