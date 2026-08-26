# ROGUELETTE — Game Design Specification

This document is the source of truth for **what Roguelette is**: its core loop, scoring rules, prototype scope, design principles, and open design questions.

Implementation architecture belongs in `ROGUELETTE_SOFTWARE_ARCHITECTURE.md`. Development order and milestones belong in `ROGUELETTE_DEVELOPMENT_ROADMAP.md`.

## High Concept

Roguelette is a European roulette–based roguelike inspired by the design philosophy of games like Balatro: a very simple base action, expanded through modifiers that let the player gradually bend and break the underlying rules.

The goal is not to simulate real gambling or bankroll management. The player is trying to reach a target score within a limited number of spins by placing a small number of fixed-value chips and building synergies over the course of a run.

The fantasy is:

> Start with a mostly normal roulette wheel. End with a broken machine whose outcomes interact with your build in increasingly powerful ways.

---

## Core Loop

**Choose a starting modifier → start a round → place 3 chips on numbers → spin the wheel → compare the result against each bet → gain score from matching relationships → repeat for the remaining spins → after 5 spins, check the target score → if successful, choose a new modifier → enter the next round with a higher target → repeat until the run ends.**

---

## Base Betting Rules

Each spin gives the player **3 fixed-value chips**.

The player does **not** choose how much money to wager. The only decision is where to place the chips.

Each chip is placed directly on a number from the European roulette wheel.

When the wheel lands on a number, the result is compared separately against every number the player bet on.

The current prototype uses five base relationships:

| Relationship | Prototype Multiplier |
|---|---:|
| Exact number | 12x |
| Physical neighbor on the roulette wheel | 4x |
| Same dozen | 2x |
| Same odd/even group | 1x |
| Same half (1–18 / 19–36) | 1x |

These values are placeholders and should be balanced through playtesting.

Multiple matching conditions can stack.

Example:

The player bets on **34** and the wheel lands on **32**.

34 and 32 are:

- in the same dozen,
- both even,
- both in the 19–36 half.

The bet therefore produces:

**2x + 1x + 1x = 4x score.**

An exact hit also receives the other relationships that apply unless later testing shows that exact hits should use a separate calculation.

---

## Why Bets Can Still Matter When They Miss

Classic roulette is mostly:

**Choose a bet → random result → win or lose.**

Roguelette changes this by making the landed number an input into a scoring system rather than a simple verdict.

A bet can still create value without landing on the exact number.

The player is therefore not only asking:

> “Will my number hit?”

They are also asking:

> “How many possible outcomes interact positively with the numbers I chose?”

This is the core strategic difference between Roguelette and ordinary roulette.

---

## Round Structure

Current prototype assumption:

- 3 bets per spin
- 5 spins per round
- One target score per round

The player must reach or exceed the target score before the round ends.

Passing the target lets the player continue the run and obtain another modifier.

Target scores rise as the run progresses.

The exact progression curve is intentionally undecided until the expected score output of the base system and early modifiers has been simulated and playtested.

---

## Roguelike Modifiers

The player chooses one modifier at the beginning of the run.

Additional modifiers are earned between successful rounds.

Modifiers should alter how the roulette system behaves rather than merely adding flat numerical bonuses.

Possible modifier directions include:

- strengthening an existing relationship,
- causing certain outcomes to retrigger,
- changing how specific numbers behave,
- modifying the roulette wheel itself,
- making normally weak outcomes valuable,
- creating interactions between number properties,
- changing how placed chips score.

The design goal is for modifiers to combine with one another so that builds emerge naturally.

A strong late-game build should make the player feel that they have gradually transformed the roulette wheel into a machine designed around their own strategy.

---

## Strategic Goal

Early in a run, the player mostly reasons about the basic relationships between numbers.

As modifiers accumulate, the central question becomes:

> “Where should I place these 3 chips to extract the most value from my current build?”

The game should gradually move from **roulette prediction** toward **build optimization**.

Randomness remains important, but the player increasingly shapes which random outcomes are useful.

The goal is not to remove randomness.

The goal is to let the player progressively make more of the wheel work in their favor.

---

## Readability / UX

The player should not be required to memorize every relationship or modifier interaction.

After bets are placed, hovering over a possible wheel result should be able to preview what would happen if that number landed.

For example:

**32 → 4x**

Reason:
- Same dozen
- Same parity
- Same half

As builds become more complicated, this preview system becomes increasingly important.

Complexity should come from building the machine, not from fighting the interface.

---

## Scope Rules

For the initial prototype, avoid adding systems that are not required to prove the core loop.

Currently excluded:

- bankroll management,
- variable bet sizes,
- realistic casino economy,
- street bets,
- column bets,
- complex roulette betting layouts,
- story systems,
- meta progression,
- large shops,
- elaborate boss systems.

These can be reconsidered later.

The first prototype only needs to answer:

> Is placing 3 bets, spinning the wheel, reading the resulting interactions, and developing a modifier build fun?

If this is not fun on its own, additional systems should not be used to hide the problem.

---

## Current Design Principles

1. **Simple input, deep consequences.**  
   The player performs very few actions, but those actions should interact with an increasingly complex system.

2. **Randomness creates situations; it should not completely dictate outcomes.**  
   Missing an exact number does not necessarily mean the spin was worthless.

3. **Builds should change decision-making.**  
   A good modifier should affect where the player wants to place chips.

4. **The wheel should become increasingly “broken.”**  
   Progression should visibly and mechanically distort normal roulette logic.

5. **Roguelike elements should enhance the core loop, not rescue it.**  
   The betting and scoring system must already contain meaningful decisions before large amounts of content are added.

6. **Keep the initial rules readable.**  
   The current base relationships are limited to exact number, physical neighbor, dozen, odd/even, and half.

---

## Important Open Questions

These are intentionally unresolved and should be tested rather than assumed.

- What should the final base multipliers be?
- Should exact hits stack with all other matching relationships?
- What is the ideal number of chips per spin?
- Is 5 spins per round the correct pacing?
- What should the target-score progression curve look like?
- How strong should the starting modifier be?
- How many modifiers should a typical run contain?
- Which modifiers create genuine positional decisions rather than automatic bonuses?
- How should 0 interact with the base relationship system?
- How much information should the outcome-preview UI reveal?
- Should late-game modifiers modify bets, the wheel, or both?
- At what point should boss-style rule changes be introduced, if at all?

---

## Current Prototype Definition

The minimum playable Roguelette prototype is:

**European roulette wheel + 3 fixed bets per spin + 5 spins per round + five base scoring relationships + one starting modifier + rising target scores + one modifier reward after each successful round.**

Nothing else is required until this version has been tested.
