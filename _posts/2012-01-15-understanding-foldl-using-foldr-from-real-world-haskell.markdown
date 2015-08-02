---
layout: post
title: Understanding foldl Using foldr from Real World Haskell
date: 2012-01-15 21:28:35.000000000 +05:30
---
[Real World Haskell](http://book.realworldhaskell.org/read) presents an interesting challenge in the chapter [How to Think About Loops](http://book.realworldhaskell.org/read/functional-programming.html). How do you explain this `foldl` implemented in terms of `foldr`?

```haskell
myFoldl :: (a -> b -> a) -> a -> [b] -> a
myFoldl f z xs = foldr step id xs z
    where step x g a = g (f a x)
```

If you haven't given understanding it a try, I'd really encourage you to invest some time into it before reading this blog post.

Now, let's get into it. Look at the expression `foldr step id xs z`. What is the signature of `foldr`, by the way? It is `foldr :: (a -> b -> b) -> b -> [a] -> b`. It needs three parameters. How many are we passing it here? Four! WTH?

What if we are still passing only three parameters? Just look at `foldr step id xs` without the `z`. Well, this is a valid `foldr` application. But why the `z` in the end? It's as if we are passing `z` to whatever that comes out of `foldr`. Hang on, what if `foldr` really returned a function? Yeah, what if `foldr step id xs` returned a function that takes `z`? This makes sense.

So `foldr step id xs` must return a function. Then its accumulator must be a function. Let's look at the starting value. `id`. Yeah, `id` is a function. We are on the track. Now, the `step` is a mystery here. Before, we look at it, we must know what it must do. step must take an element from `xs`, of type `b` (the type variable). Its accumulator must be a function. And, it must return a function.

Now, every function that we've seen go into `foldr` took two parameters and returned one. But, `step` here seems to take three! Let's again forget the last parameter `a`. `step x g` is a valid function to go into `foldl`. What should `x` be? It must be a value from the `xs` list. Ok, this seems reasonable. Now, what should the next parameter, the accumulator, be? We agreed it must be a function. So, `g` is a function. Have a look the implementation. `g (f a x)`. We don't know what's going on over there but we do know that `g` is a function there! Right? On the right track are we?

Come back once again to the `step x g`. We know that `foldr` will supply only two parameters to `step`. So, `foldr` will give our step only the `x` and `g`. What happens if a function is missing the last parameter? Function currying! It will return a function! `foldr` will get a function for not supplying the last parameter to `step`. Isn't this what we wanted? We wanted our accumulator to be a function and it is. So, do you get the implementation of `foldr`? I mean, we still don't know what `step` does, but we at least know that `foldr` takes the list we are supposed to fold over and a function that takes a function and and returns a function so that the whole thing folds up into a function. We finally supply a `z` to it and get our result. Are we clear? Let's move on.

Let's look at the `step` function now. First, what is f? `f` is a function that takes an accumulator and an element from the list and returns another accumulator. Right, so `f a x` means we are passing an accumulator, `a`, and an element from the list, `x`. We know `x` is an element from the list. So we are right. But what is this `a` anyway? It's that `a` that our step won't get from `foldr`. So, the function that `foldr` will finally return will be a function that takes a variable of the same type as `a`.

We know that `step` will get, as its accumulator, the value it returned previously. Now, since it's being partially applied, we'll get a function, `g`, that takes `a` as a parameter. Since, it won't have a, it'll return another function that takes a new `a`. This goes on until we reach the beginning of the list (we're actually folding right, so we start from the end) and we'll finally have a chain of functions all composed into one function that takes only one `a`. When, we supply the `z` in the end, we initiate a sort of chain reaction that computes the whole result.

To illustrate this point further, let's run an example mentally. Let's run `myFoldl (+) 0 [1..3]`. First look at `f a x`. We know `f` is `(+)` and `z` is the initial value that we pass for `a`. `x` is the last element. So, `f a x` is `(+) 0 3`.

We have `g (f a x)` where `g is id`. Hence, we have `id ((+) 0 3)`. But, `a` is not passed, remember? The value that is returned is a function that takes an `a`, right? The actual value return value is `(\a -> id ((+) a 3))`.
Now this value is again fed as `g` to which the next `f a x` is applied. First let's assume `a` is once again `0`. So, `f a x` here is `(+) 0 2`. So, we have the whole value as `(\a -> id ((+) a 3)) ((+) 0 2)`, right? Now, since we don't have that `0` there, we need to introduce a new variable. So, we have `(\b -> (\a -> id ((+) a 3)) ((+) b 2))`.

If we do this one more time, we get `(\c -> (\b -> (\a -> id ((+) a 3)) ((+) b 2)) ((+) c 1))`.

To this we apply the final `z`, (`0` here), and we get `(\c -> (\b -> (\a -> id ((+) a 3)) ((+) b 2)) ((+) c 1)) 0`. Go ahead, paste this into ghci and you'll get `6`.

Do you see how this evaluates? First, `c` gets substituted by `0` which leaves us with `((+) 0 1)` or `(0 + 1)`. This `(0 + 1)` is substituted for `b`, and we get `((0 + 1) + 2)`. Do this once again, and you get `((0 + 1) + 2) + 3`. This is what a `foldl` does.

This turned out to be a very long post than I thought but I hope it'll give you a clear idea of what the function does. This was an amazing example that displayed the power of functional programming. You knew that functions are like other values but the idea of folding over functions to create new functions is awesome. How awesome? Barney Stinson awesome :). Anyway, have a good day and enjoy hacking in Haskell.
