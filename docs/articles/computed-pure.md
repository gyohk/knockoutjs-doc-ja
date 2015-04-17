# PureComputedObservable

Pure computed observables, introduced in Knockout 3.2.0, provide performance and memory benefits over regular computed observables for most applications. This is because a pure computed observable doesn’t maintain subscriptions to its dependencies when it has no subscribers itself. This feature:

* Prevents memory leaks from computed observables that are no longer referenced in an application but whose dependencies still exist.
* Reduces computation overhead by not re-calculating computed observables whose value isn’t being observed.

A pure computed observable automatically switches between two states based on whether it has `change` subscribers.

1. Whenever it has no `change` subscribers, it is * sleeping * . When entering the sleeping state, it disposes all subscriptions to its dependencies. During this state, it will not subscribe to any observables accessed in the evaluator function (although it does keep track of them). If the computed observable’s value is read while it is sleeping, it is automatically re-evaluated if any of its dependencies have changed.

2. Whenever it has any `change` subscribers, it is awake and * listening * . When entering the listening state, it immediately subscribes to any dependencies. In this state, it operates just like a regular computed observable, as described in [how dependency tracking works](#) .

#### Why “pure”?

We’ve borrowed the term from [pure functions](#) because this feature is generally only applicable for computed observables whose evaluator is a pure function as follows:

1. Evaluating the computed observable should not cause any side effects.
2. The value of the computed observable shouldn’t vary based on the number of evaluations or other “hidden” information. Its value should be based solely on the values of other observables in the application, which for the pure function definition, are considered its parameters.

#### Syntax

The standard method of defining a pure computed observable is to use `ko.pureComputed` :

```javascript
this.fullName = ko.pureComputed(function() {
    return this.firstName() + " " + this.lastName();
}, this);
```

Alternatively, you can use the `pure` option with `ko.computed` :

```javascript
this.fullName = ko.computed(function() {
    return this.firstName() + " " + this.lastName();
}, this, { pure: true });
```

For complete syntax, see the [computed observable reference](#) .

### When to use a pure computed observable {#when-to-use-a-pure-computed-observable}

You can use the pure feature for any computed observable that follows the [pure function guidelines](#). You’ll see the most benefit, though, when it is applied to application designs that involve persistent view models that are used and shared by temporary views and view models. Using pure computed observables in a persistent view model provides computation performance benefits. Using them in temporary view models provides memory management benefits.

In the following example of a simple wizard interface, the `fullName` pure computed is only bound to the view during the final step and so is only updated when that step is active.

# (ここにライブビューが入ります)

#### Source code: View

```html
<div class="log" data-bind="text: computedLog"></div>
<!--ko if: step() == 0-->
    <p>First name: <input data-bind="textInput: firstName" /></p>
<!--/ko-->
<!--ko if: step() == 1-->
    <p>Last name: <input data-bind="textInput: lastName" /></p>
<!--/ko-->
<!--ko if: step() == 2-->
    <div>Prefix: <select data-bind="value: prefix, options: ['Mr.', 'Ms.','Mrs.','Dr.']"></select></div>
    <h2>Hello, <span data-bind="text: fullName"> </span>!</h2>
<!--/ko-->
<p><button type="button" data-bind="click: next">Next</button></p>
```

#### Source code: View model

```javascript
function AppData() {
    this.firstName = ko.observable('John');
    this.lastName = ko.observable('Burns');
    this.prefix = ko.observable('Dr.');
    this.computedLog = ko.observable('Log: ');
    this.fullName = ko.pureComputed(function () {
        var value = this.prefix() + " " + this.firstName() + " " + this.lastName();
        // Normally, you should avoid writing to observables within a pure computed
        // observable (avoiding side effects). But this example is meant to demonstrate
        // its internal workings, and writing a log is a good way to do so.
        this.computedLog(this.computedLog.peek() + value + '; ');
        return value;
    }, this);

    this.step = ko.observable(0);
    this.next = function () {
        this.step(this.step() === 2 ? 0 : this.step()+1);
    };
};
ko.applyBindings(new AppData());
```

### When not to use a pure computed observable {#when-not-to-use-a-pure-computed-observable}

#### Side effects

You should not use the pure feature for a computed observable that is meant to perform an action when its dependencies change. Examples include:

* Using a computed observable to run a callback based on multiple observables.

```javascript
ko.computed(function () {
    var cleanData = ko.toJS(this);
    myDataClient.update(cleanData);
}, this);
```

* In a binding’s init function, using a computed observable to update the bound element.

```javascript
ko.computed({
    read: function () {
        element.title = ko.unwrap(valueAccessor());
    },
    disposeWhenNodeIsRemoved: element
});
```

The reason you shouldn’t use a pure computed if the evaluator has important side effects is simply that the evaluator will not run whenever the computed has no active subscribers (and so is sleeping). If it’s important for the evaluator to always run when dependencies change, use a [regular computed](#) instead.

### State-change notifications {#state-change-notifications}

A pure computed observable notifies an `awake` event (using its current value) whenever it enters the listening state and notifies an `asleep` event (using an `undefined` value) whevener it enter the sleeping state. You won’t normally need to know about the internal state of your computed observables. But since the internal state can correspond to whether the computed observable is bound to the view or not, you might use that information to do some view-model initialization or cleanup.

```javascript
this.someComputedThatWillBeBound = ko.pureComputed(function () {
    ...
}, this);

this.someComputedThatWillBeBound.subscribe(function () {
    // do something when this is bound
}, this, "awake");

this.someComputedThatWillBeBound.subscribe(function () {
    // do something when this is un-bound
}, this, "asleep");
```

(The `awake` event also applies to normal computed observables created with the `deferEvaluation` option.)
