---
layout: post
title:  "Watch out for groovy string!"
date:   2016-12-31 19:45:32 -0200
categories: general
---

Today, I want to talk about something that really bugged me last Thursday. It's about the [Groovy String][groovy-string-docs]. It's not the same object as the String, but behaves in a way that can cause confusion.

Let's take this code snippet:

```
def var = 'name'
def map = ["$var": 'Erico']

println map // prints out [name:Erico]
println map.keySet() // prints out [name]
```

So, if you want to get the value `Erico`, you would use `map.name`, right? Let's see:

```
println map.name // prints out null
```

What's happening here? Well, the thing is, the key in your map is not a String. There is no `name` key in your map. That's why you receive a `null` value.

It can be confusing since you the map prints out as `[name:Erico]`. The right way to do it would be:

```
def var = 'name'
def map = [(var): 'Erico']

println map.name // prints out Erico
```

No code for this one, sorry!

[groovy-string-docs]: http://docs.groovy-lang.org/latest/html/api/groovy/lang/GString.html
