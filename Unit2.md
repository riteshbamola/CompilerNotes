# Unit 2: SYNTAX ANALYSIS

## Overview

```
Lexical Analysis ----tokens----> Syntax Analysis -----> Parse Tree
```

## Roles of Syntax Analysis

1. **Take list of tokens** from lexical analysis
2. **Analyze all syntax** of the program according to grammar rules
3. **Create parse tree** using defined grammar
4. **Report syntax errors** if any violations are found

---

## Types of Parsers

### 1. Top-Down Parser

Constructs parse tree from root to leaves (start symbol to terminals)

#### a) Recursive Descent Parser

- Uses recursive procedures for each non-terminal
- Easy to implement and understand
- May have issues with left recursion
- **Characteristics:**
  - Predictive parsing
  - No backtracking in predictive version
  - Each non-terminal has a parsing procedure

#### b) LL(1) Parser

- **L**: Left-to-right scanning
- **L**: Leftmost derivation
- **(1)**: One lookahead symbol
- Uses parsing table for decision making
- Cannot handle left recursion or ambiguous grammars

### 2. Bottom-Up Parser

Constructs parse tree from leaves to root (terminals to start symbol)

#### a) LR(K) Parsers

**L**: Left-to-right scanning, **R**: Rightmost derivation, **K**: lookahead symbols

##### i) LR(0)

- **LR(0)**: Basic LR parser with no lookahead
  - Uses only current state for decisions
  - Limited in handling conflicts
- **SLR(1)**: Simple LR with 1 lookahead
  - Uses FOLLOW sets to resolve conflicts
  - More powerful than LR(0)

##### ii) LR(1)

- **LALR(1)**: Look-Ahead LR

  - Merges LR(1) states with same core
  - Compromise between LR(1) and SLR(1)
  - Used by many parser generators (like Yacc)

- **CLR(1)**: Canonical LR
  - Most powerful LR parser
  - Handles largest class of grammars
  - Can be large in size

#### b) Operator Precedence Parser

- Specialized for expressions with operators
- Uses precedence and associativity rules
- Simpler than LR parsers for arithmetic expressions

---

## Grammar Fundamentals

### Grammar Definition: G = (V, T, P, S)

- **V**: Set of Variables/Non-terminals (e.g., {S, A, B})
- **T**: Set of Terminals (e.g., {a, b, c, +, -, \*})
- **P**: Set of Production Rules (e.g., S → AB, A → a)
- **S**: Start Symbol (distinguished non-terminal)

### Context-Free Grammar (CFG)

A grammar where:

- Left side of production has exactly one non-terminal
- Right side can have any combination of terminals and non-terminals
- Form: A → α (where A ∈ V, α ∈ (V ∪ T)\*)

---

## Left Recursion and Right Recursion

### Left Recursion

A grammar is **left recursive** if there exists a non-terminal A such that:
**A ⇒⁺ Aα** (A derives itself as the leftmost symbol)

#### Types of Left Recursion:

**1. Immediate Left Recursion:**

```
A → Aα | β
```

Where α, β are strings and β doesn't start with A.

**2. Indirect Left Recursion:**

```
A → Bα
B → Aβ | γ
```

A derives B, and B derives A.

#### Problem with Left Recursion:

- **Top-down parsers** (like LL parsers) go into **infinite loops**
- Recursive descent parsers cannot handle left recursion
- Must be eliminated before applying LL parsing

### Right Recursion

A grammar is **right recursive** if:
**A ⇒⁺ αA** (A derives itself as the rightmost symbol)

**Example:**

```
A → αA | β
```

#### Characteristics:

- **No problem** for top-down parsers
- Can cause issues with **bottom-up parsers** in some cases
- Generally preferred for LL parsers

---

## Conversion from Left to Right Recursion

### Method 1: Direct Conversion for Immediate Left Recursion

**Original Left Recursive Grammar:**

```
A → Aα₁ | Aα₂ | ... | Aαₘ | β₁ | β₂ | ... | βₙ
```

**Converted Right Recursive Grammar:**

```
A → β₁A' | β₂A' | ... | βₙA'
A' → α₁A' | α₂A' | ... | αₘA' | ε
```

#### Example:

**Left Recursive:**

```
E → E + T | E - T | T
```

**Right Recursive:**

```
E → TE'
E' → +TE' | -TE' | ε
```

---

## Non-Deterministic Grammar

### Definition

A grammar is **non-deterministic** (or ambiguous) if:

- There exists a string that has **more than one parse tree**
- Multiple **leftmost** or **rightmost** derivations exist for the same string

### Types of Non-Determinism:

#### 1. Ambiguous Productions

**Example:**

```
S → if E then S else S | if E then S | other
```

