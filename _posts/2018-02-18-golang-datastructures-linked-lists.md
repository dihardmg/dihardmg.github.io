---
layout: post
tags: [go, datastructures, linked, lists]
title: "Data structures in Go: Linked lists"
card-img: img/cards/golang-datastructures-linked-lists.png
---

Data structures and algorithms are the bread and butter of computer science.
Although sometimes they appear scary to people, most of them have a simple
explanation. Also, when explained well with a problem algorithms can be
interesting and fun to learn and apply.

This post is aimed at people that are not comfortable with linked lists, or
folks that want to see and learn how to build one with Golang. We will see
how we can implement them with Go via a (somewhat) practical example, instead of
the plain theory and code examples.

But first, let's touch on a bit of theory.

## Linked lists

Linked lists are one of the simpler data structures out there. Wikipedia's
article on linked lists states that:

> In computer science, a linked list is a linear collection of data elements, in
> which linear order is not given by their physical placement in memory.
> Instead, each element points to the next. It is a data structure consisting of
> a group of nodes which together represent a sequence. Under the simplest form,
> each node is composed of data and a reference (in other words, a link) to the
> next node in the sequence.

While all this might look like too much or confusing, let's break it down. A
linear data structure is the one where it's elements form a sequence of some
sort. Simple as that. Now, why the physical placement in memory doesn't matter?
Well, when you have arrays the amount of memory the array takes is fixed, in the
sense that if you have an array of 5 items, the language will grab only 5 memory
addresses in the memory, one after another. Because these addresses create a
sequence, the array knows in what memory range its values will be stored and
thus the physical placement in-memory of these values create a sequence.

With linked lists, it's a bit different. In the definition you will notice that
"each element points to the next", using "data and a reference (in other words,
a link) to the next node". This means that each node of the linked list stores
two things: a value and a reference to the next node in the list. Simple as
that.

## Streams of data

Everything that humans perceive around them is some sort of information or
rather data that our senses and our minds know how to process and convert it
into a useful piece of information. Whether we look, or smell, or touch we
process data and find meaning from that data. Or, when we browse our social
media networks we always resort to a feed of data, ordered chronologically and
with *no end in sight*.

So, how can we use linked lists to model such a news feed? Let's first quickly
take a glance at a simple Tweet for example:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">just setting up my twttr</p>&mdash; jack (@jack) <a href="https://twitter.com/jack/status/20?ref_src=twsrc%5Etfw">March 21, 2006</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

For the purposes of our example social network, let's get inspired by Twitter
and create a `Post` type, that has a `body`, a `publishDate` and a link to the
`next` post:

{% highlight go %}
type Post struct {
	body         string
	publishDate int64 // Unix timestamp
	next *Post // link to the next Post
}
{% endhighlight %}

Next, how can we model a feed of posts? Well, if we know that feeds consist of
posts that are linked to another post, then we can try to create a type like
that:

{% highlight go %}
type Feed struct {
	length int // we'll use it later
	start  *Post
}
{% endhighlight %}

The `Feed` struct will have a beginning (or `start`) which will point to the
first `Post` in the feed and a `length` property which will store the size of
the `Feed` at any given moment.

So, let's say we want to create a `Feed` with two posts, the first step is to
create an `Append` function on the `Feed` type:

{% highlight go %}
func (f *Feed) Append(newPost *Post) {
	if f.length == 0 {
		f.start = newPost
	} else {
		currentPost := f.start
		for currentPost.next != nil {
			currentPost = currentPost.next
		}
		currentPost.next = newPost
	}
	f.length++
}
{% endhighlight %}

Then we can invoke it, twice:

