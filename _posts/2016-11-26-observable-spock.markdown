---
layout: post
title:  "Writing async unit tests"
date:   2016-11-26 11:55:23 -0200
categories: test
---

Writing tests for asynchronous functions is a little different than we are used to.

For this post, I'm taking the [ReactiveX][reactivex-page] library to create our async functions and the [Spock][spock-page] framework as our test framework. I'm picking those since they are the ones I'm currently using in the startup I'm working in. The code is available [here][post-code].

Basically, when dealing with asynchronous programming, we assign a handler to our method call, which will be triggered when the method call completes its execution. Let's take a `Calculator` class that emits an `Observable` containing the sum of all values within a list.

```
class Calculator {

    static Observable sum(values) {
        def result = values.sum()
        // Let's pretend this is a resource hunger operation
        [1..1000000].each {
            println it
        }
        Observable.just(result)
    }

}
```

We retrieve the result by doing: `Calculator.sum([1, 2, 3]).subscribe { println it } // prints out 6`

To write our unit test for this class, we must have a way to wait for the result before evaluating it. To do so, we can use [AsyncConditions][asyncconditions-docs] class, which waits for the `evaluate` call.

```
class CalculatorUnitSpec extends Specification {

    static final Double WAIT_TIME = 5.0

    def 'Should sum all positive values'() {
        def cons = new AsyncConditions()
        def result

        given:
        def values = [1, 2, 3, 4, 5]

        when:
        Calculator.sum(values).subscribe {
            result = it
            cons.evaluate { true }
        }

        then:
        cons.await(WAIT_TIME)
        result == 15

    }
}
```

That's all I got for now, thank you!

[reactivex-page]: http://reactivex.io
[spock-page]: http://spockframework.org/
[asyncconditions-docs]: http://spockframework.org/spock/javadoc/1.0/spock/util/concurrent/AsyncConditions.html
[post-code]: https://github.com/augustoerico/observable-spock
