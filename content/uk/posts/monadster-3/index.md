---
layout: post
title: "Refactoring the Monadster"
description: "Dr Frankenfunctor and the Monadster, part 3"
date: 2015-07-09
categories:
  - Partial Application
  - Currying
  - Combinators
seriesId: "Handling State"
seriesOrder: 3
---

*UPDATE: [Slides and video from my talk on this topic](/monadster/)*

*Warning! This post contains gruesome topics, strained analogies, discussion of monads*

Welcome to the third installment in the gripping tale of Dr Frankenfunctor and the Monadster!

We saw [in the first installment](/posts/monadster/) how Dr Frankenfunctor created life out of dead body parts using "Monadster part generators" (or "M"s for short), that would, on being supplied with some vital force, return a live body part.

We also saw how the leg and arms of the creature were created, and how these M-values could be processed and combined using `mapM` and `map2M`.

In [the second installment](/posts/monadster-2/) we learned how the head, heart and body were built using other powerful techniques such as `returnM`, `bindM` and `applyM`.

In this last installment, we'll review all the techniques used, refactor the code, and compare Dr Frankenfunctor's techniques to the modern-day state monad.

Links for the complete series:

* [Part 1 - Dr Frankenfunctor and the Monadster](/posts/monadster/)
* [Part 2 - Completing the body](/posts/monadster-2/)
* [Part 3 - Review and refactoring](/posts/monadster-3/) (*this post*)

## Review of the the techniques used

Before we refactor, let's review all the techniques we have used.

### The `M<BodyPart>` type

We couldn't create an actual live body part until the vital force was available, yet we wanted to manipulate them, combine them, etc. *before* the lightning struck. We did this by creating a type `M` that wrapped a "become alive" function for each part. We could then think of `M<BodyPart>` as a *recipe* or *instructions* for creating a `BodyPart` when the time came.

The definition of `M` was:

```fsharp
type M<'a> = M of (VitalForce -> 'a * VitalForce)
```

### mapM

Next, we wanted to transform the contents of an `M` without using any vital force. In our specific case, we wanted to turn a broken arm recipe (`M<BrokenLeftArm>`) into a unbroken arm recipe (`M<LeftArm>`). The solution was to implement a function `mapM` that took a normal function `'a -> 'b` and turned it into a `M<'a> -> M<'b>` function.

The signature of `mapM` was:

```fsharp
val mapM : f:('a -> 'b) -> M<'a> -> M<'b>
```

### map2M

We also wanted to combine two M-recipes to make a new one. In that particular case, it was combining an upper arm (`M<UpperRightArm>`) and lower arm (`M<LowerRightArm>`) into a whole arm (`M<RightArm>`). The solution was `map2M`.

The signature of `map2M` was:

```fsharp
val map2M : f:('a -> 'b -> 'c) -> M<'a> -> M<'b> -> M<'c>
```

### returnM

Another challenge was to lift a normal value into the world of M-recipes directly, without using any vital force. In that particular case, it was turning a `Skull` into an `M<Skull>` so it could be used with `map2M` to make a whole head.

The signature of `returnM` was:

```fsharp
val returnM : 'a -> M<'a>
```

### Monadic functions

We created many functions that had a similar shape. They all take something as input and return a M-recipe as output. In other words, they have this signature:

```fsharp
val monadicFunction : 'a -> M<'b>
```

Here are some examples of actual monadic functions that we used:

```fsharp
val makeLiveLeftLeg : DeadLeftLeg -> M<LiveLeftLeg>
val makeLiveRightLowerArm : DeadRightLowerArm -> M<LiveRightLowerArm>
val makeLiveHeart : DeadHeart -> M<LiveHeart>
val makeBeatingHeart : LiveHeart -> M<BeatingHeart>
// and also
val returnM : 'a -> M<'a>
```

### bindM

The functions up to now did not require access to the vital force. But then we found that we needed to chain two monadic functions together. In particular, we needed to chain the output of `makeLiveHeart` (with signature `DeadHeart -> M<LiveHeart>`) to the input of `makeBeatingHeart` (with signature `LiveHeart -> M<BeatingHeart>`). The solution was `bindM`, which transforms monadic functions of the form `'a -> M<'b>` into functions in the M-world (`M<'a> -> M<'b>`) which could then be composed together.

The signature of `bindM` was:

```fsharp
val bindM : f:('a -> M<'b>) -> M<'a> -> M<'b>
```

### applyM

