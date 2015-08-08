---
layout: post
title: Learning Idris (Part 1)
date: 2015-04-29 22:28:22.000000000 +05:30
---
I've been trying to learn [Dependently Typed Languages](http://en.wikipedia.org/wiki/Dependent_type) for a while now. [Idris](http://idris-lang.org/) has dependent types and is very similar to Haskell. This series of blog posts chronicles my efforts into learning Idris.

Fair warning, I don't know a lot of Idris. There may be errors in this post that I hope to correct as I keep learning Idris. However, if you get confused by what I wrote below, do NOT let that turn you off Dependent Types. They are actually really cool and at the forefront of Programming Language research. This article may not be doing justice to them.

With that out, let's try to solve a simple problem.

> Return the first element of a non-empty array

Here's a simple implementation in Ruby.

```ruby
def first(arr)
  arr[0]
end
```

But, you know, this is flawed. Here are the scenarios in which it can fail.

1. You pass value that is not an array<sup>1</sup>
2. You pass null
3. You pass an empty array

To solve this, you have to write code that handles those cases at runtime.

Let's write this in Java now.

```java
public int first(int[] list) {
  return list[0];
}
```

You can be assured that the argument passed to `first` will be an `int`. Java handles the first flaw at compile time itself.

1. ~~You pass value that does not have type array<sup>1</sup>~~
2. You pass null
3. You pass an empty array

However, the other two points still remain. Let's write this problem in another language: Haskell.

```haskell
first :: [a] -> a
first xs = xs !! 0
```

Haskell doesn't allow you to pass `null` when calling `first`. So, we can strike point no. 2 too.

1. ~~You pass value that does not have type array<sup>1</sup>~~
2. ~~You pass null~~
3. You pass an empty array

Point no. 3 still remains. To strike it off, we must be able to make sure, at compile time, that the argument passed to `first` will always be a non-empty array. That is, the length of the array should also be part of the type of the argument. Those types are called *Dependent Types*. They depend on the value they hold. We'll use Idris to write the definition of `first` that does not have any of those flaws.

Compilers don't really know the difference between 0 and 1. An `int` is something that doesn't have a decimal value for compilers. 0 and 1 have different values at runtime, but at compile time, they are just `int`s. We need to teach the compiler that there's a difference between them so that we can achieve our ultimate goal of accepting a non-empty array.

Let's work with only Natural Numbers: 0 and more. We'll recursively define Natural Numbers. So, a natural number is either zero or a successor of another natural number.

```haskell
data Nat = Z | S Nat

zero = Z
one  = S Z
two  = S (S Z)
-- and so on.
```

Now, let's define an array that has its length as part of its type. The length will be a natural number and it'll hold any type of elements. It's usually called a *Vector*.

```haskell
data Vect : Nat -> Type -> Type where
```

The `Nat` is the length of the Vector and the first `Type` is the type of elements of the Vector. The second `Type` is the resultant dependent type.

Let's implement Vect recursively. A Vector can be of length zero. That is, it can be `Vect Z a` (a is a generic type).

```haskell
data Vect : Nat -> Type -> Type where
  Nil : Vect Z a
```

Or it can be an element appended to another Vector. In that case its length will be exactly 1 more than the other Vector. So, if the other vector is of length n, its length will be n+1. If n is `Nat`, it'll be `S n`.

```haskell
data Vect : Nat -> Type -> Type where
  Nil : Vect Z a
  (::) : a -> Vect k a -> Vect (S k) a
  
zeroVect : Vect 0 Int
zeroVect = Nil

oneVect : Vect 1 Int
oneVect = 3 :: Nil

threeVect : Vect 3 String
threeVect = "I" :: "hope" :: "i'm not confusing you" :: Nil
```

Now that we have this, writing `first` should be fairly easy. It's just a function that takes any `Vect` with length greater than zero, i.e. `S n` where `n` is any `Nat`. Even if `n` is `Z`, `S n` is `1`.

```haskell
first : Vect (S k) a -> a
first (x::xs) = x
```

We can always guarantee that `first` always receives an array that is neither null nor empty. There you go, strike off the 3<sup>rd</sup> point from the list too.

1. ~~You pass value that does not have type array<sup>1</sup>~~
2. ~~You pass null~~
3. ~~You pass an empty array~~

I've read that Dependent Types allow you to encode any runtime possibility into the type and have it checked at compile time. This allows for having **no** runtime errors at all. I still have hard time grasping that concept.

If you're interested in Idris, join me in learning it by reading [this tutorial](http://docs.idris-lang.org/en/latest/tutorial/introduction.html).

[1] Technically, any value that can't be indexed over.
