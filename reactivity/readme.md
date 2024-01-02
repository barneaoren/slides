# Main Aspects of Reactivity

Hello, my name is Dmitry Karlovsky and I... flew to you on a turbo-jet plane. The main essence of a jet engine is shown in the picture..

![Reactivity](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-reactivity.svg)

Here, the seemingly chaotic interaction between molecules leads to the fact that the escaping molecules indirectly transfer momentum to the engine body. Well, let's think about how reactive principles solve or, on the contrary, aggravate problems in programming. Let's compare different approaches to reactive programming. And we will bring to the surface all their pitfalls.

This is a text transcript of the speech at [SECON.Weekend Frontend'21](https://vk.com/event207371976). You can [watch the video recording](https://youtu.be/__iGudoQUN8), [read it as an article](https://github.com/nin-jin/slides/tree/master/reactivity) (most current version) , or [open in the presentation interface](https://nin-jin.github.io/slides/reactivity/).

# Reactive-man

Сперва вкратце о себе..
- 🎶 15 years in frontend
-    8 years with reagents
- 😭 I worked on Angular, RXJS and MobX
- ✨ Your own reactive libs with unique features
- 💞 A whole framework based on them ([$mol](https://mol.hyoo.ru/))
  
I played reactivity up and down, and on this basis I caught a bunch of insights, which I will share with you later.

#Attention!

I will try to be as objective as possible, but.. there may be side effects..

- 💥 Burning sensation in the lower back
- 👐Itchy fingertips
- 📢Increasing the volume of the speech apparatus
- 🧠 Increased tension in the gyrus area

I hope you are well refreshed, because the report will be long, intense and in many ways contrary to the usual picture of the world.

# Activities

Let's start from afar. What types of activities are there in our code?

- 🌠Interactivity
- 🚀Reactivity

## 🌠Interactivity

The system has only done what was asked... And is waiting for further commands.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-interactive.svg)

All other parts of the system, if you look at them, are now in an obsolete state. So you need to explicitly go and ask them to update too.

## 🚀Reactivity

The system did what was asked. Plus it updated the entire application itself, since it knows how different states depend on each other.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-reactive.svg)

Now, if you look at any state, it will correspond to the changes made. Although we clearly didn't ask for it.

# Components

Reactivity can significantly reduce the complexity of implementing reliable programs. So let's look at what we need to implement it..

- 📦States
- 🎬Actions
- 💨Reactions
- 💫Invariants
- 🌉Cascade
- 🧙‍♂️Runtime
- 
## Component: 📦States

First of all, we need *states* - containers that store some values.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-states.svg)

## Component: 🎬Actions

States themselves are useless until we can interact with them. So we need *actions* to change them.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-actions.svg)

## Component: 💨Reactions

But changing states without the ability to see them also makes no sense. So we need *reactions* - some procedures that run when the state changes and produce side effects.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-reactions.svg)

## Component: 💫Invariants

If a side effect of the reaction is the update of another state, then we get an *invariant* - a relationship between states that remains unchanged for any changes in these states.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-invariants.svg)

An invariant can be expressed explicitly, such as a formula in a spreadsheet. So it can be assembled in code from other abstractions. For example, as a combination of an event handler, a transformation stream and a side effect. Or, for example, a template that forms the DOM from component parameters.

## Component: 🌉Cascade

The beauty of invariants is that we can tie all application states into a single graph.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-cascade.svg)

Thus, a change in one state will *cascade* affect the entire application automatically. That is, we got that very reactivity. We can use them to tie all the application states into a single graph.

## Component: 🧙Runtime

And for reactivity to finally work, we need some *runtime* that will track changes in some states and update the values ​​of others in accordance with the invariants we have specified.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-runtime.svg)

If you don't understand how it works, then reactivity will look like magic to you. But once you figure it out, it becomes another technology in your arsenal.

# Wishes

Let's formulate what qualities we want to get from reactivity, and which ones, on the contrary, to avoid..

- 🤹‍♂️Reasonability
- 🐵Stability
- 🐘Economy
- 💫Consistency

## Wish: 🤹Reasonability

The extra computation itself gradually slows down the application. But this is half the trouble. Every extra calculation leads to other extra calculations. As a result, unnecessary calculations grow like a snowball.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-snowboll.svg)

Therefore, the sooner we stop them, the less resources we will spend in total. This means we’ll get a more responsive application that consumes less battery.

## Wish: 🐵Stability

After changing the state, the result should be the same as starting from scratch in the same state. Otherwise, the actual behavior of the user may differ from that on which the developer is debugging.
![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-stable.svg)

It sounds self-evident, but you will be horrified when you find out that stability of behavior is almost never guaranteed. As a result, a situation is possible where the programmer took the same code, opened the same windows, entered the same values... but the user has a bug, and the programmer cannot reproduce it. And then the fun debugging begins.

## Wish: 💸Economy

It is important to understand that all this reactivity is not free. In addition to the actual values, it is necessary to store various meta-information, the volume of which can be several times larger.

Let's take V8, for example, and see how much memory different data types require in the most optimistic case, when JIT has optimized everything as much as possible..

| Value    | Place   | Cost
|----------|---------|-----
| Obj      | Heap    | 12
| Array    | Heap    | 24
| Unit     | Inplace | 4
| Int      | Inplace | 4
| Float    | Heap    | 12
| BigInt   | Heap    | 16+
| String   | Heap    | 12+
| Ref      | Inplace | 4
| Closure  | Heap    | 24
| Context  | Heap    | 16

What lies in Heap eats an additional 4 bytes per link (Ref). Unit is all sorts of undefined, null, false, true and other small non-variable primitive values. Int after a billion is stored as a Float, the mantissa of which is 48 bits. Please note that this is already a reference type, like BigInt, which means it consumes an additional 4 bytes per link. The context for the closure is stored only if the function is closed to any variables. The size of the context accordingly depends on the number of these variables. As you can see (Inplace), each variable adds 4 bytes to the context.

It is easy to notice that the objects are relatively cheap. Arrays are already more expensive, because they are actually composite objects. But closures are very expensive things in themselves, even without taking into account the data stored in them.

