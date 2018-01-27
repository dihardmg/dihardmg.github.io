---
layout: post
tags: [go, golang, multiple, binaries, structure]
title: "Giving Launchd a hug using Go"
card-img: img/cards/golang-package-multiple-binaries.png
---

If you have ever tried writing a daemon for OSX you have met with Launchd. For
those that don't have the experience, think of it as a framework for starting,
stopping and managing daemons, applications, processes, and scripts. If you have
any \*nix experience the word daemon should not be too alien to you.

For those unfamiliar, a daemon is a program running in the background without
requiring user input. A typical daemon might for instance perform daily
maintenance tasks, or scan a device for malware when it is connected.

This post is aimed at folks that know a little bit about what daemons are, what
is the common way of using them, and know a bit about Go. Also, if you have
ever written a daemon for any other \*nix system, you will have a really good
idea about what we are going to talk here. If you are an absolute beginner in
Go or systems, this might prove to be challenging article. Nevertheless, feel
free to give it a shot and let me know how it goes.

Back on track - if you ever find yourself in a place where you want to write a
daemon with Go for OSX you would like to know most of the stuff we are going
to talk in this article. So, let's dive in.

## What is `launchd` and how it works?

As we mentioned at the beginning of this article, it is a unified
service-management framework, that starts, stops and manages daemons,
applications, processes, and scripts in macOS.

One of it's key features is that it differentiates between agents and daemons.
In `launchd` land, an agent is run on behalf of the logged in user while a
daemon runs on behalf of the root user or any specified user.

### Defining agents and daemons

Simply put, a agent/daemon is defined in an XML file, which states all of the
properties of the program that will be executed, among a list of other properties.
Another aspect to keep in mind is that `launchd` decides if a program will be
treated as an daemon on ar agent just by where the process XML is placed.

Over at [launchd.info](launchd.info), there's a really simple table that shows
where you would (or not) place your program's XML:

{% highlight text %}
+----------------+-------------------------------+----------------------------------------------------+
| Type           | Location                      | Run on behalf of                                   |
+----------------+-------------------------------+----------------------------------------------------+
| User Agents    | ~/Library/LaunchAgents        | Currently logged in user                           |
| Global Agents  | /Library/LaunchAgents         | Currently logged in user                           |
| Global Daemons | /Library/LaunchDaemons        | root or the user specified with the key 'UserName' |
| System Agents  | /System/Library/LaunchAgents  | Currently logged in user                           |
| System Daemons | /System/Library/LaunchDaemons | root or the user specified with the key 'UserName' |
+----------------+-------------------------------+----------------------------------------------------+
{% endhighlight %}

This means that when we set our XML file in, for example, the `/Library/LaunchAgents`
our process will be treated as a global agent. The main difference between the
daemons and agents is that LaunchDaemons will run as root, and are generally
background processes. On the other hand LaunchAgents are jobs, that will run as
a user or in the context of userland. These may be scripts or other foreground
items and they also have access to the MacOS UI (e.g. you can send notifications
in them).

So, how do we define an agent? Let's take a look at a simple XML file that
`launchd` understands:

{% highlight xml %}
<!--- Example blatantly ripped off from http://www.launchd.info/ -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
	<dict>
		<key>Label</key>
		<string>com.example.app</string>
		<key>Program</key>
		<string>/Users/Me/Scripts/cleanup.sh</string>
		<key>RunAtLoad</key>
		<true/>
	</dict>
</plist>
{% endhighlight %}

The XML is quite self-explanatory, unless it's the first time you are seeing
an XML file. The file has three main properties, with values. In fact, if you
take a better look you will see the `dict` keyword which means `dictionary`. This
actually means that the XML represents a key-value structure, so in Go it would
look like:

{% highlight go %}
map[string]string{
        "Label":     "com.example.app",
        "Program":   "/Users/Me/Scripts/cleanup.sh",
        "RunAtLoad": "true",
}
{% endhighlight %}