Finally, we needed a way to combine a large number of M-parameters to make the live body. Rather than having to create special versions of map (`map4M`, `map5M`, `map6M`, etc), we implemented a generic `applyM` function that could apply an M-function to an M-parameter. From that, we could work with a function of any size, step by step, using partial application to apply one M-parameter at a time.

The signature of `applyM` was:

```fsharp
val applyM : M<('a -> 'b)> -> M<'a> -> M<'b>
```

### Defining the others functions in terms of bind and return

Note that of all these functions, only `bindM` needed access to the vital force.

In fact, as we'll see below, the functions `mapM`, `map2M`, and `applyM` can actually be defined in terms of `bindM` and `returnM`!


## Refactoring to a computation expression

A lot of the functions we have created have a very similar shape, resulting in a lot of duplication. Here's one example:

```fsharp
let makeLiveLeftLegM deadLeftLeg  =
    let becomeAlive vitalForce =
        let (DeadLeftLeg label) = deadLeftLeg
        let oneUnit, remainingVitalForce = getVitalForce vitalForce
        let liveLeftLeg = LiveLeftLeg (label,oneUnit)
        liveLeftLeg, remainingVitalForce
    M becomeAlive  // wrap the function in a single case union
```

In particular, there is a lot of explicit handling of the vital force.

In most functional languages, there is a way to hide this so that the code looks much cleaner.

In Haskell, developers use "do-notation", in Scala people use "for-yield" (the "for comprehension"). And in F#, people use computation expressions.

To create a computation expression in F#, you just need two things to start with, a "bind" and a "return", both of which we have.

Next, you define a class with specially named methods `Bind` and `Return`:

```fsharp
type MonsterBuilder()=
    member this.Return(x) = returnM x
    member this.Bind(xM,f) = bindM f xM
```

And finally, you create an instance of this class:

```fsharp
let monster = new MonsterBuilder()
```

When this is done, we have access to the special syntax `monster {...}`, just like `async{...}`, `seq{...}`, etc.

* The `let! x = xM` syntax requires that the right side is an M-type, say `M<X>`.\
  `let!` unwraps the `M<X>` into an `X` and binds it to the left side -- "x" in this case.
* The `return y` syntax requires that return value is a "normal" type, `Y` say.\
  `return` wraps it into a `M<Y>` (using `returnM`) and returns it as the overall value of the `monster` expression.

So some example code would look like this:

```fsharp
monster {
    let! x = xM  // unwrap an M<X> into an X and bind to "x"
    return y     // wrap a Y and return an M<Y>
    }
```

*If you want more on computation expressions, I have an [in-depth series of posts about them](/series/computation-expressions.html).*

### Redefining mapM and friends

With `monster` expressions available, let's rewrite `mapM` and the other functions.

**mapM**

`mapM` takes a function and a wrapped M-value and returns the function applied to the inner value.

Here is an implementation using `monster`:

```fsharp
let mapM f xM =
    monster {
        let! x = xM  // unwrap the M<X>
        return f x   // return M of (f x)
        }
```

If we compile this implementation, we get the same signature as the previous implementation:

```fsharp
val mapM : f:('a -> 'b) -> M<'a> -> M<'b>
```

**map2M**

`map2M` takes a function and two wrapped M-values and returns the function applied to both the values.

It's also easy to write using `monster` expressions:

```fsharp
let map2M f xM yM =
    monster {
        let! x = xM  // unwrap M<X>
        let! y = yM  // unwrap M<Y>
        return f x y // return M of (f x y)
        }
```

If we compile this implementation, we again get the same signature as the previous implementation:

```fsharp
val map2M : f:('a -> 'b -> 'c) -> M<'a> -> M<'b> -> M<'c>
```

**applyM**

`applyM` takes a wrapped function and a wrapped value and returns the function applied to the value.

Again, it's trivial  to write using `monster` expressions:

```fsharp
let applyM fM xM =
    monster {
        let! f = fM  // unwrap M<F>
        let! x = xM  // unwrap M<X>
        return f x   // return M of (f x)
        }
```

And the signature is as expected

```fsharp
val applyM : M<('a -> 'b)> -> M<'a> -> M<'b>
```

## Manipulating the vital force from within a monster context

We'd like to use the monster expression to rewrite all our other functions too, but there is a stumbling block.

Many of our functions have a body that looks like this:

```fsharp
// extract a unit of vital force from the context
let oneUnit, remainingVitalForce = getVitalForce vitalForce

// do something

// return value and remaining vital force
liveBodyPart, remainingVitalForce
```