Let me give you a few examples of calculating memory consumption..
```
function make_ints_state( ... state: number[] ) {
	return { get: ()=> state }
}

const state1 = make_ints_state( 777 )
// Ref + Obj + Ref + Closure + Ref + Context + Ref + Array + Int
// 4   + 12  + 4   + 24      + 4   + 16      + 4   + 24    + 4   = 96


const state2 = { state: 777 }
// Ref + Obj + Int
// 4   + 12  + 4   = 20


const state3 = 777
// Int
// 4
```

To summarize: depending on the selected abstractions, memory consumption can differ by an order of magnitude. And if the difference between 1 and 10 megabytes is not particularly noticeable. The difference between 100 megabytes and a gigabyte will be clearly noticeable. At best, everything will slow down. And in the worst case, the application will simply crash.

A real-life example: we open a 200-page XPath specification in Google Docs and get half a gigabyte of memory consumption.

Or another example: I'm currently working on a new implementation of reactivity. And while I was flying on the plane yesterday, I meditated over this sign. As a result, I figured out how to reduce memory consumption by 2 times, turning an object with two arrays into... just one array. Of course, for this I needed inheritance, which is so unfashionable now.

## Wish: 💫Consistency

And, of course, all application states must be consistent with each other at any given time.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-interactive.svg)

If the user (or other software system) sees a discrepancy, even momentarily, he or she will be discouraged at best. At worst, both you and he will lose money, reputation and other goodies.

# Aspects

Now let's look at various aspects of implementing reactivity that you should pay attention to when choosing architecture, libraries and frameworks.

- **Style**: Code style
- **Watch**: Monitoring changes
- **Dupes**: Equivalent changes
- **Origin**: Recalculation initiator
- **Tonus**: Vigor of reactions
- **Order**: Order of reactions
- **Flow**: Data flow configuration
- **Error**: Emergency situations
- **Cycle**: Cyclic dependencies
- **Depth**: Depth limit
- **Atomic**: Atomicity of changes
- **Extern**: External interactions

## Style

Conventionally, there are 3 styles of writing code.

- 🧐Proc: Procedural
- 🤯Func: Functional
- 🤓Obj: Object

Different libraries can mix them in different proportions, but as a rule there is a clear inclination towards one of them.

### Style: 🧐Proc

Here an update procedure is sporadically launched, which reads some states, calculates others and writes them. Let's write a simple, although not very effective, implementation...

```javascript
let Name = 'Jin'
let Count
let Short

setInterval( ()=> {
	Count = Name.length
} )

setInterval( ()=> {
	Short = Count < 4
} )
```

Invariants are described approximately this way, for example, in Meteor and Angular by default. Of course, they do not start the recalculation every millisecond, but more optimally, but this does not change the general essence: runtime periodically restarts the invariants, not knowing which states can be changed by them. But the actual values ​​of these states may not be interesting to us, but they will be calculated in any case. Therefore, this approach is still not very effective.

### Style: 🤯Func

In the wake of hype, many focus on pure functions, turning their code into a puzzle...

```javascript
const Name = new BehaviorSubject( 'Jin' )

const Count = Name.pipe(
	map( Name => Name.length ),
	distinctUntilChanged(),
	debounceTime(0),
	share(),
)

const Short = Count.pipe(
	map( Count => Count < 4 )
	distinctUntilChanged(),
	debounceTime(0),
	share(),
)
```

Even an experienced *streamer* cannot immediately tell what this RxJS code does and why. But this is the simplest example, far from the real thing.

However, smart programmers love puzzles. So they spend a lot of time studying cunning abstractions that are equally distant from both how a machine works and how the human brain works. They write concise but intricate code. And they are proud that they understand what few others are able to understand. This has a rather negative effect on the project, introducing unnecessary complexity to an area that is already full of difficult things.

I used to write tricky code, too, but life taught me that it’s better to write the simplest code possible, accessible even to a beginner in programming, and not just to winners of computer science Olympiads.

In addition, the abundance of closures inherent in functional code leads to increased memory consumption.

### Style: 🤓Obj

Here the program consists of many objects with states connected by invariants into a single graph. Code in this style looks the same as regular OOP code, but with the addition of reactive memoizers.

```javascript
class State {
	
	@mem Name( next = 'Jin' ) {
		return next
	}
	
	@mem Count() {
		return this.Name().length
	}
	
	@mem Short() {
		return this.Count() < 4
	}
	
}
```

Many people have probably heard the statement that “cache invalidation is one of the most difficult issues in programming.” So, in reactive runtime, such a question does not arise at all.

This approach seems to me to be the most optimal, since it fits well with the way a person thinks (and he is accustomed to interacting with objects) and with the way a computer works (an object is simply a mutable structure in memory). Runtime clearly understands which method calculates which state. And object decomposition makes it all easy to scale. This is why the object style is used in $mol, MobX and Vue.

##Watch

How can runtime know about changes?

- 🔎Polling: Periodic reconciliation
- 🎇Events: Occurrence of an event
- 🤝Links: List of subscribers

### Watch: 🔎Polling

States store only values ​​and that’s it. Runtime periodically checks the current value with the previous one. And if they differ, it triggers reactions.

```javascript
// sometimes
if( state !== state_prev ) reactions()
```

This is how Angular, Svelte, and React work, for example. The problem with this approach is that for every sneeze, a lot of work is done, only to find out that almost nothing has changed.

It may seem to you that ordinary comparison is a trivial operation. And this is true in synthetic benchmarks. But in reality, states are scattered across memory, which results in mediocre use of processor caches. And the cherry on the cake is that such reconciliations have to be performed after each reaction in order to find out what exactly they changed in the state.

### Watch: 🎇Events

Each state additionally stores a list of change handler functions. Every time the state changes, all subscribers are called.

```javascript
// on change
for( const reaction of this.reactions ) {
	reaction()
}
```

This can be initiated manually, through a setter or a proxy. But in any case, the state knows nothing more about neighboring states, and the interaction is always one-way. This greatly limits the possible optimization algorithms. It also complicates debugging, because to find out who depends on whom in what way is a whole quest.

And the saddest thing is that storing an array of closures eats up a lot of memory. And nothing can be done about it.

### Watch: 🤝Links

