# Lexical Analysis: DFA and Regular Expressions

## Table of Contents

* [1. Overview](#1-overview)
* [2. Quick Examples](#2-quick-examples)
* [3. Formal Models](#3-formal-models)
  * [3.1 Regular Expression](#31-regular-expression)
  * [3.2 Deterministic Finite Automaton (DFA)](#32-deterministic-finite-automaton-dfa)
  * [3.3 From NFA to DFA](#33-from-nfa-to-dfa)
* [4. Implementation](#4-implementation)
  * [4.1 DFA Implementation in Prolog](#41-dfa-implementation-in-prolog)
  * [4.2 Regular Expression Implementation in Python](#42-regular-expression-implementation-in-python)
* [5. Tests](#5-tests)
  * [5.1 DFA Tests (Prolog)](#51-dfa-tests-prolog)
  * [5.2 Regex Tests (Python)](#52-regex-tests-python)
* [6. Performance Analysis](#6-performance-analysis)
  * [6.1 Time and Space Complexity](#61-time-and-space-complexity)
  * [6.2 DFA vs Regular Expression](#62-dfa-vs-regular-expression)
  * [6.3 Discussion](#63-discussion)
* [7. Limitations](#7-limitations)
  * [7.1 Lack of Semantic Validation](#71-lack-of-semantic-validation)
  * [7.2 Restricted Input Format](#72-restricted-input-format)
  * [7.3 Limited Pattern Scope](#73-limited-pattern-scope)
  * [7.4 No Context Awareness](#74-no-context-awareness)
* [8. Conclusions](#8-conclusions)
* [9. Future Work](#9-future-work)
* [10. References](#10-references)

---

## 1. Overview

The language selected for this project consists of all strings over the lowercase English alphabet that contain the substrings **"bene"** or **"scend"**.

These substrings correspond to lexical roots:

* **bene** → “good” (e.g., *beneficial, benefactor*)
* **scend** → “to climb” (e.g., *descend, transcend*)

This project models a simplified **lexical analysis task**, where specific patterns must be detected within words. Such problems are common in compilers and text processing systems, where tokens or roots are identified within input strings.

### Formal Definition of the Language

Σ* (bene | scend) Σ*

Where:

* Σ = {a, b, c, ..., z}
* Σ* = any sequence of lowercase letters

The goal is strictly **pattern detection**, not semantic validation. That means a string is accepted if it contains one of the target substrings, regardless of whether it forms a real or meaningful word.

---

## 2. Quick Examples

The following examples illustrate the behavior of the system:

| Input      | Result   | Reason                   |
| ---------- | -------- | ------------------------ |
| beneficial | ACCEPTED | Contains "bene"          |
| transcend  | ACCEPTED | Contains "scend"         |
| bene       | ACCEPTED | Exact match              |
| descend    | ACCEPTED | Contains "scend"         |
| hello      | REJECTED | No matching substring    |
| ben        | REJECTED | Incomplete pattern       |
| science    | REJECTED | Does not contain "scend" |

These examples provide an intuitive understanding before introducing the formal models.

---

## 3. Formal Models

This section presents two equivalent ways to describe the language:

* A **regular expression** (compact and practical)
* A **deterministic finite automaton (DFA)** (formal and explicit)

---

### 3.1 Regular Expression

An equivalent representation of the language using regular expressions is:

```regex
.*(bene|scend).*
```

This expression matches any string that contains either **"bene"** or **"scend"** as a substring.

* `.*` → any sequence of characters (including empty)
* `(bene|scend)` → matches either substring
* The pattern ensures the substring can appear **anywhere** in the input

This is the most concise and commonly used representation in practical applications.

---

### 3.2 Deterministic Finite Automaton (DFA)

The following diagram represents the deterministic finite automaton used to recognize the language:

![DFA Diagram](https://media.cleanshot.cloud/media/83749/DwC0wEVypT6K30V2fDBYXkTMXfOFHxfjdG6f1vBU.jpeg?Expires=1774351787&Signature=Jkj9IzrEcHyHCEG1TeiPAshlgC4-qilYhpxtAoV-xOuuA1O0f8PrJMxFci8nMbj2sTwQjYBbGuTfe1It6YFKyWfkNHdz2FslKb8BB~YNbLT3biATRw2EWbIjSVyhBYv72n~5zueI0Za~FALmDY~aYEz6EJfFgVKigUZ6YCeuUa23qT~mVAtZXNswHMaHMvrNlwtjJh22qiXuFGiWbhCXDxDzuxZ~1eD70CN9zlVehOHUCffwrlIBrACyXUx26h6oCuiE~ds8H8JY0ZWlU~FBBahakp8E6ELf~lIh-Oe2wsGJth04Z8YyZTbgPtEUBpwfanm~X2IDnJH6oXCgh7BoZw__&Key-Pair-Id=K269JMAT9ZF4GZ)

#### Diagram Interpretation

The diagram visually represents the transitions defined in the implementation.

Each arrow corresponds directly to a transition rule. For example:

```prolog
transition(q0, b, q1).
transition(q3, e, q4).
```

Self-loops (e.g., in **q4**) indicate that the automaton remains in the same state for any valid input symbol.

The automaton processes the input string **character by character**, transitioning between states that represent partial matches of the target substrings.

Once the automaton reaches the accepting state (**q4**), it remains there for any additional input. This allows detection of substrings within larger words.

---

#### DFA State Explanation

The DFA is designed to detect the substrings **"bene"** and **"scend"** through intermediate states representing partial matches.

**States for "bene":**

* **q0 (start state)**
  Initial state. No relevant input detected.

* **q1 ("b")**
  The character `'b'` has been detected.

* **q2 ("be")**
  The sequence `'be'` has been recognized.

* **q3 ("ben")**
  The sequence `'ben'` has been recognized.

* **q4 (accepting state)**
  The substring **"bene"** or **"scend"** has been fully recognized.
  This is a final state with self-loops for all inputs.



**States for "scend":**

* **q5 ("s")**
  The character `'s'` has been detected.

* **q6 ("sc")**
  The sequence `'sc'` has been recognized.

* **q7 ("sce")**
  The sequence `'sce'` has been recognized.

* **q8 ("scen")**
  The sequence `'scen'** has been recognized.

From **q8**, reading `'d'` transitions to **q4**, completing the substring **"scend"**.

---

#### DFA Execution Example

**Example 1: `beneficial`**

| Step | Input | Current State | Next State | Explanation             |
| ---- | ----- | ------------- | ---------- | ----------------------- |
| 1    | b     | q0            | q1         | Start matching "bene"   |
| 2    | e     | q1            | q2         | Matches "be"            |
| 3    | n     | q2            | q3         | Matches "ben"           |
| 4    | e     | q3            | q4         | Matches "bene" → ACCEPT |
| 5    | f     | q4            | q4         | Stay in accepting state |
| ...  | ...   | q4            | q4         | Continue                |

Final result: **ACCEPTED**


**Example 2: `transcend`**

| Step | Input | Current State | Next State | Explanation              |
| ---- | ----- | ------------- | ---------- | ------------------------ |
| 1    | t     | q0            | q0         | No match                 |
| 2    | r     | q0            | q0         | No match                 |
| 3    | a     | q0            | q0         | No match                 |
| 4    | n     | q0            | q0         | No match                 |
| 5    | s     | q0            | q5         | Start matching "scend"   |
| 6    | c     | q5            | q6         | Matches "sc"             |
| 7    | e     | q6            | q7         | Matches "sce"            |
| 8    | n     | q7            | q8         | Matches "scen"           |
| 9    | d     | q8            | q4         | Matches "scend" → ACCEPT |

Final result: **ACCEPTED**

---

#### Formal Properties of the DFA

The constructed DFA satisfies:

* **Determinism:** Exactly one transition per state and symbol
* **Completeness:** All symbols in Σ are handled
* **Single accepting state:** Both substrings converge to **q4**

These properties ensure the automaton is well-defined and suitable for implementation.

---

### 3.3 From NFA to DFA

![NFA Diagram](https://media.cleanshot.cloud/media/83749/ioRHYNKuQo9JCsfqQaDUJGjWNhtW1TAvV7RKUUxq.jpeg?Expires=1774350885&Signature=ssh9ZDK0aaXgcjzT99xpsC-WUtNImKanNwrWuytD89jkAVrOLU1gGZoapkhZEIwKYcEwaZ8qaq3PQwNoy8Dhz5YAiN2t-71hY1TL9Sl37PrmHqfQpSeBMYpGg5S4FkXs3tO-FnEzpjS-VqrIg29RHnDvQrzwM7jwliIu0MqoE27b9uVEVhkqh3M4yRWPyF5c1GlVGM3vBMNGynaBrWz7sqhy6brBe1Zcd6k42~VvbTRIMvipNmt1qBdHWZI6I~KVlM~pv5tXf4RR7hpNNm3YLlFbyyFu~-14yfY5xwCCaeuj2HprhoNj58QoMyy1BLfsZWX04Dz7uT-g6QKaY2bbHQ__&Key-Pair-Id=K269JMAT9ZF4GZ)

During the design process, the system can be conceptually viewed as a **non-deterministic finite automaton (NFA)**, since multiple matching paths may exist when detecting substrings.

For implementation, it was converted into a **deterministic finite automaton (DFA)** using the classical subset construction method.

This guarantees:

* A single transition per input symbol
* No ambiguity during execution
* Direct compatibility with programming implementations

Additionally, all missing transitions were defined to ensure the DFA handles the entire alphabet Σ, making it a total function.

This completes the formal foundation of the project. The next sections will focus on implementation and testing.

## 4. Implementation

This section describes how the formal models were translated into working programs using **Prolog** and **Python**.

---

### 4.1 DFA Implementation in Prolog

The DFA was implemented in Prolog using a rule-based approach.

#### Core Components

* `transition/3` Defines state transitions:
  ```prolog
  transition(CurrentState, Symbol, NextState).
  ```

* `run/2` Recursively processes the input string character by character.

* `accepts/1` Determines whether a string is accepted by the DFA.

* `parse_word/1` User-facing predicate that prints whether the string is **ACCEPTED** or **REJECTED**.

#### Example Usage

```prolog
?- parse_word(beneficial).
beneficial -> ACCEPTED
```

#### Implementation Notes

* The automaton strictly processes **lowercase alphabetic characters**
* Each transition is explicitly defined, ensuring determinism
* The recursive structure of `run/2` mirrors the step-by-step DFA execution

This implementation closely follows the formal definition of a DFA, making it suitable for both educational and practical purposes.

---

### 4.2 Regular Expression Implementation in Python

The same language was implemented in Python using regular expressions.

#### Core Function

```python
def matches_regex(word):
    ...
```

Returns:

* `True` → if the word contains `"bene"` or `"scend"`
* `False` → otherwise

---

#### Example Usage

```python
beneficial -> ACCEPTED
hello -> REJECTED
```

---

#### Implementation Notes

* The implementation uses Python’s built-in `re` module
* The pattern used is equivalent to:

```regex
.*(bene|scend).*
```

* A simple **menu-based interface** allows interactive testing of words
* The regex engine handles pattern matching efficiently without explicit state tracking

This approach provides a concise and practical alternative to the DFA.

---

## 5. Tests

To ensure correctness, both implementations were validated using automated tests.

---

### 5.1 DFA Tests (Prolog)

Test cases were executed using a dedicated test predicate.

#### Accepted Cases

```
beneficial
benefactor
bene
descend
transcend
```

#### Rejected Cases

```
hello
apple
ben
ascent
science
```

#### Test Behavior

* Each test checks whether the DFA correctly accepts or rejects the input
* The test runner reports results as **PASSED** or **FAILED**
* Coverage includes:

  * Exact matches
  * Substring matches
  * Near-misses (e.g., "ben")
  * Completely unrelated words

---

### 5.2 Regex Tests (Python)

Tests were implemented using Python’s `unittest` framework.

#### Function Tested

```
matches_regex(word)
```


#### Example Test Cases

```python
self.assertTrue(matches_regex("beneficial"))
self.assertFalse(matches_regex("hello"))
```

---

#### Test Coverage

* Valid substrings ("bene", "scend")
* Words containing substrings
* Invalid inputs without matches

All tests confirm that the regex implementation behaves consistently with the DFA.

These tests validate that both implementations correctly recognize the same language, ensuring functional equivalence between the formal models.

---
## 6. Performance Analysis

This section evaluates the efficiency and behavior of both implementations.

---

### 6.1 Time and Space Complexity

Both approaches process the input string sequentially.

#### DFA (Prolog)

* **Time Complexity:** O(n)
* **Space Complexity:** O(1)

The DFA reads each character exactly once and performs constant-time transitions between states. No additional memory is required beyond the current state.

---

#### Regular Expression (Python)

* **Average Time Complexity:** O(n)

The regex engine scans the input string to locate a match. In this case, the pattern is simple and does not involve nested quantifiers, so performance remains linear in practice.

---

### 6.2 DFA vs Regular Expression

| Aspect         | DFA                  | Regular Expression |
| -------------- | -------------------- | ------------------ |
| Representation | Graph-based (states) | Text-based pattern |
| Implementation | More complex         | Simpler            |
| Readability    | Lower                | Higher             |
| Flexibility    | Low                  | High               |
| Performance    | Deterministic O(n)   | Engine-dependent   |

---

### 6.3 Discussion

The DFA provides a **formal and deterministic model**, making it ideal for theoretical analysis and correctness guarantees. Each transition is explicitly defined, ensuring predictable behavior.

In contrast, the regular expression offers a **compact and practical solution**, widely used in real-world applications such as:

* Text processing
* Input validation
* Search tools

Both approaches recognize the same language but differ in abstraction:

* The **DFA** is more verbose but fully transparent
* The **regex** is concise but abstracts away internal mechanics

In practice, regular expressions are often preferred due to their simplicity. However, DFAs remain essential for understanding the theoretical foundations of pattern recognition.

---

## 7. Limitations

Although both implementations correctly recognize the defined language, they have limitations inherent to lexical analysis.

---

### 7.1 Lack of Semantic Validation

The system detects substrings purely at the **syntactic level**.

Examples:

* `"xxbenezz"` → ACCEPTED (contains "bene", but not a real word)
* `"scendxyz"` → ACCEPTED (contains "scend", but meaningless)

This behavior is expected, as lexical analyzers do not evaluate meaning.

---

### 7.2 Restricted Input Format

* Only **lowercase alphabetic characters** are supported
* Uppercase letters, numbers, and symbols are rejected
* No normalization (e.g., case folding) is performed

---

### 7.3 Limited Pattern Scope

* Only two substrings are recognized: **"bene"** and **"scend"**
* The model does not analyze:

  * Prefixes or suffixes
  * Word structure
  * Grammar or syntax beyond substring matching

---

### 7.4 No Context Awareness

The system does not consider context or linguistic rules.

For example:

* `"ascent"` is **rejected**, even though it is related to "scend"
* Only exact substring matches are recognized


These limitations are consistent with the scope of **lexical analysis**, which focuses on pattern recognition rather than full language understanding.


## 8. Conclusions

This project demonstrates how concepts from **formal language theory** can be applied to a practical lexical analysis problem.

Two equivalent approaches were developed:

* A **Deterministic Finite Automaton (DFA)**, providing a formal and explicit representation of the language
* A **Regular Expression**, offering a concise and efficient implementation

Both models successfully recognize the same language and were validated through comprehensive testing.

The comparison highlights an important trade-off:

* The **DFA** ensures determinism and is well-suited for theoretical analysis and correctness proofs
* The **regular expression** is simpler to implement and more practical for real-world applications

Additionally, the project reinforces the distinction between:

* **Lexical analysis (syntax/pattern recognition)**
* **Semantic analysis (meaning and correctness)**

The implemented system operates strictly at the lexical level, focusing only on substring detection.

Overall, this work shows that DFAs and regular expressions are **complementary tools**, one emphasizing formal rigor, the other practical usability.

---

## 9. Future Work

Several improvements could extend the functionality and realism of this project:

* **Support for uppercase and mixed-case input**
  Implement normalization or case-insensitive matching

* **Expansion of lexical patterns**
  Include additional roots or more complex patterns

* **DFA minimization**
  Apply formal algorithms to reduce the number of states

* **Enhanced input validation**
  Allow broader character sets (e.g., digits or symbols)

* **Context-aware analysis**
  Incorporate basic semantic checks or dictionary validation

* **Visualization tools**
  Provide interactive or programmatic visualization of the automaton

These extensions would bring the system closer to real-world lexical analyzers used in compilers and natural language processing systems.

---

## 10. References

[1] J. E. Hopcroft, R. Motwani, and J. D. Ullman, *Introduction to Automata Theory, Languages, and Computation*, 2nd ed. Boston, MA, USA: Addison-Wesley, 2001.

[2] T. H. Cormen, C. E. Leiserson, R. L. Rivest, and C. Stein, *Introduction to Algorithms*, 3rd ed. Cambridge, MA, USA: MIT Press, 2009.
