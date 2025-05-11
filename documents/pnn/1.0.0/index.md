# Piece Name Notation (PNN) Specification

**Version**: 1.0.0
**Author**: [Sashité](https://sashite.com/)
**Published**: May 1, 2025
**License**: MIT License

---

## Overview

Piece Name Notation (PNN) defines a consistent and rule-agnostic format for representing pieces in abstract strategy board games. This specification establishes a standardized way to identify and represent pieces independent of any specific game rules or mechanics.

---

## Properties

* PNN is **rule-agnostic** and does not encode legality, validity, or game-specific conditions.
* PNN provides a **canonical representation**, ensuring that equivalent pieces yield identical strings.
* PNN allows for **state modifiers** to express special conditions without compromising rule neutrality.

---

## Constraints

* PNN supports exactly **two players**.
* Players are assigned distinct casing: **uppercase letters** (`A-Z`) represent pieces of the player who moves first in the initial position; **lowercase letters** (`a-z`) represent the second player.
* A maximum of **26 unique piece types per player** is allowed, as identifiers must use single letters (`a-z` or `A-Z`).
* Modifiers can only be applied to pieces **on the board**, as they express state information. Pieces held off the board (e.g., pieces in hand or captured pieces) must never include modifiers; only the base letter identifier is used.

---

## Format

A piece is represented by an identifier composed of a single ASCII letter, optionally modified by prefixes and/or suffixes.

### Piece Identifier Structure

```
[<prefix>]<letter>[<suffix>]
```

Where:
* `<letter>` is a single ASCII letter (`a-z` or `A-Z`).
* `<prefix>` is an optional modifier preceding the letter (`+` or `-`).
* `<suffix>` is an optional modifier following the letter (`=`, `<`, or `>`).

---

## Modifiers

Modifiers provide a mechanism to express the state of pieces on the board without compromising the rule-agnostic nature of the notation.

### Modifier Meanings

| Modifier | Meaning |
|----------|---------|
| `+`      | Alternative or enhanced state |
| `-`      | Diminished or restricted state |
| `=`      | Bidirectional or dual-option state |
| `<`      | Left-side constraint or condition |
| `>`      | Right-side constraint or condition |

### Modifier Application Rules

1. Modifiers are strictly optional.
2. A piece may have a prefix, a suffix, both, or neither.
3. Modifiers do not change the fundamental identity of the piece but only express variations in state.
4. Modifiers only apply to pieces **on the board**. Pieces off the board (in hand, captured) must use the **plain letter only**.

---

## Formal Grammar (BNF)

```bnf
<piece> ::= <letter>
         | <prefix> <letter>
         | <letter> <suffix>
         | <prefix> <letter> <suffix>

<prefix> ::= "+" | "-"
<suffix> ::= "=" | "<" | ">"

<letter> ::= <letter-lowercase> | <letter-uppercase>

<letter-lowercase> ::= "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
                    | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
                    | "u" | "v" | "w" | "x" | "y" | "z"

<letter-uppercase> ::= "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
                    | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
                    | "U" | "V" | "W" | "X" | "Y" | "Z"
```

---

## JSON Schema

The following JSON Schema defines the validation pattern for a PNN piece string:

```json
{
  "type": "string",
  "pattern": "^[-+]?[a-zA-Z][=<>]?$"
}
```

---

## PNN Implementations

This section lists available libraries and tools that implement the PNN specification, allowing developers to easily integrate this format into their applications.

### Ruby

* **[Pnn.rb](https://github.com/sashite/pnn.rb)** – Implementation of the Piece Name Notation specification for Ruby.

---

Copyright © 2020–2025 [Sashité](https://sashite.com/)
