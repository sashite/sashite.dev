# Portable Move Notation (PMN) Specification

**Version**: 1.0.0
**Author**: [Sashité](https://sashite.com/)
**Published**: May 1, 2025
**License**: MIT License

---

## Overview

Portable Move Notation (PMN) is a **rule‑agnostic**, JSON‑based format for describing **state‑changing actions** in abstract‑strategy board games. A PMN *record* lists those actions *sequentially*, allowing any engine to reproduce the exact, deterministic transformation of a game position—independent of the rules that validate or forbid such transformations.

PMN deliberately **does not** embed legality, turn order, or game‑specific concepts such as check, mate, or repetition. It focuses on a *minimal yet exhaustive* vocabulary for moving, capturing, dropping, promoting, or otherwise transforming pieces on an arbitrary‑dimension board.

---

## Core Concepts

### Action Item

An *action item* is the atomic unit of PMN.
Each item is a JSON object with four fields:

```json
{
  "src_square": <string-or-null>,
  "dst_square": <string>,
  "piece_name": <string>,
  "piece_hand": <string-or-null>
}
```

The items in a PMN array **must be applied in the order they appear**. This guarantees that composite moves—such as castling, en‑passant captures, or multi‑piece fairy moves—can be expressed unambiguously.

### Square Labels

* A *square label* is an **arbitrary non‑empty UTF‑8 string** agreed upon by all actors of a game session.
* Labels can be purely numeric (e.g. `"42"`), purely alphabetic (e.g. `"c"`), alphanumeric (e.g. `"c3"`), or use any other convention (e.g. `"@4,7"`).
* PMN **never interprets** the content of the label; it only requires that identical strings refer to the same board location during a session.

### Piece Identifiers (PNN)

PMN reuses **Piece Name Notation (PNN)** for all piece strings. Review §PNN for the grammar; in short:

* A piece *on the board* may have **one prefix** (`+` or `-`) and/or **one suffix** (`'`).
* A piece *in hand* (reserve) **must be the bare letter**—no modifiers.
* Case (**A–Z** vs **a–z**) distinguishes first‑player pieces from second‑player pieces.

---

## Semantics & Consequences

The meaning of each field is intentionally simple yet powerful:

| Field        | Semantics                                                                                                                                                                   |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `src_square` | **Where the piece comes from.** `null` means “from hand / outside the board.”                                                                                               |
| `dst_square` | **Where the piece ends up.** Always required.                                                                                                                               |
| `piece_name` | **What now sits on `dst_square`.** It describes the *post‑action* state of that piece. Promotion, demotion, modifier stripping, colour change, etc. must be reflected here. |
| `piece_hand` | **What (if anything) enters the mover’s reserve** as a result of this action. It must be a *bare* PNN letter or `null`.                                                     |

Given those definitions, every action item produces **deterministic side‑effects**:

1. **Board Removal** – If `src_square` is not `null`, that square becomes *empty*.
2. **Board Placement** – `dst_square` contains `piece_name` *after* the action.
3. **Hand Addition** – If `piece_hand` is not `null`, add *one* such piece to the mover’s reserve.
4. **Hand Removal** – If `src_square` is `null`, remove *one* unmodified copy of `piece_name` (letter only) from the mover’s reserve.

> PMN **does not** verify that the mover actually *had* such a piece in reserve, or that the capture is legal. Engines must enforce game rules on top of PMN.

These consequences are **invariant**: they apply to Chess, Shogi, Xiangqi, Fairy variants, or any hybrid game.

---

## JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Portable Move Notation (PMN) v1.0.0",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "src_square": { "type": ["string", "null"] },
      "dst_square": { "type": "string" },
      "piece_name": { "type": "string", "pattern": "^[-+]?[A-Za-z]['']?$" },
      "piece_hand": { "type": ["string", "null"], "pattern": "^[A-Za-z]$" }
    },
    "required": ["dst_square", "piece_name"],
    "additionalProperties": false
  },
  "minItems": 1
}
```

*The schema intentionally leaves square‑label strings unconstrained so that any coordinate convention may be used.*

---

## Examples

Coordinates below are illustrative only; any label convention may be substituted.

### Shogi · Promotion

A pawn promotes upon reaching the enemy camp.

```json
[{ "src_square": "27", "dst_square": "18", "piece_name": "+P", "piece_hand": null }]
```

* **Before**: square `"27"` contained `P`.
* **After**: square `"27"` is empty; square `"18"` contains `+P`.

### Shogi · Capture → Piece in Hand

A bishop captures a promoted pawn. The pawn enters the bishop‑owner’s hand **demoted** to its letter.

```json
[{ "src_square": "36", "dst_square": "27", "piece_name": "B", "piece_hand": "P" }]
```

* The captured board piece was `+p`; the reserved piece is `P`.

### Shogi · Drop

Dropping a pawn from hand to the board.

```json
[{ "src_square": null, "dst_square": "27", "piece_name": "p", "piece_hand": null }]
```

* Removes one `p` from hand; square `"27"` now contains `p`.

### Chess · Kingside Castling

```json
[
  { "src_square": "e1", "dst_square": "g1", "piece_name": "K", "piece_hand": null },
  { "src_square": "h1", "dst_square": "f1", "piece_name": "R", "piece_hand": null }
]
```

* The rook’s suffix (`R' → R`) is stripped to mark that it has moved and can no longer castle.

### Chess · Castling with Dual‑Rook Suffix Removal

If *both* rooks originally carried a suffix (`R'`), remove the suffix from the idle rook via a **zero‑distance action**:

```json
[
  { "src_square": "e1", "dst_square": "g1", "piece_name": "K", "piece_hand": null },
  { "src_square": "h1", "dst_square": "f1", "piece_name": "R", "piece_hand": null },
  { "src_square": "a1", "dst_square": "a1", "piece_name": "R", "piece_hand": null }
]
```

A zero‑distance move is legal in PMN because legality is outside this specification; it merely expresses *“replace whatever is on `a1` by an un‑suffixed R.”*

### Chess · En Passant

White advances a pawn two squares; Black captures en passant.

```json
// White
[{ "src_square": "e2", "dst_square": "e4", "piece_name": "P'", "piece_hand": null }]

// Black
[
  { "src_square": "d4", "dst_square": "e3", "piece_name": "p", "piece_hand": null },
  { "src_square": "e3", "dst_square": "e4", "piece_name": "p", "piece_hand": null }
]
```

### Hybrid Example

A Chess knight captures a Makruk rook in a cross‑variant game.

```json
[{ "src_square": "c4", "dst_square": "d6", "piece_name": "N", "piece_hand": null }]
```

---

## Related Specifications

* **[Piece Name Notation (PNN)](https://sashite.dev/documents/pnn/1.0.0/)** — piece identifiers.
* **[Forsyth‑Edwards Enhanced Notation (FEEN)](https://sashite.dev/documents/feen/1.0.0/)** — static positions.

## Implementations

### Ruby

* **[Pmn.rb](https://github.com/sashite/pmn.rb)** – Reference implementation.

---

Copyright © 2020–2025 [Sashité](https://sashite.com/)
