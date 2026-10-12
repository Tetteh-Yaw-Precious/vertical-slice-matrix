# vertical-slice-matrix

> **In plain terms:** Two things go wrong when software gets built. You plan the work and end up
> with a list of "phases" that nobody outside the room can tell are finished. And you build a small
> piece of code without noticing that the same piece has already been written four times, in four
> places, that now disagree with each other. This is a checklist an AI assistant follows at exactly
> those two moments. It is the kitchen-drawer problem: four people each bought their own tin opener,
> and the recipe is pinned in four places with different ingredients on each copy.

The rest of this page explains both halves, then how to install it. You don't need to already know
the tools to follow along.

## What am I looking at?

This is a **skill for AI coding agents** like Claude Code, Cursor, Codex, and Gemini CLI, the tools
that write code alongside you. A skill is a set of written instructions the agent reads and
follows. You install it once and don't call it. The agent notices the two moments below and uses
it.

> **Not to be confused with "vertical slice architecture."** If you came here for the .NET pattern
> (one folder per feature, MediatR handlers, FluentValidation), that's a different thing that
> happens to share the words. This cares about which capabilities to build in what order, and about
> finding the one place a list should be written down. It says nothing about folder layout and works
> in any language.

## The story that explains the second half

A list of ways a customer could pay had been typed into about fourteen different files across the
codebase. Nobody chose that. Each time somebody added a payment method, they typed it into whichever
files they happened to have open.

One of those fourteen was supposed to be the official one, and it was out of date by a single entry.
Now, normally a programming language will catch that for you: if the official list has five items
and a screen only handles five items, and you add a sixth, the language complains. **But the
official list was the thing that was stale.** Five against five looked perfect. Every check
downstream went quiet.

So gift cards shipped as a way to pay, and the receipt printed `gift_card` at the customer.

The lesson isn't "we missed a file". It's that **one out-of-date copy of a list doesn't just add one
more wrong place. It silently switches off the safety checks everywhere else**, which is how
something can be wrong in a dozen places and still look completely fine.

## Half one: before you write a small piece of code

The agent asks one question first, because two things that look identical have opposite right
answers.

**Is it behaviour, or is it a fact?**

**Behaviour** is something that could reasonably be written more than one way and none of them are
wrong: a date formatter, a bit of validation, a button component. Duplicating it costs you
maintenance and nothing worse. Here the advice is relaxed: one or two copies, leave them alone,
because two copies is cheaper than the wrong shared abstraction. At four or more you're paying rent
every month, so your next piece of work folds one of them in, deletes it, and says so in writing:

```
formatBytes: 5 -> 4
remaining: ui/FileRow.tsx, admin/UsageTable.tsx, reports/ExportSummary.tsx
```

Adding a new "official" version without deleting a copy isn't tidying up, it's making a fifth one
that claims to be in charge. The next person believes it and never learns that four screens go
round the back.

**A fact** is one truth that lots of places have to agree about: the ways you can pay, the states an
order can be in, who's allowed to do what, the kinds of notification you send. The test is a single
question: *if somebody adds one more tomorrow, how many files have to change for the code to still
be right?* One means it has a home. More than one means it's leaked.

For a fact the relaxed advice is thrown out. Two copies is already broken, it's just waiting for the
next entry to prove it. And the way to find the copies is the most useful trick in the whole skill:

```bash
grep -rn "mobile_money" .          # search for a thing in the list
```

**Search for an item, never the name of the list.** The copies almost never share a name. Three
places holding the same payment methods were called `formatPaymentMethod`,
`getPaymentMethodLabel`, and an unnamed lump of text inside a screen. Searching by name finds one of
them and reads like a definitive answer. Searching for `mobile_money` finds all three, because
whatever anyone called the wrapper, every copy has to contain the actual items.

## Half two: before you propose a plan

A numbered list of work is a build order, and most build orders come out shaped like layers.

**The layer test.** If the name of a piece of work is a layer or a phase, it isn't a real piece of
work. `Schema Hardening`, `Safety Infrastructure`, `Operational Hardening`, `Phase 2`. Each of
those can be completely honest and worth doing, and they're still wrong as plan items, because
**nothing reaches anybody when one of them finishes**. There's no moment where you can tell your
customer something happened.

So name each piece of work by what changes when it lands: `See the week`, not `Calendar module`.
Someone who has never read the code should be able to tell you what's different afterwards.

**Then write the finish line.** Every item gets a "done when" that **someone who wasn't in the
conversation could check**. Not "implement the command table", which is just the title again, but
"the app refuses to start if the table is missing an entry". If you can't write that sentence, the
item isn't understood well enough to be on the list yet. That's a finding, not an obstacle.

**And order by what gets worse if you wait.** The instinct is to build the most-depended-on thing
first. The skill says don't: build a shared thing when its *second* user shows up, so its shape is
settled by two real cases instead of one guess. One line per item answers "what gets worse if this
arrives later?", and for most of them the honest answer is "nothing, it's the same work either way".

The exception is the thing whose answer is *"retrofitting this across everything we've already built
is a different and much worse project"*: how money is represented, how customer data is kept apart,
the record of who did what. That one goes first even though nothing visible changes. There's
usually exactly one, and if you think you have three you haven't looked closely enough.

## Requirements

