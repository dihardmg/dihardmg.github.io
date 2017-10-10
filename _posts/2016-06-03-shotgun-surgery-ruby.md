---
layout: post
title: "Refactoring in Ruby: Shotgun Surgery"
tags: [ruby, dci]
image: shotgun-surgery-ruby.png
---

In our everyday job as software engineers, we try to build businesses while 
caring about code quaility and cost of maintenance. The always try to write the
best code we can, we do code reviews and try to improve the general quality of
the software. But, we are often faced with code that has a high cost of
maintenance, or as we like to call it - bad code. In this post, we will see one
very popular design smell with some steps of refactoring that smell, which
ultimately leads to decrease that cost.

## Shotgun surgery

The design smell that we will go over in this post is called shotgun surgery. 
So, what is it? Shotgun surgery is a smell that is represented by enforcing 
changes in multiple classes when we are trying to introduce a single coherent 
change in the behavior of our software. Basically, if you try to change 
something very small, you end up with a huge Git diff.

But, what does actually shotgun surgery mean? If you think a bit more broadly,
it means that there are multiple classes that have a single responsibility. 
Instead of having one class that has one responsibility, which is one of the
S.O.L.I.D. principles, we have multiple classes that share that responsibility.
This design smell although might seem obvious, it's quite tricky to detect. Why?
Well, first you need to define what "single coherent change" means. Usually,
to discover shotgun surgery you need to find methods with similar behaviour, that
require the same (or very similar) modificaiton to introduce the change in
behaviour. If you detect a shotgun surgery, you are most likely looking at code
that is not DRYed up. In other words, there are entities (methods, functions,
classes, modules, etc) that have the same or very similar code (or algorithm).
Also, if you take a step back and take a good look, you will see bad (or 
nonexistent) separation of concerns.

So, let's see an example, of what does shotgun surgery looks and feels like, with
the required steps to refactor it.

## Bank accounts

Let's imagine we are working on a small Ruby application that allows users to
track their budget. Mainly it will support users, bank accounts, credit, debit,
and history of money traffic. We will have a `User` class, and a `BankAccount`
class. The `User` object encapsulates the user data, while having a associated
`BankAccount` object, that holds the state of the user's bank account.

Let's see this in code:

{% highlight ruby %}
class User
  def initialize(first_name, last_name)
    @first_name = first_name
    @last_name = last_name
    @account = BankAccount.new
  end
end
{% endhighlight %}

Then, the `BankAccount` class:

{% highlight ruby %}
class BankAccount
  def initialize
    @balance = 0
  end

  def credit(amount)
    @balance += amount
  end

  def debit(amount)
    @balance -= amount
  end
end
{% endhighlight %}

Great stuff. So, now, we want to 

