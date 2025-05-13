# Forsyth–Edwards Enhanced Notation (FEEN) Specification

**Version**: 1.0.0
**Author**: [Sashité](https://sashite.com/)
**Published**: May 1, 2025
**License**: MIT License

---

## Overview

FEEN is a compact, canonical, and rule-agnostic textual format for representing static board positions in two-player piece-placement games. It is designed to support multiple game types and hybrid configurations, while remaining neutral with regard to the rules of any specific game.

---

## Properties

- FEEN is **rule-agnostic** and does not encode legality, move counters, or game-specific conditions.
- FEEN enforces a **canonical representation**, ensuring that equivalent positions yield identical strings.
- FEEN supports **hybrid** or **cross-game** positions involving different piece sets.
- FEEN supports **arbitrary-dimensional** board configurations.
- FEEN is suitable for **static storage**, **comparison**, and **visual reconstruction** of positions.

---

## Constraints

- FEEN supports exactly **two players**.
- A maximum of 26 unique pieces per player is allowed, as identifiers must be mapped to `a-z` or `A-Z`.
- Players must be assigned **distinct casing** (uppercase/lowercase) for their associated game identifiers to ensure proper disambiguation.

---

## Format

A FEEN record consists of the following three space-separated fields:

1. **`<PIECE-PLACEMENT>`** — Specifies the configuration of pieces on the board, possibly in multiple dimensions.
2. **`<PIECES-IN-HAND>`** — Lists any off-board pieces available to each player, sorted lexicographically.
3. **`<GAMES-TURN>`** — Indicates which player is to move, and associates piece casing with game identities.

These fields are combined into a single FEEN string like:

```txt
<PIECE-PLACEMENT> <PIECES-IN-HAND> <GAMES-TURN>
```

### Examples

#### Initial Positions from Traditional Games

The following are examples of valid FEEN strings representing the initial positions of various traditional games. These are for illustration only — **FEEN does not define or assume any game-specific rules**.

* **Makruk** *(Thai chess)*:

  ```txt
  rnbqkbnr/8/pppppppp/8/8/PPPPPPPP/8/RNBKQBNR - MAKRUK/makruk
  ```

* **Shogi** *(Japanese chess)*:

  ```txt
  lnsgkgsnl/1r5b1/ppppppppp/9/9/9/PPPPPPPPP/1B5R1/LNSGKGSNL - SHOGI/shogi
  ```

* **Xiangqi** *(Chinese chess)*:

  ```txt
  rheagaehr/9/1c5c1/s1s1s1s1s/9/9/S1S1S1S1S/1C5C1/9/RHEAGAEHR - XIANGQI/xiangqi
  ```

* **Chess** *(Western chess, starting position)*:

  ```txt
  r'nbqkbnr'/pppppppp/8/8/8/8/PPPPPPPP/R'NBQKBNR' - CHESS/chess
  ```

* **Chess** *(after 1.e4)*:

  ```txt
  r'nbqkbnr'/pppppppp/8/8/4P3/8/PPPP1PPP/R'NBQKBNR' - chess/CHESS
  ```

---

## Piece Representation

FEEN relies on the Piece Name Notation (PNN) specification for representing individual pieces. This provides a consistent way to identify pieces and their states across various board games.

Key points from PNN relevant to FEEN:

- Pieces are denoted using single ASCII characters `a-z` or `A-Z`.
- Optional modifiers may be applied to pieces **on the board**:
  - A prefix: `-` or `+`
  - A suffix: `'`
- Pieces in hand (Field 2) must use only the base letter without any modifiers.
- Case distinguishes between players (uppercase for first player, lowercase for second).

Refer to the full [PNN specification](https://sashite.dev/documents/pnn/1.0.0/) for detailed information about piece representation.

---

## Field 1: `<PIECE-PLACEMENT>`

This field encodes the spatial distribution of pieces across the board. The position is always expressed **from the point of view of the player who plays first in the initial position**, according to the standard rules of the game(s) represented.

- The board is traversed starting from the outermost dimension and proceeding inward, dimension by dimension.
- **Pieces** are denoted using PNN notation.
- **Empty spaces** are compressed using digits `1`-`n`, representing consecutive empty cells.
- **Dimension separators**:
  - `/` separates consecutive elements of the first inner dimension.
  - Each higher-level dimension adds an additional `/`, resulting in `//`, `///`, etc.
- **Pieces are not separated** by delimiters; their identity and grouping are inferred from their individual characters.

This generalization allows FEEN to describe *n*-dimensional boards with arbitrary shapes.

---

## Field 2: `<PIECES-IN-HAND>`

This field lists the pieces available for future placement on the board:

- Droppable pieces (also referred to as _pieces in hand_) are represented using their symbolic identifiers, as defined in PNN, including case but **without modifiers** (prefixes or suffixes).
- If multiple occurrences of the same piece are available, they must be grouped and preceded by a number indicating their count.
  - For a single occurrence, no numeric prefix is used (e.g., "P").
  - For two or more occurrences, a numeric prefix must be used (e.g., "2P", "10P").
  - The numeric prefix cannot start with "0".
  - The numeric prefix cannot be exactly "1" (simply use the letter without a prefix).
- The list must be sorted in ASCII lexicographic order.
- No delimiters are used between entries.
- If no pieces are droppable by either player, this field is set to a single hyphen (`-`).
- The case of pieces in this field indicates their current ownership:
  - Uppercase pieces belong to the player identified by the uppercase game name in `<GAMES-TURN>`.
  - Lowercase pieces belong to the player identified by the lowercase game name in `<GAMES-TURN>`.
- The case may differ from the piece's original representation in prior positions, reflecting potential changes in ownership.

### Examples of Pieces in Hand Encoding

| Format | Description |
|--------|------------|
| `P` | One "P" piece in hand |
| `AB` | One "A" piece and one "B" piece |
| `P3QRS` | One "P", three "Q", one "R", one "S" |
| `2AB2C2p2q` | Two "A", one "B", two "C", two "p", two "q" |
| `10P` | Ten "P" pieces |
| `-` | No pieces in hand |

### Practical Implications

This format provides a significant reduction in FEEN string size, particularly in games like Shogi where many identical pieces can be captured. For example, in Shogi, a late-game position might have up to 18 pawns ("P" or "p") in hand, which would require only 3 characters ("18P" or "18p") in this format.

The numeric compression of identical pieces is mandatory when there are two or more occurrences of the same piece, in order to maintain the canonicity of FEEN representations.

---

## Field 3: `<GAMES-TURN>`

This field identifies the game type associated with each player and specifies whose turn it is.

- The format is `<FIRST-GAME>/<SECOND-GAME>`, where:
  - The **first name** refers to the player **to move**.
  - The **second name** refers to the opponent.
- Each name consists of alphabetic characters and must be valid identifiers.
- **Casing is semantically significant**: one of the names must be in uppercase, and the other in lowercase.
  - The uppercase name identifies the game to which **uppercase-labeled pieces** belong.
  - The lowercase name identifies the game to which **lowercase-labeled pieces** belong.

This dual function enables unambiguous attribution of pieces to games and identification of the player to move, even in hybrid configurations.

---

## Formal Grammar (BNF)

The following Backus-Naur Form (BNF) grammar defines the FEEN format. It supports arbitrarily nested board dimensions through a recursive structure of dimension groups, and describes all three main fields: `<PIECE-PLACEMENT>`, `<PIECES-IN-HAND>`, and `<GAMES-TURN>`.

```bnf
<feen> ::= <piece-placement> <whitespace> <pieces-in-hand> <whitespace> <games-turn>
<whitespace> ::= " "
```

---

### Piece Placement (Recursive n-Dimensional Support)

```bnf
<piece-placement> ::= <dim-group>

<dim-group> ::= <dim-element>
             | <dim-element> <dim-separator> <dim-group>

<dim-element> ::= <dim-group>        ; nested dimension (recursive case)
               | <rank>              ; base case: 1D sequence of cells

<rank> ::= <cell>
        | <cell> <rank>

<cell> ::= <piece> | <number>

<piece> ::= <pnn-piece>              ; As defined in the PNN specification

<number> ::= <non-zero-digit>
          | <non-zero-digit> <digits>

<non-zero-digit> ::= "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
<digit> ::= "0" | <non-zero-digit>
<digits> ::= <digit>
          | <digit> <digits>

<dim-separator> ::= <slash> <separator-tail>
<separator-tail> ::= ""                  ; base level: one slash
                  | <slash> <separator-tail> ; additional nesting: //, ///, etc.

<slash> ::= "/"
```

---

### Pieces in Hand

```bnf
<pieces-in-hand> ::= "-"
                  | <pieces-sequence>

<pieces-sequence> ::= <uppercase-sequence> <lowercase-sequence>

<uppercase-sequence> ::= <A-part> <B-part> <C-part> <D-part> <E-part> <F-part> <G-part> <H-part> <I-part> <J-part> <K-part> <L-part> <M-part>
                        <N-part> <O-part> <P-part> <Q-part> <R-part> <S-part> <T-part> <U-part> <V-part> <W-part> <X-part> <Y-part> <Z-part>

<lowercase-sequence> ::= <a-part> <b-part> <c-part> <d-part> <e-part> <f-part> <g-part> <h-part> <i-part> <j-part> <k-part> <l-part> <m-part>
                        <n-part> <o-part> <p-part> <q-part> <r-part> <s-part> <t-part> <u-part> <v-part> <w-part> <x-part> <y-part> <z-part>

<A-part> ::= "" | "A" | <count> "A"
<B-part> ::= "" | "B" | <count> "B"
<C-part> ::= "" | "C" | <count> "C"
<D-part> ::= "" | "D" | <count> "D"
<E-part> ::= "" | "E" | <count> "E"
<F-part> ::= "" | "F" | <count> "F"
<G-part> ::= "" | "G" | <count> "G"
<H-part> ::= "" | "H" | <count> "H"
<I-part> ::= "" | "I" | <count> "I"
<J-part> ::= "" | "J" | <count> "J"
<K-part> ::= "" | "K" | <count> "K"
<L-part> ::= "" | "L" | <count> "L"
<M-part> ::= "" | "M" | <count> "M"
<N-part> ::= "" | "N" | <count> "N"
<O-part> ::= "" | "O" | <count> "O"
<P-part> ::= "" | "P" | <count> "P"
<Q-part> ::= "" | "Q" | <count> "Q"
<R-part> ::= "" | "R" | <count> "R"
<S-part> ::= "" | "S" | <count> "S"
<T-part> ::= "" | "T" | <count> "T"
<U-part> ::= "" | "U" | <count> "U"
<V-part> ::= "" | "V" | <count> "V"
<W-part> ::= "" | "W" | <count> "W"
<X-part> ::= "" | "X" | <count> "X"
<Y-part> ::= "" | "Y" | <count> "Y"
<Z-part> ::= "" | "Z" | <count> "Z"

<a-part> ::= "" | "a" | <count> "a"
<b-part> ::= "" | "b" | <count> "b"
<c-part> ::= "" | "c" | <count> "c"
<d-part> ::= "" | "d" | <count> "d"
<e-part> ::= "" | "e" | <count> "e"
<f-part> ::= "" | "f" | <count> "f"
<g-part> ::= "" | "g" | <count> "g"
<h-part> ::= "" | "h" | <count> "h"
<i-part> ::= "" | "i" | <count> "i"
<j-part> ::= "" | "j" | <count> "j"
<k-part> ::= "" | "k" | <count> "k"
<l-part> ::= "" | "l" | <count> "l"
<m-part> ::= "" | "m" | <count> "m"
<n-part> ::= "" | "n" | <count> "n"
<o-part> ::= "" | "o" | <count> "o"
<p-part> ::= "" | "p" | <count> "p"
<q-part> ::= "" | "q" | <count> "q"
<r-part> ::= "" | "r" | <count> "r"
<s-part> ::= "" | "s" | <count> "s"
<t-part> ::= "" | "t" | <count> "t"
<u-part> ::= "" | "u" | <count> "u"
<v-part> ::= "" | "v" | <count> "v"
<w-part> ::= "" | "w" | <count> "w"
<x-part> ::= "" | "x" | <count> "x"
<y-part> ::= "" | "y" | <count> "y"
<z-part> ::= "" | "z" | <count> "z"

<count> ::= <digit-except-0-1>
         | <digit-except-0> <digits>

<digit-except-0-1> ::= "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
<digit-except-0> ::= "1" | <digit-except-0-1>
<digit> ::= "0" | <digit-except-0>
<digits> ::= "" | <digit> <digits>
```

---

### Game Turn

```bnf
<games-turn> ::= <game-id-uppercase> <slash> <game-id-lowercase>
              | <game-id-lowercase> <slash> <game-id-uppercase>

<slash> ::= "/"

<game-id-uppercase> ::= <identifier-uppercase>
<game-id-lowercase> ::= <identifier-lowercase>

<identifier-uppercase> ::= <letter-uppercase> | <letter-uppercase> <identifier-uppercase>
<identifier-lowercase> ::= <letter-lowercase> | <letter-lowercase> <identifier-lowercase>

<letter-lowercase> ::= "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j"
                     | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r" | "s" | "t"
                     | "u" | "v" | "w" | "x" | "y" | "z"

<letter-uppercase> ::= "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J"
                     | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R" | "S" | "T"
                     | "U" | "V" | "W" | "X" | "Y" | "Z"
```

---

## Related Specifications

FEEN is part of a family of specifications designed to provide a comprehensive and rule-agnostic representation system for abstract strategy board games:

- [Piece Name Notation (PNN)](https://sashite.dev/documents/pnn/1.0.0/) - Defines the format for representing individual pieces.
- [Portable Move Notation (PMN)](https://sashite.dev/documents/pmn/1.0.0/) - Defines the format for representing moves and actions.

Together, these specifications form a complete system for representing both static positions (FEEN) and dynamic transitions (PMN) in arbitrary board games.

---

## FEEN Implementations

This section lists available libraries and tools that implement the FEEN specification, allowing developers to easily integrate this format into their applications.

### Ruby

- **[Feen.rb](https://github.com/sashite/feen.rb)** - Full implementation of the FEEN specification for Ruby, including conversion from/to various formats, as well as support for hybrid games and arbitrary dimensions.

---

Copyright © 2011–2025 [Sashité](https://sashite.com/)
