---
layout: post
title: "Writing 'Finite State Machines' in Kotlin (Part 2) - 'Delta Hat'"
excerpt: "In this part we shall move on to defining and implementing delta hat, better known as the 'extended transition function'"
categories: [Kotlin, Computer Science]
comments: true
medium: https://medium.com/@spaghetticode/writing-finite-state-machines-in-kotlin-part-2-delta-hat-3f2ae5eb42bd
image:
  feature: /post/state_2.jpg
  credit: Karim Manjra
  creditlink: https://unsplash.com/photos/pWlM5L6PFis
---

This is part **two** of my mini-series on programming finite state machines. If you have yet to check out part one, I highly recommend tackling it before moving on to this part.
 
In this part we shall move on to defining and implementing 'delta hat' which is better known as the 'extended transition function' a function that takes a finite state automaton and tests whether a sequence of inputs is part of the *language* it represents.

☕️ Hold on to your delta hats, because here comes the mathematics! ☕️

# Mind your language

First, let me do some housekeeping. Going forward we shall be calling inputs such as "aa" to our DFA 'strings' or 'words' and the set of input symbols we defined in part one, denoted by **Σ**, an 'alphabet' that is made up of individual 'characters'. These will both help us define the language that a DFA represents and are also the terms most commonly associated with finite state automata.

When we talk about 'languages' in this context, what we are referring to the set of _all_ possible words that result in a sequence of state transitions that ultimately end in an accepting state. 

Let's revisit our simple light-switch example DFA from part 1 of this series through the state transition diagram we created. 