In other words, we're *getting* some of the vital force and then *putting* a new vital force to be used for the next step.

We are familiar with "getters" and "setters" in object-oriented programming, so let's see if we can write similar ones that will work in the `monster` context.

### Introducing getM

Let's start with the getter. How should we implement it?

Well, the vital force is only available in the context of becoming alive, so the function must follow the familiar template:

```fsharp
let getM =
    let doSomethingWhileLive vitalForce =
        // what here ??
        what to return??, vitalForce
    M doSomethingWhileLive
```

Note that getting the `vitalForce` doesn't use any up, so the original amount can be returned untouched.

But what should happen in the middle? And what should be returned as the first element of the tuple?

The answer is simple: just return the vital force itself!

```fsharp
let getM =
    let doSomethingWhileLive vitalForce =
        // return the current vital force in the first element of the tuple
        vitalForce, vitalForce
    M doSomethingWhileLive
```

`getM` is a `M<VitalForce>` value, which means that we can unwrap it inside a monster expression like this:

```fsharp
monster {
    let! vitalForce = getM
    // do something with vital force
    }
```

### Introducing putM

For the putter, the implementation is a function with a parameter for the new vital force.

```fsharp
let putM newVitalForce  =
    let doSomethingWhileLive vitalForce =
        what here ??
    M doSomethingWhileLive
```

Again, what should we do in the middle?

The most important thing is that the `newVitalForce` becomes the value that is passed on to the next step. We must throw away the original vital force!

Which in turn means that `newVitalForce` *must* be used as the second part of the tuple that is returned.

And what should be in the first part of the tuple that is returned? There is no sensible value to return, so we'll just use `unit`.

Here's the final implementation:

```fsharp
let putM newVitalForce  =
    let doSomethingWhileLive vitalForce =
        // return nothing in the first element of the tuple
        // return the newVitalForce in the second element of the tuple
        (), newVitalForce
    M doSomethingWhileLive
```

With `getM` and `putM` in place, we can now create a function that

* gets the current vital force from the context
* extracts one unit from that
* replaces the current vital force with the remaining vital force
* returns the one unit of vital force to the caller

And here's the code:

```fsharp
let useUpOneUnitM =
    monster {
        let! vitalForce = getM
        let oneUnit, remainingVitalForce = getVitalForce vitalForce
        do! putM remainingVitalForce
        return oneUnit
        }
```


## Using the monster expression to rewrite all the other functions

With `useUpOneUnitM`, we can start to rewrite all the other functions.

For example, the original function `makeLiveLeftLegM` looked like this, with lots of explicit handling of the vital force.

```fsharp
let makeLiveLeftLegM deadLeftLeg  =
    let becomeAlive vitalForce =
        let (DeadLeftLeg label) = deadLeftLeg
        let oneUnit, remainingVitalForce = getVitalForce vitalForce
        let liveLeftLeg = LiveLeftLeg (label,oneUnit)
        liveLeftLeg, remainingVitalForce
    M becomeAlive  // wrap the function in a single case union
```

The new version, using a monster expression, has implicit handling of the vital force, and consequently looks much cleaner.

```fsharp
let makeLiveLeftLegM deadLeftLeg =
    monster {
        let (DeadLeftLeg label) = deadLeftLeg
        let! oneUnit = useUpOneUnitM
        return LiveLeftLeg (label,oneUnit)
        }
```

Similarly, we can rewrite all the arm surgery code like this:

```fsharp
let makeLiveRightLowerArm (DeadRightLowerArm label) =
    monster {
        let! oneUnit = useUpOneUnitM
        return LiveRightLowerArm (label,oneUnit)
        }

let makeLiveRightUpperArm (DeadRightUpperArm label) =
    monster {
        let! oneUnit = useUpOneUnitM
        return LiveRightUpperArm (label,oneUnit)
        }

// create the M-parts
let lowerRightArmM = DeadRightLowerArm "Tom" |> makeLiveRightLowerArm
let upperRightArmM = DeadRightUpperArm "Jerry" |> makeLiveRightUpperArm

// turn armSurgery into an M-function
let armSurgeryM  = map2M armSurgery

// do surgery to combine the two M-parts into a new M-part
let rightArmM = armSurgeryM lowerRightArmM upperRightArmM
```

And so on. This new code is much cleaner.

