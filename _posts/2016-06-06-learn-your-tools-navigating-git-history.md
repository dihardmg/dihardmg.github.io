---
layout: post
title: "Learn your tools: Navigating your Git History"
tags: [git, history]
image: learn-your-tools-navigating-git-history.png
header-img: img/posts/learn-your-tools-navigating-git-history.jpg
card-img: img/cards/learn-your-tools-navigating-git-history.png
---

Starting a greenfield application everyday is nearly impossible, especially in
your daily job. In fact, most of us are facing (somewhat) legacy codebases on a
daily basis, and regaining the context of why some feature, or line of code
exists in the codebase is very important. This is where `git`, the distributed 
version control system, is invaluable. Let's dive in and see how we can use our
`git` history and easily navigate through it.

## Git history

First and foremost, what is `git` history? As the name says, it is the commit
history of a `git` repo. It contains a bunch of commit messages, with their 
authors' name, the commit hash and the date of the commit. The easiest way to 
see the history of a `git` repo, is the `git log` command. 

Sidenote: For the purpose of this post, we will use Ruby on Rails' repo, the 
`master` branch. The reason behind this is because Rails has a very good `git` 
history, with nice commit messages, references and explanations behind every 
change. Given the size of the codebase, the age and the number of maintainers, 
it's certainly one of the best repositories that I have seen. Of course, I am 
not saying there are no other repositories built with good `git` practices, but 
this is one that has caught my eye.

So back to Rails' repo. If you run `git log` in the Rails' repo, you will see 
something like this:

{% highlight text %}
commit 66ebbc4952f6cfb37d719f63036441ef98149418
Author: Arthur Neves <foo@bar.com>
Date:   Fri Jun 3 17:17:38 2016 -0400

    Dont re-define class SQLite3Adapter on test

    We were declaring  in a few tests, which depending of
    the order load will cause an error, as the super class could change.

    see https://github.com/rails/rails/commit/ac1c4e141b20c1067af2c2703db6e1b463b985da#commitcomment-17731383

commit 755f6bf3d3d568bc0af2c636be2f6df16c651eb1
Merge: 4e85538 f7b850e
Author: Eileen M. Uchitelle <foo@bar.com>
Date:   Fri Jun 3 10:21:49 2016 -0400

    Merge pull request #25263 from abhishekjain16/doc_accessor_thread

    [skip ci] Fix grammar

commit f7b850ec9f6036802339e965c8ce74494f731b4a
Author: Abhishek Jain <foo@bar.com>
Date:   Fri Jun 3 16:49:21 2016 +0530

    [skip ci] Fix grammar

commit 4e85538dddf47877cacc65cea6c050e349af0405
Merge: 082a515 cf2158c
Author: Vijay Dev <foo@bar.com>
Date:   Fri Jun 3 14:00:47 2016 +0000

    Merge branch 'master' of github.com:rails/docrails

    Conflicts:
        guides/source/action_cable_overview.md

commit 082a5158251c6578714132e5c4f71bd39f462d71
Merge: 4bd11d4 3bd30d9
Author: Yves Senn <foo@bar.com>
Date:   Fri Jun 3 11:30:19 2016 +0200

    Merge pull request #25243 from sukesan1984/add_i18n_validation_test

    Add i18n_validation_test

commit 4bd11d46de892676830bca51d3040f29200abbfa
Merge: 99d8d45 e98caf8
Author: Arthur Nogueira Neves <foo@bar.com>
Date:   Thu Jun 2 22:55:52 2016 -0400

    Merge pull request #25258 from alexcameron89/master

    [skip ci] Make header bullets consistent in engines.md

commit e98caf81fef54746126d31076c6d346c48ae8e1b
Author: Alex Kitchens <foo@bar.com>
Date:   Thu Jun 2 21:26:53 2016 -0500

    [skip ci] Make header bullets consistent in engines.md
{% endhighlight %}

