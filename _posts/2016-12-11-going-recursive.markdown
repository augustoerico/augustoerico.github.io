---
layout: post
title:  "Going recursive and functional!"
date:   2016-12-11 13:40:12 -0200
categories: general
---

Let's say you want to apply a function to all the values in a map structure. Take Josh's employment history for instance:

```
[
    name: 'Josh',
    location: [
        currentCity: 'San Francisco',
        pastCities: ['Berkeley', 'Los Angeles', 'Santa Monica'],
        employment: [
            pastEmployers: [
                [employer: 'Code Happy'],
                [employer: 'House of Code']
            ]
        ]
    ],
    programmingLanguages: ['Groovy']
]
```

We can write a [recursive function][recursive-function] that applies the desired transformation as:

```
class DataTransformer {

    static applyMyFunction(Map map) {
        def newMap = [:]
        map.each { key, value ->
            if (value instanceof Map || value instanceof List) {
                newMap."$key" = applyMyFunction(value)
            } else {
                newMap."$key" = myFunction(value)
            }
        }
        newMap
    }

    static applyMyFunction(List list) {
        def newList = []
        list.each { value ->
            if (value instanceof Map || value instanceof List) {
                newList << applyMyFunction(value, closure)
            } else {
                newList << myFunction(value)
            }
        }
        newList
    }

}
```

But, what if we want to apply another function? Since Groovy is a functional programming language, we can easily refactor this code to accept a [Closure][closure-docs] as a parameter:

```
class DataTransformer {

    static transform(Map map, Closure closure) {
        def newMap = [:]
        map.each { key, value ->
            if (value instanceof Map || value instanceof List) {
                newMap."$key" = transform(value, closure)
            } else {
                newMap."$key" = closure(value)
            }
        }
        newMap
    }

    static transform(List list, Closure closure) {
        def newList = []
        list.each { value ->
            if (value instanceof Map || value instanceof List) {
                newList << transform(value, closure)
            } else {
                newList << closure(value)
            }
        }
        newList
    }

}
```

This is one of the main advantages of using a functional language: we can decouple the function that transforms the data and the one that iterates on our data structure. Notice that, even if we had more complex objects as values instead of Strings, the `transform` function is the same.

As usual, the code for this post is at [GitHub][code-post]. See ya!

[recursive-function]: https://en.wikipedia.org/wiki/Recursion_(computer_science)
[code-post]: https://github.com/augustoerico/going-recursive
[closure-docs]: http://groovy-lang.org/closures.html
