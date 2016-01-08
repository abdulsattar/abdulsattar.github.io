---
layout: post
title: Closures and Mutability
date: 2016-01-08T15:48:08+0530
---
Closures are the feature that have been most heavily borrowed from functional languages. However, in a language with a lot of mutable state, it's quite a challenge to get them right.

A closure closes over the variables in its scope. It has access to the variables whenever it is run.

```haskell
map (\i -> runLater i) [1..10]
```

Here, `runLater` was passed the `i` that closure received. Now, `map` calls that closure 10 times with a different value for `i`. However, in languages where `i` does change, we can run into problems. Here is a sample in Javascript:

```javascript
for(i=1; i<=10; i++) {
  setTimeout(function() {
    console.log(i);
  }, i*100);
}
```

That won't print 1..10 to the console. Instead, it will print `11`, 10 times. That is because, by the time the first second elapses, the for loop would have modified `i` to `11`. Now, each of the time, the closure captured the variable `i`. But, `i` itself changed.

Now, you can come around this with a lot of different solutions. Pass `i` via another closure.

```javascript
for(i=1; i<=10; i++) {
  setTimeout(function(i) {
    return function() { console.log(i); };
  }(i), i*100);
}
```

Or, you can use the `bind` method:

```javascript
for(i=1; i<=10; i++) {
  setTimeout(function() {
    console.log(this.toString());
  }.bind(i), i*100);
}
```

But, I find, it's better to not mutate `i` itself. Do it the way the first example in Haskell does it.

```javascript
[1,2,3,4,5,6,7,8,9,10].forEach(function(i) {
  setTimeout(function() { console.log(i); }, i*100);
});
```
