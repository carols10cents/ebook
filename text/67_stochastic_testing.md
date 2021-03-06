# Stochastic and Property-Based Testing

The term "stochastic" comes from the Greek _stokhastikos_, which is a form of _stokhazesthai_, which means "aim at" or "guess".  It's a good etymology for __stochastic testing__, which uses random processes that can be analyzed using statistics, but not exactly predicted.  At first blush, it may seem ridiculous to use randomness in software testing; after all, isn't the fundamental concept of testing to determine what the observed behavior is and if it equal to the expected behavior?  If you don't know what the input is, how would you know what the expected output it?

The answer is that there are expected behaviors and properties you expect from a system, no matter what the input.  For example, no matter what code is passed to a compiler, you will expect it not to crash.  It may generate an error message saying that the code is unparseable.  It may make an executable.  That executable may run, or it may not.  You do expect the system not to have a segmentation fault.  Thus, you can still run tests where the expected behavior is something like "does not crash the system".

By providing a method for the system to use random data as an input, you also reduce the cost of testing.  No longer do you have to imagine lots of specific test cases, and then painstakingly program all of them in or write them down as test cases.  Instead, you just tie together some sort of random number generator and some way to generate data based on it, and your computer can do all the work of generating test cases.  Even though the random number generator probably won't be nearly as good as a dedicated testing professional to coming up with edge cases, varying equivalence classes, etc., it will often find many problems simply by the sheer number of tests it can generate and how quickly it can do it.

Stochastic testing is also referred to as __monkey testing__, by analogy with a monkey banging on computer keys.  However, "stochastic testing" sounds much more impressive if you are talking to your manager, or, say, writing a book on software testing.

## Infinite Monkeys and Infinite Typewriters

There is an old parable about monkeys and typewriters, which normally would not seem like two things that go together well.  The parable states that given a million monkeys and infinite amount of time, the monkeys will eventually write the works of Shakespeare. (NB: The monkeys in this scenario are immortal.)  Stochastic testing follows a similar principle - given a large enough set of random input (a million monkeys banging on keys) and a large enough period of time, defects will be found.  In this analogy, our tester is William Shakespeare (don't let the comparison go to your head).  He could certainly write the works of Shakespeare in less time than the million monkeys.  However, monkeys (like the computer's random number generator) are much cheaper than re-animating Zombie William Shakespeare.  Similarly, even if the random number generator isn't as good a writer of tests as you (or Shakespeare), by sheer dint of numbers, it's bound to hit on numerous interesting edge cases and perhaps find defects.

Of course, since you - or more precisely, the stochastic testing system - may not know exactly what the expected behavior should be for a given input, you need to check for properties of the system.  At the unit testing level, where you are checking individual methods, this is called __property-based testing__.

## Property-Based Testing

Let's say once again that we are testing our sorting function, `billSort`.  As you'll recall, it's meant to be twenty times faster than any other sorting algorithm out there, but there are questions about its correctness, and so you have been tasked to test that it works in all cases.  What kind of input values would you test it with?  Assume the method signature looks like this:

```java
public int[] billSort(int[] arrToSort) {
  ...
}
```

We'd definitely want to pass in a wide variety of values that hit different base, edge, and corner cases.  Single-element arrays.  Zero-element arrays.  Arrays with negative integers.  Arrays that are already sorted.  Arrays that are sorted the opposite way as the sort works (ascending vs descending, or vice versa).  Very long arrays.  Arrays that contain multiple values, and arrays that consist of the same value repeated over and over again.  This doesn't even take into account the fact that Java is a statically typed language, so we don't have to worry about what happens when an array contains strings, or references to other arrays, or complex numbers, or a variety of other kinds of things.  If this sort were implemented in a dynamically typed language such as Ruby, we'd really have a lot to worry about.  Just considering the Java method above, though, let's think about some of the different inputs and their respective expected output values for this method:

```
[] => []
[1] => [1]
[1, 2, 3, 4, 5] => [1, 2, 3, 4, 5]
[5, 4, 3, 2, 1] => [1, 2, 3, 4, 5]
[0, 0, 0, 0] => [0, 0, 0, 0]
[9, 3, 1, 2] => [1, 2, 3, 9]
[-9, 9, -4, 4] => [-9, -4, 4, 9]
[3, 3, 3, 2, 1, 1, 1] => [1, 1, 1, 2, 3, 3, 3]
[-1, -2, -3] => [-3, -2, -1]
[1000000, 10000, 100, 10, 1] => [1, 10, 100, 10000, 1000000]
[2, 8, ... (many ints) ... -3, 7] => [-900, -874, ... (many ints) ... 989, 991]
```

Even without any additional wrinkles, there's an absolutely huge number of possible combinations of numbers to test.  That was just a taste.  There's even a huge number of equivalence cases to test, ignoring the fact there could be a problem with, say, a specific number; maybe only sorts with the number 5 don't work, for example.  Writing tests for all of these various kinds of input would be extremely tedious and error-prone.  How can we avoid having to write such a large number of tests?

### Climbing The Abstraction Ladder