![](https://i.imgur.com/Sl02pTl.png)

As **S1** is our accepting state, we can see straight away that the string of **"a"** is part of the language this DFA represents as it would transition and terminate in the accepting state. From this, we also know **"aa"** would not be part of the language as we'd find ourselves in **S0**, a non-accepting state, once we complete the transitions.

To generalise this, for an automaton **A** that represents language **L** with alphabet **Σ**, we would define the language formally using the following notation

![](https://latex.codecogs.com/svg.latex?%5CLARGE%20L%28A%29%20%5Csubseteq%20%5CSigma%5E*)

**Σ\*** in this example is the 'Kleene operator' applied to our alphabet which represents the set of  _all_ words of finite length consisting of symbols within **Σ** but also including the 'empty symbol' denoted as **ε**, the Greek letter epsilon. For our light-switch example the set of produced when we apply the Kleene operator to our alphabet would be {"ε", "a", "aa", "aaa", "aaaa", "aaaaa" ... } and so on ad infinitum. The definition simply states that the language the DFA represents is equal to this infinite set or is (more likely) some subset of it.

So what does it take to confirm whether a word is contained in the language the DFA represents?

This is where 'delta hat', the extended transition function, makes an appearance.  

# Our friend Delta Hat

Casting our minds back to part one of this series, we defined **δ** as the state transition function for a DFA that, given a state and a character, would provide the resulting state after that transition. 

The extended transition function works in a similar fashion but is used to determine if strings are part of the DFA's language. This is achieved by taking an initial state **Q** in addition to the string contained within **Σ\*** and producing the output state **Q** after processing the entire word.

From that, we would define delta hat, the extended transition function, as follows.

![](https://latex.codecogs.com/svg.latex?%5CLARGE%20%5Chat%7B%5Cdelta%7D%20%5Cin%20Q%20%5Ctimes%20%5CSigma%5E*%20%5Crightarrow%20Q)


But how do we use it? The answer is provided after defining it inductively as follows using some fancy, but ultimately primitive recursion 🤓


![](https://latex.codecogs.com/svg.latex?%5CLARGE%20%5C%3B%5C%3B%5C%3B%5C%3B%5C%3B%5Chat%7B%5Cdelta%7D%28s%2C%20%5Cepsilon%29%20%3D%20s%20%5Cnewline%20%5Chat%7B%5Cdelta%7D%28s%2C%20xw%29%20%3D%20%5Chat%7B%5Cdelta%7D%28%5Cdelta%28s%2C%20x%29%2C%20w%29)

If you think that looks slightly like alphabet soup, I do not blame you. To understand what this is, let's break down what is going on here. The first line states that when we apply delta hat to a state **s** and the empty string **ε** we return the exact same state we used as an input. Simple enough.

The second line is the *real* magic. When we apply delta hat to a state **s** and a word **xw** where **x** is the head of the word (i.e. the first symbol) and **w** is the rest of the word. We call delta hat *recursively* on **w** after first using our previously defined **δ** from the DFA applied to the head of our word to give us the new state.

If that didn't make an awful lot of sense, then consider this example from our light-switch DFA, testing if **"aaa"** is part of the language.

![](https://latex.codecogs.com/svg.latex?%5CLARGE%20%5Cbegin%7Balign*%7D%20%5Chat%7B%5Cdelta%7D%28S_%7B0%7D%2C%20aaa%29%20%3D%20%5Chat%7B%5Cdelta%7D%28%5Cdelta%28S_%7B0%7D%2C%20a%29%2C%20aa%29%20%5C%5C%20%3D%20%5Chat%7B%5Cdelta%7D%28%5Cdelta%28S_%7B1%7D%2C%20a%29%2C%20a%29%20%5C%5C%20%3D%20%5Chat%7B%5Cdelta%7D%28%5Cdelta%28S_%7B0%7D%2C%20a%29%2C%20%5Cepsilon%29%20%5C%5C%20%3D%20%5Chat%7B%5Cdelta%7D%28S_%7B1%7D%2C%20%5Cepsilon%29%20%5C%5C%20%3D%20S_%7B1%7D%20%5C%5C%20%5Cend%7Balign*%7D)

Character by character, we apply the extended transition function until we end at our base case, a single state and the empty string. In this case, the string **"aaa"** *is* an element of our DFA's language as we finish in **S1**, the accepting state.

That's all well and good, but how can we implement this and apply it to our existing Kotlin DFA?

Good question!


# How do we Kotlin-ify this?
To start exploring how we might create our extended transition function in Kotlin, we should first revisit and revise the Kotlin DFA we constructed from [part one](https://medium.com/@spaghetticode/finite-state-machines-in-kotlin-part-1-57e68d54d93b).

The main difference to note here is the use of the [sealed classes](https://kotlinlang.org/docs/reference/sealed-classes.html) as the `State` and `Input` to ensure we have exhaustive `when` statements and avoid any invalid input funny business.

{% highlight kotlin %}
// Using State and Input as Sealed Classes (See part 1)
sealed class State
object S0 : State()
object S1 : State()

sealed class Input
object A : Input()

// Our trusty light-switch DFA

val dfa = DFA(
   states = setOf(S0, S1),
   inputs = setOf(A),
   delta = { state, input ->
     when(input) {
       A -> when (state) {
              S0 -> S1
              S1 -> S0
       }
     }
   },
   initialState = S0,
   isFinalState = { it == S1 }
)
{% endhighlight %}

Let's first take the slightly naive, but nonetheless perfectly valid, approach of creating our extended transition function by defining its behaviour iteratively.

{% highlight kotlin %}
fun deltaHatIterative(dfa: DFA, input: List<Input>): State {
    var state = dfa.initialState
    for (character in input) {
        state = dfa.delta(state, character)
    }
    return state
}
{% endhighlight %}

I hope that if thus far you are slightly confused how delta hat works, this example will have hopefully cleared up any confusion. We are quite simply applying the transitions in the DFA by using the word as an input.

😬 This is great and the solution would be perfect, _if_ it was not for the `var`. Having mutable state within our method is something we'd generally like to avoid where possible to allow us to create 'pure' functions. Why pure functions? I will let [Alvin Alexander](https://alvinalexander.com/scala/fp-book/benefits-of-pure-functions) explain that in his excellent blog post, but trust me it makes our lives as developers much simpler.

On that note, let's take a stab at creating the extended transition function using the more formal recursive definition we looked at in the last section.

{% highlight kotlin %}
// Helper extension to easily split list into head and tail
fun <T> List<T>.dequeue() = Pair(first(), drop(1)) 

tailrec fun deltaHatRecursively(dfa: DFA, state: State, input: List<Input>): State {
    return when {
        input.isNullOrEmpty() -> state
        else -> {
            val (head, other) = input.dequeue()
            deltaHatRecursively(dfa, dfa.delta(state, head), other)
        }
    }
}
{% endhighlight %}

Lovely, through the use of `when` we can match our mathematical inductive definition and handle both the base case (when the list of inputs is empty) and the general case (in which it isn't).

The first obvious point of note is the use of extension functions to allow lists to be easily split into their head and tail elements. For a list such as `[A, B, C, D]` this method would simply return the tuple `(A, [B, C, D])`.

The second thing that likely jumps out to most Kotlin devs is the `tailrec` modifier. The [tail recursion modifier](https://kotlinlang.org/docs/reference/functions.html#tail-recursive-functions) within Kotlin provides optimisation through the compiler translating the recursive element to a simple loop during compilation and thus removes the possibility of rearing the ugly head of a `StackOverflowError`. It can only be applied to functions that call itself as the _last_ operation it performs.

For more on tail recursion within Kotlin, [Jorge Castillo](https://medium.com/@JorgeCastilloPr/tail-recursion-and-how-to-use-it-in-kotlin-97353993e17f) has written an excellent piece on this in 2017, which is well worth reading.

Believe it or not, we can actually refactor this entire function into a single line. Through the use of the `fold` function, we can condense our scary 'delta hat' into something that looks relatively simple

{% highlight kotlin %}
fun deltaHat(dfa: DFA, input: List<Input>) = input.fold(dfa.initialState, dfa.delta)
{% endhighlight %}

Wait what. Where has all of our boilerplate gone? 

No magic here, we have tapped into the power of 'functional programming'.
`fold` when applied to a collection, takes an 'initial value' and a lambda function. The first invocation of the lambda you pass will receive the initial value and the first element of the collection as the parameters of the lambda. The lambda function should complete by returning the _next_ value to be passed to the lambda alongside the next element within the collection.

Visualising this `fold` with a word **w** containing **n** letters, an initial state **S0** and using the DFA's **δ** as our lambda, expands the fold operation into the following

![](https://latex.codecogs.com/svg.latex?%5Chuge%20Q%20%3D%20%5Cdelta%28%5Cdelta%28%5Cdelta%28S_%7B0%7D%2C%20w_%7B0%7D%29%2C%20w_%7B1%7D%29%2C%20w_%7Bn%7D%29)

Which, thanks to what we learned earlier, we can see is the same definition as 'delta hat'.

After running each delta hat implementation on DFAs with 1 to 5000 inputs, it is clear the advantage a functional approach has when compared to recursion and standard iterative programming.

```
Iterative: 94 ms (0s)
Recursive: 39953 ms (39s) // Eww
Functional: 84 ms (0s)

```

And relax, that's our delta hat implementation completed! 🎉

# Awesome, what comes next?

In the next part of this series, we will look at how we can the DFA's not-so-distant relative the non-deterministic finite automaton (NFA) and use Kotlin to generalise our definitions of NFAs and DFAs.

_I hope you found this post interesting, please feel free to tweet me with any feedback at [@Sp4ghettiCode](https://twitter.com/sp4ghetticode) and don't forget to like, tweet, share etc_
