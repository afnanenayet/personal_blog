---
title: "Semigroups and Monoids"
date: 2019-10-07T09:59:52-07:00
---

In functional programming there are two concepts that are mentioned a lot that
may sound intimidating: the semigroup and the monoid (*not monad*). These
concepts originate from 
[category theory](https://en.wikipedia.org/wiki/Category_theory), which is a
branch of math that aims to reason about the entirety of math through graphs
called "categories". While monoids and semigroups sound complex, they are a
simple but powerful abstraction that allow you to generically combine data
types.

## Semigroups

Before we get into monoids, we first need to define a `Semigroup`, since the
monoid is a strict subset of the semigroup. A semigroup is basically a set with
an associative multiplication operation defined on it. In Haskell, we don't
have a multiplication operation, but rather, the `<>` operator. Both of these
things allude to the same thing: a method of coalescing elements across a set
in a associative manner. Note that all I mean by "associative operation" is
that `a <> (b <> c) = (a <> b) <> c`, so order shouldn't matter when chaining
these operations together (just like with multiplication).

So how do we go about implementing a semigroup in Haskell? You only need to
implement one function: the binary `<>` operator. There's more to it, but the
other functions have default implementations that you probably won't need to
worry about.

### An Example

So how would we define a semigroup on some arbitrary data type?

First, let's start out by defining a new data type: a 2D point which consists
of an x and y value:

```haskell
data Point2D x y = Point2D Int Int
```

Now how are we going to combine `Point2D` values? It seems natural that we
would add them together by adding the x and y values to create a new point.
This is easy to implement, and it has the bonus of being already associative.

```haskell
instance Semigroup (Point2D a b) where
    (<>) (Point2D x1 y1) (Point2D x2 y2) = Point2D newX newY
      where
        newX = x1 + x2
        newY = y1 + y2
```

It really is that easy. We have implemented a `Semigroup` instance on a data
type by implementing one function.

## Monoids

Monoids are a strict subset of semigroups. They have the same properties as
semigroups save for one thing: the identity element. Monoids need to have some
identity element (`ident`) such that `a <> ident = a` and `ident <> a = a`.
For addition, the identity element is 0, because `a + 0 = a` and `0 + a = a`.
For multiplication, the identity element is 1 (proof is left as an exercise to
the reader).

We can see from the following declaration of the `Monoid` class that you only
have to implement one function, the identity element. The rest is derived from
your semigroup implementation by default.

```haskell
class Semigroup m => Monoid m where
  mempty :: m

  -- defining mappend is unnecessary, it copies from Semigroup
  mappend :: m -> m -> m
  mappend = (<>)

  -- defining mconcat is optional, since it has the following default:
  mconcat :: [m] -> m
  mconcat = foldr mappend mempty
```

### An Example

Implementing the identity element for `Point2D` is trivial, because we already
know the identity element in addition.

```haskell
instance Monoid (Point2D a b) where
    mempty = Point2D 0 0
```

We can play around with using the methods provided by the monoid class. For
example, let's define a few points:

```haskell
p1 = Point2D 1 2
p2 = Point2D 2 3
```

We can combine these points using `mconcat`, `<>`, or `mappend`:

```txt
mconcat [p1, p2]

> Point2D 3 5

--

p1 <> p2

> Point2D 3 5
```

You can combine any number of these points in any arbitrary manner using
semigroups and monoids, which provide a powerful abstraction over elements of
sets, or more concretely, elements of a data/type class.
