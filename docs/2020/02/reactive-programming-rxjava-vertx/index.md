# Reactive Programming, Rxjava and Vert.x


## Reactive Programming

https://medium.com/@mahdichtioui/reactivex-reactive-programming-principles-dbb1bafa8384

**Reactive programming is programming with *asynchronous data streams*.**

When using reactive programming, data streams are going to be the spine of your
application. Events, messages, calls, and even failures are going to be conveyed by a
data stream. With reactive programming, you observe these streams and **react** when a
value is emitted.

## Rxjava

**Reactive eXtension** (http://reactivex.io, aka. RX) is an implementation of the reactive
programming principles to “*compose asynchronous and event-based programs by using
observable sequence*”. With RX, your code creates and subscribes to data streams named
**Observables**. While Reactive Programming is about the concepts, RX provides you an
amazing toolbox. By combining the **observer** and **iterator patterns** and **functional
idioms**, RX gives you superpowers. You have an arsenal of functions to combine, merge,
filter, transform and create the data streams. The next picture illustrates the usage of
RX in Java (using https://github.com/ReactiveX/RxJava).

**RxJava** is a popular library for creating asynchronous and event-based programs, it
takes inspiration from the main ideas brought forward by the **Reactive Extensions** initiative.

## Vert.x

**Vert.x**, a project under Eclipse‘s umbrella, offers several components designed from
the ground up to fully leverage the reactive paradigm.

Used together with **Rxjava**, they could prove to be a valid foundation for any Java 
program that needs to be reactive.

```java
FileSystem fileSystem = vertx.fileSystem();
HttpClient httpClient = vertx.createHttpClient();
```

`Vert.x's FileSystem` gives access to the file system in a reactive way, while `Vert.x's
HttpClient` does the same for HTTP.