**Problem:** Dangling else problem

#### 2. Operator Precedence Ambiguity

**Example:**

```
E → E + E | E * E | id
```

**String:** `id + id * id`
**Two parse trees possible** due to unclear precedence

#### 3. Left Factoring Issues

**Example:**

```
A → αβ₁ | αβ₂
```

**Problem:** Common prefix α causes confusion

### Resolving Non-Determinism:

#### Method 1: Left Factoring

**Original:**

```
A → αβ₁ | αβ₂ | γ
```

**After Left Factoring:**

```
A → αA' | γ
A' → β₁ | β₂
```

## Comprehensive Examples

### Example 1: Left Recursion Elimination

**Original Grammar:**

```
E → E + T | T
T → T * F | F
F → (E) | id
```

**After Elimination:**

```
E → TE'
E' → +TE' | ε
T → FT'
T' → *FT' | ε
F → (E) | id
```

### Example 2: Left Factoring

**Original:**

```
S → iEtS | iEtSeS | a
```

**After Left Factoring:**

```
S → iEtSS' | a
S' → eS | ε
```

### Example 3: Removing Ambiguity

**Ambiguous Grammar:**

```
E → E + E | E - E | E * E | E / E | (E) | id
```

**Unambiguous Grammar:**

```
E → E + T | E - T | T
T → T * F | T / F | F
F → (E) | id
```

---

---

## FIRST and FOLLOW Sets - Complete Guide

### FIRST Set

**FIRST(α)** = set of terminals that can appear at the beginning of strings derived from α

#### Detailed Rules for Computing FIRST:

**Rule 1: Terminal**

```
If X is a terminal: FIRST(X) = {X}
```

**Rule 2: Epsilon Production**

```
If X → ε is a production: add ε to FIRST(X)
```

**Rule 3: Non-terminal Production**

```
For production X → Y₁Y₂...Yₖ:
1. Add FIRST(Y₁) - {ε} to FIRST(X)
2. If ε ∈ FIRST(Y₁), add FIRST(Y₂) - {ε} to FIRST(X)
3. If ε ∈ FIRST(Y₁) ∩ FIRST(Y₂), add FIRST(Y₃) - {ε} to FIRST(X)
4. Continue until Yᵢ where ε ∉ FIRST(Yᵢ) OR all symbols processed
5. If ε ∈ FIRST(Yᵢ) for all i = 1 to k, add ε to FIRST(X)
```

**Rule 4: String of Symbols**

```
For string α = X₁X₂...Xₙ:
FIRST(α) includes FIRST(X₁) - {ε}
If ε ∈ FIRST(X₁), include FIRST(X₂) - {ε}
Continue as above...
```

### FOLLOW Set

**FOLLOW(A)** = set of terminals that can appear immediately after non-terminal A in some sentential form

#### Detailed Rules for Computing FOLLOW:

**Rule 1: Start Symbol**

```
Add $ to FOLLOW(S) where S is start symbol
```

**Rule 2: Production A → αBβ**

```
Add FIRST(β) - {ε} to FOLLOW(B)
If ε ∈ FIRST(β), add FOLLOW(A) to FOLLOW(B)
```

**Rule 3: Production A → αB (B at end)**

```
Add FOLLOW(A) to FOLLOW(B)
```

**Rule 4: Repeat until no changes**

```
Apply rules repeatedly until FOLLOW sets don't change
```

---

## Comprehensive FIRST/FOLLOW Example

### Grammar:

```
S → ACB | CbB | Ba
A → da | BC
B → g | ε
C → h | ε
```

### Step-by-Step FIRST Computation:

**Step 1: Terminals**

- FIRST(d) = {d}, FIRST(g) = {g}, FIRST(h) = {h}
- FIRST(a) = {a}, FIRST(b) = {b}

**Step 2: Non-terminals with ε**

- B → ε: FIRST(B) = {ε}
- C → ε: FIRST(C) = {ε}
- B → g: FIRST(B) = {g, ε}
- C → h: FIRST(C) = {h, ε}

**Step 3: Other Productions**

- A → da: FIRST(A) includes {d}
- A → BC: FIRST(A) includes FIRST(B) - {ε} = {g}
  Since ε ∈ FIRST(B), add FIRST(C) - {ε} = {h}
  Since ε ∈ FIRST(C), add ε
  So FIRST(A) = {d, g, h, ε}

- S → ACB: FIRST(S) includes FIRST(A) - {ε} = {d, g, h}
  Since ε ∈ FIRST(A), add FIRST(C) - {ε} = {h} (already there)
  Since ε ∈ FIRST(C), add FIRST(B) - {ε} = {g} (already there)

- S → CbB: FIRST(S) includes FIRST(C) - {ε} = {h}
  Since ε ∈ FIRST(C), add FIRST(b) = {b}

- S → Ba: FIRST(S) includes FIRST(B) - {ε} = {g}
  Since ε ∈ FIRST(B), add FIRST(a) = {a}

**Final FIRST Sets:**

- FIRST(S) = {d, g, h, b, a}
- FIRST(A) = {d, g, h, ε}
- FIRST(B) = {g, ε}
- FIRST(C) = {h, ε}

### Step-by-Step FOLLOW Computation:

**Step 1: Start Symbol**

- FOLLOW(S) = {$}

**Step 2: Apply Rules**

From S → ACB:

- FOLLOW(A) includes FIRST(CB) = FIRST(C) - {ε} ∪ FIRST(B) = {h, g}
- FOLLOW(C) includes FIRST(B) - {ε} = {g}
  Since ε ∈ FIRST(B), add FOLLOW(S) = {$}
- FOLLOW(B) includes FOLLOW(S) = {$}

From S → CbB:

- FOLLOW(C) includes FIRST(bB) = {b}
- FOLLOW(B) includes FOLLOW(S) = {$} (already there)

From S → Ba:

- FOLLOW(B) includes FIRST(a) = {a}

From A → da:

- No additional FOLLOW entries

From A → BC:

- FOLLOW(B) includes FIRST(C) - {ε} = {h}
  Since ε ∈ FIRST(C), add FOLLOW(A) = {g, h}
- FOLLOW(C) includes FOLLOW(A) = {g, h, $}

**Final FOLLOW Sets:**

- FOLLOW(S) = {$}
- FOLLOW(A) = {g, h}
- FOLLOW(B) = {$, a, h, g}
- FOLLOW(C) = {g, $, b, h}

---

## Advanced FIRST/FOLLOW Example

### Grammar:

```
E → TE'
E' → +TE' | ε
T → FT'
T' → *FT' | ε
F → (E) | id
```

### FIRST Sets:

- FIRST(E) = FIRST(T) = FIRST(F) = {(, id}
- FIRST(E') = {+, ε}
- FIRST(T') = {\*, ε}

### FOLLOW Sets:

- FOLLOW(E) = {$, )}
- FOLLOW(E') = FOLLOW(E) = {$, )}
- FOLLOW(T) = FIRST(E') - {ε} ∪ FOLLOW(E') = {+, $, )}
- FOLLOW(T') = FOLLOW(T) = {+, $, )}
- FOLLOW(F) = FIRST(T') - {ε} ∪ FOLLOW(T') = {\*, +, $, )}

---

## Example: Computing FIRST and FOLLOW

### Simple Grammar:

```
S → AB
A → a | ε
B → b | ε
```

### FIRST Sets:

- FIRST(A) = {a, ε}
- FIRST(B) = {b, ε}
- FIRST(S) = FIRST(A) - {ε} ∪ (since ε ∈ FIRST(A)) FIRST(B) = {a, b, ε}

### FOLLOW Sets:

- FOLLOW(S) = {$}
- FOLLOW(A) = FIRST(B) - {ε} ∪ (since ε ∈ FIRST(B)) FOLLOW(S) = {b, $}
- FOLLOW(B) = FOLLOW(S) = {$}

---

## Parser Comparison

| Parser Type | Grammar Class | Lookahead | Complexity | Usage               |
| ----------- | ------------- | --------- | ---------- | ------------------- |
| LL(1)       | LL(1)         | 1         | O(n)       | Recursive descent   |
| SLR(1)      | SLR(1)        | 1         | O(n)       | Simple applications |
| LALR(1)     | LALR(1)       | 1         | O(n)       | Yacc, Bison         |
| CLR(1)      | LR(1)         | 1         | O(n)       | Theoretical         |

---

## Error Handling in Parsing

### Types of Errors:

1. **Lexical errors**: Invalid characters
2. **Syntax errors**: Grammar violations
3. **Semantic errors**: Type mismatches
4. **Logical errors**: Incorrect program logic

### Error Recovery Strategies:

1. **Panic mode**: Skip tokens until synchronizing token
2. **Phrase level**: Local correction
3. **Error productions**: Add error rules to grammar
4. **Global correction**: Minimum cost correction

---

## Key Concepts Summary

- **Parse Tree**: Concrete representation of derivation
- **Abstract Syntax Tree (AST)**: Simplified parse tree
- **Ambiguous Grammar**: Grammar with multiple parse trees for same string
- **Left Recursion**: A → Aα (problematic for top-down parsers)
- **Left Factoring**: Removing common prefixes from productions

This completes the comprehensive coverage of Syntax Analysis fundamentals, parser types, and FIRST/FOLLOW set computation.