States store direct references to each other, forming a global graph. Arrays of links are relatively memory-efficient, because each link is only 4-8 bytes. To communicate with neighbors, you just need to run through the array and pull the desired method from the neighboring state.

```javascript
// on change
for( const slave of this.slaves ) {
	slave.obsolete()
}


// on complete
for( const master of this.masters ) {
	master.finalize()
}
```

In the first example, you can see that when one state changes, we tell all dependents that they are out of date. And in the second, when the calculation of one state is completed, we tell all dependencies that the calculation is finished, and the caches that they could hold in case of repeated access can be freed. There can be many such interaction options, which gives maximum flexibility in the supported algorithms.

In addition, when debugging, it is much easier to follow direct links between objects than to extract the necessary information from contexts captured by closures.

## Dupes

Sometimes the value changes to an equivalent one. And here there are different approaches to cutting off degenerate calculations..

- 👯Every: Reaction to every action
- 🆔Identity: Comparison by link
- 🎭Equality: Structural comparison
- 🔀Reconcile: Structural reconciliation

### Dupes: 👯Every

In libraries like RxJS, each value is a unique event, which results in reactions being fired unnecessarily.

```ts
777 != 777
```

To prevent this from happening, you need to write additional code, which is often forgotten and then screwed up.

### Dupes: 🆔Identity

Many libraries can still compare values. And if the state has not changed, then the reactions do not work. And if it has changed, even by an equivalent value, then they work.

```ts
        777 == 777

[ 1, 2, 3 ] != [ 1, 2, 3 ]
```

If we filtered a new array with the same content, then most likely we do not need to run a cascade of calculations. But manually keeping track of all such places is not very realistic.

### Dupes: 🎭Equality

The most advanced libraries, like $mol_atom, do a deep comparison of the new and old value. In some others, such as CellX, it can be enabled in the settings.

```ts
        777 == 777

[ 1, 2, 3 ] == [ 1, 2, 3 ]

[ 1, 2, 3 ] != [ 3, 2, 1 ]
```

This allows you to cut off unnecessary calculations as early as possible - at the time of making changes. And not at the moment of rendering the newly generated VDOM into the real DOM, as often happens in React, in order to find out that there is nothing to change in the house.

Deep comparison is, of course, a more expensive operation in itself than simply comparing two links. However, sooner or later, you will still have to compare all the contents. But it’s much faster to do this while the data is nearby, and not when it scatters across a thousand components during the rendering process.

However, if the data has changed, it will fly further through the application and will be deeply compared again and again, which is no good. Therefore, it is important to implement caching of the result of a deep comparison for each pair of compared objects.

### Dupes: 🔀Reconcile

Finally, you can go even further and not just compare values ​​deeply, but also reconcile them to preserve references to old objects when they are equivalent to new ones.

```ts
const A = { foo: 1, bar: [] }
const B = { foo: 2, bar: [] }

reconcile( A, B )

assert( B.foo === 2 )
assert( B.bar === A.bar )
```

As you can see, `A` and `B` are different here, but the `bar` property remains the same. This is good from the GC's point of view, because we are reusing objects that were in the old generation of garbage collector. And the object from the younger generation was discarded during reconciliation, which was very quick.

In addition, when later the component that renders `bar` is checked again, this will happen extremely quickly, because the old and new values ​​will be identical. On the other hand, if the objects still differ, then the re-verification will again go through all the insides of the object. Here again caching is necessary. But..

Changing the fields of a new object to values ​​from the old one is not always a safe operation. For example, such tricks cannot be performed with DOM elements. At best it won't work, and at worst it will break it altogether. In some cases, you will receive an exception when you try to modify an object. Sometimes changes will simply be ignored, and sometimes setters will be launched that do some weird things.

In addition, if the objects are virtually identical, we need to compare them deeply first. And if they are the same, then return the old one, and if not, start changing the new one. Well, either immediately start changing the new one, only to find out later that we have changed all its properties, so we need to return the old one.

In short, this approach is not the fastest or most reliable, so in $mol we abandoned it in favor of a deep comparison with caching.

##Origin

Despite the fact that it all starts with the fact that someone changed something, the final decision whether to recalculate this or that invariant can take both a dependence and a dependent state.

- 🥌Push: Addiction pushes
- 🚂Pull: The addict pulls

### Origin: 🥌Push

When the dependency changes, reactions are certainly triggered, which calculate and write new values ​​to the dependent states. This is how, for example, RxJS, Effector and other procedural/functional libraries/frameworks work.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-push.svg)

And this works great for a static invariant graph. However, in any not entirely trivial application we have dynamics. Well, it’s trivial: if we switch between pages, then we need to release the resources of the previous page (and in particular unsubscribe from data changes) and seize resources for the new page (and in particular subscribe to data changes).

That is, our graph of invariants must be able to change in the process of recalculating these invariants. This means that when acting on the push principle, we will often find ourselves in situations like this: we spent a long, long time calculating some value, but in the end no one needed it, because the consumer was destroyed.

### Origin: 🚂Pull

When accessing a dependent state, an invariant is calculated, which pulls values ​​from the dependencies and returns the current value. This is how $mol_wire, CellX, MobX and Vue work.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-pull.svg)

Here, purely logically, we always know that if a calculation has occurred, then someone needs its result. And if it is not needed, then the calculation will not occur. Therefore, the delaying approach seems to me more practical.

## Tonus

You can calculate dependent states as early as possible, or as late as possible, even to the point of abandoning calculations, if possible.

- 🍔Instant: Instant reactions
- ⏰Defer: Deferred reactions
-   Lazy: Lazy calculations

### Tonus: 🍔Instant

In libraries such as RxJS, dependent states are recalculated immediately when the dependency changes. If we need to change several states in a row, this may lead to unnecessary calculations.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-instant.svg)

Moreover, these extra intermediate calculations produce an inconsistent state, calculated partly from states that have already been updated, and partly from states that have not yet been updated. And an inconsistent state, even if temporary, is a very dangerous thing. At best, the user will observe *glitches* - visual flickering. At worst, the application will not work correctly and contain various errors.

### Tonus: ⏰Defer

