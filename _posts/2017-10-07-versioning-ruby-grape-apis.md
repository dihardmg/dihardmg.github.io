---
layout: post
title: "Versioning REST APIs: The Theory and Using Grape in Ruby"
tags: [ruby, api, rest, versioning]
image: versioning-ruby-grape-apis.png
header-img: img/posts/versioning-ruby-grape-apis.jpg
card-img: img/cards/versioning-ruby-grape-apis.png
---

Nowadays, having an API on top of your application is considered common.
I've often been disapointed when I've been expecting an API of a product I like
to find none. There are powerful tools out there that allow easy API
integrations, like IFTTT.com. Also, if you want to build a mobile application to
work aside your product (or maybe your product is mobile-first), then an API is
a must-have - there’s no way around it.

Although APIs are so widespread today, designing, maintaining and expanding them
hasn’t become less hard than ever before. Just today, your APIs are expected to
work properly, be well designed and documented. So, basically, I believe that
the tools haven’t caught up with the expectation of the market - APIs are a
must-have, but building them is as hard as ever.

For one to build a great API, whether it’s a shiny GraphQL or a RESTful one, the
first thing that we need to understand is versioning. Let’s take a dive at this
often underestimated aspect of all APIs, and expand our knowledge about it.

## What is versioning?

Imagine this: you work for a product company, like Twitter. As you can imagine,
Twitter provides an API where it’s content can be consumed through. Having
millions of users, Twitter normally would like to enable other developers to use
their platform and build add-ons or products on top of their APIs.

As the product advances and gets newer features, and the old ones get updated,
some of the changes that occur to the product might be breaking to some of the
consumers of the APIs. So, that becomes a problem - if you do the breaking
change a large number of integrations with the API will break. On the other
hand, the product must keep on evolving and you simply cannot just stop
development because you are scared you will break other’s people apps that
consume your APIs.

As you can imagine, the best way to approach this is to apply API versioning.
That means, when you want to introduce a breaking change to your API, you will
have to add a new version of the API, so people that want to use the changed
feature can use (for example) version two of the API, while version one will
stay intact (but will probably become deprecated). Therefore, you will keep
backwards compatibility for other consumers of the old API, while advancing the
product.

## Versioning strategies

Having said that, let’s look at some strategies of API versioning. Over the
years, as we’ve been building APIs, people have invented different ways to
version their APIs. One of the most popular are the following:

- Media type versioning
- Header
- URI
- Domain
- Parameter versioning

Let’s take a look at a quick example of each of them, and how we can implement
them using Ruby.

### Media type versioning

This type of versioning is also known as "content negotiation" or "accept
header". It basically utilises the `Accept` media header to specify the version
of the API where the request should be processed. For example, it usually looks
something like this:

```
Accept: application/vnd.mycompany.api+json; version=1.0
```

This is called a "vendor header" (hence the `vnd` in the header name). It
supplies the name of the API, the negotiated content type and the version of
the API.

One of it’s good sides is that you can version the API at the resource level,
which will allow you to preserve your URIs between versions and get the resource
representation/behaviour based on the negotiated version. On the other hand,
exploring is much harder, since just a browser won’t cut it. Additionally, the
HTTP headers get distorted to fit in the versioning in them. This makes the
clients know the media type for each resource, and hence they need to request
the same media type throughout their use of the API, to make sure the
integration works normally at all times.

### Header versioning

Another way of versioning APIs is through HTTP headers, usually custom ones. In
the wild you can see many variations of this type of versioning, but they all
revolve around a common theme - there’s a custom header that controls the
versioning for the request lifecycle. By convention, custom HTTP headers are
prefixed with `X`, like `X-Custom-Header`. When it comes to versioning, you can
often see `X-API-Version` or `X-Version`.

The one very good side about this versioning is that you can preserve all the
URIs between versions, because the headers are transparent to the URI.

### URI Versioning

URI versioning is achieved by, obviously, specifying the API version in the URI.
You have probably seen this type of versioning if you’ve ever worked with a
vendor API, like Facebook’s or Twitter’s. It is achieved by adding the version
in the URI, like:

