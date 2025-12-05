---
title: Inheritance give you no power, composition does
date: '2025-09-20'
categories:
    - post
tags:
    - cs 
    - design
draft: true
---

## Three (actually more) ways of doing polymorphism

- why do generic in the first place? only for reduce code duplication?
- runtime polymorphism (have time to explain it??)
- Inheritance, but not just python, look ma, java/c++.
- Rust/Golang trait/interface
- Julia's dispatch
- FP v.s OOP: https://www.youtube.com/watch?v=08CWw_VD45w

- Python can do anything above (same for c++)! But that is another chaos -> coding standard heavily exist in c++ and python projects, but less bit a problem in other programming language. That is why golang shines (or even java), and rust should be mentioned.
- the toolings may not the best for any of that. (duck typing with Protocol. but why that is not ideal? (the types!!))
- functools.singledispath (julia like), protocol. Redo it in those style.
- You just refactor it into your style.
- my experiment with David Perl's library. 30 mins to see the big picture and then able contribute a performance optimization.
- DFTK and julia's lively community.

- Python is good in the sense of it's dominance, and we (I) will live with it for another 10 years, but how?

- [ ] for myself, need to read https://lab.abilian.com/Tech/Python/OOP/The%20Zen%20of%20Polymorphism%20-%204%20Ways%20to%20Write%20Cleaner%20Python/

## How inheritance fails hard

- political, UK, eu ...
- fun side-track: https://www.cambridge.org/core/journals/modern-intellectual-history/article/revolution-in-property-tocqueville-and-beaumont-on-democratic-inheritance-reform/9F1575B4F000784CFA707A3B83E2BA2A
- when a new "hard to categorized type added in".
- scattered dependencies.
- real life case with aiida-core's scheduler, with plumpy, with workgraph.

```python
class Animal:
    def eat(self): pass

class Bird(Animal):
    def fly(self): pass

class Fish(Animal):
    def swim(self): pass

class Duck(Bird):   # Ducks fly, so inherit Bird
    def quack(self): pass
```

```python
class Penguin(Bird):
    def fly(self):
        raise Exception("Penguins don't fly")
```

```python
class Platypus(Animal):
    # we need swim(), but we don't want to be a Fish...
```

```rust
trait Animal {
    fn eat(&self);
}

trait Fly {
    fn fly(&self);
}

trait Swim {
    fn swim(&self);
}

struct Duck;
impl Animal for Duck { fn eat(&self) {} }
impl Fly for Duck { fn fly(&self) {} }
impl Swim for Duck { fn swim(&self) {} }

struct Penguin;
impl Animal for Penguin { fn eat(&self) {} }
// Penguins can't fly—so we simply don't impl Fly
impl Swim for Penguin { fn swim(&self) {} }
```

## Composition with interface comes to rescue

## When inheritance is not too bad?

- it all started with one big function, dos as example.
- Alan Kay, and .. Casys's talk.

## bonus, duck tyting impl without dependencies.

## abstraction is nice, too much of it can be lead to nothing.

- rust philosophy: abstraction is allowed only if it's zero-cost and clear