To avoid *glitches* the recalculation can be delayed until later, so that it is performed only once, no matter how many dependencies are updated.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-defer.svg)

However, the recalculation will be carried out in any case, even if the result is not useful to us.

### Tonus:   Lazy

In drag-on reactivity models, it is possible to lazily evaluate invariants - only at the moment when the dependent state is actually needed.****

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-lazy.svg)

When the initial states change, we do not calculate the dependent ones and do not even schedule their calculation, but only mark them as outdated. And if you subsequently access them, they will begin to be calculated.

This is both the most economical approach and the most consistent, since it guarantees that whenever we access the state, the resulting value will be relevant.

## Order

The order of execution of reactions has its own peculiarities, which are often left to random. However, let's look at all the possible options...

- 📰Subscribe: By subscription time
-   Event: By time of event occurrence
- 📶Deep: According to the depth of dependence
- 👨‍💻Code: By position in the program
- 
### Order: 📰Subscribe

Whichever reaction appeared earlier is triggered earlier. In any non-trivial application, the list of reactions changes over time, which means they can be arranged in almost any order.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-order-subscribe.svg)

The result is a hidden state that affects the operation of the application through a different order of side effects, which can give various mutual interferences. We get unstable behavior, which complicates debugging and testing.

This is a typical problem with most libraries.

### Order: 🧨Event

Let's assume we managed to fix the order of subscriptions in one way or another. However, there is another source of instability - the order of actions.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-order-event.svg)

By explicitly or implicitly changing states in different orders, we can again get different orders of reactions. Unfortunately, most libraries are susceptible to this problem.

### Order: 📶Deep

Some libraries use so-called *topological graph sorting* to recalculate invariants in an optimal order from less dependent to more dependent.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-order-deep.svg)

In this example, `Post` is changed to one that we do not have access to. First, the contents of this page will be updated, which will not only lead to unnecessary recalculations, but they may also result in errors or just garbage as side effects. And only then, when calculating `Page`, it will be found out that `PostPage` should be completely destroyed, and instead a `Forbidden` access error message should be displayed.

Note that the existence of `Title` and `Body` depends on the value of `Page`. But the `Title` and `Body` values ​​themselves no longer depend on the `Page` value. Conversely, the value of `Page` is independent of the value of `Title` and `Body`. That is, the connection between them is non-reactive. But it is there. And this is already a connection between owner and property. That is, the `Page` value owns the `Title` and `Body` reactive states, and therefore controls their lifetime.

This problem cannot be solved by analyzing the reactive graph alone. Unless you can supplement it with a possession graph. But this will require even more complexity in the runtime logic. And I'm not sure that topological sorting of such a double graph can be done with acceptable algorithmic complexity. Otherwise, our whole struggle for efficiency will work slower than a much dumber but simpler architecture.

### Order: 👨‍💻Code

It is preferable that reactions are processed in the order that is explicitly specified in the code. This ensures that the owner is updated before everything he owns.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-order-code.svg)

Here, `Allow` will be updated first, then `Page`, which will lead to the loss of `PostPage` and, as a result, the destruction of `PostPage` with all the states inside, without calculating them.

## Flow

In a reactive system, all states are connected to each other by invariants into a single graph. When we change something on one side of this graph, runtime provides a cascade recalculation of dependent states. Such sequences of recalculations are nothing more than *information flows* (data flow). The more straightforward these flows are, the less they branch out and affect states that are irrelevant to changes, the more efficiently the system operates. And here there are two approaches to optimizing information flows..

-   Manual: Manual
- 🚕Auto: Automatic

### Flow: 🦽Manual

In push-based libraries, it is difficult to automate flows, so manual management of them thrives. This means we get two types of errors...

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-manual.svg)

Firstly, we may forget to subscribe to something, resulting in inconsistency. In the example, we forgot to subscribe to `Title`, and when it changes, `Greeting` is not recalculated.

Secondly, we may forget to unsubscribe from something, resulting in unnecessary calculations. In the example, we forgot to unsubscribe from `Name`, and when it changes, `Greeting` is recalculated, but gets the same value.

But, if you can still somehow cope with errors, then it is no longer easy to cope with the complexity of manual optimizations. For banal logical branching, we need to actually implement a transistor by hand, where we have a control flow that switches the output between two inputs. For loops and indirect addressing, everything becomes so complicated that [few people are even able to adequately describe it](https://qna.habr.com/q/427478). In the end, it all comes down to the fact that, instead of point-by-point recalculations, many states are recalculated for any sneeze, which is quite slow.

### Flow: 🚕Auto

Draw-based libraries typically use dependency auto-tracking. Not only is this much more reliable, it is also extremely simple for an application programmer. He does not need to think about data streams at all - they are dynamically configured by runtime in the most optimal (for a given application state) way.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-auto.svg)

Here application programmers are divided into two camps: some are afraid of this “magic” because they don’t understand how it works, while others simply don’t give a damn - it works and works, one less headache.

Well, on the sidelines stands the camp of those who simply know how it works and use this knowledge to benefit. After all, as you know: any sufficiently developed technology is indistinguishable from magic... for the uninitiated person.

Practice shows that (with automation) the application code is orders of magnitude smaller, the code itself is much simpler and more reliable, and the application runs faster.

## Error

Very often programmers do not think about emergency situations. It’s especially sad when these are not application developers, but authors of libraries and frameworks.

For example, when I was preparing this material, I asked in the Effector chat how the system behaves when exceptions occur. To which they answered that there should be no exceptions in pure functions (exceptions, by the way, do not actually contradict purity, but that’s another story) and if you allowed them, then you yourself are a fool. When I asked whether the author even knew how his library would behave in an emergency situation, I was accused of being toxic and banned.

Well, we digress. There shouldn't be bugs either, like other bad things in life, however, they sometimes happen. The fault of the programmer, the browser, its extensions, the operating system, the stellar wind - it doesn’t matter. You have to be able to take a punch and not bury your head in the sand. But in different libraries we will encounter no matter what kind of behavior...

- 🎲Unstable: Unstable operation
- ⛔Stop: Stop work
-   Store: Error indication and waiting for recovery
- ⏮Revert: Revert to a stable state

### Error: 🎲Unstable