{% highlight go %}
func main() {
        f := &Feed{}
        p1 := Post{
            body: "Lorem ipsum",
        }
	f.Append(&p1)

	fmt.Printf("Length: %v\n", f.length)
	fmt.Printf("First: %v\n", f.start)

        p2 := Post{
            body: "Dolor sit amet",
        }
	f.Append(&p2)

	fmt.Printf("Length: %v\n", f.length)
	fmt.Printf("First: %v\n", f.start)
	fmt.Printf("Second: %v\n", f.start.next)
}
{% endhighlight %}

So, what does this code do? First, the `main` function - it creates a pointer to
a `Feed` struct, two `Post` structs with some dummy content and it invokes twice
the `Append` function on the `Feed`, which results in it having a length of 2.
We inspect the two values that the `Feed` has by accessing the `start` of the
`Feed` (which is, in fact, a `Post`) and the `next` item after the `start`,
which is the second `Post`.

When we run the program, the output will look something like:

{% highlight text %}
Length: 1
First: &{Lorem ipsum 1257894000 <nil>}
Length: 2
First: &{Lorem ipsum 1257894000 0x10444280}
Second: &{Dolor sit amet 1257894000 <nil>}
{% endhighlight %}

You can see, after we add the first `Post` to the `Feed`, the length is `1` and
the first `Post` has a `body` and a `publishte` (as a Unix timestamp), while
it's `next` value is `nil` (because there's no more `Post`s in the `Feed`).
Then, we add the second `Post` to the `Feed` and when we inspect the two `Post`s
we see that the first one has the same content as before, but with a pointer to
the `next` `Post` in the list. The second also has a `body` and a `publishDate`
but with no pointer to the next `Post` in the list. Also, the length of the
`Feed` increases as we add more `Post`s to it.

Let's go back to the `Append` function now and deconstruct it so we understand
better how to work with linked lists. First, the function creates a pointer to a
`Post` value, with the `body` argument as `body` of the `Post` and the
`publishDate` is set to the Unix timestamp representation of the current time.

Then, we check if the `length` of the `Feed` is `0` - this means that it has no
`Post`s and the first one that is added should be set as the starting `Post`,
conveniently named `start`.

But, if the length of the `Feed` is more than 0, then our algorithm takes a
different turn. It will start with the `start` of the `Feed` and it will walk
through all the `Post`s until it finds one that doesn't have a pointer to a
`next` one. Then, it will attach the new `Post` as the `next` value on the last
`Post` of the list.

### Optimising `Append`

Imagine we have a user that scrolls through their `Feed`, like on any other
social network. Since posts are chronologically ordered, based on the
`publishDate`, the `Feed` will grow more and more as the user scrolls through it
and more `Post`s are attached to the `Feed`. Given the approach, we took in the
`Append` function, as the `Feed` gets longer and longer the `Append` function
will become more and more expensive. Why? Because we have to traverse the whole
`Feed` to add a `Post` at the end of it. If you have heard about *Big-O
notation*, this algorithm has an `O(n)` time complexity, which means that it
will always traverse the whole `Feed` before it adds a `Post` to it.