In fact, we can make it cleaner yet by eliminating the intermediate values such as `armSurgery` and `armSurgeryM` and putting everything in the one monster expression.

```fsharp
let rightArmM = monster {
    let! lowerArm = DeadRightLowerArm "Tom" |> makeLiveRightLowerArm
    let! upperArm = DeadRightUpperArm "Jerry" |> makeLiveRightUpperArm
    return {lowerArm=lowerArm; upperArm=upperArm}
    }
```

We can use this approach for the head as well. We don't need `headSurgery` or `returnM` any more.

```fsharp
let headM = monster {
    let! brain = makeLiveBrain deadBrain
    return {brain=brain; skull=skull}
    }
```

Finally, we can use a `monster` expression to create the whole body too:

```fsharp
// a function to create a M-body given all the M-parts
let createBodyM leftLegM rightLegM leftArmM rightArmM headM beatingHeartM =
    monster {
        let! leftLeg = leftLegM
        let! rightLeg = rightLegM
        let! leftArm = leftArmM
        let! rightArm = rightArmM
        let! head = headM
        let! beatingHeart = beatingHeartM

        // create the record
        return {
            leftLeg = leftLeg
            rightLeg = rightLeg
            leftArm = leftArm
            rightArm = rightArm
            head = head
            heart = beatingHeart
            }
        }

// create the M-body
let bodyM = createBodyM leftLegM rightLegM leftArmM rightArmM headM beatingHeartM
```

