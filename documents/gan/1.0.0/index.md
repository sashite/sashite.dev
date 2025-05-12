# General Actor Notation (GAN) Specification

**Version**: 1.0.0
**Author**: [Sashité](https://sashite.com/)
**Published**: May 4, 2012
**License**: MIT License

---

## Overview

General Actor Notation (GAN) defines a consistent and rule-agnostic format for representing game actors in abstract strategy board games. Building upon Piece Name Notation (PNN), GAN eliminates ambiguity by associating each piece with its originating game, allowing for unambiguous gameplay application and cross-game distinctions.

---

## Properties

- GAN is **rule-agnostic** and does not encode legality, validity, or game-specific conditions.
- GAN provides a **canonical representation**, ensuring that equivalent actors yield identical strings.
- GAN allows for **game-specific identification** while maintaining the state modifiers from PNN.
- GAN resolves **collision problems** between identical piece notations from different games.

---

## Constraints

- GAN inherits PNN constraints: support for exactly **two players**, with a maximum of **26 unique piece types per player**.
- Players are assigned distinct casing: **uppercase letters** (`A-Z`) represent pieces of the player who moves first in the initial position; **lowercase letters** (`a-z`) represent the second player.
- Game identifiers must follow the same casing convention as in the FEEN specification's third field.

---

## Terminology

- An **actor** is the complete identifier in GAN format: `<game-id>:<piece-id>`.
- A **piece** is the part after the colon: `<piece-id>`, which follows PNN rules.

---

## Format

An actor is represented by a game identifier, followed by a colon, followed by a piece identifier that follows the PNN specification.

### Actor Structure

```
<game-id>:<piece-id>
```

Where:
- `<game-id>` is a sequence of alphabetic characters identifying the game variant.
- `:` is a literal colon character, serving as a separator.
- `<piece-id>` is a piece representation following the PNN specification: `[<prefix>]<letter>[<suffix>]`.

### Casing Rules

The casing of the game identifier reflects the player:
- **Uppercase** game identifiers (e.g., `CHESS:`) denote pieces belonging to the first player.
- **Lowercase** game identifiers (e.g., `chess:`) denote pieces belonging to the second player.

This ensures consistency with the FEEN specification's third field, which uses casing to associate game identities with players.

---

## Examples

### Chess Pieces

| PNN   | GAN (First Player) | GAN (Second Player) |
|-------|--------------------|--------------------|
| `K=`  | `CHESS:K=`         | `chess:k=`         |
| `Q`   | `CHESS:Q`          | `chess:q`          |
| `R`   | `CHESS:R`          | `chess:r`          |
| `B`   | `CHESS:B`          | `chess:b`          |
| `N`   | `CHESS:N`          | `chess:n`          |
| `P`   | `CHESS:P`          | `chess:p`          |

### Shogi Pieces

| PNN   | GAN (First Player) | GAN (Second Player) |
|-------|--------------------|--------------------|
| `K`   | `SHOGI:K`          | `shogi:k`          |
| `G`   | `SHOGI:G`          | `shogi:g`          |
| `S`   | `SHOGI:S`          | `shogi:s`          |
| `+S`  | `SHOGI:+S`         | `shogi:+s`         |
| `N`   | `SHOGI:N`          | `shogi:n`          |
| `+N`  | `SHOGI:+N`         | `shogi:+n`         |
| `L`   | `SHOGI:L`          | `shogi:l`          |
| `+L`  | `SHOGI:+L`         | `shogi:+l`         |
| `R`   | `SHOGI:R`          | `shogi:r`          |
| `+R`  | `SHOGI:+R`         | `shogi:+r`         |
| `B`   | `SHOGI:B`          | `shogi:b`          |
| `+B`  | `SHOGI:+B`         | `shogi:+b`         |
| `P`   | `SHOGI:P`          | `shogi:p`          |
| `+P`  | `SHOGI:+P`         | `shogi:+p`         |

### Disambiguated Collisions

These examples show how GAN resolves ambiguities between pieces that would have identical PNN representation:

| Description | PNN   | GAN                 |
|-------------|-------|---------------------|
| Chess Rook (white) | `R`   | `CHESS:R`           |
| Makruk Rook (white) | `R`   | `MAKRUK:R`          |
| Shogi Rook (sente) | `R`   | `SHOGI:R`           |
| Promoted Shogi Rook (sente) | `+R`  | `SHOGI:+R`          |

---

## Formal Grammar (BNF)

```bnf
<actor> ::= <uppercase-actor> | <lowercase-actor>

<uppercase-actor> ::= <game-id-uppercase> ":" <piece-id-uppercase>
<lowercase-actor> ::= <game-id-lowercase> ":" <piece-id-lowercase>

<game-id-uppercase> ::= <identifier-uppercase>
<game-id-lowercase> ::= <identifier-lowercase>

<identifier-uppercase> ::= <letter-uppercase> | <letter-uppercase> <identifier-uppercase>
<identifier-lowercase> ::= <letter-lowercase> | <letter-lowercase> <identifier-lowercase>

<piece-id-uppercase> ::= <letter-uppercase>
                      | <prefix> <letter-uppercase>
                      | <letter-uppercase> <suffix>
                      | <prefix> <letter-uppercase> <suffix>

<piece-id-lowercase> ::= <letter-lowercase>
                      | <prefix> <letter-lowercase>
                      | <letter-lowercase> <suffix>
                      | <prefix> <letter-lowercase> <suffix>

<prefix> ::= "+" | "-"
<suffix> ::= "=" | "<" | ">"

<letter-lowercase> ::= "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
                    | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
                    | "u" | "v" | "w" | "x" | "y" | "z"

<letter-uppercase> ::= "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
                    | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
                    | "U" | "V" | "W" | "X" | "Y" | "Z"
```

---

## JSON Schema

The following JSON Schema defines the validation pattern for a GAN actor string:

```json
{
  "type": "string",
  "pattern": "^([A-Z]+:[-+]?[A-Z][=<>]?|[a-z]+:[-+]?[a-z][=<>]?)$"
}
```

---

## Relationship with PNN and FEEN

GAN is designed to work seamlessly with existing specifications:

- **PNN (Piece Name Notation)**: GAN extends PNN by prefixing the piece notation with a game identifier. The piece portion of a GAN string (`<piece-id>`) follows all PNN rules.
- **FEEN (Forsyth–Edwards Enhanced Notation)**: GAN uses game identifiers consistent with the third field of FEEN (`<GAMES-TURN>`), which follows the format `<FIRST-GAME>/<SECOND-GAME>`. The casing of game identifiers in GAN reflects the same casing convention used in FEEN to associate pieces with games.

---

## Applications

GAN is particularly useful in the following scenarios:

1. **Cross-game analysis**: When analyzing positions involving pieces from multiple games or variants.
2. **Engine development**: When implementing game engines that need to apply different movement rules based on a piece's original game.
3. **Hybrid games**: When creating or notating positions from hybrid games that combine elements from different chess variants.
4. **Database systems**: When storing game data that needs to disambiguate between similar pieces from different games.

---

## GAN Implementations

This section lists available libraries and tools that implement the GAN specification.

### Ruby

- **[Gan.rb](https://github.com/sashite/gan.rb)** - Implementation of the General Actor Notation specification for Ruby.

---

Copyright © 2012-2025 [Sashité](https://sashite.com/)