Why not hop up a rung on the abstraction ladder and instead of thinking about the specific values that you want as input and output, you think about the *properties* you'd expect of your input and output?  That way, you don't have to consider each individual test.  You can let the computer know that you expect all of the output to have certain properties, and what kind of values you expect as input, and let the computer write and execute the tests for you.

For example, what kinds of properties did all of the correct output values of the `billSort` method have, in relationship to the input values?  There are quite a few.  These properties should hold for all sorted lists.  Thus, they are called *invariants*.

Some invariants for a sort function would be:

1. The output array has the same number of elements as the input array
2. Every value in the output array corresponds to one in the input array
3. The value of each successive element in the output array is greater than or equal to the previous value
4. No element not in the input array is found in the output array
5. The function is idempotent; that is, running the sort method on a list, and then running the sort again on the output, should produce the same output array as just running it once
6. The function is pure; running it two times on the same input array should always produce the same output array

Now that we have some of the properties we expect from *any* output of the `billSort` method, we can let the computer do the grunt work of thinking up random arrays of data, passing them in to our method, and then checking that whatever output array is produced meets all of the properties that we set.  If an output array does not meet one of the invariants, we can then report the error to the tester.  Producing output that does not meet the specified invariant is called __falsifying the invariant__.

There are multiple libraries for Java which perform property-based testing, but no standard.  Property-based testing is much more popular in the functional programming world, with programs like QuickCheck for Haskell being more used than standard unit tests.  In fact, the concept of automated property-based testing of this sort comes from the functional world, and from the Haskell community in particular.  For more information, check out the paper _QuickCheck: A Lightweight Tool for Random Testing of Haskell Programs_ by Koen Claessen and John Hughes.

## Smart, Dumb, Evil, and Chaos Monkeys

As mentioned above, stochastic testing is often called monkey testing.  What is not as well known is that there are different kinds of monkeys out there doing our testing work for us!

__Dumb monkey__ testing is sending in just any old input you can think of.  "`Mfdsjbkfd`", "`1 + snorf`", and "`(*@()`" all seem like good inputs to the dumb monkey.  There is no rhyme or reason, just lots of different randomized input.  This can be helpful for catching edge cases, but it is not very focused.  Remember that the world of possible values is absolutely huge.  The chances of finding a specific defect might be minimal when using dumb monkey testing.

As an example, let us assume that you have a defect where arbitrary JavaScript code can be executed by entering it into a text box on your web application.  However, if the JavaScript code is not syntactically correct, nothing bad happens.  Think of how long it would take for a dumb monkey to randomly generate a valid JavaScript string that would take advantage of this obvious vulnerability!  It may be years.  Even then, it may be some JavaScript code which does not cause any problems, or even any noticeable output, such as a comment or logging a message to the console.  Meanwhile, even a novice tester is going to try to enter some JavaScript on any text box they can find.

__Smart monkey__ testing involves using input which a user might conceivably enter, as opposed to being strictly random.  For example, suppose you are testing a calculator program, which accepts a string of numbers and arithmetic operators and displays a result.  It is rational to assume that an ordinary user is much more likely to enter the following input:

```
> 1 + 2
3
> 4 + + 6
ERROR
> 4 + 6
10
```

than

```
> jiwh0t34h803h8t32h8t3h8t23
ERROR
> aaaaaaaaaaaaa 
ERROR
> 084_==wjw2933
ERROR
```

While this assumption may not hold when it comes to toddlers, in general it is most likely true.  Thus, in order to focus our testing resources on finding defects which are more likely to occur, we can use smart monkey testing to act as a user.  Since the smart monkey test is automated, however, it will be able to operate much more quickly and on many more possible inputs than manual testing by an actual user.

Creating a smart monkey test can be difficult, because not only do you have to first understand what users would likely do with the application, but then develop a model for it.  However, the benefits are that it is much more likely to discover a defect in the system under test.

__Evil monkey__ testing simulates a malicious user who is actively trying to hurt your system.  This can be through sending very long strings, potential injection attacks, malformed data, or other inputs which are designed to cause havoc in your system.  In today's networked world, systems are almost always under attack if they are connected to the Internet for more than a few milliseconds.  It is much better to have an evil monkey under our control determine that the system is vulnerable than let some actual malicious user figure it out!

Perhaps the best-named kind of monkey is the __Chaos Monkey__.  Chaos Monkey is a tool developed by Netflix which randomly shuts down servers that their system is running on, in order to simulate random outages.  For any large system, servers will go down on a regular basis, and at any given time some percentage of systems will be unavailable.  Chaos monkey testing ensures that the system as a whole will be able to operate effectively even when individual machines are not responding.

You do not have to use the official Chaos Monkey tool to do this kind of testing, however.  Think of all the things that can go wrong with a multiple-server system, and simulate them.  What happens when the network topography changes?  Does the system stay active when somebody pulls out some power or networking cables?  What happens if latency is increased to several seconds?  A distributed system is ripe for problems.  Testing that it can handle them now will allow you to prepare for when they happen in reality.  After all, the best way to avoid a problem is to induce it repeatedly; soon, you will have automated procedures to ameliorate it or ensure that it doesn't happen.



