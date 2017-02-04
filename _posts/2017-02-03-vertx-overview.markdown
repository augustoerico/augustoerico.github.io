---
layout: post
title:  "We need to talk about Vert.x - a quick overview"
date:   2017-02-04 11:30:14 -0200
categories: vertx
---

# Why should you care about Vert.x?

[Vert.x][vertx] is a toolkit to write event driven applications. And it is great to handle concurrency with minimal hardware required.

For those with hardware background, it's the same idea behind working with [interrupt signals][interrupt-wikipedia]. Think of a microprocessor delegating some operation to another microprocessor. While the second one is doing the math required, the first one can keep doing other tasks instead of waiting for the result.

With Vert.x, we can do the same thing: you can have your server running on a thread while you have other thread dealing with database access, for instance. This approach can save you infrastructure resources.

# Say "Hello!" to our 1 million dollars idea -  a test case

According to our research, there are many lonely people in the world. Our solution is to build a RESTful API that says "Hello, <name>!" and "Bye, <name>!". These two operations are computational expensive (no, they are not), so we may want to run them on separated threads, keeping our server free to handle other requests.

# A quick note about verticles

In our application, the [verticles][verticle-docs] are responsible for the computational expensive process.

First, we deploy both verticles (`HelloVerticle` and `ByeVerticle`). These verticles will run on a different threads than the main application.

```
vertx.deployVerticle(HelloVerticle.name, [worker: true], {
    vertx.deployVerticle(ByeVerticle.name, {

        Router router = Router.router(vertx)
        router.route('/hello').handler { // route handler  }
        router.route('/bye').handler { // route handler }

        vertx.createHttpServer()
                .requestHandler(router.&accept)
                .listen(8080)        
    })
})
```

Assume the application is running on `Thread A`. The route handlers will be running in `Thread A` as well. Both services (`Hello` and `Bye`) send their messages to the event bus using `Thread A` and they handle the results in this very same thread.

Let's take a look in how the `Hello` service sends a message to the event bus:

```
def sayHello(String name, ...) {
    vertx.eventBus().send(
        Channel.HELLO.name(),
        name,
        { // result handler }
    )
```

Now, assume the `ByeVerticle` is started in the `Thread B`. The message sent by the service will be processed in this `Thead B`. In other words, it won't block the main application thread.

Since we defined `HelloVerticle` as a worker verticle, it starts in the `Thread C` but, once it receives the message from the service, it borrows a thread from the worker thread pool. So, at each call from the service, the message will be processed in threads `D`, `E`, `F`...

The main difference between these two types of verticle is the amount of time it can execute before throwing `Thread blocked` warning. For "standard verticle", it is 2 seconds, by default. For "worker verticle", 60 seconds.

Before being able to handle the messages, at the verticle start, it need to register the handler in a given channel of the event bus:

```
vertx.eventBus().consumer('HELLO', { // message handler })
```

For the sake of the curiosity, this is how we are simulating the message processing.

```
def helloHandler = { Message message ->
    // Thread from worker pool (worker verticle)
    this.thread = Thread.currentThread()
    println "HelloVerticle.helloHandler: $thread.id"

    def name = message.body()
    def now = new SimpleDateFormat().format(new Date())

    // Let's pretend it is a resource hunger operation that takes up to 1s
    Thread.sleep(new Random().nextInt(10) * 100)

    message.reply("Hello, $name at $now".toString())
}
```

# Testing the application

You can run the Application main and make requests using [Postman][postman]. Just make GET requests for `http://http://localhost:8080/hello` and `http://localhost:8080/bye`

Here is some output I had in my tests:

```
HelloVerticle.start       : 13 | vert.x-worker-thread-0
ByeVerticle.start         : 16 | vert.x-eventloop-thread-1
All verticles deployed
Application.main          : thread: 15 | vert.x-eventloop-thread-0
HelloHandler.handle       : thread: 15 | vert.x-eventloop-thread-0
Sending message to event bus
HelloVerticle.helloHandler: 19
Hello.sayHello: handling reply
Hello.sayHello: thread    : 15 | vert.x-eventloop-thread-0
Hello.sayHello: success
HelloHandler.handleResult : thread: 15 | vert.x-eventloop-thread-0
ByeHandler.handle: thread : 15 | vert.x-eventloop-thread-0
ByeVerticle.byeHandler    : 16
Bye.sayBye: handling reply
Bye.sayBye                : thread: 15 | vert.x-eventloop-thread-0
Bye.sayBye: success
ByeHandler.handleResult   : thread: 15 | vert.x-eventloop-thread-0
```

As usual, you can access the source code in [GitHub][github-vertx-overview], and I hope it makes you interested in learning more about Vert.x!


[vertx]: http://vertx.io/
[interrupt-wikipedia]: https://en.wikipedia.org/wiki/Interrupt
[verticle-docs]: http://vertx.io/docs/apidocs/io/vertx/core/Verticle.html
[github-vertx-overview]: https://github.com/augustoerico/vertx-event-bus-timeout/tree/v1.1
[postman]: https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop
