---
layout: post
title:  "Code's Deeper Truths"
date:   2025-05-26 21:55:34 +0300
categories: jekyll update
---
Welcome to Lattice of Thought. While I intend for this space to primarily share technical thoughts, ideas, and experiences, I want to dedicate this initial series of posts to a somewhat different topic. As a software developer whose main work is around backend systems, my workdays are often spent tackling practical, well-defined problems and building concrete solutions. But every now and then, I find myself asked about my thesis – a topic that dives into more theoretical computer science. While it might seem a bit academic at first glance, the concepts are profoundly relevant to the quality, reliability, and ultimately, the future of software development. These posts are my attempt to bridge that gap, offering fellow software engineers and developers a clear, non-academic glimpse into the ideas behind my thesis and how they relate to our everyday work. If you're curious and want to jump straight to the source, you can find my thesis PDF [here](https://www.tau.ac.il/~sharonshoham/papers/cav19b.pdf).


This first post marks the beginning of a short series dedicated to unpacking those theoretical foundations. To better understand the core of my thesis, we need to establish a common understanding of software verification (sometimes referenced as **formal verification**). While we'll touch on concepts that can get quite mathematical, I'll attempt to explain them with as little formal notation as possible. We'll start by defining what software verification actually entails, how it differs from the more familiar practice of testing, and explore some of its theoretically proven limitations. Finally, we'll look at a few examples of where software verification concepts are already being applied in the wild, even if not always explicitly called out. This foundational knowledge will then set the stage for future posts, which will cover **safety hyperproperties**. Then the ground will be set to better explain my thesis.

Note that for the rest of this post while most code examples are written in Python, the principles apply to all languages.