Often, in the event of an exception, the application goes into an inconsistent state, resulting in unstable operation.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-error-unstable.svg)

In the example, let's say an incorrect `codepoint` has crept into the name. And, let's say, an attempt to take the length of a string leads in this case to an exception. The example is quite synthetic, later I will show more realistic ones, but for now that’s it.

And so, when calculating the invariant, an exception occurred, which is why the runtime did not update `Count`. As a result, all states split into 2 subgraphs, which themselves are consistent, but are no longer consistent with each other.

### Error: ⛔Stop

An equally strange solution is to simply stop functioning, as, for example, RxJS does. If an exception occurs anywhere in the stream, then all streams after it are finalized and will never work again.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-error-stop.svg)

This strategy is suitable for a single task - you either do it or fail with an error. But reactivity is for long-lived systems that constantly display something. This means that if reactivity fails, the application will simply break without signaling this to the user.

Moreover, it will only break by half. And to restore operation, you will need to restart either the entire application, or at least this half.

An incident from life: A crowd of athletes moved in with me on the hotel floor. And these are guys with 100 kg of pure meat. And so, we crammed into the elevator with them this morning, which predictably led to overload. The elevator raised its paws and said "that's it."

Well, okay, the couple left - nothing happens. Ok, another half came out - still nothing. So, everyone got out - the elevator still didn’t work. And we all had to do a run up the stairs today. I think the software for this elevator was written in RxJS, no less.

### Error: ⏮Revert

Libraries like reatom, in principle, do not allow inconsistency by recalculating invariants within a transaction. So if something happens, all states are rolled back to the last consistent one.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-error-revert.svg)

Formally it doesn't sound bad. But for the user this is terrible behavior, because because of one black sheep somewhere in the corner of the application, which constantly throws exceptions, our entire application comes to a standstill and does not react in any way to the user’s actions. Or simply - it hangs tightly. Which is no good.

### Error: 🦺Store

It is much more practical to consider the error as a possible result of the calculation, along with the return value.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-error-store.svg)

Here, all states that depend on the incorrect one are also marked as incorrect. And the rendering system can automatically show a failure indicator for parts of the application that fail to update. Well, or you can catch the exception and draw your own beautiful message. In any case, the user will understand what is happening and that the faulty part of the application should not be relied upon. But you can continue to use other parts.

Even though part of the application is broken, the state of the application is still consistent. Because the error message at the output is precisely consistent with the incorrect value at the input.

At the same time, eliminating the cause of the failure will automatically restore the correct operation of this part of the application. Without additional movements on the part of the programmer!

## Cycle

Sometimes we can end up with cyclical dependencies. Sometimes we may want to do them intentionally. For example, when implementing a converter between degrees Celsius and Fahrenheit, where the user can change any of the two values, and the second should be recalculated automatically.

However, in the vast majority of cases, cyclic dependencies indicate a problem with logic, so they are usually avoided. Fortunately, the logic of even a degree converter can always be rewritten so that there are no cyclic dependencies.

So let's look at how different systems react to this emergency situation.

- 🚫Unreal: Impossible
- 💤Infinite: Endless loop
- 🎰Limbo: Arbitrary result
- 🌋Fail: Causes an error
### Cycle: 🚫Unreal

It's quite tempting to think of making it syntactically impossible to create loops. For example, we can require, when creating a state, that all of its dependencies already exist. This is typically the case with push libraries.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-cycle-unreal.svg)

It doesn't sound bad. However, we threw out the baby with the bathwater. That is, we have extremely limited ourselves in what kind of logic of invariants we are able to describe. In particular, this practically puts an end to the dynamic configuration of data streams. For example, it will no longer be possible to implement a spreadsheet on such an architecture.

### Cycle: 💤Infinite

A number of libraries simply go into an endless loop, constantly updating the same states.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-cycle-allow.svg)

For Angular and React, for example, this is typical behavior. There is even a crutch - a limit on the number of recalculations of one invariant. But we'll talk about this later.

### Cycle: 🎰Limbo

There is also a very strange solution - when indirectly accessing the state that is currently being calculated, its previous value is used.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-cycle-limbo.svg)

Depending on the order of calculations, this approach gives different results. That is, not only does the state turn out to be inconsistent, but also the behavior of the application becomes not stable, but begins to depend on the weather on Mars.

### Cycle: 🌋Fail

The best solution is to detect the loop at runtime and throw an exception.

