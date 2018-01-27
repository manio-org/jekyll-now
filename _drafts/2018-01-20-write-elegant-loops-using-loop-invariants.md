---
layout: post
title:  "Write Elegant Loops Using Loop Invariants"
date:   2018-01-20
categories: coding
---

Writing loops is an essential part of programming; but even writing loops to
handle simple tasks is not easy. For example, Linus showed an ugly but commonly
used piece of code which deletes an entry in a link list (we will discuss this
example). Writing loops is large becuase multiple variables changes. Keeping
track of these variables and making sure they are in the right states when
exiting require lots of mental energy.

How can we reason about the logic of loops better and write elegant loops?
Before answering the questions, let me describe what elegant loops are.
Elegant loops have the following characteristics.

1. They look beautiful.
2. They are easy to understand.
3. Their correctness is easy to prove.

Here are some smells of ugly loops:

1. Muliple exit conditions in the loop body, which make it hard to determine
the exit status of the loop.
2. Specical case handling before or after the loop. For example, treating
the first or the last element of an array differently.


So, how can we reason about the logic of loops better and write elegant loops?
My answer is to find a good loop invariant and use it to construct the loop. I 
will review what a loop invariant is and demonstrate the idea using two examples:
deleting an entry in a linked list (by Linus) and find a number that is larger than
a given number in a sorted array.

Let us explain what a loop invariant is by the following code, which moves the `1`s
from `a` to `b`.

``` c
int a = 10
int b = 0;

// invariant: a + b == 10                (A)
while (a != 0) {                         (B)
    // invariant: a + b == 10            (C)
    a--;
    b++;
    // invariant: a + b == 10            (D)
}
// invariant: a + b == 10                (E)

// post condition: a == 0 && a + b == 10
```

`a + b == 10` is one of the loop invariants of the loop. The invariant is true
at marked locations. `a != 0` is the loop condition. We know the loop condition
is _false_ after the loop; we also know that the loop invariant is _true_ after
loop.  Combining these two conditions, we get the post condition 
(`a == 0 && a + b == 10`) which indicates `b == 0`. Now we know for sure that all the `1` in `a` has been moved to `b`.

Loop invariant (A) must be true right before `while`, otherwise it will not be
true at the beginning of the loop (C), because line (B) changes nothing.

Note that if the loop body does not execute at all (because the loop codition
is false), the loop invariant set at (A) still holds after the loop (E).

Now, writing an elegant loop becomes:

1. Find an elegant loop invariant
2. Determine a loop condition so `loop condition == False && loop invariant == true` 
is what you want when the loop finishes.

Let's practice by examining two examples.

# Example 1: Linus's Taste

In a TED interview, Linus gave the following piece of code to show what a
bad taste of code is. 

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

The diagram below is an illustration of the basic data structure. 

![Simple Link List]({{ "/assets/images/simple-link-list.png" | absolute_url }})

Linus hates the code because it has to handle a special case (the `if` statement)
after the loop. He argued that the programmer failed to see the common pattern
between the specical case and the regular case. 

I think the programmer wrote the ugly `remove_list_entry_01()` because 
he/she did not find a good loop invariant, while the loop invariant of 
`remove_list_entry_02()` is better. Now let us see what their loop invariants
are.

## Loop invariant of the ugly loop and its improvements

In `remove_list_entry_01()`, the loop invariant is:

```
None of the nodes before `walk` is equal to `entry`.
```



So the post condition is:

```
`walk == entry` && none of the nodes before `walk` is equal to `entry`.
```

The reason that `remove_list_entry_01()` has to handle a specical case is 
that `prev` is not in the loop invariant. Depending on whether the loop
has been executed, `prev` can be `NULL`, or pointing to a valid node.

If the loop invariant is the following, then we would not need to handle
two cases after the loop.

```
None of the nodes before `walk` is equal to `entry` 
&& 
`prev` points to a node before `walk`
```

We can make this loop invariant happen by adding header node, so that even
if the loop never executes, `prev` always points to a node before `walk`.
The initial structure is shown below.

![Link List with header]({{ "/assets/images/link-list-with-header.png" | absolute_url }})


``` c
/*
Header -> Node1 -> Node2 -> ...
*/
remove_list_entry_02(entry)
{
    prev = head; // head points to Header
    walk = head->next;

    while (walk != entry) {
        prev = walk;
        walk = walk->next;
    }

    walk->next = entry->next;
}
```

But this is overkill because it consumes more memory. 


## Loop invariant of the elegant loop

Linus gave the following elegant solution, which eliminates the special case by
introducing a pointer that points to the variable that will update.

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
there is a imaginary header node just like the one in `remove_list_entry_02()`.
`head` can be thought as the `next` member of the header node. `indirect` could
have pointed to the imaginary header node if it really existed. In reality, we
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

The diagram below shows a possible state after the loop, where `indirect` points
to the `next` of the node to be modified.

![Link List last state]({{ "/assets/images/link-list-last-state.png" | absolute_url }})

**To find an elegant loop invariant, you probably need to draw diagrams like
the ones above to find the common pattern between special cases and regular cases.**


# Example 2: looking for a number











