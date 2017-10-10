---
layout: post
title: "What is the best trait a developer should have?"
header-img: img/post-bg-01.jpg
---

## What is a programmer? Or developer? Or software engineer?

Whichever term you want to use, we all think about the same thing. We are people
that teach machines to do whatever we want, right? Back in the day, engineers
were writing machine instructions to make machines do things. Later, we
invented programming languages.

But, what really is a software engineer? Yes, we write code that a machine
understands, and we make it does magic for us. We have machines available in the
whole world, where with a single command we can push our code and affect the lives
of thousands, if not millions, of people.

We shape the current reality that we live in. We have the power and the
responsibility to make our users' lives easier. And the coolest part is that we
are builders, we make things out of thin air.

## What does **being** a programmer mean?

Scenario 1: You are employed by a company, just like Catawiki. You are given
tasks, you communicate with the product owners, the product managers, ops,
customer support. You solve issues, you fix bugs, you mentor less experienced
folks, you help your colleagues, you do code reviews. But most important - you
build an experience. The product that you build is YOURS, you embrace it and you
improve the experience to the best of your knowledge.

Scenario 2: You enjoy FOSS. You are hired by a company to work on architectural
changes, contribute to the open source software that your company uses. Think
about well-known people in the Ruby community as Aaron Patternson and the likes.

Scenario 3: You work as a consultant, which means you charge companies to help
them solve problems. You bill by the hour, or per week, and you face clients and
various issues very frequently. Although it's a bit more of a risky job (compared
to the traditional approaches), it's a bit more rewarding.

There are plenty of scenarios, but after all, what does being a programmer mean?

I like to think that it’s about being a problem solver. You are given a problem,
i.e. speed up the auction lot wizard. You analyse the problem by diving into the
code, you try to find the bottleneck and you put on paper (or in “Notes”) a
proposed solution. Maybe you could build a prototype, evaluate it and, if
plausible, throw it away and build a proper solution.

Wherever you are employed, however you do your job, there’s only one general
truth - you are a problem solver.

## Doing THE job

We come to work every day. We sit down and open our laptops, our editor and we
dive into the code. You pull the Jira ticket out, you check the description of
the issue at hand. Then you probably message your Project Manager (or whoever
added the ticket in Jira) about more clarifiaction, because "Put a link in the
header to the profile page" sounds easy, but it involves a lot of detail.

Being an engineer you are dealing with so much detail every day, and it requires
a massive amount of focus and energy. It also involves pure intelligence - to
build "the brain of a machine" requires a more advanced brain. But as we all know
there's no perfect engineer. Even the smartest one get it wrong sometimes. That's
why it's hard.

Keeping in mind that doing this job is quite hard without thinking about the code
at all, let's jump right into the code. Building a "brain" requires a lot of
mental power, and the complexity of it can be flabbergasting. So, how do we make
our lives easier? How do we reduce the brainpower we need to use to complete a
task? How can we save some brain-cycles, so we can feel fresher and more
productive?

Well, there are couple of things - some of them are easy to achieve, others are
quite hard.

## Writing readable and idiomatic code

Ruby is a very pretty language. Yeah, I know what I said. Pretty. When you see
a piece of clean Ruby code, it's pretty! It feels like human language. It’s like
you opened a novel that you want to read on a Sunday night.

Ruby was built as a language to make developers happy. Yes, remember that - it's
focus is not performance, nor parallelism, nor concurrency. Plain happinness.
That's why it's striving, and succeeding, to be the most expressive language on
the market right now.

So, how do we write readable and idiomatic code? First and foremost, it's good
to note that writing readable and idiomatic code *is not harder or time consuming*
than any other form of code, whether you call it bad or something else. Like
everything else in life, achieving this skill only takes practice and a clear
goal. Then as your skillset gets better your code will get better. There's no
shortcut to achieving this.

Well used Ruby idioms allow us to express our logic in a clean and maintainable
way. Jokingly, the other day, me and my colleagues were trying to make up some
absurd conditional code, like:

```ruby
def is_invalid?
  return false unless valid?
  return true
end
```

