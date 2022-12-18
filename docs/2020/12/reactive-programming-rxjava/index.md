# Reactive Programming, RX and RxJava


## Reactive programming

Reactive means **acting in response** to a situation rather than **creating** or **controlling** it: reacting to it. Reactive programming is actually similar to this definition, meaning writing code that **reacts to changes**.

It’s actually a way of coding with **asynchronous data streams** that will make it easier for us to code apps and interfaces that respond dynamically to changes in data. Meaning that whenever we have changes in the data, our app will respond reactively to those changes.

In programming, **data streams** are a sequence of data elements made available over time which can be accessed in sequential order. In other words, **a flow of data ordered in time**.

So, reactive programming is way of coding **event-based **programs with **data streams asynchronously**.

## ReactiveX or Rx

Reactive eXtension is a **library** for **event-based** and **asynchronous** programming by using **data streams** sequences.

Rx abstracts away concerns about low-level threading, synchronization, thread-safety and much more and it’s a non-blocking I/O: as the tasks runs on **the background thread**, this will allow us to perform smooth and continuous user experience tasks **without blocking the main thread**.

In Rx, everything is a data stream, which can be observed, and whenever a value is emitted to the stream some actions will take place. **Those data streams can be anything: events (for examples click events), http calls, variable changes … and even errors.**

We will be creating data streams all the time, which we can **observe**, **iterate** over and make some operations on those data using **operators** like filter, transform, select, combine…

- Observe: observer pattern.
- Iterate: iterator pattern.
- Operations: functional programming.

## Why ReactiveX?

Rx will enhance the **performance** and specially the **user experience** of our applications so that our users **won’t be blocked** waiting for results and have a nice and smooth running application.

## Observables and Observers in Rx

![](https://miro.medium.com/max/700/1*WlUfDpwXYdT7RVh0Ds1eyw.png)

```java
Integer[] numbers = {10,20,30};

// Create observable of integers
Observable<Integer> observable = Observable.from(numbers);

// Subscribe to the created observable
Subscription subscription = observable.subscribe(new Observer<Integer>() {

  @Override
  public void onNext(Integer value){
    System.out.println("Current value: ", value);
  }
  
  @Override
  public void onError(Throwable e){
    System.out.println("Error occured: ", e);
  }

  @Override
  public void onCompleted(){
    System.out.println("Completed");
  }

};

// Unsubscribing the subscription
subscription.unsubscribe();
```

## Single
A Single is something like an Observable, but it emits only either one value or an error. When subscribing, it only uses 2 methods instead of 3 (`onNext`, `onError`, `onCompleted`). Those 2 methods are:
- onSuccess
- onError

A Single can call only one method either `onSuccess` or `onError`.

## Subject

A subject is special type of reactive that can be an observable and an observer. It can emit values or events, accept subscriptions and add new elements into the sequence.

- AsyncSubject
- BehaviorSubject
- PublishSubject
- ReplaySubject

## Hot and Cold Observables

- **Cold** observables are those whom **data are created inside them**, and they begin emitting values only when an observer **subscribes** to those type of observables meaning it **emits values only when requested**. Some use cases examples: database queries, web services, reading or downloading files...

    The data are produced **inside** of the observables so it’s **not shared** between subscribers, since each subscriber have its own private source.

    `Of`, `From`, `Range`, `Interval`, `Timer` operators create cold observables.

- **Hot** observables are those who start emitting values whether the observer is ready or not. Meaning that these observables start emitting values before the subscription is made it does not wait for observers to subscribe (No control on emission rate).

    The data is produced **outside** of the observable, for example: mouse and keyboard events, UI events, system events, time...

    `fromEvent()` operator creates hot observables.

    Those type of observables are able to **share** data between multiple observers and the subscription has no side effects.


## Operators

Operate on an observable and return an observable.

Categories:

- Creating: `Create`, `From`, `Interval`, `Just`.
- Transforming: `Map`, `FlatMap`, `Scan`.
- Filtering: `Debounce`, `Filter`, `Skip`, `Take`.
- Combining: `Merge`, `Join`, `Switch`, `Zip`.
- Error: `Catch`, `Retry`.
- Utility: `Delay`, `Do`, `Subscribe`, `SubscribeOn`, `ObserveOn`.

More detail about operators: http://reactivex.io/documentation/operators.html

```java
Observable<Integer> observable = Observable.from(new Integer[] {3,6,8,1});

observable
  .filter(i -> i<4)
  .map(i -> i+1)
  .subscribe(System.out::print);
```

## Schedulers

Rx is library for asynchronous programming, and by asynchronous we mean executing multiple code blocks simultaneously, each code block will runs on its own thread.

So we need to manage these thread to get a **better performance** and prevent problems like memory leaks, and that’s where schedulers comes in handy.

Schedulers in Rx are the ones responsible for telling the observables and observers, **on which thread they should run**. In order to do so Rx introduce 2 main operators to manage these threads:

- `SubscribeOn()`: allow you to specify on which thread the observable should operate.
- `ObserveOn()`: Specify on which scheduler an observer should observe the observable.

Note that if you don’t specify which thread to execute the operators on, by default, it keeps running on the same thread in which the subscription is created.

```java
postService.getPosts()
  .observeOn(AndroidSchedulers.mainThread())
  .subscribeOn(Schedulers.io())
  .subscribe({ result ->
       handleResponse(result)
     }, 
     { throwable ->
       throwable.printStackTrace()
   })
```