### Table of Contents
- [Software Verification: A State-Based View](#a-state-based-view)
- [Beyond Testing](#beyond-testing)
- [Unavoidable Truth: Theoretical Limitations](#theoretical-limitations)
- [Verification in the Wild](#verification-in-the-wild)

### Software Verification: A State-Based View {#a-state-based-view}

In the world of software, programs operate by transitioning through various "states." Each state represents the collective values of all the program's variables, its memory, and its control flow at a given moment. When a program functions incorrectly, it often means it has entered a "bad state" — a configuration of variables or conditions that violates its intended behavior, leading to errors or undesirable outcomes.

The challenge of software verification lies in answering a critical question (also called a **safety property**): given a program and an initial state, can we guarantee that the program will never reach a "bad state"? A bad state might represent a crash, a security vulnerability, a malicious activity, or any undesirable behavior. Verification involves systematically analyzing the program to ensure such states are unreachable.

In more practical terms, bad states in software systems often represent some of these scenarios:
- **Null Pointer Dereference**:  
  **State**:
  ```python
  ptr = None
  ```  

  **Code**: 
  ```python  
  value = ptr.some_property  # Accessing a property of 'None' causes a crash.
  ```

- **Buffer Overflow**:  
  **State**: 
  ```python 
  buffer = [0, 1, 2]
  ``` 

  **Code**: 
  ```python 
  buffer[3] = 99  # Writing beyond the allocated buffer size.
  ```

- **Arithmetic Overflow**:  
  **State**: 
  ```python
  x = 2**31 - 1 #maximum value for a 32-bit signed integer
  ```

  **Code**: 
  ```python
  x = x + 1 # Results in an overflow, wrapping around to a negative value.
  ```


<br>
<figure style="text-align: center">
  <img src="/assets/images/car.jpg" alt="Car reaching a bad state" style="margin: auto; display: block;">
  <figcaption><em>Imagine a self-driving car allowed to get into that state.</em></figcaption>
</figure>

### Beyond Testing {#beyond-testing}

Testing is our primary tool for uncovering reachable bad states in a program. When we write tests, we effectively provide the program a limited set of inputs, forcing it into a limited set of initial states, and then allow it to execute. We then examine the resulting states to see if any are "bad states." If a test fails, it means we've successfully demonstrated that for that particular set of initial conditions, the program can indeed reach a bad state. The challenge with testing is that it can only explore a finite number of these possibilities. Even with extensive test suites, there's always an enormous, often infinite, number of other states the program could potentially be in that we haven't examined. This leaves open the possibility that a bad state might exist in an untested corner of the program's behavior.

Software verification, by contrast, takes a different approach. Its goal isn't just to find bad states within a few specific scenarios. Instead, it aims to establish a conclusive argument that a program will never enter a bad state, regardless of the sequence of operations or inputs. This means considering the entire realm of all possible states the program could ever reach. When verification is successful, it provides strong assurance that your program, within its defined scope, is robustly free from specified bad states.

Think of it like this: our program is a set of activities designed for a child. Testing is like observing the child's mood during a few specific activities (e.g., playtime, mealtime) to see if any of them lead to a tantrum (a "bad state"). You might identify some triggers, but you haven't observed the child in all possible situations, so other triggers could still exist. Software verification, however, aims to prove that within the boundaries of our defined set of activities, a tantrum will never occur, ensuring that particular "bad state" can never be reached within the scope of our program. While we cannot guarantee the child will *never* be in a bad mood, we can attempt to prove it within the confines of our planned activities.

To illustrate the limitations of testing, consider a function designed to perform a seemingly simple math operation:

```python
def tricky_operation(x, y):
    # If numbers are different, add them
    if x != y:
        return x + y
    # If numbers are the same, attempt a division that leads to a bad state
    else:
        # This will cause a DivisionByZero error if x == y
        return x / (y - x)
```

Here, the "bad state" we're concerned about is one where the program attempts to divide by zero, leading to an immediate crash. This occurs specifically when `x = y`.

Now, let's look at a typical test for this function:

```python
result = tricky_operation(5, 3)
print(f"Result for (5, 3): {result}") # Expected: 8
```

When we run this test with `x = 5` and `y = 3`, the program enters an initial state where x and y are indeed different. The function correctly returns 8 and avoids the division by zero.

This single test, or even many tests where `x != y`, provides no insight into what happens when `x = y`. It offers no guarantee that the "bad state" of division by zero cannot be reached under other, untested, initial conditions. Testing shows the absence of a bad state for the states it visits; verification aims to ensure it for all states, whether visited in our tests or not.

### Unavoidable Truth: Theoretical Limitations {#theoretical-limitations}

While software verification offers the promise of a far stronger guarantee than testing, it's vital to understand that even this rigorous approach operates under certain fundamental constraints. We won't delve into the deep mathematical proofs that underpin these limitations. However, grasping what these limits will help us to understand what's truly achievable and where compromises must be made.

This brings us to [Rice's Theorem](https://en.wikipedia.org/wiki/Rice%27s_theorem), a cornerstone result in theoretical computer science. In essence, Rice's Theorem tells us that for any non-trivial definition of a bad state, it's impossible to create a general algorithm or tool that can definitively prove whether a program may reach a bad state. This includes seemingly simple questions like whether a program always terminates.

Applying this directly to our discussion: Rice's Theorem implies that we cannot create a universal verification tool that can precisely and perfectly determine whether a program will always avoid a particular bad state. This isn't a limitation of our current technology or our cleverness as engineers; it's a fundamental, proven limitation of computation itself.

Given this theoretical barrier, any practical software verification tool must make accuracy compromises. These compromises typically fall into one (or more) of two directions:

#### Soundness (No False Positives): 

A sound tool will never claim a program is free of bad states if it actually contains one. If a sound tool says "no bad state found," you can trust that assertion. The compromise here is that it might sometimes fail to give a definitive answer (e.g., say "I don't know") even if the program is free of bad states, meaning it might be incomplete.

#### Completeness (No False Negatives): 

A complete tool will always find a bad state if one exists. If a program truly has a bad state, a complete tool will eventually tell you. The compromise here is that it might sometimes issue "false alarms," claiming a program has a bad state when it actually doesn't (meaning it might be unsound).

A verification tool may be sound and incomplete, unsound and complete, or unsound and incomplete.

To better grasp these idea, think about a "Child Mood Detector" app (some will call it a "parent"). This app is designed to alert if a child enters a "bad mood" state – perhaps they're grumpy, crying, or throwing a tantrum. The challenge in building such a tool, just like with software verification, is the inherent complexity of detecting this "bad state" perfectly for all situations.

- **Soundness** (No False Positives):
If your "Child Mood Detector" is sound, it means that every time it tells you your child is in a bad mood, your child is genuinely in a bad mood. It never gives you a false alarm. You can trust its warnings completely – if it flags a bad mood, you know it's truly there.

    The Compromise: To be this reliable, the tool might be very strict in its criteria. It might sometimes miss actual bad moods, e.g., if your child is quietly sulking, the tool might not pick up on it because it's only looking for very clear, undeniable signs like crying. So, while its "bad mood" alerts are always true, it might not detect every instance of a bad mood.

- **Completeness** (No False Negatives):
If your "Child Mood Detector" is complete, it means that every time your child is actually in a bad mood, the tool will always detect it and alert you. It never misses a single bad mood. You can be confident that if your child is grumpy, the tool will know.

    The Compromise: To ensure it catches every bad mood, the tool might be very sensitive. It might sometimes give you "false alarms," flagging your child as being in a bad mood when they are just quietly concentrating, or simply tired, but not actually grumpy. It sacrifices precision for thoroughness.

In the world of software verification, building a tool that is both perfectly sound and perfectly complete for complex, non-trivial programs and states, is what Rice's Theorem tells us is generally impossible. Developers usually have to choose which type of error is more acceptable for their specific use case: missing a potential bug (incompleteness) or getting false alarms about non-existent bugs (unsoundness). For critical systems, soundness is almost always prioritized, accepting incompleteness in return for certainty when a "no bad state" guarantee is provided. 

One can think of the accuracy of any static analysis tool as a spectrum. On one end we have very "quiet" tools that almost never alerts on any issue, but when they do we should take them very seriously. On the other end of the spectrum, we have very "noisy" tools that we can be pretty certain that they did not miss any, or at least many, issues. Of course, some tools are in neither ends of this spectrum and their usability very much depends on our needs. 

### Verification in the Wild {#verification-in-the-wild}

While software verification might sound abstract, its principles are already being applied in real-world systems. For instance, modern compilers and static analysis tools often use verification techniques to detect potential bugs, such as null pointer dereferences or race conditions, before the code is even executed. Another example is in safety-critical industries like aerospace or automotive, where verification ensures that systems like autopilots or braking systems behave correctly under all possible conditions.

One particularly common example of verification being frequently employed is **static type checking**. Static type checkers, such as those used in languages like TypeScript or Python (via tools like `mypy`), analyze the code at compile-time (or before execution) to ensure that variables and functions are used in ways consistent with their declared types. This process helps catch a wide range of errors early in the development cycle, such as type mismatche or using undefined variables.

For example, consider the following Python code:

```python
def add_numbers(a: int, b: int) -> int:
    return a + b

result = add_numbers("hello", 5)  # Static type checker will flag this as an error
```

This simple example shows how a type checker can catch potential runtime errors before execution. However, like any verification tool, `mypy` must also balance between soundness and completeness. In fact, `mypy` provides various configuration options to tune this balance according to your needs. For instance, you can configure how it will behave when it fails to infer the types of global and class variables (see [allow-untyped-globals](https://mypy.readthedocs.io/en/stable/command_line.html#cmdoption-mypy-allow-untyped-globals)). A more permissive configuration might let more potential errors slip through (sacrificing soundness) but generate fewer false alarms, while a stricter configuration might catch more real issues but also flag legitimate code patterns as problematic (sacrificing completeness). This practical example illustrates how the theoretical trade-offs we discussed earlier manifest in real-world tools.

### Conclusion

We've covered a lot of ground in this initial exploration into software verification. We've established its core purpose, distinguished it from testing by focusing on their differing approaches and even touched upon the fundamental limitations imposed by Rice's Theorem. We also saw glimpses of its application in real-world scenarios. This foundational understanding is crucial, as it sets the stage for our next discussions. Stay tuned for the next post, where we'll delve into safety hyperproperties.
