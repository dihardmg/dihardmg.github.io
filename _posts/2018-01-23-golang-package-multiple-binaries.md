---
layout: post
tags: [go, golang, multiple, binaries, structure]
title: "Packing multiple binaries in a Golang package"
card-img: img/cards/golang-package-multiple-binaries.png
---

Recently, while writing a small Golang program for setting reminders I came
across a small confusion that I guess most newcomers to Golang will have - how
to organise a package in a way that will enable it to cleanly contain two or
more binaries.

This post is not aimed at experienced Golang programmers, it's mostly aimed at
beginners to understand how to compose more complex packages, beyond making the
usual "one package one binary" ones. It's essentially what I would like to have
read (or understand) after spinning my wheels for a bit while building my first
(more complex) package.

But, if you are one of the experienced folks I would be very grateful if you
finish reading this article and call out any mistakes you might find. Also I,
and probably other beginners, would be happy to find out any other alternative
approaches to this problem.

That being said, let's begin by seeing what an elementary package layout looks
like, and then we can continue with building a more complex one.

## Fortune telling

Imagine we need to build a CLI program in Golang. One that needs to tell us
our fortune, or provide advice on a subject that we are thinking about. Just like
a [Magic 8-Ball](https://en.wikipedia.org/wiki/Magic_8-Ball).

Let's begin by fleshing out the `main` file of this package:

{% highlight go %}
// eight_ball.go
package main

import (
	"bufio"
	"fmt"
	"math/rand"
	"os"
	"time"
)

func main() {
	rand.Seed(time.Now().Unix())
	reader := bufio.NewReader(os.Stdin)
	answers := []string{
		"It is certain",
		"It is decidedly so",
		"Without a doubt",
		"Yes, definitely",
		"You may rely on it",
		"As I see it, yes",
		"Most likely",
		"Outlook good",
		"Yes",
		"Signs point to yes",
		"Reply hazy try again",
		"Ask again later",
		"Better not tell you now",
		"Cannot predict now",
		"Concentrate and ask again",
		"Don't count on it",
		"My reply is no",
		"My sources say no",
		"Outlook not so good",
		"Very doubtful",
	}

	fmt.Print("What is your question? ")
	reader.ReadString('\n')

	fmt.Println(answers[rand.Intn(len(answers))])
}
{% endhighlight %}

The implementation is quite straightforward. We have a list of predefined
answers, blatantly ripped off from
[Wikipedia's article on Magic 8-ball](https://en.wikipedia.org/wiki/Magic_8-Ball). The
program takes the input from `STDIN`, ignores it (obviously) and with the help of
the `rand` package takes a random answer from the array, which is then printed
to `STDOUT`.

We can test this by running `go run eight_ball.go`:

{% highlight bash %}
› go run main.go
What is your question? Will I win the lottery this year?
My sources say no
{% endhighlight %}

I guess I won't be getting rich by wining the lottery. Now, let's throw in
another program in our package. This one, following the same theme, can be about
fortune cookies.

{% highlight go %}
// cookie.go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	rand.Seed(time.Now().Unix())
	quotes := []string{
		"The beginning of wisdom is to desire it.",
		"You will have a very pleasant experience.",
		"You will inherit some money or a small piece of land.",
		"You will live a long, happy life.",
		"You will spend old age in comfort and material wealth.",
		"You will step on the soil of many countries.",
		"You will take a chance in something in the near future.",
		"You will witness a special ceremony.",
		"Your everlasting patience will be rewarded sooner or later.",
		"Your great attention to detail is both a blessing and a curse.",
		"Your heart is a place to draw true happiness.",
		"Your ability to juggle many tasks will take you far.",
		"A friend asks only for your time, not your money.",
		"You will be invited to an exciting event.",
	}

	fmt.Println(quotes[rand.Intn(len(quotes))])
}
{% endhighlight %}

If you are carefully reading the code, you can see the approach is basically the
same. We have a list of predefined quotes, and using the `math/rand` package
we print out a random quote when the program is executed.

If we run the program, we would get the following output:

{% highlight bash %}
› go run cookie.go
You will live a long, happy life.
{% endhighlight %}

Nice and simple. The program works flawlessly.

## Building it

So how can we build this now in a binary, and distrubite it
for different operating systems and architectures. It's really one of the things
that Go really shines for, right?

If you have done any Go you would immediately say: using `go build`. Let's give
that a shot:

{% highlight bash %}
› go build
# github.com/fortune_telling
./eight_ball.go:11:6: main redeclared in this block
	previous declaration at ./cookie.go:9:6
{% endhighlight %}

Whoops, what happenned? Well, the error is quite self-descriptive. Basically we
have the `main` function defined in both of these files. But, on the other hand
if we rename those functions they will not be run when we create binaries out
of the programs. Let's try to rename the `main` function as `foo` in
`eight_ball.go`, and then try to run the program using `go run`:

{% highlight bash %}
› go run eight_ball.go
# command-line-arguments
runtime.main_main·f: relocation target main.main not defined
runtime.main_main·f: undefined: "main.main"
{% endhighlight %}

As you can see, then the binary cannot be built, since Go's compiler complains
about the missing `main` function. On the other hand, the missing `main`
function in `eight_ball.go` will actually make `go build` work.

Why is that? Well you see, in a package with a name `foo` there can only be one
`main` function. When the package is built, that `main` function will be the
entry point for the program (in the binary), so it's mandatory the package has it.
Since `eight_ball.go` is missing it, and `cookie.go` has it, the package `main`
will have a single `main` function (in `cookie.go`), rendering the `main` package
as valid.

## Split the packages

Okay, so by now hopefully it's clear that we cannot have two binaries with two
entry points in a single package. So, why don't we split the files in two packages.
For example, `cookie.go` can have a `cookie` package, while `eight_ball.go` can
have an `eight_ball` package. Both of them will have a `main` function and
everything should work smoothly, right?

Let's do that and give it a shot. First, let's rename the packages in the
`cookie.go` and `eight_ball.go` files respectively:

{% highlight go %}
// cookie.go
package cookie

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	// Snipped
}
{% endhighlight %}