Let's look at the keys individually:
1. `Label` - The job definition, basically the name of the job. This is the
unique identifier for the job within the `launchd` instance. Usually the label
(and hence the name) is written in
[Reverse domain name notation](en.wikipedia.org/wiki/Reverse_domain_name_notation).
2. `Program` - This key defines what the job should start, in our case a script
with the path `/Users/Me/Scripts/cleanup.sh`.
3. `RunAtLoad` - This key specifies when the job should be run, in this case
right after it has been loaded.

As you can see, the keys used in this XML file are quite self-explanatory. This
is the case for the remaining 30-40 keys that `launchd` supports. Last, but not
least, this files although have an XML syntax, they in fact have a `.plist`
extension which means `Property List`. Makes a lot of sense, right?

## `launchd` v.s. `launchctl`

Before we continue with our little exercise of creating daemons/agents with
Go, let's first see how `launchd` allows us to control these jobs. While
`launchd`'s job is to boot the system and to load and maintain services, there
is a different command used for jobs management - `launchctl`. With `launchd`
facilitating jobs, the control of services is centralized in the `launchctl`
command.

`launchctl` has a long list of subcommands that we can use. For example, loading
or unloading a job is done via:

{% highlight bash %}
launchctl unload/load ~/Library/LaunchAgents/com.example.app.plist
{% endhighlight %}

Or, starting/stopping a job is done via:

{% highlight bash %}
launchctl start/stop ~/Library/LaunchAgents/com.example.app.plist
{% endhighlight %}

To get any confusion out of the way, `load` and `start` are different. The
important difference is that if a job is configured to be run on load, when the
job is loaded it will be started automatically. This is achieved by setting
the `RunAtLoad` property in the property list XML of the job:

{% highlight xml %}
<!--- Example blatantly ripped off from http://www.launchd.info/ -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
	<dict>
		<key>Label</key>
		<string>com.example.app</string>
		<key>Program</key>
		<string>/Users/Me/Scripts/cleanup.sh</string>

		<key>RunAtLoad</key><true/>

	</dict>
</plist>
{% endhighlight %}

If you would like to see what other commands `launchctl` supports, you can see
`man launchctl` in your terminal and see all of the options in detail.

## Automating with Go

After getting the basics of `launchd` and `launctl` out of the way, why don't we
see how we can add an agent for any Go package we need? For our example, we are
going to write a simple way of plugging in a `launchd` agent for any of your
Go packages.

As we already established before, `launchd` speaks in XML. Or, rather, it
understands XML files, called _property lists_ (or `.plist`). This means, for
our Go package to have an agent running on OSX, it will need to tell `launchd`
"hey, `launchd`, run this thing!". And since `launch` speaks only in `.plist`,
that means our package needs to be capable of generating XML files.

### Templates in Go