NOTE: The complete code using `monster` expressions is [available on GitHub](https://gist.github.com/swlaschin/54489d9586402e5b1e8a#file-monadster2-fsx).

### Monster expressions vs applyM

We previously used an alternative way to create the body, using `applyM`.

For reference, here's that way using `applyM`:

```fsharp
let createBody leftLeg rightLeg leftArm rightArm head beatingHeart =
    {
    leftLeg = leftLeg
    rightLeg = rightLeg
    leftArm = leftArm
    rightArm = rightArm
    head = head
    heart = beatingHeart
    }

let bodyM =
    createBody
    <!> leftLegM
    <*> rightLegM
    <*> leftArmM
    <*> rightArmM
    <*> headM
    <*> beatingHeartM
```

So what's the difference?

Aesthetically there is a bit of difference, but you could legitimately prefer either.

However, there is a much more important difference between the `applyM` approach and the `monster` expression approach.

The `applyM` approach allows the parameters to be run *independently* or *in parallel*, while the `monster` expression approach requires that the parameters are run *in sequence*, with the output of one being fed into the input of the next.

That's not relevant for this scenario, but can be important for other situations such as validation or async. For example, in a validation context, you may want to collect all the validation errors at once, rather than only returning the first one that fails.


{{< linktarget "statemonad" >}}

## Relationship to the state monad

Dr Frankenfunctor was a pioneer in her time, blazing a new trail, but she did not generalize her discoveries to other domains.

Nowadays, this pattern of threading some information through a series of functions is very common, and we give it a standard name: "State Monad".

Now to be a true monad, there are various properties that must be satisfied (the so-called monad laws), but I am not going to discuss them here, as this post is not intended to be a monad tutorial.

Instead, I'll just focus on how the state monad might be defined and used in practice.

First off, to be truly reusable, we need to replace the `VitalForce` type with other types.  So our function-wrapping type (call it `S`) must have *two* type parameters, one for the type of the state, and one for the type of the value.

```fsharp
type S<'State,'Value> =
    S of ('State -> 'Value * 'State)
```

With this defined, we can create the usual suspects: `runS`, `returnS` and `bindS`.

```fsharp
// encapsulate the function call that "runs" the state
let runS (S f) state = f state

// lift a value to the S-world
let returnS x =
    let run state =
        x, state
    S run

// lift a monadic function to the S-world
let bindS f xS =
    let run state =
        let x, newState = runS xS state
        runS (f x) newState
    S run
```

Personally, I'm glad that we got to understood how these worked in the `M` context before making them completely generic. I don't know about you, but signatures like these

```fsharp
val runS : S<'a,'b> -> 'a -> 'b * 'a
val bindS : f:('a -> S<'b,'c>) -> S<'b,'a> -> S<'b,'c>
```

would be really hard to understand on their own, without any preparation.

Anyway, with those basics in place we can create a `state` expression.

```fsharp
type StateBuilder()=
    member this.Return(x) = returnS x
    member this.ReturnFrom(xS) = xS
    member this.Bind(xS,f) = bindS f xS

let state = new StateBuilder()
```

`getS` and `putS` are defined in a similar way as `getM` and `putM` for `monster`.

```fsharp
let getS =
    let run state =
        // return the current state in the first element of the tuple
        state, state
    S run
// val getS : S<State>

let putS newState =
    let run _ =
        // return nothing in the first element of the tuple
        // return the newState in the second element of the tuple
        (), newState
    S run
// val putS : 'State -> S<unit>
```

### Property based testing of the state expression

Before moving on, how do we know that our `state` implementation is correct? What does it even *mean* to be correct?

Well, rather than writing lots of example based tests, this is a great candidate for a [property-based testing](/pbt/) approach.

The properties we might expect to be satisfied include:

* [**The monad laws**](https://stackoverflow.com/questions/18569656/explanation-of-monad-laws-in-f).
* **Only the last put counts**. That is, putting X then putting Y should be the same as just putting Y.
* **Get should return the last put**. That is, putting X then doing a get should return the same X.

and so on.

I won't go into this any more right now. I suggest watching [the talk](/pbt/) for a more in-depth discussion.

### Using the state expression instead of the monster expression

We can now use `state` expressions exactly as we did `monster` expressions. Here's an example:

```fsharp
// combine get and put to extract one unit
let useUpOneUnitS = state {
    let! vitalForce = getS
    let oneUnit, remainingVitalForce = getVitalForce vitalForce
    do! putS remainingVitalForce
    return oneUnit
    }

type DeadLeftLeg = DeadLeftLeg of Label
type LiveLeftLeg = LiveLeftLeg of Label * VitalForce

// new version with implicit handling of vital force
let makeLiveLeftLeg (DeadLeftLeg label) = state {
    let! oneUnit = useUpOneUnitS
    return LiveLeftLeg (label,oneUnit)
    }
```

Another example is how to build a `BeatingHeart`:

```fsharp
type DeadHeart = DeadHeart of Label
type LiveHeart = LiveHeart of Label * VitalForce
type BeatingHeart = BeatingHeart of LiveHeart * VitalForce

let makeLiveHeart (DeadHeart label) = state {
    let! oneUnit = useUpOneUnitS
    return LiveHeart (label,oneUnit)
    }

let makeBeatingHeart liveHeart = state {
    let! oneUnit = useUpOneUnitS
    return BeatingHeart (liveHeart,oneUnit)
    }

let beatingHeartS = state {
    let! liveHeart = DeadHeart "Anne" |> makeLiveHeart
    return! makeBeatingHeart liveHeart
    }

let beatingHeart, remainingFromHeart = runS beatingHeartS vf
```

As you can see, the `state` expression automatically picked up that `VitalForce` was being used as the state -- we didn't need to specify it explicitly.

So, if you have a `state` expression type available to you, you don't need to create your own expressions like `monster` at all!

For a more detailed and complex example of the state monad in F#, check out the [FSharpx library](https://github.com/fsprojects/FSharpx.Extras/blob/master/src/FSharpx.Extras/ComputationExpressions/Monad.fs#L409).

*NOTE: The complete code using `state` expressions is [available on GitHub](https://gist.github.com/swlaschin/54489d9586402e5b1e8a#file-monadster3-fsx).*

## Other examples of using a state expression

The state computation expression, once defined, can be used for all sorts of things. For example, we can use `state` to model a stack.

Let's start by defining a `Stack` type and associated functions:

```fsharp
// define the type to use as the state
type Stack<'a> = Stack of 'a list

// define pop outside of state expressions
let popStack (Stack contents) =
    match contents with
    | [] -> failwith "Stack underflow"
    | head::tail ->
        head, (Stack tail)

// define push outside of state expressions
let pushStack newTop (Stack contents) =
    Stack (newTop::contents)

// define an empty stack
let emptyStack = Stack []

// get the value of the stack when run
// starting with the empty stack
let getValue stackM =
    runS stackM emptyStack |> fst
```

Note that none of that code knows about or uses the `state` computation expression.

To make it work with `state`, we need to define a customized getter and putter for use in a `state` context:

```fsharp
let pop() = state {
    let! stack = getS
    let top, remainingStack = popStack stack
    do! putS remainingStack
    return top
    }

let push newTop = state {
    let! stack = getS
    let newStack = pushStack newTop stack
    do! putS newStack
    return ()
    }
```

With these in place we can start coding our domain!

### Stack-based Hello World

Here's a simple one. We push "world", then "hello", then pop the stack and combine the results.

```fsharp
let helloWorldS = state {
    do! push "world"
    do! push "hello"
    let! top1 = pop()
    let! top2 = pop()
    let combined = top1 + " " + top2
    return combined
    }

let helloWorld = getValue helloWorldS // "hello world"
```

### Stack-based calculator

Here's a simple stack-based calculator:

```fsharp
let one = state {do! push 1}
let two = state {do! push 2}

let add = state {
    let! top1 = pop()
    let! top2 = pop()
    do! push (top1 + top2)
    }
```

And now we can combine these basic states to build more complex ones:

```fsharp
let three = state {
    do! one
    do! two
    do! add
    }

let five = state {
    do! two
    do! three
    do! add
    }
```

Remember that, just as with the vital force, all we have now is a *recipe* for building a stack. We still need to *run* it to execute the recipe and get the result.

Let's add a helper to run all the operations and return the top of the stack:

```fsharp
let calculate stackOperations = state {
    do! stackOperations
    let! top = pop()
    return top
    }
```

Now we can evaluate the operations, like this:

```fsharp
let threeN = calculate three |> getValue // 3

let fiveN = calculate five |> getValue   // 5
```

## OK, OK, some monad stuff

People always want to know about monads, even though I do not want these posts to degenerate into [yet another monad tutorial](/posts/why-i-wont-be-writing-a-monad-tutorial/).

So here's how they fit in with what we have worked with in these posts.

A **functor** (in a programming sense, anyway) is a data structure (such as Option, or List, or State) which has a `map` function associated with it. And the `map` function has some properties that it must satisfy (the ["functor laws"](https://en.wikibooks.org/wiki/Haskell/The_Functor_class#The_functor_laws)).

A **applicative functor** (in a programming sense) is a data structure (such as Option, or List, or State) which has two functions associated with it: `apply` and `pure` (which is the same as `return`). And these functions have some properties that they must satisfy (the ["applicative functor laws"](https://en.wikibooks.org/wiki/Haskell/Applicative_functors#Applicative_functor_laws)).

Finally, a **monad** (in a programming sense) is a data structure (such as Option, or List, or State) which has two functions associated with it: `bind` (often written as `>>=`) and `return`. And again, these functions have some properties that they must satisfy (the ["monad laws"](https://en.wikibooks.org/wiki/Haskell/Understanding_monads#Monad_Laws)).

Of these three, the monad is most "powerful" in a sense, because the `bind` function allows you to chain M-producing functions together, and as we have seen, `map` and `apply` can be written in terms of `bind` and `return`.

So you can see that both our original `M` type and the more generic `State` type, in conjunction with their supporting functions, are monads, (assuming that our `bind` and `return` implementations satisfy the monad laws).

For a visual version of these definitions, there is a great post called [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html).

## Further reading

There are a lot of posts about state monads on the web, of course. Most of them are Haskell oriented, but I hope that those explanations will make more sense after reading this series of posts, so I'm only going to mention a few follow up links.

* [State monad in pictures](http://adit.io/posts/2013-06-10-three-useful-monads.html#the-state-monad)
* ["A few monads more", from "Learn You A Haskell"](http://learnyouahaskell.com/for-a-few-monads-more)
* [Much Ado About Monads](http://codebetter.com/matthewpodwysocki/2009/12/31/much-ado-about-monads-state-edition/). Discussion about state monad in F#.

And for another important use of "bind", you might find my talk on [functional error handling](/rop/) useful.

If you want to see F# implementations of other monads, look no further than [the FSharpx project](https://github.com/fsprojects/FSharpx.Extras/blob/master/src/FSharpx.Extras/ComputationExpressions).

## Summary

Dr Frankenfunctor was a groundbreaking experimentalist, and I'm glad that I have been able to share insights on her way of working.

We've seen how she discovered a primitive monad-like type, `M<BodyPart>`, and how `mapM`, `map2M`, `returnM`, `bindM` and `applyM` were all developed to solve specific problems.

We've also seen how the need to solve the same problems led to the modern-day state monad and computation expression.

Anyway, I hope that this series of posts has been enlightening. My not-so-secret wish is that monads and their associated combinators will no longer be so shocking to you now...

![shocking](./monadster_shocking300.gif)

...and that you can use them wisely in your own projects. Good luck!

*NOTE: The code samples used in this post are [available on GitHub](https://gist.github.com/swlaschin/54489d9586402e5b1e8a)*.