{% highlight go %}
// eight_ball.go
package eight_ball

import (
	"bufio"
	"fmt"
	"math/rand"
	"os"
	"time"
)

func main() {
	// Snipped
}
{% endhighlight %}

Let's try to build these packages now:

{% highlight bash %}

› go build cookie.go

{% endhighlight %}

{% highlight bash %}
› go build eight_ball.go

{% endhighlight %}

Okay, cool. We ran `go build` on the files and there were no problems. That means
that Go produced the binaries for each of these files and we could run them.

Nope. It didn't.

If we see [Go's documentation](https://golang.org/cmd/go/#hdr-Compile_packages_and_dependencies)
on the `go build` command, we will find this segment:

> When compiling a single main package, build writes the resulting executable to
> an output file named after the first source file ('go build ed.go rx.go' writes
> 'ed' or 'ed.exe') or the source code directory ('go build unix/sam' writes 'sam'
> or 'sam.exe'). The '.exe' suffix is added when writing a Windows executable.

Also, this:

> When compiling multiple packages or a single non-main package, build compiles
> the packages but discards the resulting object, serving only as a check that
> the packages can be built.

This means that, to produce a binary of a package, whose name will be derived
from the folder name it is stored in, we need to build a `main` package. That
means that our `cookie.go` and `eight_ball.go` files will have to be contained
in their own folders, while their package names have to stay `main`.

## Reorganizing our files

What we need to do is quite simple actually. In our working directory let's
introduce a folder called `cmd`, which will store both of our commands.

Let's run the following commands:

{% highlight bash %}
› mkdir -p cmd/{cookie,eight_ball}/

› mv cookie.go cmd/cookie/main.go

› mv eight_ball.go cmd/eight_ball/main.go
{% endhighlight %}

If we run `tree` on our working directory, we will see the following structure:

{% highlight bash %}
› tree
.
└── cmd
    ├── cookie
    │   └── main.go
    └── eight_ball
        └── main.go
{% endhighlight %}

If we try to build the packages now we won't get far as well:

{% highlight bash %}
› go build
can't load package: package github.com/fortune_teller: no Go files in /Users/ie/projects/go/src/github.com/fortune_teller
{% endhighlight %}

But, the restructuring of the packages allows us to now use the `go install ./...`
command, which will install our packages in our `$GOPATH/bin` directory:

{% highlight bash %}
› go install ./...

› cookie
A thrilling time is in your immediate future.

› eight_ball
What is your question? Will I win the lottery this year?
Don't count on it
{% endhighlight %}

Whoa, so, what happenned? How did these programs just got installed?

What happenned is the following - since we moved the programs to their own
subfolders in our workspace, when built (and installed) they will inherit the
name of the folder they are in. Therefore, `cmd/cookie/main.go` will compile
into a `cookie` binary, while `cmd/eight_ball/main.go` will compile into a
`eight_ball` binary. Then, after buildilng, these binaries will be installed
into the Go binaries path (`$GOPATH/bin`).

If you would like to build the binaries for each of the two packages, without
installing them to `$GOPATH/bin`, then you will need to `cd` in to the paths
where the `main.go` files are, and run `go build` there:

{% highlight bash %}
› cd cmd/cookie

› go build

› ./cookie
Your greatest fortune is the large number of friends you have.
{% endhighlight %}

{% highlight bash %}
› cd fortune_teller/cmd/eight_ball

› go build

› ./eight_ball
What is your question? Will I win the lottery?
Yes, definitely
{% endhighlight %}

## What if I want a library as part of the package?

This question makes sense - what if with the package one wants to ship some
additional code. Or even better, what if you would like to `import` code from
the package in the CLI programs?

Let's extract the `answers` and `quotes` variables from the CLI programs into a
new package, and try to import that one back in:

{% highlight go %}
// content.go
package fortune_teller

func Answers() []string {
	return []string{
		"It is certain",
		"It is decidedly so",
		"Without a doubt",
		"Yes, definitely",
		"You may rely on it",
		"As I see it, yes",
		"Most likely",
		"Outlook good",
		"Yes",
		"Signs point to yes",
		"Reply hazy try again",
		"Ask again later",
		"Better not tell you now",
		"Cannot predict now",
		"Concentrate and ask again",
		"Don't count on it",
		"My reply is no",
		"My sources say no",
		"Outlook not so good",
		"Very doubtful",
	}
}

func Quotes() []string {
	return []string{
		"There is a true and sincere friendship between you and your friends.",
		"You find beauty in ordinary things, do not lose this ability.",
		"Ideas are like children; there are none so wonderful as your own.",
		"It takes more than good memory to have good memories.",
		"A thrilling time is in your immediate future.",
		"Your blessing is no more than being safe and sound for the whole lifetime.",
		"Plan for many pleasures ahead.",
		"The joyfulness of a man prolongeth his days.",
		"Your everlasting patience will be rewarded sooner or later.",
		"Make two grins grow where there was only a grouch before.",
		"Something you lost will soon turn up.",
		"Your heart is pure, and your mind clear, and your soul devout.",
		"Excitement and intrigue follow you closely wherever you go!",
		"A pleasant surprise is in store for you.",
		"May life throw you a pleasant curve.",
		"As the purse is emptied the heart is filled.",
	}
}
{% endhighlight %}

After extracting these two functions in a top level package, which will return 
the content for the two commands, we can `import` the package in the `cookie`
and `eight_ball` commands, which will utilise the content. By being able to
extract code in a common package, it allows us to easily reuse any
of the shared code between multiple commands.

For example, if we had a database driver that would write to a database
(i.e. `sqlite`), we could easily `import` this driver in both `cookie` and
`eight_ball` and use the code within the context of the program.

Let's modify now the CLI programs to `import` the top-level package:

{% highlight go %}
// cmd/cookie/main.go
package main

import (
	"fmt"
	"math/rand"
	"time"

	ft "github.com/fteem/fortune_teller"
)

func main() {
	rand.Seed(time.Now().Unix())
	quotes := ft.Quotes()
	fmt.Println(quotes[rand.Intn(len(quotes))])
}
{% endhighlight %}

{% highlight go %}
// cmd/eight_ball/main.go
package main

import (
	"bufio"
	"fmt"
	"math/rand"
	"os"
	"time"

	ft "github.com/fteem/fortune_teller"
)

func main() {
	rand.Seed(time.Now().Unix())
	reader := bufio.NewReader(os.Stdin)
	answers := ft.Answers()
	fmt.Print("What is your question? ")
	reader.ReadString('\n')

	fmt.Println(answers[rand.Intn(len(answers))])
}
{% endhighlight %}

It's as simple as that. Here we purposely alias the package to `ft`, so we don't
have to type `fortune_teller` every time we want to invoke these functions. To
test the commands manually, the simplest way is to install them using
`go install ./...`, which will install all of the packages that can be found in
the path (and all of it's subpaths).

## It works!

After trying this approach out, I noticed that some other popular packages use
a similar structure of building packages. For example, you can see that `BoltDB`
[places](https://github.com/boltdb/bolt) it's command line program
in a `cmd/bolt` path. So does [packr](https://github.com/gobuffalo/packr), and
other various libraries that I have noticed. So, I would safely assume that
this pattern of organising your packages is clean and safe to use.

Since you got to the end, I will assume that you would like some more reading
resources around building and installing packages, and file structure organisation.
I recommend you continue your journey in Go with the following links:

- [How to Write Go Code](https://golang.org/doc/code.html)
- [Command go - Compile packages and dependencies](https://golang.org/cmd/go/#hdr-Compile_packages_and_dependencies)
- [Structuring Applications in Go](https://medium.com/@benbjohnson/structuring-applications-in-go-3b04be4ff091) - I think this is probably the origin of the structural pattern used in this article

If you would like to see the code used in this article, you can get it
[from my Github](https://github.com/fteem/fortune_teller).