Although this was a joke, when another colleagues entered the room after the
fooling-around was done, he freaked out when he saw the code. But, believe it
or not, some of us still write this type of code. So, how do we refactor this to
make it more readable?

Well, although to most of us it might seem obvious on how to refactor this, it's
always good to think about intention. What is the intenion of this method? To
check if an object has an invalid state. So, given that you already have the
`valid?` method implemented:

```ruby
def is_invalid?
  !valid?
end
```

Next thing is naming. Do you think that this method is properly named? I don't.
Since Ruby allows you to use the question mark in the method signature, why not
use the chance to be expressive?

```ruby
def invalid?
  !valid?
end
```

The question mark implies that the result of the method will be a boolean, which
already implies that it will answer the question "is this invalid?". Therefore,
we can freely drop the `is_` from the method name.

Another example is sorting. Ruby's `Enumerable` module allows us to do all sorts
of operations to collections, and it's awesome. But, with great power comes
somewhat of a great responsibility. Let's look at a very simple example:

```ruby
max_item = array.sort.last
```

We want to get the max item in an array. The solution sorts the array and then we
return the last item, which is the largest (or the maximum) item. Sure, the
solution works. But, think about the mental load that this simple operation
imposes on you as an engineer. Sure, you might think, this is super simple and
I am a cry-baby. I disagree. Do you know how many times in a normal work day
you do these logical “exercises”? Think about every line of code you write and
you read, and I am sure you will end up with a number larger than 10 thousand.
And that's **per day**.

So how can we lower the cognitive load in our code? Clean, simple and
battle-tested idioms.

Why not:

```ruby
max_item = array.max
```

Our brains, or at least mine, can much more clearly see the intent behind this
snippet, over the previous one. It's not a huge improvement, but it's the small
refactoring steps that we take that improve the overall condition of the
codebase. A huge refactor almost never helps - on the contrary, it most often
creates more mess.

## Abstractions

OOP is fun, especially in Ruby, right? Right now on the market, it's the purest
OOP language out there. Also, it's dynamic, which makes working with it even
smoother.
One of my colleagues that has Java background, started with Ruby couple of months
ago. Every time we talk about Ruby he says that he is surprised by how easy is to
do stuff with Ruby. It’s super flexible and powerful. But we always forget that
with great power comes a shit-ton of responsibility.

Think about maintenance of the code you write. How easy is it going to be
maintained in the future? Will your colleagues know how to extend that code? A
wise man once said:

> When you write your code, think that the maintainer of that code will be a
> psychopath that knows where you live.

Or, in Commitstrip's words:

![](http://www.commitstrip.com/wp-content/uploads/2016/01/Strip-Voyage-dans-le-temps-650-finalenglish-2.jpg)

### So, how much abstraction is good?

Let’s talk about Active Record. AR is the Object-Relational Mapper for Ruby on
Rails. It allows us to use Ruby to query the database, fetch data and it builds
objects for the associations and records in the database. It allows us to easily
fetch the attributes of the objects, their associations, use the methods from the
Enumerable module and so much more. Some say AR is bloated, other say it’s
awesome.

As always, it’s about personal preference. You might like it, or you might hate
it, but the truth is that Active Record is very powerful.

Let's look at a very tiny example of Active Record usage:

```ruby
user = User.find(params[:user_id])
posts = user.posts
```

Although most of us write something like this snippet almost every day, most of
the time we do it mechanically. What does this say to us? A user has an `id`?
This is probably a piece of code from a controller? User has many posts?

Well, sure. But let's take a step back and look at it again? This small piece of
code, although simple, tells a lot about an application. This is a User-centric
application. Users have complex behaviour on it. Probably complex things like
authentication, authorisation and sessions are included as well.

One more step back? We are talking about a behaviour of a user on a web
application. The User can (probably) create posts. This means that we are talking
about a blog CMS or something similar.

You see, you can say so much about an app from couple of lines of code. This is
why abstraction is powerful - it won't just hide your code complexity, but it
will also help with the application behaviour. But, to choose the right
abstraction to apply you **must know the domain** inside out.

## Cut to the chase
