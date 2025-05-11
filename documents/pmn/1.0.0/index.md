# Portable Move Notation (PMN) Specification

**Version**: 1.0.0
**Author**: [Sashité](https://sashite.com/)
**Published**: May 11, 2025
**License**: MIT License

---

## Overview

Portable Move Notation (PMN) is a rule-agnostic JSON-based format for representing moves in abstract strategy board games. It provides a consistent representation system for game actions across both traditional and non-traditional board games, supporting arbitrary dimensions and hybrid configurations while maintaining neutrality toward game-specific rules.

---

## Properties

- PMN is **rule-agnostic** and does not encode game-specific legality, validity, or conditions.
- PMN supports **arbitrary-dimensional** board configurations through a unified coordinate system.
- PMN is **game-neutral**, providing a common structure applicable to all abstract strategy games with piece movement.
- PMN facilitates **hybrid** or **cross-game** scenarios where multiple game types coexist.
- PMN is designed for **deterministic representation** of all possible state transitions in piece-placement games.

---

## Constraints

- PMN represents actions only, not the resulting state or legality of those actions.
- PMN does not encode turn order, game phase, or similar dynamic properties.
- PMN is concerned solely with the spatial transformation of game elements, not with the rules governing those transformations.

---

## Format

A PMN record consists of an array of one or more action items, where each action item is a JSON object with precisely defined fields:

```json
[
  {
    "src_square": <source-coordinate-or-null>,
    "dst_square": <destination-coordinate>,
    "piece_name": <piece-identifier>,
    "piece_hand": <captured-piece-identifier-or-null>
  },
  ...
]
```

### Action Item Fields

Each action item must include the following fields:

1. **`src_square`** — An unsigned integer representing the source coordinate, or `null` for placements from outside the board.
2. **`dst_square`** — An unsigned integer representing the destination coordinate (required).
3. **`piece_name`** — A string identifier for the piece being moved (required).
4. **`piece_hand`** — A string identifier for any captured piece that becomes droppable (also referred to as a _piece in hand_), or `null` if none.

Notes about `piece_hand`:

- In some games (such as Shogi), when a piece is captured and becomes droppable, it is common practice to convert the piece's identifier to reflect its new ownership (by changing its casing). PMN does not enforce this convention; game engines are responsible for applying any game-specific ownership transformations.
- In games where captured pieces are eliminated from play (such as Chess), this field must always be `null`.
- Piece's identifier must be recorded without any modifiers (prefixes or suffixes).

---

## Coordinate System

PMN uses a one-dimensional coordinate system to represent board positions:

- All board positions are indexed using unsigned integers, starting from 0.
- The coordinate system follows a flattened representation of the board's geometry.
- For rectilinear boards, coordinates increase from left to right, then top to bottom (from the first player's perspective).
- For higher-dimensional boards, the flattening order is standardized as follows: innermost dimensions vary most rapidly.

### Common Board Dimensions

Common board geometries map to the following coordinate ranges:

| Game Type | Board Dimensions | Coordinate Range |
|-----------|------------------|------------------|
| Chess     | 8×8              | 0–63             |
| Shogi     | 9×9              | 0–80             |
| Xiangqi   | 9×10             | 0–89             |
| Go        | 19×19            | 0–360            |
| Draughts  | 10×10            | 0–99             |

### Arbitrary Dimensions

For boards with dimensions beyond the standard rectilinear forms, the following mapping applies:

- For a board with dimensions d₁ × d₂ × ... × dₙ, the coordinate of position (i₁, i₂, ..., iₙ) is:
  i₁ + i₂×d₁ + i₃×d₁×d₂ + ... + iₙ×d₁×d₂×...×dₙ₋₁

---

## Piece Representation

### Formal Definition of Piece Identifiers

The piece identifiers in PMN follow specific formal rules:

#### BNF Grammar for `piece_name`

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

#### BNF Grammar for `piece_hand`

```bnf
<piece-hand> ::= <letter>

<letter> ::= <letter-lowercase> | <letter-uppercase>

<letter-lowercase> ::= "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
                     | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
                     | "u" | "v" | "w" | "x" | "y" | "z"

<letter-uppercase> ::= "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
                     | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
                     | "U" | "V" | "W" | "X" | "Y" | "Z"
```

### Piece Naming Semantics

- Piece identifiers are case-sensitive strings.
- Case typically distinguishes between players (e.g., uppercase for one player, lowercase for the other).
- Modifiers may be applied to `piece_name` identifiers to denote special states:
  - Prefix modifiers: `-` or `+` (e.g., `+P` for a promoted piece)
  - Suffix modifiers: `=`, `<`, or `>` (e.g., `K=` for a king with castling rights)
- Modifiers are NOT permitted in `piece_hand` values.

### Null Values

- `null` is a valid value for the `src_square` field, indicating a piece placement from outside the board.
- `null` is a valid value for the `piece_hand` field, indicating no piece was captured or retained.

---

## JSON Schema Definition

The following JSON Schema formally defines the structure of a PMN record:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Portable Move Notation (PMN)",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "src_square": {
        "type": ["integer", "null"],
        "minimum": 0
      },
      "dst_square": {
        "type": "integer",
        "minimum": 0
      },
      "piece_name": {
        "type": "string",
        "pattern": "^[-+]?[a-zA-Z][=<>]?$"
      },
      "piece_hand": {
        "type": ["string", "null"],
        "pattern": "^[a-zA-Z]$"
      }
    },
    "required": ["dst_square", "piece_name"],
    "additionalProperties": false
  },
  "minItems": 1
}
```

---

## Examples

The following examples illustrate how PMN represents common move types.
Coordinates refer to the flattened board representation.

### Chess: Initial Pawn Move

A white pawn moves from e2 (coordinate 52) to e4 (coordinate 36).

```json
[{ "src_square": 52, "dst_square": 36, "piece_name": "P", "piece_hand": null }]
```

### Shogi: Dropping a Pawn

A pawn is dropped onto square 27 from the player's hand (a Shogi drop move).

```json
[{ "src_square": null, "dst_square": 27, "piece_name": "p", "piece_hand": null }]
```

### Chess: Castling Kingside

White castles kingside:

* The king moves from e1 (60) to g1 (62),
* The rook moves from h1 (63) to f1 (61).

Both moves are recorded as a composite action.

```json
[
  { "src_square": 60, "dst_square": 62, "piece_name": "K", "piece_hand": null },
  { "src_square": 63, "dst_square": 61, "piece_name": "R", "piece_hand": null }
]
```

### Shogi: Promotion

A pawn moves from square 27 to 18 and promotes upon arrival.
The promoted piece is represented as `+P`.

```json
[{ "src_square": 27, "dst_square": 18, "piece_name": "+P", "piece_hand": null }]
```

### Shogi: Piece Capture

A bishop (B) captures a promoted pawn (+p) at square 27. The captured pawn becomes a droppable piece for the bishop's owner.

```json
[{ "src_square": 36, "dst_square": 27, "piece_name": "B", "piece_hand": "P" }]
```

Note that even if the captured piece had a modifier on the board (such as `+p`), the `piece_hand` value must be recorded without any modifier (just `P`). The change in case from lowercase to uppercase indicates the piece's new ownership.

### Chess: En Passant Capture

After White plays a double pawn move from e2 (52) to e4 (36),

```json
[{ "src_square": 52, "dst_square": 36, "piece_name": "P", "piece_hand": null }]
```

Black captures the pawn _en passant_ from d4 (35) to e3 (44):

```json
[
  { "src_square": 35, "dst_square": 36, "piece_name": "p", "piece_hand": null },
  { "src_square": 36, "dst_square": 44, "piece_name": "p", "piece_hand": null }
]
```

### Cross-Game Capture

In a hybrid game (e.g., Chess-Makruk crossover),
a chess knight (N) captures a Makruk rook (r) at destination square 44.

```json
[{ "src_square": 27, "dst_square": 44, "piece_name": "N", "piece_hand": null }]
```

---

## Compatible Formats

PMN can be used alongside other notation formats such as Forsyth-Edwards Enhanced Notation (FEEN) for representing board positions. While PMN specializes in move representation, these complementary formats focus on position representation, allowing for a complete description of a game's state and transitions.

---

## PMN Implementations

This section lists available libraries and tools that implement the PMN specification, allowing developers to easily integrate this format into their applications.

### Ruby

- **[PMN.rb](https://github.com/sashite/pmn.rb)** - Full implementation of the PMN specification for Ruby, including conversion from/to various formats, as well as support for arbitrary dimensions.

---

Copyright © 2020–2025 [Sashité](https://sashite.com/)