As you can see, the `git log` shows the commit hash, the author and his email
and the date of when the commit was created. Of course, `git` being super 
customisable, it allows you to customise the output format of the `git log` 
command. Let's say, we want to just see the first line of the commit message,
we could run `git log --oneline`, which will produce a more compact log:

{% highlight text %}
66ebbc4 Dont re-define class SQLite3Adapter on test
755f6bf Merge pull request #25263 from abhishekjain16/doc_accessor_thread
f7b850e [skip ci] Fix grammar
4e85538 Merge branch 'master' of github.com:rails/docrails
082a515 Merge pull request #25243 from sukesan1984/add_i18n_validation_test
4bd11d4 Merge pull request #25258 from alexcameron89/master
e98caf8 [skip ci] Make header bullets consistent in engines.md
99d8d45 Merge pull request #25254 from kamipo/fix_debug_helper_test
818397c Merge pull request #25240 from matthewd/reloadable-channels
2c5a8ba Don't blank pad day of the month when formatting dates
14ff8e7 Fix debug helper test
{% endhighlight %}

To see all of the `git log` options, I recommend checking out manpage of 
`git log`, available in your terminal via `man git-log` or `git help log`. 
A tip: if `git log` is a bit scarse or complicated to use, or maybe you are just
bored, I recommend checking out various `git` GUIs and command line tools. In
the past I've used [GitX](http://gitx.frim.nl/) which was very good, but since 
the command line feels like home to me, after trying 
[tig](https://github.com/jonas/tig) I've never looked back.

## Finding Nemo

So now, since we know the bare minimum of the `git log` command, let's see how
we can explore the history more effectively in our everyday work.

Let's say, hypothetically, we are suspecting an unexpected behaviour in the
`String#classify` method and we want to find how and where it has been 
implemented.

One of the first commands that you can use, to see where the method is defined,
is `git grep`. Simply said, this command prints out lines that match a certain
pattern. Now, to find the definition of the method, it's pretty simple - we can
grep for `def classify` and see what we get:

{% highlight bash %}
➜  git grep 'def classify'

activesupport/lib/active_support/core_ext/string/inflections.rb:  def classify
activesupport/lib/active_support/inflector/methods.rb:    def classify(table_name)
tools/profile:    def classify
{% endhighlight %} 

Now, although we can already see where our method is created, we are not sure
on which line it is. If we add the `-n` flag to our `git grep` command, `git` 
will provide the line numbers of the match:

{% highlight bash %}
➜  git grep -n 'def classify'

activesupport/lib/active_support/core_ext/string/inflections.rb:205:  def classify
activesupport/lib/active_support/inflector/methods.rb:186:    def classify(table_name)
tools/profile:112:    def classify
{% endhighlight %} 

Much better, right? Having the context in mind, we can easily figure out that
the method that we are looking for lives in `activesupport/lib/active_support/core_ext/string/inflections.rb`, on line 205. The `classify` method, in all of it's glory looks like
this:

{% highlight ruby %}
# Creates a class name from a plural table name like Rails does for table names to models.
# Note that this returns a string and not a class. (To convert to an actual class
# follow +classify+ with +constantize+.)
#
#   'ham_and_eggs'.classify # => "HamAndEgg"
#   'posts'.classify        # => "Post"
def classify
  ActiveSupport::Inflector.classify(self)
end
{% endhighlight %}

Although the method we found is the one we usually call on `String`s, it invokes
another method on the `ActiveSupport::Inflector`, with the same name. Having our
`git grep` result available, we can easily navigate there, since we can see the
second line of the result being 
`activesupport/lib/active_support/inflector/methods.rb` on line 186. The method
that we are are looking for is:

{% highlight ruby %}
# Creates a class name from a plural table name like Rails does for table
# names to models. Note that this returns a string and not a Class (To
# convert to an actual class follow +classify+ with #constantize).
#
#   classify('ham_and_eggs') # => "HamAndEgg"
#   classify('posts')        # => "Post"
#
# Singular names are not handled correctly:
#
#   classify('calculus')     # => "Calculus"
def classify(table_name)
  # strip out any leading schema name
  camelize(singularize(table_name.to_s.sub(/.*\./, ''.freeze)))
end
{% endhighlight %}

Boom! Given the size of Rails, finding this should not take us more than 30 
seconds with the help of `git grep`.

## So, what changed last?

Now, since we have the method available, we need to figure out what were the
changes that this file has gone through. The since we know the correct file name
and line number, we can use `git blame`. This command shows what revision and 
author last modified each line of a file. Let's see what were the latest changes
made to this file:

{% highlight bash %}
git blame activesupport/lib/active_support/inflector/methods.rb
{% endhighlight %}

Whoa! Although we get the last change of every line in the file, we are more
interested in the specific method (lines 176 to 189). Let's add a flag to
the `git blame` command, that will show the blame of just those lines. Also, we
will add the `-s` (suppress) option to the command, to skip the author names and 
the timestamp of the revision (commit) that changed the line:

{% highlight bash %}
git blame -L 176,189 -s activesupport/lib/active_support/inflector/methods.rb

9fe8e19a 176)     # Creates a class name from a plural table name like Rails does for table
5ea3f284 177)     # names to models. Note that this returns a string and not a Class (To
9fe8e19a 178)     # convert to an actual class follow +classify+ with #constantize).
51cd6bb8 179)     #
6d077205 180)     #   classify('ham_and_eggs') # => "HamAndEgg"
9fe8e19a 181)     #   classify('posts')        # => "Post"
51cd6bb8 182)     #
51cd6bb8 183)     # Singular names are not handled correctly:
5ea3f284 184)     #
66d6e7be 185)     #   classify('calculus')     # => "Calculus"
51cd6bb8 186)     def classify(table_name)
51cd6bb8 187)       # strip out any leading schema name
5bb1d4d2 188)       camelize(singularize(table_name.to_s.sub(/.*\./, ''.freeze)))
51cd6bb8 189)     end
{% endhighlight %}