While one could have just a hardcoded `.plist` file in their project and just
copy it across to the `~/Library/LaunchAgents` path, a more programatical way
to do this would be to use a template to generate these XML files. Good thing is
Go's standard library has us covered in these cases - the `text/template` package
([docs](https://godoc.org/text/template)) does exactly what we need.

In a nutshell, `text/template` implements data-driven templates for generating
textual output. Or in other words, you give it a template and a data structure,
it will mash them up together and produce a nice and clean text file. Perfect.

Let's say the `.plist` we need to generate in our case is the following:

{% highlight xml %}
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE plist PUBLIC \"-//Apple Computer//DTD PLIST 1.0//EN\" \"http://www.apple.com/DTDs/PropertyList-1.0.dtd\" >
<plist version='1.0'>
  <dict>
    <key>Label</key><string>Ticker</string>
    <key>Program</key><string>/usr/local/bin/ticker</string>
    <key>StandardOutPath</key><string>/var/log/ticker/access.log</string>
    <key>StandardErrorPath</key><string>/var/log/ticker/error.log</string>
    <key>KeepAlive</key><true/>
    <key>RunAtLoad</key><true/>
  </dict>
</plist>
{% endhighlight %}

We want to keep it quite simple for the purpose of our little exercise. It will
contain only six properties: `Label`, `Program`, `StandardOutPath`,
`StandardErrorPath`, `KeepAlive` and `RunAtLoad`. To generate such a XML, it's
template would look something like this:

{% raw %}
```xml
<?xml version='1.0' encoding='UTF-8'?>
<!DOCTYPE plist PUBLIC \"-//Apple Computer//DTD PLIST 1.0//EN\" \"http://www.apple.com/DTDs/PropertyList-1.0.dtd\" >
<plist version='1.0'>
  <dict>
    <key>Label</key><string>{{.Label}}</string>
    <key>Program</key><string>{{.Program}}</string>
    <key>StandardOutPath</key><string>/var/log/{{.Label}}/access.log</string>
    <key>StandardErrorPath</key><string>/var/log/{{.Label}}/error.log</string>
    <key>KeepAlive</key><{{.KeepAlive}}/>
    <key>RunAtLoad</key><{{.RunAtLoad}}/>
  </dict>
</plist>
```
{% endraw %}

As you can see, the difference between the two XMLs is that the second one has
the double curly braces with expressions in them in places where the first XML
has some sort of a value. These are called "actions", which can be data
evaluations or control structures and are delimited by "{{" and "}}". Any of the
text outside actions is copied to the output untouched.

### Injecting your data

Now that we have our template with all of it's glorious XML and curly braces
(or, actions), let's see how we can inject our data in it. Since stuff is
generally simple in Go, especially when it comes to it's standard library, you
should not worry - this will be easy!

To keep thing simple, we will store the whole XML template in a plain old string.
Yes, weird, I know. The best way would probably be to store it in a file and
read it from there, but for the purpose of our little example let's keep it
simple:

{% raw %}
```go
// template.go
package main

func Template() string {
	return `
<?xml version='1.0' encoding='UTF-8'?>
 <!DOCTYPE plist PUBLIC \"-//Apple Computer//DTD PLIST 1.0//EN\" \"http://www.apple.com/DTDs/Prope    rtyList-1.0.dtd\" >
 <plist version='1.0'>
   <dict>
     <key>Label</key><string>{{.Label}}</string>
     <key>Program</key><string>{{.Program}}</string>
     <key>StandardOutPath</key><string>/var/log/{{.Label}}/access.log</string>
     <key>StandardErrorPath</key><string>/var/log/{{.Label}}/error.log</string>
     <key>KeepAlive</key><{{.KeepAlive}}/>
     <key>RunAtLoad</key><{{.RunAtLoad}}/>
   </dict>
</plist>
`
}
```
{% endraw %}

And the program that will use our little template function:

{% highlight go %}
// main.go
package main

import (
	"log"
	"os"
	"text/template"
)

func main() {
	data := struct {
		Label     string
		Program   string
		KeepAlive bool
		RunAtLoad bool
	}{
		Label:     "ticker",
		Program:   "/usr/local/bin/ticker",
		KeepAlive: true,
		RunAtLoad: true,
	}
	t := template.Must(template.New("launchdConfig").Parse(Template()))
	err := t.Execute(os.Stdout, data)
	if err != nil {
		log.Fatalf("Template generation failed: %s", err)
	}
}
{% endhighlight %}

So, what happens there, in the `main` function? It's actually quite simple:

1. We declare a small `struct`, which has only the properties that will be needed
in the template, and we immediatelly initialize it with the values for our program.
2. We build a new template, using the `template.New` function, with the name
`launchdConfig`. Then, we invoke the `Parse` function on it, which takes the
XML template as an argument.
3. We invoke the `template.Must` function, which takes our built template as
argument. From the documentation, `template.Must` is a helper that wraps a call
to a function returning `(*Template, error)` and panics if the error is non-`nil`.
Actually, `template.Must` is built to be used exactly as we do in our program.
4. Lastly, we invoke `Execute` on our built template, which takes a data structure
and applies it's attributes to the actions in the template. Then it sends the
output to `os.Stdout`, which does the trick for our example. Of course, the
output can be sent to any struct that implements the `io.Writer` interface, like
an `os.File`.

### Make and load my `.plist`

Instead of sending all this nice XML to standard out, let's throw in an open
file descriptor to the `Execute` function and finally save our `.plist` file in
`~/Library/LaunchAgents`. There are a couple of main points we need to change.

First, getting the location of the binary. Since it's a Go binary, and we will
install it via `go install`, we can assume that the path will be at `$GOPATH/bin`.
Second, since we don't know the actual `$HOME` of the current user, we will have
to get it through the environment. Both of these can be done via `os.Getenv`
([docs](https://godoc.org/os#Getenv)) which takes a variable name and returns
it's value.

{% highlight go %}
// main.go
package main

import (
	"log"
	"os"
	"text/template"
)

func main() {
	data := struct {
		Label     string
		Program   string
		KeepAlive bool
		RunAtLoad bool
	}{
		Label:     "com.ieftimov.ticker", // Reverse-DNS naming convention
		Program:   fmt.Sprintf("%s/bin/ticker", os.Getenv("GOPATH")),
		KeepAlive: true,
		RunAtLoad: true,
	}

        plistPath := fmt.Sprintf("%s/Library/LaunchAgents/%s.plist", os.Getenv("HOME"), data.Label)
        f, err := os.Open(plistPath)
	t := template.Must(template.New("launchdConfig").Parse(Template()))
	err := t.Execute(f, data)
	if err != nil {
		log.Fatalf("Template generation failed: %s", err)
	}
}
{% endhighlight %}

That's about it. The first part, about setting the correct `Program` property,
is done by concatenating the name of the program and `$GOPATH`:

{% highlight go %}
fmt.Sprintf("%s/bin/ticker", os.Getenv("GOPATH"))
// Output: /Users/<username>/go/bin/ticker
{% endhighlight %}

The second part is slightly more complex, and it's done by concatenating three
strings, the `$HOME` environment variable, the `Label` property of the program
and the `/Library/LaunchAgents` string:

{% highlight go %}
fmt.Sprintf("%s/Library/LaunchAgents/%s.plist", os.Getenv("HOME"), data.Label)
// Output: /Users/<username>/Library/LaunchAgents/com.ieftimov.ticker.plist
{% endhighlight %}

By having these two paths, opening the file and writing to it is very trivial -
we open the file via `os.Open` and we pass in the `os.File` structure to
`t.Execute` which writes to the file descriptor.

## What about the Launch Agent?

We can keep this one simple. Let's throw in a simple command to our package,
make it installable via `go install` (not that there's much to it) and make it
runnable by our `.plist` file.

{% highlight go %}
// cmd/ticker/main.go
package ticker

import (
  "time"
  "fmt"
)

func main() {
    for range time.Tick(30 * time.Second) {
            fmt.Println("tick!")
    }
}
{% endhighlight %}

This the `ticker` program will use `time.Tick`, to execute an action every 30
seconds. Since this will be an infinte loop, `launchd` will kick off the program
on boot (because `RunAtLoad` is set to `true` in the `.plist` file), and will
keep it running. But, to make the program controllable from the operating system,
we need to make the program react to some OS signals, like `SIGINT` or `SIGTERM`.

### Understanding and handling OS signals

While there's quite a bit to be learned and taught about OS signals, for the
purpose of our example we will really just scratch a bit off the surface.

The best way to think about a signal is that it's a message from the operating
system to a process. It is basically an asynchronous notification sent to a
process or to a specific thread within the same process in order to notify it of
an event that occurred. 

There are quite a bit of various signals that can be sent to a process
(or a thread), like `SIGKILL` (which kills a process), `SIGSTOP` (stop), `SIGTERM`
(termination), `SIGILL` and so on and so forth. There's an exhaustive list of
signal types on [Wikipedia's page](https://en.wikipedia.org/wiki/Signal_(IPC))
on signals.

To get back to `launchd`, if we look at it's documenation about stopping a job
we will notice the following:

> Stopping a job will send the signal `SIGTERM` to the process. Should this not
stop the process launchd will wait `ExitTimeOut` seconds (20 seconds by default)
before sending `SIGKILL`.

Pretty self-explanatory, right? We basically need to handle one signal - `SIGTERM`.
Why not `SIGKILL`? Because `SIGKILL` is a special signal that cannot be caught -
it directly kills the process without any chance for a graceful shutdown. That's
why there's a termination signal and a "kill" signal.

{% highlight go %}
package main

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	sigs := make(chan os.Signal, 1)
	signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)

	go func() {
		<-sigs
		os.Exit(0)
	}()

	for range time.Tick(30 * time.Second) {
		fmt.Println("tick!")
	}
}
{% endhighlight %}
