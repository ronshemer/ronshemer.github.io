---
layout: post
title:  "From Safety to Hyper-Safety"
date:   2025-08-09 12:00:00 +0300
categories: jekyll update
---

In the previous post, we explored **safety properties**—guarantees that a program will never enter a "bad state" during any single execution. While this model is powerful for catching many bugs, it falls short for properties that depend on comparing multiple runs. For example, how do we ensure a password checker doesn't leak secrets, or that a program behaves deterministically? In this post, we introduce **hyperproperties**: a framework for reasoning about relationships across multiple executions.

### Thinking in Sets of Executions

Safety properties focus on individual executions, but many crucial guarantees require looking at sets of runs. A **hyperproperty** is a property of a set of execution traces, not just one. Think of it like a restaurant: inspecting one dish checks safety, but ensuring consistent quality means comparing many dishes over time. Hyperproperties let us reason about such consistency and relationships.

### The Limits of "Simple" Safety

The difference between single-trace and multi-trace properties highlights a key limitation: safety properties only consider one execution at a time. Yet, many important behaviors—especially in security—can only be understood by comparing multiple runs.

Take non-interference: we want to ensure that secret inputs (like passwords) don't leak to public outputs. To check this, we must compare at least two runs with different secrets and see if the public output changes. This can't be captured by looking at just one execution.

For example, consider constant-time password checking. If a function returns faster when the first character is wrong, an attacker can measure response times and learn the password one character at a time—a side-channel attack. Safety properties might check for crashes or valid outputs, but only hyperproperties can express whether timing leaks information.

The diagram below illustrates two runs of the same program, each with different secret inputs but the same public input. Non-interference means that changes in the secret should not affect the public output—so an observer can't learn anything about the secret. In password checking, this means execution time should remain constant, regardless of the password.

<figure style="text-align: center; margin: 0 auto; display: flex; flex-direction: column; align-items: center;">
  <img src="/assets/images/non-interference.svg" alt="Non-interference illustration" style="max-width: 400px;">
  <figcaption style="margin-top: 10px;"><em>Figure 1: Illustration of the non-interference property. Two executions of the same program with different secret inputs but identical public inputs should produce identical public outputs.</em></figcaption>
</figure>

### From Safety to Hyper-Safety

<!-- 
**4. From Hyperproperties to "Hyper-Safety"**
    *   Connect the new concept back to the familiar one. If a "safety property" means a "bad thing" (a bad state) never happens in a single run, then a **"hyper-safety"** property means a "bad combination" of things never happens across a set of runs.
    *   Emphasize that, like safety, hyper-safety violations can be demonstrated with finite evidence (e.g., "Here are the two execution traces that, together, prove the violation").
-->
Safety properties prevent "bad states" in single runs; **hyper-safety** properties prevent "bad combinations" across multiple runs. Like safety, hyper-safety violations can be shown with finite evidence—such as two traces that together reveal a leak.

Theoretically, hyper-safety generalizes safety to any number of executions. We call this k-safety: 1-safety is traditional safety, 2-safety involves pairs of runs, and so on.

Hyper-safety properties matter in practice:
- **Non-Interference (Security):** Ensures high-privilege actions don't affect low-privilege observations.
- **Observational Determinism (Reliability):** Guarantees consistent outputs for the same inputs.
- **Constant-Time Execution (Security):** Prevents timing attacks by making execution time independent of secrets.

Hyperproperties aren't just for security experts—they show up in everyday programming. A familiar example is the Java `equals` contract, which most developers know but rarely connect to hyperproperties. Examining this contract reveals how these ideas shape code correctness.

### Hyperproperties in Practice: Java's `equals` Contract

To correctly implement the `equals` method in Java, it must satisfy three key properties: **reflexivity**, **symmetry**, and **transitivity**. These requirements are part of the [Java equals and hashCode contract](https://www.baeldung.com/java-equals-hashcode-contracts).

- **Reflexivity:** For any non-null reference value `x`, `x.equals(x)` should return `true`.
- **Symmetry:** For any non-null reference values `x` and `y`, `x.equals(y)` should return `true` if and only if `y.equals(x)` returns `true`.
- **Transitivity:** For any non-null reference values `x`, `y`, and `z`, if `x.equals(y)` returns `true` and `y.equals(z)` returns `true`, then `x.equals(z)` should also return `true`.

Both **symmetry** and **transitivity** are *hyperproperties*: they cannot be verified by looking at a single execution of the `equals` method. Instead, they require reasoning about multiple executions and their relationships. For example, symmetry requires comparing the results of `x.equals(y)` and `y.equals(x)`—two separate executions. Transitivity involves three executions.

#### Example: Failing Symmetry

Consider the following incorrect implementation:

```java
public class Person {
  private String name;

  public Person(String name) {
    this.name = name;
  }

  @Override
  public boolean equals(Object obj) {
    if (obj instanceof Person) {
      Person other = (Person) obj;
      // Incorrect: only checks if this.name starts with other's name
      return this.name.startsWith(other.name);
    }
    return false;
  }
}
```

Suppose we have:
```java
Person a = new Person("Alice");
Person b = new Person("Ali");
```
Here, `a.equals(b)` returns `true` (since "Alice" starts with "Ali"), but `b.equals(a)` returns `false` (since "Ali" does not start with "Alice"). This violates **symmetry**, a hyperproperty, because the result depends on the direction of comparison.

#### Correct Implementation

To fix this, ensure the comparison is symmetric:

```java
@Override
public boolean equals(Object obj) {
  if (this == obj) return true;
  if (obj == null || getClass() != obj.getClass()) return false;
  Person other = (Person) obj;
  return Objects.equals(this.name, other.name);
}
```

Now, `a.equals(b)` and `b.equals(a)` will always return the same result, satisfying symmetry. This demonstrates how hyper-safety properties are essential for correct and trustworthy code.


### Verifying Hyper-Safety Properties

To verify hyper-safety properties, we can often use the same tools as for safety properties—but with a twist. Since most tools work on single executions, we adapt by using **self-composition**: we build a new program by combining multiple copies of the original, one for each required execution. This lets us express hyper-safety as a safety property over the combined program, making verification practical. More on self-composition in the next post.



### Conclusion

In summary, hyperproperties extend our ability to reason about software by focusing on sets of executions, not just individual runs. This perspective is essential for capturing guarantees like security, determinism, and correctness in everyday code. We saw how even the Java `equals` contract depends on hyper-safety, and how verification can be made practical through self-composition. By broadening our lens, we unlock new tools for building secure, reliable, and trustworthy systems. Next, we'll explore self-composition in depth and connect these ideas to the main contributions of my thesis.