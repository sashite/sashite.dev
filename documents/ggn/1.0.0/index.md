# General Gameplay Notation (GGN) Specification

**Version**: 1.0.0
**Author**: [Sashité](https://sashite.com/)
**Published**: 8 March 2014
**License**: MIT License

---

## Overview

General Gameplay Notation (GGN) is a rule‑agnostic, JSON‑based format for describing **pseudo‑legal moves** in abstract strategy board games.
Whereas PMN expresses *what* a move does, GGN expresses *whether* that move is **possible** under basic movement constraints.
GGN is deliberately silent about higher‑level, game‑specific legality questions (e.g. check, ko, repetition, castling paths).
This neutrality makes the format universal: any engine can pre‑compute and share a library of pseudo‑legal moves for any mix of games.

---

## Core Concepts

A single GGN **entry** answers the question :

> Can this piece, currently on this square, reach that square?

It therefore encodes :

1. **Which piece** (via GAN identifier)
2. **From where** (source square label, or `"*"` for pieces not currently on the board)
3. **To where** (destination square label)
4. **Which pre‑conditions** must hold (`board_check`)
5. **Which post‑conditions** result (`board_patch`, plus optional hand changes)

No other information (turn order, player rights, game outcome, etc.) belongs to GGN.

---

## JSON Structure

```json
{
  "<Source piece GAN>": {
    "<Source square>": {
      "<Destination square>": [
        {
          "board_check":  { "<square>": "<required state>", … },
          "board_patch":  { "<square>": "<new state | null>", … },
          "in_hand_add":  "<piece GAN>" | null,
          "in_hand_del":   "<piece GAN>" | null
        }
      ]
    }
  }
}
```

### Fields

| Field                  | Type            | Required | Meaning                                                                      |
| ---------------------- | --------------- | -------- | ---------------------------------------------------------------------------- |
| **Source piece GAN**   | string          | yes      | Identifier of the *current* piece state (GAN)                                |
| **Source square**      | string \| `"*"` | yes      | Origin square, or `"*"` for drops                                            |
| **Destination square** | string          | yes      | Target square                                                                |
| **board\_check**       | object          | yes      | Square → **Occupation State** map that *must* be satisfied *before* the move |
| **board\_patch**       | object          | yes      | Square → new piece state (or `null` for empty) *after* the move              |
| **in\_hand\_add**      | string \| null  | no       | Piece added to mover’s hand                                                  |
| **in\_hand\_del**      | string \| null  | no       | Piece removed from mover’s hand                                              |

If `in_hand_add` or `in_hand_del` is `null`, the field may be omitted.  Both may be omitted entirely when unused.

---

## Occupation States

GGN recognises **five** states, all orthogonal to attack/defence concepts:

| State            | Meaning                                             |
| ---------------- | --------------------------------------------------- |
| `"empty"`        | Square must be empty                                |
| `"occupied"`     | Square must contain any piece                       |
| `"enemy"`        | Square must contain an opposing piece               |
| `"ally"`         | Square must contain a friendly piece                |
| *GAN identifier* | Square must contain **exactly** the specified piece |

---

## Examples (Game‑Neutral)

> Coordinates are illustrative; any labelling convention is acceptable.

### Simple Move

A piece slides from **c3** to **c5** along an empty file.

```json
{
  "GAME:X": {
    "c3": {
      "c5": [
        {
          "board_check": { "c4": "empty", "c5": "empty" },
          "board_patch": { "c3": null, "c5": "GAME:X" }
        }
      ]
    }
  }
}
```

### Capture

The same piece captures an enemy on **d4**.

```json
{
  "GAME:X": {
    "c3": {
      "d4": [
        {
          "board_check": { "d4": "enemy" },
          "board_patch": { "c3": null, "d4": "GAME:X" }
        }
      ]
    }
  }
}
```

### Promotion (Variant‑Agnostic)

A piece on the penultimate rank moves forward and promotes.

```json
{
  "GAME:P": {
    "e7": {
      "e8": [
        {
          "board_check": { "e8": "empty" },
          "board_patch": { "e7": null, "e8": "GAME:Q" }
        }
      ]
    }
  }
}
```

### Piece Drop (Reserve to Board)

Dropping a piece from hand onto **f5**.

```json
{
  "GAME:P": {
    "*": {
      "f5": [
        {
          "board_check": { "f5": "empty" },
          "board_patch": { "f5": "GAME:P" },
          "in_hand_del": "GAME:P"
        }
      ]
    }
  }
}
```

---

## JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "General Gameplay Notation (GGN)",
  "type": "object",
  "patternProperties": {
    "^[A-Za-z]+:[\\+\\-]?[A-Za-z]['']?$": {
      "type": "object",
      "additionalProperties": {
        "type": "object",
        "additionalProperties": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "board_check": {
                "type": "object",
                "additionalProperties": {
                  "type": "string",
                  "pattern": "^(empty|occupied|enemy|ally|[A-Za-z]+:[\\+\\-]?[A-Za-z]['']?)$"
                }
              },
              "board_patch": {
                "type": "object",
                "additionalProperties": {
                  "anyOf": [
                    { "type": "string", "pattern": "^[A-Za-z]+:[\\+\\-]?[A-Za-z]['']?$" },
                    { "type": "null" }
                  ]
                }
              },
              "in_hand_add": {
                "type": ["string", "null"],
                "pattern": "^[A-Za-z]+:[A-Za-z]$"
              },
              "in_hand_del": {
                "type": ["string", "null"],
                "pattern": "^[A-Za-z]+:[A-Za-z]$"
              }
            },
            "required": ["board_check", "board_patch"],
            "additionalProperties": false
          },
          "minItems": 1
        }
      }
    }
  },
  "additionalProperties": false
}
```

---

## Relationship to Other Notations

| Notation | Purpose                                 |
| -------- | --------------------------------------- |
| **GAN**  | Unique, unambiguous piece identifiers   |
| **FEEN** | Complete board state (static positions) |
| **PMN**  | Sequential, deterministic state changes |

GGN sits **between** FEEN and PMN: it lists *potential* transformations without asserting their validity under full game rules.

---

Copyright © 2014–2025 [Sashité](https://sashite.com/)