As you can imagine, this can be quite inefficient, especially if the `Feed`
grows to be quite long. How can we improve the `Append` function and decrease
the [asymptotic
complexity](https://en.wikipedia.org/wiki/Asymptotic_computational_complexity)
of it?

See, since our `Feed` data structure is just a list of `Post`s, to traverse it
we have to know the beginning of the list (called `start`) that's a pointer of
type `Post`. Because in our example `Append` always adds a `Post` to the end of
the `Feed`, we could drastically improve the performance of the algorithm if
`Feed` knows not just of its `start`ing element, but also about it's `end`ing
element. Of course, there's always a tradeoff with optimisations, and the
tradeoff here is that the data structure will consume a bit more memory (for the
new attribute on the `Feed` structure).

Extending our `Feed` data structure is quite easy:

{% highlight go %}
type Feed struct {
	length int
	start  *Post
	end    *Post
}
{% endhighlight %}

But our `Append` algorithm will have to be tweaked to work with the new
structure of a `Feed`. This is the version of `Append` using the `end` attribute
of `Post`:

{% highlight go %}
func (f *Feed) Append(newPost *Post) {
	if f.length == 0 {
		f.start = newPost
		f.end = newPost
	} else {
		lastPost := f.end
		lastPost.next = newPost
		f.end = newPost
	}
	f.length++
}
{% endhighlight %}

That looks a bit simpler, doesn't it? Let me give you some good news:
1. The code is simpler and shorter now, and
2. We drastically improved the time complexity of the function

If you look now at the algorithm, it does two things: if the `Feed` is empty it
will set the new `Post` as the beginning and the end of the `Feed`, otherwise it
will set the new `Post` as the `end` item and it will attach it to the previous
`Post` in the list. On top of how simple it is, this will algorithm now has a
big-O complexity of `O(1)`, also known as "constant time". That means that
`Append` will perform the same regardless of the length of the `Feed` structure.

Pretty simple, right? But let's imagine that the `Feed` is actually a list of
`Post`s on our profile. Hence they are ours, we should be able to delete them. I
mean, what kind of a social network doesn't allow it's users to (at least)
delete their posts?

## Removing a `Post`

As we established in the previous section, we want the users of our `Feed`s to
be able to delete their posts. So, how can we model that? If our `Feed`s were an
array, we would just remove the item and be done with it, right?

Well, this is actually where linked lists shine. When arrays sizes change, the
runtime has to capture a new memory block to store the items of the array in.
Linked lists due to their design, each item having a pointer to the next node in
the list, can be scattered throughout the memory space meaning adding/removing
nodes from the list is cheap from a space perspective. When one wants to remove
a node from a linked list only the neighbours of the removed node need to be
linked, and that is. Garbage collected languages (like Go) make this even easier
since we don't have to worry about releasing the allocated memory - the GC will
kick in and remove all unused objects.

To make our lives a bit easier for this example, let's put a constraint that
each of the `Posts` on a `Feed`will have a unique `publishDate`.This means a
publisher can create one `Post` per second on their `Feed`. Taking that into
effect, this is how we can easily remove a `Post` from a `Feed`:

{% highlight go %}
func (f *Feed) Remove(publishDate int64) {
	if f.length == 0 {
		panic(errors.New("Feed is empty"))
	}

	var previousPost *Post
	currentPost := f.start

	for currentPost.publishDate != publishDate {
		if currentPost.next == nil {
			panic(errors.New("No such Post found."))
		}

		previousPost = currentPost
		currentPost = currentPost.next
	}
	previousPost.next = currentPost.next

	f.length--
}
{% endhighlight %}

The `Remove` function will take a `publishDate` of a `Post` as an argument by
which it will detect what `Post` needs to be deleted (or unlinked). The function
is rather small. If it detects that the `start` item of the `Feed` is to be
removed it will just reassign the `start` of the `Feed` with the second `Post`
in the `Feed`. Otherwise, it jumps through each of the `Post`s in the `Feed`
until it runs into a `Post` that has a matching `publishDate` to the one passed
as the function argument. When it finds one, it will just connect the previous
and the next `Post` in the `Feed` with each other, effectively dropping the
middle (matching) one from the `Feed`.

There's one edge case that we need to make sure that we cover in our `Remove`
function - what if the `Feed` doesn't have a `Post` with the specified
`publishDate`? To keep it simple, the function checks for the absence of the
`next` `Post` in the `Feed` before it jumps to it. If the `next` is `nil` the
function `panic`s, telling us that it couldn't find a `Post` with such
`publishDate`.

## Inserting a `Post`

Now that we got appending and removing out of the way, let's take a look at (a
bit of) a hypothetical case. Imagine that the source that produces the `Post`s
sends them to our application in a non-chronological order. This means that the
`Post`s need to be put in the correct order in the `Feed`, based on the
`publishDate`. This is how that would look like:

