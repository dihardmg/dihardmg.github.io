---
layout: post
tags: [go, datastructures, queues, priority queues, stacks]
title: "Data structures in Go: Stacks and queues"
card-img: img/cards/golang-datastructures-stacks-queues.png
acknowledgements: "Big thanks to <a href='https://bytes-and-bites.com/' target='_blank'>Vincent Boisard</a> for reviewing a final draft of this post and his constructive criticism on it."
---

In a previous post, we took a look at linked lists and how we can apply them in
a hypothetical use-case. In this post, we will look at two similar but powerful
data structures.

## Modelling actions and history

Think about Excel or Google docs. You know, the most ubiquitous applications
for composing documents that humanity has invented. We've all used them in some
capacity. As you might know, these apps come with various actions one can
apply to a text. Like adding colour to the text, underline, various fonts and
sizes or organising the content in tables. The list is long, but there's one
ubiquitous functionality that we expect from these tools - the ability to
"Undo" and "Redo" the actions we have applied.

Have you ever considered how you would program such functionality if given the
opportunity? Let's explore a data structure that can help us with such a task.

![](https://i.imgur.com/Y9Jbm5A.png)

Let’s try to think how we would model the actions each of these applications
have. Also, later on, we will see how we can keep the history of the actions
and "Undo" them.

A simple `Action` `struct` would look something like:

{% highlight go %}
type Action struct {
	name string
	metadata Meta
	// Probably some other data here...
}
{% endhighlight %}

We will store only the name at the moment, while the real-deal would have much
more options compared to our example. So now, when the user of the editor
applies a function to a bunch of text, we want to store that action in some
sort of a collection so we can "Undo" it later.

{% highlight go %}
type ActionHistory struct {
  top *Action
  size int
}
{% endhighlight %}

Looking at this `ActionHistory` data structure, we store a pointer to the
`Action` at the top of the stack and the size of the stack. Once an action is
applied, we will link it to the `top` item. So, when applying an action to the
document, this is what could go under the hood.

{% highlight go %}
func (history *ActionHistory) Apply(newAction *Action) {
	if history.top != nil {
		oldTop := history.top
		newAction.next = oldTop
	}
	history.top = newAction
	history.size++
}
{% endhighlight %}

An `Add` function would add the latest `Action` to the top of the
`ActionHistory`. If the history struct has an action present at the top, it
would push it down the history by linking it to the new action. Otherwise, it
will attach the new action to the top of the list. Now, if you know about
linked lists (or read [my last post about linked lists in
Go](https://ieftimov.com/golang-datastructures-linked-lists)) you might see the
resemblance here. So far, what we use here is essentially a linked list, none
the less.

What about undoing an action? This is what an `Undo` function would look like:

{% highlight go %}
func (history *ActionHistory) Undo() *Action {
	topAction := history.top
	if topAction != nil {
		history.top = topAction.next
	} else if topAction.next == nil {
		history.top = nil
	}
	historyAction.size--
	return topAction
}
{% endhighlight %}

If you are paying close attention you can notice that this is a bit different
than just removing a node from a linked list. Due to the nature of an
`ActionHistory`, we want the **last** applied action to be the one that
will be `Undo`ed **first**. Which makes sense, right?

This is the default behaviour of a stack. A stack is a data structure where you
can only insert or delete items at the top of the stack. Think of it like a
stack of papers, or stack of plates in your kitchen drawer.  If you want to
take the bottom plate from a stack you are going to have a hard time. But
taking the one at the top is plain simple. Stacks are also considered LIFO
structures - meaning Last In First Out, because of the behaviour we explained
above.

That is basically what our `Undo` function does. If the stack (a.k.a.
`ActionHistory`) has more than one `Action` in it, it will set the top link for
the second item. Otherwise, it will empty up the `ActionHistory` by setting the
`top` element to `nil`.

From a big-O perspective, searching through a stack has `O(n)` complexity,
but insertion and deletion from a stack are super quick - `O(1)`. This is
because traversing the whole stack, in worst case scenario, will still take
going all `n` items in it, while inserting and removing items is constant time
because we always insert and remove from the top of the stack.

You can play with the working version of this code
[here](https://play.golang.org/p/Eu8_-HTDBY_A).

## Luggage control

Most of us have travelled by plane and know all of the security checks that a
person has to go through to get on the plane. Sure, it’s for own our safety but
sometimes it’s unnecessarily stressful to get through all of the scans, checks
and tests.

One of the usual sights at airports security checkpoints are the long queues of
people and luggage being put on the strip of the X-ray machines while people
walk through metal detector gates. Probably there’s more to it that we even
don’t know about it, but let’s focus on the X-ray machines that scan our bags.

Have you thought how would you model the interactions that happen on that
machine? Of course, at least the visible ones. Let’s explore this idea for a
moment. We would have to somehow represent the luggage on the strip as a
collection of items, and the x-ray machine that scans the luggage one piece at
a time.

The `Luggage` would be simple, for now:

{% highlight go %}
type Luggage struct {
	weight int
	passenger string
}
{% endhighlight %}

We can also add a simple constructor function for the `Luggage` type as well:

{% highlight go %}
func NewLuggage(weight int, passenger string) *Luggage {
	l := Luggage{
		weight:    weight,
		passenger: passenger, // just as an identifier
	}
	return &l
}
{% endhighlight %}

Then, we can create a `Belt` where the `Luggage` is put on and taken to the
x-ray scanner:

{% highlight go %}
type Belt []*Luggage
{% endhighlight %}

Not what you were expecting? Well, what we create is a `Belt` type that is
actually a slice of `Luggage` pointers. Which is what exactly the machine belt
is - just a collection of bags that get scanned one by one.

So now we need to add a function that knows how to add `Luggage` to the `Belt`:

{% highlight go %}
func (belt *Belt) Add(newLuggage *Luggage) {
	*belt = append(*belt, newLuggage)
}
{% endhighlight %}

Since `Belt` is actually a slice, we can use the built-in `append` function
which will add the `newLuggage` to the `Belt`.  The cool part about this
implementation the time complexity - because we use `append`, which is a
built-in method, we get an **amortised** `O(1)` for insertion. Of course, this
comes at space cost, due to [the way slices in Go
work](https://blog.golang.org/go-slices-usage-and-internals).

As the `Belt` moves and takes `Luggage` to the x-ray machine, we will need to
somehow take this luggage off and load it into the machine for inspection. But
because of the nature of the `Belt`, the first item that’s put on it is the
first item to be scanned. And the last one that is added will be the last one
to be scanned. So, we can say that the `Belt` is a FIFO (First in, first out)
data structure.

This is what the function `Take`  would look like, keeping in mind the details
above:

{% highlight go %}
func (belt *Belt) Take() *Luggage {
	first, rest := (*belt)[0], (*belt)[1:]
	*belt = rest
	return first
}
{% endhighlight %}

It will take the first item and return it, and it will assign everything else
in the collection to the beginning of it,  so the second item becomes first and
so on. If you were wondering, this has a `O(1)` time complexity since it always
takes the first item from the queue.

Using our new types and functions can be done as such:

{% highlight go %}
func main() {
	belt := &Belt{}
	belt.Add(NewLuggage(3, "Elmer Fudd"))
	belt.Add(NewLuggage(5, "Sylvester"))
	belt.Add(NewLuggage(2, "Yosemite Sam"))
	belt.Add(NewLuggage(10, "Daffy Duck"))
	belt.Add(NewLuggage(1, "Bugs Bunny"))

	fmt.Println("Belt:", belt, "Length:", len(*belt))
	first := belt.Take()
	fmt.Println("First luggage:", first)
	fmt.Println("Belt:", belt, "Length:", len(*belt))
}
{% endhighlight %}

The output of the `main` function will be something like:
```
Belt: &[0x1040a0c0 0x1040a0d0 0x1040a0e0 0x1040a100 0x1040a110] Length: 5
First luggage: &{3 Elmer Fudd}
Belt: &[0x1040a0d0 0x1040a0e0 0x1040a100 0x1040a110] Length: 4
```

Basically what happens is we add five different `Luggage` structs on the `Belt`
and we take the first one off, which is the one printed on the second line of
the output.

You can play with the code from this example
[here](https://play.golang.org/p/DTFUkWeZ4H8).

### What about first-class passengers?

Yeah, what about them? I mean, they’ve paid so much money for their ticket that
it just doesn’t make sense for them to wait in queues as the economy ticket
holders. So, how can we prioritise these passengers? What if their luggage has
some sort of priority, where the higher the priority is the faster they get
through the queue?

Let’s modify our `Luggage` struct to enable this:

{% highlight go %}
type Luggage struct {
	weight    int
	priority  int
	passenger string
}
{% endhighlight %}

Also, all `Luggage` created in the `NewLuggage` function will have to take the
`priority` level as an argument:

{% highlight go %}
func NewLuggage(weight int, priority int, passenger string) *Luggage {
	l := Luggage{
		weight:    weight,
		priority:  priority,
		passenger: passenger,
	}
	return &l
}
{% endhighlight %}

Let’s think this through again. Basically, when a new `Luggage` is put on the
`Belt`, we need to detect it’s `priority` and put it as close to the beginning
of the `Belt` as possible based on the detected `priority`.

Let’s modify our `Add` function:

{% highlight go %}
func (belt *Belt) Add(newLuggage *Luggage) {
	if len(*belt) == 0 {
		*belt = append(*belt, newLuggage)
	} else {
		added := false
		for i, placedLuggage := range *belt {
			if newLuggage.priority > placedLuggage.priority {
				*belt = append((*belt)[:i], append(Belt{newLuggage}, (*belt)[i:]...)...)
				added = true
				break
			}
		}
		if !added {
			*belt = append(*belt, newLuggage)
		}
	}
}
{% endhighlight %}

Compared to the previous implementation, this is rather complicated. There are
a couple of things going on here and the first case is quite simple. If the
belt is empty,  we just put the new luggage on the belt and we are done. The
`Belt` will have only one item on it and that one will be the first that can be
taken off of it.

The second case, when there’s more than one item on the `Belt`, loops through
all of the luggage on the `Belt` and compares their `priority` with the new one
coming on. Then it finds a `Luggage` that has a smaller priority it bypasses it
and puts the new luggage in front of it. This means, the higher the `priority`
the `Luggage` has the further to the beginning of the `Belt` it will be placed.

Of course, if the loop doesn’t find such a `Luggage` it will just append it to
the end of the `Belt`. Our new `Add` function has a time complexity of `O(n)`
because in worst case scenario it will have to traverse the whole slice before
it inserts the new `Luggage` struct. Inherently, searching and accessing any
item in our queue will cost us the same - `O(n)`.

To demonstrate the new `Add` functionality, we can run the following code:

{% highlight go %}
func main() {
	belt := make(Belt, 0)
	belt.Add(NewLuggage(3, 1, "Elmer Fudd"))
	belt.Add(NewLuggage(3, 1, "Sylvester"))
	belt.Add(NewLuggage(3, 1, "Yosemite Sam"))
	belt.Inspect()

	belt.Add(NewLuggage(3, 2, "Daffy Duck"))
	belt.Inspect()

	belt.Add(NewLuggage(3, 3, "Bugs Bunny"))
	belt.Inspect()

	belt.Add(NewLuggage(100, 2, "Wile E. Coyote"))
	belt.Inspect()
}
{% endhighlight %}

First, it will create a `Belt` with three `Luggage` structs on it, each of them
with a priority of `1`:

```
0. &{3 1 Elmer Fudd}
1. &{3 1 Sylvester}
2. &{3 1 Yosemite Sam}
```

Then, it will add a new `Luggage` with a priority of `2`:

```
0. &{3 2 Daffy Duck}
1. &{3 1 Elmer Fudd}
2. &{3 1 Sylvester}
3. &{3 1 Yosemite Sam}
```

You see, the new luggage with the highest priority got promoted to the first
place in the `Belt`. Next, we add a new one with even higher priority (`3`):
```
0. &{3 3 Bugs Bunny}
1. &{3 2 Daffy Duck}
2. &{3 1 Elmer Fudd}
3. &{3 1 Sylvester}
4. &{3 1 Yosemite Sam}
```

As expected, the one with the highest priority was put on the first spot in the
`Belt`. Lastly, we add another `Luggage`, this time with priority `2`:
```
0. &{3 3 Bugs Bunny}
1. &{3 2 Daffy Duck}
2. &{100 2 Wile E. Coyote}
3. &{3 1 Elmer Fudd}
4. &{3 1 Sylvester}
5. &{3 1 Yosemite Sam}
```

The new `Luggage` was added right after the `Luggage` with the same priority as
the new one and of course, not at the beginning of the `Belt`. Basically, our
`Belt` gets implicitly sorted by the priority as we add new `Luggage` to it.

If you are knowledgeable about queues you might think that these are not the
most performant ways of implementing priority queues and you are quite right.
Implementing priority queues can be done more performant using heaps, that
we’ll take a look at in another post.

There’s more interesting knowledge around priority queues that we can explore.
Until then you can take a look at priority queues’ [Wiki
page](https://en.wikipedia.org/wiki/Priority_queue). If you are knowledgeable
about queues you might think that these are not the most performant ways of
implementing priority queues and you are quite right.  Implementing priority
queues can be done more performant using heaps, that we’ll take a look at in
another post.), especially the “Implementation” section.

You can see and play with the code from this example
[here](https://play.golang.org/p/Eu8_-HTDBY_A).

## Closing thoughts

In this post, we took a look at what stacks and queues are and we tried to mimic
some interesting real-world application to them. I strongly believe that when
we learn a topic (especially in computer science) it’s much better to do it
through examples and application to a problem and I hope I managed to do
exactly that.

In further posts we will look at some more priority queues (using heaps), but
before we get there we will take a look at some a bit simpler data structures.