- An AI coding agent that supports the Agent Skills standard: Claude Code, Cursor (2.4+), Codex,
  Gemini CLI, or GitHub Copilot CLI. They all read the same `SKILL.md` format, so one copy works in
  any of them.

No other dependencies. It's instructions and tables, and it doesn't care what language your project
is written in.

## Install

| Agent | Where skills live |
|-------|-------------------|
| Claude Code | `~/.claude/skills/` (your home folder, used by every project) |
| Codex | `~/.agents/skills/` |
| Gemini CLI | `~/.agents/skills/` |
| GitHub Copilot CLI | `~/.agents/skills/` |
| Cursor (2.4+) | `.cursor/skills/` **inside the project** you want to use it in |

```bash
# Claude Code
git clone https://github.com/Tetteh-Yaw-Precious/vertical-slice-matrix.git \
  ~/.claude/skills/vertical-slice-matrix

# Codex, Gemini CLI, or Copilot CLI
git clone https://github.com/Tetteh-Yaw-Precious/vertical-slice-matrix.git \
  ~/.agents/skills/vertical-slice-matrix
```

If your project has a `CLAUDE.md` or `AGENTS.md`, naming the moments is what makes it actually fire
rather than sit there unused:

```markdown
| Moment                                                     | Skill                     |
| ---------------------------------------------------------- | ------------------------- |
| About to propose a sequence of work, or asked "what next"   | vertical-slice-matrix     |
| About to write any helper, formatter, hook or component     | vertical-slice-matrix     |
| About to add a value to a fixed list of options             | vertical-slice-matrix     |
```

You can also just read `SKILL.md` and apply it by hand. It's prose and tables, not code.

## Honest limits

- **The planning half is for new work.** On a live product with real customers, the order comes from
  what they tell you and what broke last week, not from a dependency diagram. Use it for a new
  project or a new milestone, not to re-cut a backlog that's already moving.
- **Don't retrofit it onto an existing tracker.** A board nobody reopens is decoration.
- **The copy counts are a signal, not arithmetic.** Draw the boundaries differently and the numbers
  change.
- **Items you're deliberately not building never become tickets.** They're worth listing, because
  knowing they're coming shapes what you build now, and that's free. The moment one becomes a
  ticket, somebody works on it.

## How this sits next to Matt Pocock's `to-tickets`

[`to-tickets`](https://github.com/mattpocock/skills) is the nearest neighbour in the most widely
installed skill set there is, and it is worth knowing where the two agree, where they genuinely
disagree, and where it is simply better than this one.

**Shared ground, arrived at separately.** `to-tickets` already insists each ticket cuts "a narrow
but COMPLETE path through every layer (schema, API, UI, tests): vertical, NOT a horizontal slice of
one layer", and that "a completed slice is demoable or verifiable on its own". That is the layer
test and the outside-checkable finish line, in different words. Two people reached the same place,
which is decent evidence the place is right.

**The real disagreement: what orders the list.** `to-tickets` gives every ticket its blocking
edges and tells the agent to "work the frontier", meaning any ticket whose blockers are all done.
This skill demotes dependency deliberately, calling it "rare, and usually less binding than it
first looks", and sorts by cost of deferral instead.

Both answers are defensible, and they answer different questions. A dependency graph tells you what
*can* start now. It does not tell you what you will regret not having done first. Those come apart
exactly where it hurts: how money is represented has no blockers at all, so it sits available on the
frontier from day one and gets picked up in month four, by which point every feature already built
has an opinion about it.

**The thing neither of them shares.** `to-tickets` takes a plan and cuts it up. It never asks
whether a shared capability should exist yet. This skill does, and the answer is the strongest claim
here: build the shared thing when its **second** user turns up, so two real cases settle its shape
instead of one guess. There is exactly one exception, the floor, where there is no later. Nothing in
`to-tickets` contradicts that, so running both is reasonable.

**Where `to-tickets` is better, and this skill has no answer.** Wide refactors. A single mechanical
change whose blast radius fans across the codebase (rename a column, retype a shared symbol) cannot
be made into a vertical slice, because one edit breaks thousands of call sites and nothing can land
green. `to-tickets` handles it properly with expand then contract: add the new form beside the old
so nothing breaks, migrate the call sites in batches sized by blast radius with each batch its own
ticket, then delete the old form once no caller remains. The layer test in this skill would simply
reject "rename the column everywhere" as not a slice and tell you to split it across the slices that
consume it, which is not an answer for a mechanical rename. Use the expand-contract sequence there.

`to-tickets` also sizes every slice to fit in a single fresh context window, which is a constraint
about working with an agent rather than about the shape of the work. This skill has nothing to say
about that, and probably should.

## Companion skills

| Skill | How it relates |
|---|---|
| [`architecture-guardrails`](https://github.com/Tetteh-Yaw-Precious/architecture-guardrails) | Once you have a count of duplicate copies, that skill is how you stop it getting worse: an automatic check that fails if the number goes up. The "done when" line here assumes something like it exists. |
| [`feature-cycle`](https://github.com/Tetteh-Yaw-Precious/feature-cycle) | The loop around everything. It calls this skill at the point where it's about to write a small piece of code or add a value to a fixed list. |
| [`lifecycle-feature`](https://github.com/Tetteh-Yaw-Precious/lifecycle-feature) | A list of statuses is the textbook example of a fact, so check here before adding one, then build it with that. |

## License

MIT.