![](https://raw.githubusercontent.com/nin-jin/slides/master/reactivity/reactivity-cycle-fail.svg)

Further processing proceeds in the same way as with any other emergency situations. So it is especially important here that the system handles exceptions correctly.

## Depth

As a rule, the depth of dependencies remains relatively small, not exceeding a couple of dozen states.

But sometimes addictions can grow to indecent depths. This is especially true for applications where the user himself can control who depends on whom and how. Typical examples: spreadsheet or Gantt chart.

And not all reactivity models will allow you to implement this. And sometimes you only find out about it when it’s too late to change horses. And the crutching begins. So let's take a closer look at this aspect...

- 🚧Limit: Limited by a constant
- 🗻Stack: Limited by stack
- 🌌Heap: Unlimited

### Depth: 🚧Limit

Some libraries deal with circular dependencies by introducing a limit on the number of recalculations at a time. Usually this is a dozen or two recounts.

```ts
for( let i = 0; i < MAX_REPEATS; ++i ) {
	if( !dirty ) return
	changeDetection()
}

throw new Error( 'Too many change detection repeats' )
```

This prevents the application from freezing completely. But it also fundamentally limits the depth of dependencies. It will be painful to implement a spreadsheet in such conditions.

### Depth: 🗻Stack

The situation is slightly better with reactivity models, where there are no artificial restrictions. However, they initiate some calculations inside others, which leads to the growth of the stack.

```ts
first() {
	this.second()
}

second() {
	this.third()
}

thisrd() {
	this.etc()
}
```

And since the stack size is not infinite, it is only sufficient for a depth of several thousand states. This may already be enough for even average spreadsheets. However, if you go beyond the stack, and that’s it, an exception is thrown.

Moreover, it may or may not fly out, depending on the order in which the recalculations took place. That is, we also get instability of behavior. For example, this can manifest itself like this: when you open an application, everything is fine, but as soon as you change one state, the recalculation of a state that is deeply dependent on it drops.

However, the advantage of this approach is that the stack shows in what order the recalculation was performed, which can be useful for debugging.

### Depth: 🌌Heap

The best option does not grow the stack, which allows it to work with dependencies of arbitrary depth. Well, how much RAM is enough, of course.

```ts
while( reactions.length ) {
	reactions.shift().execute()
}
```

Unfortunately, here stack traces become less informative. But logging can come to the rescue when debugging, which, if desired, can even be pasted into the stack trace manually.

## Atomic

So far we have talked about emergency situations when calculating invariants. However, they can also arise on approach - during changes to several initial states at the same time. Let's look at what could go wrong here...

- 👻Alone: ​​One separate state
-   Base: For primary states
- 🤼Full: For all conditions

### Atomic: 👻Alone

As a rule, a change in one state is atomic everywhere. That is, it will either happen or it won’t happen. Let's consider a simple example: we need to update two states, but after updating the first one, an abnormal situation arose..

```ts
Name = 'John'
Count = 4

Name = 'Jin'
throw 'function is not a function'
Count = 3 // still 4
```

The result is an inconsistent application state. After all, one state has been updated, but the second has not.

You can work around this problem by storing both values ​​in the same state. But this is not always possible.

### Atomic: 🦶Base

It's good if the runtime supports transactions. They guarantee that either all initial states will receive their updates, and dependent states will be updated, or no one will change, and dependent states will not be updated either.

```ts
Name = 'John'
Count = 4

@transaction update() {
	this.Name = 'Jin' // will still 'John'
	throw 'function is not a function'
	this.Count = 3
}
```

### Atomic: 🤼Full

In some libraries, a transaction can be rolled back not only by exceptions that occurred directly when changes were made, but also by exceptions in invariants that were calculated as a result of the changes made.

```ts
Name = 'John'
Count = 4

@derived get Greeting() {
	// Fails on 'Jin' name
	return this.Name.split('')[3].toUppercase()
}

@transaction update() {
	this.Name = 'Jin' // will still 'John'
	this.Count = 3 // will still 4
}
```

In the example, we have a secondary state `Greeting`, which, with a short name, throws an exception and cannot be calculated. Runtime, seeing this, rolls back the entire transaction. As a result, we again get a situation where one crooked view somewhere in the corner of the application does not allow us to update the model and the entire application comes to a standstill.

## Extern

Sometimes an invariant requires asynchronous communication. For example, for heavy calculations in a separate worker. Most reactive libraries do not support asynchronous invariants, but there are some that do. Let's consider both options..

- 🏊Sync: Synchronous invariants
- 🏇Async: (A) synchronous invariants
  
### Extern: 🏊Sync

If only synchronous reactivity is supported, and we need to perform some kind of asynchronous call, then it usually goes somewhere to the side. Let's take a simple example using RxJS...

```ts
const image = source_element.pipe( map( capture ) )
const data = image.pipe( map( recognize ) )
const text = data.pipe( map( data => data.text ) )

text.subscribe( text => {
	output.innerText = text
} )
```

The `capture` and `recognize` functions are asynchronous, since the first one needs to wait for the image to load, and the second one launches neurons on a pool of workers. When we change `source_element`, `output.innerText` will not change at all. That is, the states will no longer be consistent. And they will come to consistency only when all asynchronous operations are completed.

This problem is usually solved by interactively setting some `isLoading` flag at the beginning and interactively resetting it at the end. And when this flag is raised, the waiting indicator is drawn reactively.

Not only is this a routine, but it is also often prone to bugs when several tasks being performed are tied to one indicator. Which, with interactive logic, can cause a so-called *race condition*.

### Extern: 🏇Async

If asynchronous invariants are also supported, then runtime maintains consistency automatically. A typical solution is through a mechanism for dealing with emergency situations. Let's write what the code might look like using, for example, generators..

```ts
@computed
text*() {
	const image = yield capture( this.source_element )
	const data = yield recognize( image )
	return data.text
}
```

Why not asynchronous functions? Yes, because they are made in JS through the ass. So the authors of libraries have to rely on generators that are made through the opposite place, but also not through what they should.

In fact, you can even do without generators. $mol, Vue and React support SuspenseAPI, which allows you to write pseudo-synchronous code and not have to worry about `yield` and `await`. Well, it doesn’t matter, the generators for my story will be clearer.

When runtime calls the `text` generator, it is given a promise instead of a string. It understands that the final result will come later, subscribes to the finalization of the promise, and in the meantime marks the state as “pending value.” This wait flag applies to all dependent states. And the rendering system, seeing this, automatically draws a waiting indicator. Cool!

# Practicality

Let's now take what we know about reactivity and try to formulate what the most practical model of reactivity might look like. What properties should it have to make it pleasant to use, so that it gives us a minimum of problems, so that the user has everything stable, fast, and always understands what is going on..

| Aspect  | ✅Usable   | ❌Unusable
|---------|------------|---------
| Style   | 🤓Obj      | 🧐Proc 🤯Func
| Watch   | 🤝Links    | 🔎Polling 🎇Events
| Dupes   | 🎭Equality | 🆔Identity 👯‍♀️Every 🔀Reconcile
| Origin  | 🚂Pull     | 🥌Push
| Order   | 👨‍💻Code     | 📰Subscribe 🧨Event 📶Deep
| Flow    | 🚕Auto     | 🦽Manual 

| Aspect  | ✅Usable   | ❌Unusable
|---------|-------------|---------
| Tonus   | 🦥Lazy     | 🍔Instant ⏰Defer
| Error   | 🦺Store    | ⛔Stop ⏮Revert 🎲Unstable
| Cycle   | 🌋Fail     | 💤Infinite 🎰Limbo 🚫Unreal
| Depth   | 🌌Heap     | 🗻Stack 🚧Limit
| Atomic  | 🦶Base     | 🤼‍♂️Full 👻Alone
| Extern  | 🏇Async    | 🏊‍♂️Sync

Let's now take various well-known libraries and frameworks and see how close they are to ideal... But first, a small note...

#Default

Below we consider only the default behavior and the code style recommended by the author. It is clear that you can always somehow get around the problems. Somewhere the behavior can be changed with a config parameter. Somewhere you need to remember to write additional code here and there. Somewhere you need to get creative with hellish crutches. And in some cases you will have to abandon one library altogether and add another one on the side.

However, it is important to understand that the author of the library, even if he is deeply wrong, most likely has greater expertise than an ordinary application developer. In this topic in general, and in my library in particular. Therefore, most third-party code using it will be written in the canonical style, designed for default behavior. And any departure from the default will require additional code, which you must remember to write, and spend time writing it correctly.

- 🎓 Expert's choice
- 🐭 Minimum code
- 👀 Increased attention
- 👾 Third party code
  
Further comparison is useful not so much in order to understand which one should be taken urgently and which one should be thrown away immediately. But also in order to understand what you need to be prepared for when starting a project on a particular technology.

Some aspects may be completely unimportant to you. Some may turn out to be *show stoppers*. And some can be easily bypassed. And it would be nice to lay some straws in advance so as not to have to deal with painful debugging and optimization later.

# Reactive Libraries

| Lib                                                                | Style | Watch | Dupes | Origin | Tonus | Order | Flow  | Error | Cycle | Depth | Atomic | Extern 
|--------------------------------------------------------------------|-------|-------|-------|--------|-------|-------|-------|-------|-------|-------|--------|--------
| [$mol_wire](https://github.com/hyoo-ru/mam_mol/tree/master/wire)   | 🤓✅ | 🤝✅ | 🎭✅ | 🚂✅  | 🦥✅ | 👨‍💻✅ | 🚕✅ | 🦺✅ | 🌋✅ | 🗻❌ | 👻❌  | 🏇✅
| [CellX](https://github.com/Riim/cellx)                             | 🤓✅ | 🤝✅ | 🆔❌ | 🚂✅  | 🦥✅ | 👨‍💻✅ | 🚕✅ | 🦺✅ | 🌋✅ | 🗻❌ | 🦶✅  | 🏇✅
| [MobX](https://mobx.js.org/)                                       | 🤓✅ | 🤝✅ | 🆔❌ | 🚂✅  | 🦥✅ | 👨‍💻✅ | 🚕✅ | 🦺✅ | 🌋✅ | 🗻❌ | 👻❌  | 🏊‍♂️❌
| [ChronoGraph](https://www.npmjs.com/package/@bryntum/chronograph)  | 🧐❌ | 🤝✅ | 🆔❌ | 🚂✅  | ⏰❌ | 👨‍💻✅ | 🚕✅ | ⏮❌ | 🌋✅ | 🌌✅ | 🤼‍♂️❌  | 🏊‍♂️❌
| [Reatom](https://reatom.js.org/)                                   | 🤯❌ | 🤝✅ | 🆔❌ | 🚂✅  | 🦥✅ | 🧨❌ | 🦽❌ | ⏮❌ | 🎰❌ | 🗻❌ | 🤼‍♂️❌  | 🏊‍♂️❌
| [Effector](https://effector.dev/)                                  | 🤯❌ | 🤝✅ | 🆔❌ | 🥌❌  | 🍔❌ | 📰❌ | 🦽❌ | 🎲❌ | 💤❌ | 🌌✅ | 👻❌  | 🏊‍♂️❌
| [RxJS](https://rxjs.dev/)                                          | 🤯❌ | 🤝✅ | 👯‍♀️❌ | 🥌❌  | 🍔❌ | 📰❌ | 🦽❌ | ⛔❌ | 🚫❌ | 🗻❌ | 👻❌  | 🏊‍♂️❌

There are two main camps here: “Object Reactive Programming” and “Functional Reactive Programming”. As you can see, the currently fashionable functional approach is not very practical, unlike the more old-school object approach.

It’s also worth noting that RxJS itself is not about reactivity. It is fundamentally about controlling the flow of execution. However, with its help it is possible to describe invariants connecting states, and then we get a reactive system.

Many thanks to the library authors for their help in preparing this table. Write to me if you want to add yours to the comparison. I will try to keep this sign up to date, if the community, of course, helps me keep track of all the news.

# $mol_wire

The leader, of course, turned out to be $mol_wire, which I implemented after this analysis. There is a detailed article about why it is good and why it works the way it does, which I highly recommend reading, because at the end there will be some pleasant surprises waiting for you..

> [Designing an ideal reactivity system](https://mol.hyoo.ru/#!section=articles/author=nin-jin/repo=HabHub/article=48)

Its predecessor `$mol_atom2` had all the same qualities, so in the $mol framework we quite quickly completely moved to the new implementation of reactivity. And by the way, about frameworks...

# Reactive Frameworks

| Lib                               | Style | Watch | Dupes | Origin | Tonus | Order | Flow  | Error | Cycle | Depth | Atomic | Extern 
|-----------------------------------|-------|-------|-------|--------|-------|-------|-------|-------|-------|-------|--------|--------
| [Vue](https://vuejs.org/)         | 🤓✅ | 🤝✅ | 🆔❌ | 🚂✅  | 🦥✅ | 👨‍💻✅ | 🚕✅ | 🦺✅ | 🎰❌ | 🗻❌ | 👻❌  | 🏊‍♂️❌
| [Ember](https://emberjs.com/)     | 🤓✅ | 🔎❌ | 🆔❌ | 🚂✅  | 🦥✅ | 👨‍💻✅ | 🚕✅ | 🎲❌ | 🌋✅ | 🚧❌ | 👻❌  | 🏊‍♂️❌
| [Solid](https://www.solidjs.com/) | 🧐❌ | 🤝✅ | 🆔❌ | 🚂✅  | 🍔❌ | 👨‍💻✅ | 🚕✅ | ⛔❌ | 🚫❌ | 🗻❌ | 👻❌  | 🏊‍♂️❌
| [React](https://reactjs.org/)     | 🧐❌ | 🔎❌ | 👯‍❌ | 🥌❌  | ⏰❌ | 👨‍💻✅ | 🦽❌ | ⛔❌ | 🌋✅ | 🚧❌ | 👻❌  | 🏇✅
| [Angular](https://angular.io/)    | 🧐❌ | 🔎❌ | 🆔❌ | 🥌❌  | ⏰❌ | 👨‍💻✅ | 🚕✅ | 🎲❌ | 🎰❌ | 🚧❌ | 👻❌  | 🏊‍♂️❌
| [Svelte](https://svelte.dev/)     | 🧐❌ | 🔎❌ | 🆔❌ | 🥌❌  | ⏰❌ | 👨‍💻✅ | 🚕✅ | ⛔❌ | 🚫❌ | 🌌✅ | 👻❌  | 🏊‍♂️❌

Analysis of frameworks in terms of reactivity is somewhat arbitrary. It usually manifests itself in two aspects: invariants between the states of one component, and the connection between the state of one component and the parameters of another.

As you can see, proceduralism is more popular here, which is also not the most practical approach. And the most practical thing here is object-based Vue. Only [$mol](https://mol.hyoo.ru/) is cooler than it, but there is no point in considering it separately as a framework, because it simply uses the $mol_wire library as a circulatory system, and we have already analyzed it earlier.

It is important to note that you should not blindly trust these plates, because they are compiled manually. I, of course, tried to reflect everything as accurately as possible, but I could have messed up somewhere. That's why..

# More about Reactivity

- [state-management-specification](https://github.com/artalar/state-management-specification) / Artyom Harutyunyan
- [What Makes a Good Reactive System?](https://www.pzuraq.com/what-makes-a-good-reactive-system/) / Chris Garrett
- [A General Theory of Reactivity](https://github.com/kriskowal/gtor) / Kris Kowal
- [A Hands-on Introduction to Fine-Grained Reactivity](https://dev.to/ryansolid/a-hands-on-introduction-to-fine-grained-reactivity-3ndf)
- [Object Reactive Programming](https://github.com/nin-jin/slides/tree/master/orp) / Me @ FrontendConf'17
- [Quantum mechanics of calculations in JS](https://github.com/nin-jin/slides/tree/master/fibers) / Me @ HolyJS'18

Artyom (author of Reatom) has an interesting project to classify state managers using tests. This is a slightly broader topic, since, for example, Redux is a state manager, but it is not reactive. This is just a transactional change in the state tree and that’s it, no cascading invariants between states. If you are interested in this topic, then get involved in writing tests - it will be useful for the entire community.

In a series of articles from Chris Garrett, you can see how the reactive model was rethought in the Ember framework. I have left a link to the most relevant one.

An in-depth article by Kris Kowal looks at the issue of reactivity from a different perspective. In my opinion, he is wrong, but you can read it to broaden your horizons. But the one I agree with is the author of SolidJS - Ryan Carniato, who talks about how to make a practical reactive system.

Finally: a couple of my speeches examining the advantages of ORP and the mechanics of implementing asynchronous invariants will help to delve even deeper into the topic.

# After Party

For those who have reached the end, but are not yet tired, I can also suggest watching the discussion about state managers that unfolded after the speech between me and Sergei Sova, Effector’s maintainer..

[![About state managers: Dmitry Karlovsky VS Sergey Sova](https://www.youtube.com/embed/NhPjUO6trGY)](https://www.youtube.com/watch?v=NhPjUO6trGY)

# Pre Party

If this is not enough for you, then I invite you to last year’s discussion about state managers in a more expanded format..

[![State Management Talks](https://www.youtube.com/embed/cUSyJk6k2rk)](https://www.youtube.com/watch?v=cUSyJk6k2rk)

# More from Me

For 10 years now I have been actively sharing knowledge, ideas, and open source. And each of my materials is an original, and sometimes radical, idea, tested in practice. So it's worth checking out all of this, even if you don't plan to use it.

- [slides.hyoo.ru](https://slides.hyoo.ru/) - my performances (10+)
- [Core Dump](https://www.youtube.com/channel/UC-qEImMrqSLZ9KLee1JTcuw) - video about fundamental (4+)
- [habhub.hyoo.ru](https://habhub.hyoo.ru/) - my articles (40+)
- [`_jin_nin_`](https://twitter.com/_jin_nin_) - development news (>9000)

# Contribution

This is the second year that I have stopped working, but am engaged in computer science and super-source projects. So if you find my work useful, you can thank me by donating or even patronizing me..

- [yasobe.ru/na/mol](http://yasobe.ru/na/mol) - one-time thank you
- [boosty.to/hyoo](https://boosty.to/hyoo) - constant support
- [hyoo-ru/hyoo.ru/wiki](https://github.com/hyoo-ru/hyoo.ru/wiki/%D0%9E%D0%B1%D0%B7%D0%BE%D1% 80-%D0%BD%D0%B0%D1%88%D0%B8%D1%85-%D0%BF%D1%80%D0%BE%D0%B4%D1%83%D0%BA%D1% 82%D0%BE%D0%B2) - our open projects
  
We, in the $hyoo guild, have many interesting projects that should soon change the world. So the most valuable contribution you can make is not even money, but participation in projects.

We want to create an ecosystem of tightly integrated web services using the most advanced and cheap technologies. And to displace the open source of the current Internet giants. Join our little revolution!

You can also invite me to conduct a seminar in your company. Not only about reactivity, of course. I have something to say on many issues. And not superficially, but with deep analysis.

#LastWords

That's all for sure now. Thank you for your attention. I hope this analysis was useful to you.

![](https://habrastorage.org/webt/um/jg/hz/umjghz4nc3jqzxx5morka4jm-58.jpeg)

And now, let’s boost our jet engines and fly into a bright future!

## First Feedback

- It was interesting to watch the dispute between the lords, from which the observer could derive informational benefit.
- Informative and detailed.
- Interesting and intelligible.
“I myself have been interested in this topic for many years as a hobby, but here a person was thoroughly involved in this and covered the topic in detail.

## Last Feedback

- [Habr](https://habr.com/ru/company/timeweb/blog/586450/comments/)
- [DEV](https://dev.to/ninjin/main-aspects-of-reactivity-58co)
- [andrey_sitnik](https://twitter.com/andrey_sitnik/status/1460956157854752768)
- [_sergeysova](https://twitter.com/_sergeysova/status/1461729153359745032)