{% highlight go %}
func (f *Feed) Insert(newPost *Post) {
	if f.length == 0 {
		f.start = newPost
	} else {
		var previousPost *Post
		currentPost := f.start

		for currentPost.publishDate < newPost.publishDate {
			previousPost = currentPost
			currentPost = previousPost.next
		}

		previousPost.next = newPost
		newPost.next = currentPost
	}
	f.length++
}
{% endhighlight %}

In essence, this is a very similar algorithm to the one in the `Remove`
function, because although both of them do a very different thing (adding v.s.
removing a `Post` in the `Feed`), they are both based on a *search* algorithm.
That means that both of the functions actually traverse the whole `Feed`,
*searching* for a `Post` that matches the `publishDate` with the one received in
the argument of the function. The only difference is that `Insert` will actually
put the new `Post` in the place where the dates match, while `Remove` will
remove the `Post` from the `Feed`.

Additionally, this means that both of these functions carry the same time
complexity, which is `O(n)`. This means that in a worst-case scenario, the
functions will have to traverse the whole `Feed` to get to the item where the
new post needs to be inserted (or the removed).

## What if we used arrays?

If you are asking that yourself, let me say right up front - you have a point.
True, we could store all of the `Post`s in an array (or a slice in Go), easily
push items onto it and also even have random access with an `O(1)` complexity.

Due to the nature of arrays, whose values have to be stored in memory one right
after another, reading is really fast and cheap. Once you have something stored
in an array, retrieving it as easy as accessing it by its 0-based index. When it
comes to inserting an item, whether in the middle or at the end, then arrays
become less efficient compared to lists. That is because if the array doesn't
have more memory reserved for the new item(s), it will have to reserve it and
use it. But, if the next memory addresses are not free, it will have to "move"
to a new memory address where there would be space for all of the items in it
(new and old).

Looking at all of the examples and discussion we had so far, we can create a
table with time complexity for each of the algorithms we created, and compare
them with the same algorithms for arrays:

{% highlight text %}
╔═══════════╦═════════╦═════════════╗
║ Action    ║ Array   ║ Linked list ║
╠═══════════╬═════════╬═════════════╣
║ Access    ║ O(1)    ║ O(n)        ║
║ Search    ║ O(n)    ║ O(n)        ║
║ Prepend   ║ O(1)    ║ O(1)        ║
║ Append    ║ O(n)    ║ O(1)        ║
║ Delete    ║ O(n)    ║ O(n)        ║
╚═══════════╩═════════╩═════════════╝
{% endhighlight %}

As you can see, when you are faced with a certain problem, picking the correct
data structure can really make or break the products you create. For
ever-growing `Feed`s, where insertion of `Post`s is paramount, linked lists do a
much better job because insertions are really cheap. But, if we had a different
problem on our hands that requires frequent deletions or lots of
retrieval/searching, then we would have to pick the correct data structure for
the problem we are dealing with.

You can see the whole implementation of the `Feed` and play with it
[here](https://play.golang.org/p/fqLPjf_ekD6). Also, Go has it's own linked list
implementation, with some nice functions already built in. You can see it's
documentation [here](https://golang.org/pkg/container/list/).

*EDIT* 20.02.2018 10:00 CET: Previous verion of the article wrongly stated that
the big-O complexity of the delete function is `O(1)`. Changed to `O(n)` after
pointed out by [Bart de Goede](https://twitter.com/bartdegoede).

*EDIT* 22.02.2018 23:00 CET: Previous verion of the article wrongly stated that
the big-O complexity of searching in an array is `O(1)` instead of `O(N)`. This
was a mixup of *Access* and *Search* on my side. This is fixed by adding
separate rows in the table for *Search* and *Access* and their respective time
complexities.

*EDIT* 25.02.2018 17:00 CET: Previous version of the article had a buggy
implementation of the `Insert` function, which was pointed out by Albert Shirima
in the comments.