The output of the `git blame` command now shows all of the file lines and their
respective revisions. Now, to see a specific revision, or in other words, what
each of those revisions changed, we can use the `git show` command. When 
supplied a revision hash (like `66d6e7be`) as an argument, it will show you the 
full revision, with the author name, timestamp and the whole revision in it's 
glory. Let's see what actually changed at the latest revision that changed line 
188:

{% highlight bash %}
git show 5bb1d4d2
{% endhighlight %}

Whoa! Did you test that? If you didn't, it's an awesome 
[commit](https://github.com/rails/rails/commit/5bb1d4d288d019e276335465d0389fd2f5246bfd)
by [Schneems](https://twitter.com/schneems) that made a very interesting 
performance optimization by using frozen strings, which makes sense in our 
current context. But, since we are on this hypothetical debugging session, this 
doesn't tell much about our current problem. So, how can we see what changes has 
our method under investigation gone through?

## Searching the logs

Now, we are back to the `git` log. The question is, how can we see all the 
revisions that the `classify` method went under?

The `git log` command is quite powerful, because it has a rich list of options
to apply to it. We can try to see what the `git` log has stored for this file,
using the `-p` options, which means show me the patch for this entry in the
`git` log:

{% highlight bash %}
git log -p activesupport/lib/active_support/inflector/methods.rb
{% endhighlight %}

This will show us a big list of revisions, for every revision of this file. But,
just like before, we are interested in the specific file lines. Let's modify the
command a bit, to show us what we need:

{% highlight bash %}
git log -L 176,189:activesupport/lib/active_support/inflector/methods.rb
{% endhighlight %}

The `git log` command accepts the `-L` option, which takes the lines range and
the filename as arguments. The format might be a bit weird for you, but it
translates to:

{% highlight bash %}
git log -L <start-line>,<end-line>:<path-to-file>
{% endhighlight %}

When we run this command, we can see the list of revisions for these lines,
which will lead us to the first revision that created the method:

{% highlight bash %}
commit 51xd6bb829c418c5fbf75de1dfbb177233b1b154
Author: Foo Bar <foo@bar.com>
Date:   Tue Jun 7 19:05:09 2011 -0700

    Refactor

diff --git a/activesupport/lib/active_support/inflector/methods.rb b/activesupport/lib/active_support/inflector/methods.rb
--- a/activesupport/lib/active_support/inflector/methods.rb
+++ b/activesupport/lib/active_support/inflector/methods.rb
@@ -58,0 +135,14 @@
+    # Create a class name from a plural table name like Rails does for table names to models.
+    # Note that this returns a string and not a Class. (To convert to an actual class
+    # follow +classify+ with +constantize+.)
+    #
+    # Examples:
+    #   "egg_and_hams".classify # => "EggAndHam"
+    #   "posts".classify        # => "Post"
+    #
+    # Singular names are not handled correctly:
+    #   "business".classify     # => "Busines"
+    def classify(table_name)
+      # strip out any leading schema name
+      camelize(singularize(table_name.to_s.sub(/.*\./, '')))
+    end
{% endhighlight %}

Now, look at that - it's a commit from 2011. Practically, `git` allows us to 
travel back in time. This is a very good example of why a proper commit message 
is paramount to regain context, because from the commit message we cannot really
regain context of how this method came to be. But, on the flip side, you should
**never ever** get frustrated about it, because you are looking at someone that 
basically gives away his time and energy for free, doing open source work.

Coming back from that tangent, we are not sure how the initial implementation
of the `classify` method came to be, given that the first commit is just a
refactor. Now, if you are thinking something within the lines of "but maybe, just
maybe, the method was not on the line range 176 to 189, and we should look more
broadly in the file", you are very correct. The revision that we saw said 
"Refactor" in it's commit message, which means that the method was actually 
there, but after that refactor it started to exist on that line range. 

So, how can we confirm this? Well, believe it or not, `git` comes to the rescue
again. The `git log` command accepts the `-S` option, which looks for the code
change (additions or deletions) for the specified string as an argument to the
command. This means that, if we call `git log -S classify`, we can see all of
the commits that changed a line that contains that string.

If you call this command in the Rails repo, you will first see `git` slowing 
down a bit. But, when you realise that `git` actually parses all of the 
revisions in the repo to match the string, it's actually super fast. Again, the 
power of `git` at your fingertips. So, to find the first version of the 
`classify` method, we can run:

{% highlight bash %}
git log -S 'def classify'
{% endhighlight %}

This will return all of the revisions where this method has been introduced or
changed. If you were following along, the last commit in the log that you will
see is:

{% highlight ruby %}
commit db045dbbf60b53dbe013ef25554fd013baf88134
Author: David Heinemeier Hansson <foo@bar.com>
Date:   Wed Nov 24 01:04:44 2004 +0000

    Initial

    git-svn-id: http://svn-commit.rubyonrails.org/rails/trunk@4 5ecf4fe2-1ee6-0310-87b1-e25e094e27de
{% endhighlight %}

How cool is that? It's the initial commit to Rails, made on a `svn` repo by DHH!
This means that `classify` has been around since the beginning of (Rails) time.
Now, to see the commit with all of it's changes, we can run:

{% highlight bash %}
git show db045dbbf60b53dbe013ef25554fd013baf88134
{% endhighlight %}

Great, we got to the bottom of it. Now, by using the output from 
`git log -S 'def classify'` you can track the changes that have happened to this
method, combined with the power of the `git log -L` command.

## Until next time

Sure, we didn't really fix any bugs, because we were trying some `git` commands
and following along the evolution of the `classify` method. But, nevertheless,
`git` is a very powerful tool that we all must learn to use and to embrace. I
hope this article gave you a little bit more knowledge of how useful `git` is.

What are your favourite (or, most effective) ways of navigating through the `git`
history?

