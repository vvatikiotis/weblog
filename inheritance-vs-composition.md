---
Author: Vassilis Vatikiotis
Tags: OOP
Date: 20 July 2018
Title: Inheritance vs Composition
---

# Inheritance vs Composition

Notes on https://www.thoughtworks.com/insights/blog/composition-vs-inheritance-how-choose

## Inheritance

- Semantics
- Mechanics

Example 1: How to misuse Inheritance

```typescript
class Stack extends ArrayList {
    public void push(Object value) { … }
    public Object pop() { … }
}
```

1. Semantic flaw: A Stack is not an ArrayList, or a Stack is not a proper subtype of ArrayList, check [Liskov substitution principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle). Substituting an ArrayList with a Stack isn't semantically sound...
(I need to further explore this, not satisfied with the article. For example,in [Subtyping](https://en.wikipedia.org/wiki/Subtyping#Relationship_with_inheritance) it states that Subtyping and Inheritance are orthogonal).

2. Mechanical flaw: using an ArrayList to hold the stack's object collection is an implementation choice that should be hidden from consumers.

3. Cross-domain inheritance relationship: Domain classes should *use* implementation classes, not inherit from them. Implementation space should be invisible at the domain level.

---

A (preferred by author) solution is to inherit from utility classes as much as necessary to implement mechanical structures, then use these structures in domain classes via composition.

**and**

Unless you are creating an implementation class, you should not inherit from an implementation class.

**and**

your application domain classes should *use* implementation classes, *not* be one.

## Using inheritance well

Primarily for *additive* changes. For example, we are subclassing a Widget class with a few tweaks and enhancments (that's why the author uses the word additive).

*Do not* mix application domain level and implementation hierarchies/taxonomies.

## How to decide, Composition or Inheritance?

Split the discussion in two:

1. One dimension is the representation/implementation of domain concepts
2. Another dimension is the semantics of domain concepts and their relationship to one another.

Do not inherit across inter-dimensional boundaries!

Implementation-wise, inheritance should be used when:
1. Both classes are in the same logical domain
2. Child is a proper subtype of parent
3. the superclass implementation is necessary/appropriate for subclass
4. Subclass enchancements are additive.

- Higher-level domain modelling
- Frameworks and extensions
- Differential programming (Like the Widget example, *additive*)

I probably don't need inheritance if I'm not doing any of the above 3 things.