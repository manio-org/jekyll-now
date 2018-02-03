---
layout: post
title:  "Write Elegant Loops Using Loop Invariants"
date:   2018-02-02
categories: coding
---

Writing loops is an essential part of programming, but writing elegant loops is
not easy. For example, in [an
interview](https://www.ted.com/talks/linus_torvalds_the_mind_behind_linux)
Linus showed a commonly used piece of code which delete an entry in a
link list (we will discuss this example). The task was simple but the loop in
the solution was ugly.

Writing loops is hard because multiple variables changes. Keeping track of
these variables and making sure they are in the right states require lots of
mental energy.

How can we reason about the logic of loops better and write elegant loops?
Before answering the questions, let me describe the characteristics of elegant
loops.

1. They look beautiful.
2. They are easy to understand.
3. Their correctness is easy to prove.

Here are some smells of ugly loops:

1. Multiple exit conditions in the loop body, which make it hard to determine
the exit state of the loop.
2. Special case handling before or after the loop. For example, treating
the first or the last element of an array differently.


So, how can we reason about the logic of loops better and write elegant loops?
My answer is to **find a good loop invariant and use it to construct the loop.** I 
will review what a loop invariant is and demonstrate the idea using two examples:
deleting an entry in a linked list (by Linus) and finding a number that is larger than
or equal to a given number.

Let us explain what a loop invariant is by the following code, which moves the `1`s
from `a` to `b`.

``` c
int a = 10
int b = 0;

// invariant: a + b == 10                (A)
while (a != 0) { //                      (B)
    // invariant: a + b == 10            (C)
    a--;
    b++;
    // invariant: a + b == 10            (D)
}
// invariant: a + b == 10                (E)

// post condition: a == 0 && a + b == 10
```

`a + b == 10` is one of the loop invariants of the loop. The invariant is true
at the marked locations. `a != 0` is the loop condition. When the loop
terminates, we know the loop condition is _false_; we
also know that the loop invariant is _true_. Combining these two
conditions, we get the post condition (`a == 0 && a + b == 10`) which indicates
`b == 10`. Now we know for sure that all the `1` in `a` has been moved to `b`.

Loop invariant must be true right before `while` (at (A)); otherwise, it will not be
true at the beginning of the loop (at (C)), because line (B) changes nothing.

Note that if the loop body does not execute at all (because the loop condition
is false), the loop invariant set at (A) still holds after the loop (E).

Now, writing an elegant loop becomes:

1. Find an elegant loop invariant
2. Determine a loop condition so `loop condition == False && loop invariant == true` 
is what you want when the loop terminates.

Let's practice by examining two examples.

# Example 1: Linus's Taste

Linus gave the following piece of code to show what a bad taste of code is. 

``` c
remove_list_entry_01(entry)
{
    prev = NULL;
    walk = head;

    // Walk the list
    while (walk != entry) {
        prev = walk;
        walk = walk->next;
    }

    // Remove the entry by updating the
    // head or the previous entry
    if (!prev)
        head = entry->next;
    else
        prev->next = entry->next;
}
```

The diagram below is an illustration of the basic data structure. `head` is a 
pointer that points to the head of the linked list.

![Simple Linked List]({{ "/assets/images/simple-link-list.png" | absolute_url }})

Linus hates the code because it has to handle a special case (the `if` statement)
after the loop. He argued that the programmer failed to see the common pattern
between the special case and the regular cases. 

I think the programmer wrote the ugly `remove_list_entry_01()` because 
he/she did not find a good loop invariant.

### Loop invariant of the ugly loop

In `remove_list_entry_01()`, the loop invariant is:

```
None of the nodes before `walk` is equal to `entry`.
```

The loop condition is

```
`walk != entry`
```

The post condition is:

```
`walk == entry` && none of the nodes before `walk` is equal to `entry`.
```

The reason that `remove_list_entry_01()` has to handle a special case is 
that `prev` is not in the loop invariant. Depending on whether the loop
has been executed, `prev` can be `NULL`, or pointing to a valid node.

If the loop invariant is the following, then we would not need to handle
two cases after the loop.

```
None of the nodes before `walk` is equal to `entry` 
&& 
`prev` points to a node before `walk`
```

We can make this loop invariant happen by adding a header node, so that even
if the loop never executes, `prev` always points to a node before `walk`.
The initial structure is shown below.

![Link List with header]({{ "/assets/images/link-list-with-header.png" | absolute_url }})


The code becomes the following.

``` c
remove_list_entry_02(entry)
{
    prev = head; 
    walk = head->next;

    while (walk != entry) {
        prev = walk;
        walk = walk->next;
    }

    walk->next = entry->next;
}
```

But this is overkill because it consumes more memory. 


### Find a better loop invariant

Linus gave the following elegant solution, which eliminates the special case by
introducing a pointer that points to the variable that we will update.

``` c
remove_list_entry_03(entry)
{
    // The "indirect" pointer points to the
    // *address* of the thing we'll update
    indirect = &head;

    // Walk the list, looking for the thing that
    // points to the entry we want to remove
    while ((*indirect) != entry)
        indirect = &(*indirect)->next;

    // .. and just remove it
    *indirect = entry->next;
}
```

This solution can be illustrated by the diagram below. Let us imagine that
there is an imaginary header node just like the one in `remove_list_entry_02()`.
`head` can be thought as the `next` member of the header node. `indirect` could
have pointed to the imaginary header node if it really existed; but in reality, we
make `indirect` point to `head`, which is the `next` of the imaginary header
node.

![Link List with imaginary header]({{ "/assets/images/link-list-with-imaginary-header.png" | absolute_url }})

In `remove_list_entry_03()`, the loop invariant is

```
`indirect` points to the memory location of node.next`, 
where `node` and nodes before `node` are not equal to `entry`
```

At the very beginning, `indirect` points to the imaginary header node, which
is for sure not equal to `entry` (because the imaginary header does not exist).

The post condition is

```
`indirect` points to the memory location of node.next`, 
where `node` and nodes before `node` are not equal to `entry`
&& 
node.next == entry
```

The diagram below shows a possible state after the loop terminates, where `indirect` points
to the `next` of the node to be modified.

![Link List last state]({{ "/assets/images/link-list-last-state.png" | absolute_url }})

**To find an elegant loop invariant, you probably need to draw diagrams like
the ones above to find the common pattern between
special cases and regular cases. In this example, it is to find that there could be 
an imaginary header node.**


# Example 2: looking for a number

In the second example, we deal with arrays, which are handled by loops all the
time. The problem in this example is, given a sorted array and a number `x`,
find the index of the smallest number that is large than `x`. Another
assumption is that the number we look for is close to the beginning of the
array, so we should look for the number from the beginning.

So, this is what we will do: we will check the numbers from the beginning one by
one until we find a number that is larger than or equal to `x`. Here is an implementation:

``` c
// n is the size of the array
int skip_01(int array[], int n, int x) {
    int i = 0;

    while (array[i] < x) {
      i++;
      if ( i >= n ) {
        return n;
      }
    }
    return i;
}
```

Although the code is short, it still takes some time to reason about its correctness 
because the loop has multiple exit points. You may need to validate the following
cases one by one to see if the function returns the right value.

1. what if the loop body is never run because `array[0] < x == false`
2. what if the size of the array is 0 (in which the code above fails)
3. what if the size of the array is 1 (a common corner case)
4. what if we cannot find a number that is larger than or equal to `x`
5. what if the number we find is the last one in the array (a corner case you may want to check)
6. how about a regular case

Let's use loop invariant so we do not need to worry about any of the above. After a few trials,
you may find the following loop invariant:

```
numbers in array[-infinity...i - 1] are less than `x`
```

The loop condition is

```
i != n && array[i] < x
```

With the loop invariant and loop condition, you can come up with the code below.

``` c
int skip_02(int array[], int n, int x) {
    int i = 0;

    while (i != n && array[i] < x) { 
        i++;
    }

    return i;
}
```

This function looks better and it is easy to verify its correctness. Let's check if the
loop invariant holds true using the annotated code below.

``` c
int skip_02(int array[], int n, int x) {
    int i = 0;

    // loop inv: numbers in array[-infinity...i - 1] are less than `x`     (A)
    while (i != n && array[i] < x) {                                    // (B)
        // loop inv: numbers in array[-infinity...i - 1] are less than `x` (C)
        i++;                                                            // (D)
        // loop inv: numbers in array[-infinity...i - 1] are less than `x` (E)
    }
    // loop inv: numbers in array[-infinity...i - 1] are less than `x`     (F)

    return i;
}
```

The comments in the code above indicate places where the loop invariant should be true.

- (A): at this point, `i=0`. We can fill imaginary numbers that are smaller
than `x` to `array[-infinity...-1]`. The loop invariant is true here.
- (B): this line does not change any variable. If the loop invariant is true
before executing this line, the loop invariant will still be true after this line.
- (C): because loop invariant is true at (A) and (E) (let's just assume the loop 
invariant holds at (E) at this moment), it must be true at (C), because (B) does not change anything. 
In fact, all numbers in `array[-infinity...i]` are less than `x` in (C).
- (D): advance the index
- (E): For ease of discussion, let's assume `i=k` at (C). So at (E), `i=k+1`. We already
know that all numbers in `array[-infinity...k]` are less than `x` at (C). Because `k=i-1`
at (E), loop invariant "`all numbers in array[-infinity...i-1]` are less than `x`" holds 
at (E).
- (F): The execution can only reach (F) directly from (A) or (E), where the loop invariant
holds. So the loop invariant also holds at (F).

Now let's check if `i` points to the right number after the loop terminates.
After the loop, the following conditions are true due to the termination condition
and the loop invariant.

```
(i == n OR array[i] >= x) AND numbers in array[-infinity...i - 1] are less than `x`
```

We can expand and get:

```
(i == n AND numbers in array[-infinity...i - 1] are less than `x`)
OR
i != n AND array[i] >= x AND numbers in array[-infinity...i - 1] are less than `x`
```

In plain English, they mean either of the following

1. `i` points to the element right after the last existing element of the array and 
all numbers in the array are less than `x`.
2. `array[i]` is larger than or equal to `x` and all numbers before `array[i]` are less than `x`.

The proof is not rigid, but in practice, it is sufficient to help us write better loops.
A nice thing about using loop invariants is that it is **a uniform way of thinking about 
loops**. In contrast, people often use ad-hoc methods to reason about loops and worry 
about their correctness.


