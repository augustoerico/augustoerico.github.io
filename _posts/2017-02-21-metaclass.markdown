---
layout: post
title:  "Metaprogramming with Groovy - a tip"
date:   2017-02-21 9:54:14 -0200
categories: general
---

# Overview

Metaprogramming allows us to modify the program itself in runtime. You may refer to [metaprogramming in Groovy][metaprograming-groovy] for a more detailed explanation, but, in short lines, Groovy is capable of intercepting, injecting and synthesizing classes and interfaces methods.

# How?

Here is our class:

```
class Person {

  def name
  def age

}
```

We want to inject a method `sayHello()` in the `Person` class without updating the class itself. This can be done by:

```
Person.metaClass.sayHello = {
  println 'Hello!'
}
```

Warping all together:

```
class Person {

  def name
  def age

}

Person.metaClass.sayHello = {
  println 'Hello!'
}

def person = new Person()
person.sayHello() // prints out "Hello!"
```

# Real case scenario

Why would you need to use this? Well, I've been working with a [SOAP web service][soap-ws]. We integrate the service into our code by generating stub classes based on the WSDL provided. It's not the best practice to maintain code generate by tools, since the modifications won't be applied when regenerating it. Still, I want to add validations in the constructor in one of the generated classes and assure the `Person` is always instantiated with `name` not `null`.

Adding this behavior to the constructor is a little tricky because the constructor returns a instance by using a constructor. This is a cyclic dependency, which results a stack overflow. The work around for this situation is to instantiate by using the `Person` class constructor newInstance method, which is not intercepted by our function.

```
class Person {

  def name
  def age

  String toString() {
    "{name=$name,age=$age}"
  }

}

Person.metaClass.constructor = { Map map = null ->
  assert map
  assert map.name
  def instance = Person.class.getConstructor().newInstance()
  instance.name = map.name
  instance.age = map.age
  instance
}

def person = new Person([name: 'Erico', age: 25])
println person.toString() // prints out "{name=Erico,age=25}"

person = new Person([name: 'Erico'])
println person.toString() // prints out "{name=Erico,age=null}"

person = new Person([age: 25]) // fails at assertion
```

Notice that I'm intercepting both `new Person()` and `new Person(Map map)` constructor by putting `map` as an optional parameter in my closure. This way, I assure the `Person` instance always have a `name` property not null.

Hope you guys make a good use of this. See ya!

[metaprograming-groovy]: http://groovy-lang.org/metaprogramming.html#_runtime_metaprogramming
[soap-ws]: https://en.wikipedia.org/wiki/SOAP
