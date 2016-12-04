---
layout: post
title:  "Composing single parameter functions with lists"
date:   2016-12-04 12:07:10 -0200
categories: reactivex
---

Sometimes, you may have a list of elements and you need to apply an async function that takes only a single parameter. In a real case scenario, it could be a function that inserts a single entity in your database. For our simplified scenario, let's assume it's this function:

```
static Observable addTwo(number) {
    Observable.just(number + 2)
}
```

And we want to apply this function to our list of numbers `[1, 2, 3]`.

In this case, we can use the `Observable.from(list)` ([docs here][observable-from-docs]) to generate a list of observables containing each list element and then using [`reduce`][reduce-docs] to get our list back.

```
static Observable addTwoOb(numbers) {
    Observable.from(numbers)
        .flatMap {
            addTwo(it)
        }
        .reduce([]) { acc, elem ->
            acc << elem
            acc
        }
}
```

Here, `[]` is the initial value for our accumulator, which is returned in the callback function.

The source code for this project is [here][post-code]. See ya!

[observable-from-docs]: http://reactivex.io/documentation/operators/from.html
[reduce-docs]: http://reactivex.io/documentation/operators/reduce.html
[post-code]: https://github.com/augustoerico/observable-iterable