```
https://example.com/api/v1/users
```

Usually the versioning is in the form of  `vX`, where `v` represents `version`
and `X` is the version number.

While this type of versioning breaks the RESTfulness of the APIs, since each URI
should be a resource and not a version, on the other hand it’s the most widely
used type of versioning. It’s very easy to browse an API (and it’s versions)
through a browser and it’s dead simple to use.

### Domain versioning

This type of versioning can be considered a sub-type of the previous, URI based,
versioning. It’s quite simple, and it allows easy routing through the
infrastructure where the API is hosted, since subdomains are very easy to
configure.

It usually looks like:

```
v1.api.example.net/users
```

### Parameter versioning

Parameter versioning is achieved by setting a parameter in the request to use
the appropriate API version. For example, this can be done as:

```
GET example.net/api/users?version=1
```

As you can imagine, unlike the previous versioning strategies, the parameter can
be left out and the server should know to what version to default to.

Now, on the other hand, although this can allow you to version the resources
easily, if you throw in API versioning as well, things can easily get out of
hand. That’s why, people like to version their APIs, usually, via the URI or the
headers.

## Show me the code

Now that we’ve depleted most of the popular API version strategies, let’s see
how we can actually implement them. For the purpose of these examples, we’ll be
using Grape API, one of the most popular API DSLs for the Ruby programming
language.

Let's imagine that we have a very simple API endpoint, which will fetch a single
user, with a URI `GET /user/:id`.

{% highlight ruby %}
module API
  class Users < Grape::API
    resource :users do
      route_param :id do
        get do
          User.find(params[:id])
        end
      end
    end
  end
end
{% endhighlight %}

### Media-type versioning

Implementing this type of versioning is rather easy. For example, let's say our
company is called 'BarCo', so we want to make our `Accept` header look like:

{% highlight text %}
Accept: application/vnd.barco-v1+json
{% endhighlight %}

So, how can we implement this quickly, using a `before` hook that will be
executed before any request to our API?

{% highlight ruby %}
module API
  module V1
    VENDOR = 'barco'.freeze
    VERSION = 'v1'.freeze
    HEADER = 'application/vnd.#{API::VENDOR}-#{API::VERSION}+json'.freeze

    before do
      error!('Not found', 404) unless headers['Accept'] == HEADER
    end
  end
end
{% endhighlight %}

So, before every request comes in, the `before` hook will validate the `Accept`
header of the request and will raise a `HTTP 404` if it cannot find the
appropriate API version.

As you can see, this example to header based versioning is quite simple and
rudimentary, but lucky for us Grape has a built-in strategy for header
versioning so we don't have to reinvent the wheel every time we write an API.

To do the same, Grape has the [Header](https://github.com/ruby-grape/grape#header)
versioning, and it easy to plug it in your API, like:

{% highlight ruby %}
module API
  class Users < Grape::API
    version 'v1', using: :header, vendor: 'barco'

    resource :users do
      route_param :id do
        get do
          User.find(params[:id])
        end
      end
    end
  end
end
{% endhighlight %}

Just by adding the `version` line in the class, Grape will version this resource
for us and will expect the appropriate header to be provided to the request.

### Header versioning

Similarly to our previous approach, we could version our API using custom header
versioning. As an example, the API could require the following header:

{% highlight text %}
X-API-Version: 1
{% endhighlight %}

So, using Grape's `before` hooks, we could write something like:

{% highlight ruby %}
module API
  module V1
    VERSION = '1'.freeze
    HEADER = 'X-API-Version'.freeze

    before do
      return respond_with_404 unless headers.key?(HEADER)
      return respond_with_404 unless headers[HEADER] == VERSION
    end

    def respond_with_404
      error!('Not found', 404)
    end
  end
end
{% endhighlight %}

This `before` hook will check for the presence of a header in the request
headers, and for the value of the header if it exists. Then it will respond with
and error if any of the two requirements are not met.

Easy enough, right? Well, sure, but again, our example is a bit rudimentary and
it won't work that elegantly. But fear not - Grape has our backs, again!

Grape implements the
[Accept-Version Header](https://github.com/ruby-grape/grape#accept-version-header)
strategy, which will accept the following versioning header:

{% highlight text %}
Accept-Version: v1
{% endhighlight %}

Plugging it into the API is also very trivial, just like before:

{% highlight ruby %}
module API
  class Users < Grape::API
    version 'v1', using: :accept_version_header

    resource :users do
      route_param :id do
        get do
          User.find(params[:id])
        end
      end
    end
  end
end
{% endhighlight %}

That's it, versioning with minimal footprint.

### URI Versioning

Implementing URI versioining is a bit more involved to do manually in Grape,
because it involves basically rewriting the URI that is requested and invoking
the proper endpoint based on pieces of the URI.

The good thing is that we don't have to do that manually as well, since Grape
provides an out-of-the-box solution for URI versioning as well:

{% highlight ruby %}
module API
  class Users < Grape::API
    version 'v1', using: :path

    resource :users do
      route_param :id do
        get do
          User.find(params[:id])
        end
      end
    end
  end
end
{% endhighlight %}

### Parameter versioning

Just like our previous approaches, we could version our API using a parameter.
As an example, the API could require this parameter:

{% highlight text %}
https://example.net/users?v=1
{% endhighlight %}

So, using Grape's `before` hooks, we could write something like:

{% highlight ruby %}
module API
  module V1
    VERSION = '1'.freeze
    PARAM = 'v'.freeze

    before do
      return respond_with_404 unless params.key?(PARAM)
      return respond_with_404 unless params[PARAM] == VERSION
    end

    def respond_with_404
      error!('Not found', 404)
    end
  end
end
{% endhighlight %}

This `before` hook will check for the presence of a param in the request, and
for the value of the param if it exists. Then it will respond with and error if
any of the two requirements are not met.

Easy enough, right? Again, our example is rudimentary. You guessed it - Grape
has our backs, again and again.

Grape implements the
[Param versioning](https://github.com/ruby-grape/grape#param) strategy, which
work very similarly, but with a very small footprint:

{% highlight ruby %}
module API
  class Users < Grape::API
    version '1', using: :param, parameter: 'v'

    resource :users do
      route_param :id do
        get do
          User.find(params[:id])
        end
      end
    end
  end
end
{% endhighlight %}

## Rolling your own v.s. using built-in strategy

In our examples, although quite small, we saw how you could roll out your own
versioning strategy. Whichever strategy you pick, I recommend first knowing the
differences between all of them so you can do a educated pick instead of blindly
following a podcast or a tutorial or a blog post (like this one).

When it comes to rolling your own, I recommend against it. Gems like Grape, that
we explored briefly in this post, have perfected these versioning strategies and
if you roll your own you can expose your API to various vulnerabilities or bugs.

Of course, if you still need to do it yourself, or your company uses some custom
or non-standard versioining strategy, I recommend looking at what the community
has already done. For example, you can see all of Grape's versioning middleware
in
[their Github repo](https://github.com/ruby-grape/grape/tree/master/lib/grape/middleware/versioner)
and see how they've implemented it. The code is quite clean and straightforward.


## In closing

In this post we saw what API versioning means, what are the various versioning
strategies, and how we could do it ourselves. Also, we saw how one of the most
popular REST API DSLs for Ruby do it. I strongly recommend checking out the
source code of Grape's versioning middleware, so you can get a good grasp of how
this is actually done under the hood.

Also, it's always smart to look at the big players in the field and learn from
their experiences. For example, you can take a look at Facebook's Graph API
[documentation](https://developers.facebook.com/docs/graph-api/), Github's API
[documentation](https://developer.github.com/v3/) or Stripe's API
[documentation](https://stripe.com/docs/api). They've done a tremendous job
documenting their API's. Also, you can see how Stripe uses a custom header
versioining, in combination with dates - it's quite interesting.
