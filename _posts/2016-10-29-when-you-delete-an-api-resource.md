---
layout: post
title: "What happens when you DELETE a resource?"
tags: [rest, api, design]
header-img: img/posts/when-you-delete-an-api-resource.jpg
card-img: img/cards/when-you-delete-an-api-resource.png
---
# What happens when you DELETE a resource?

Have you ever found yourself publishing an API, either an internal or a public
one? Have your ever heard from the consumers of those same APIs back? Are they
happy about the functionality of your APIs and their design? You already know,
there is no perfect design, but API design has to be taken very seriously. Why?
Because evolving and  changing APIS is **hard and time consuming**.

Imagine you are on a team in a company, that has set off to design a
company-wide API guideline and a blueprint. As you could imagine, a feat like
this one takes time - first you have to put in a draft, then let it settle like
a fine French wine, then review it and apply any needed changes. And repeat
over, until you have a team wide consensus on the design.

In situations like this, I have found interesting how your API should behave
when a resource is `DELETE`d? Of course, aside from the obvious, that the record
will be deleted.

Let's look at couple of points that might be a good food for thought in these
occasions.

## Idempotency

We have all heard about idempotency and we have heard that all calls to a REST
API should be idempotent, except the `POST` requests. But, what does this mean?

<small>_Crickets sound on_</small>

From Wikipedia:

> Idempotence is the property of certain operations in mathematics and computer
> science, that can be applied multiple times without changing the result beyond
> the initial application.

<small>_Crickets sound off_</small>

Simply put, a call to an idempotent endpoint with a given set of parmeters,
should always have the same effect. For example, if you have a `User` resource,
everytime you call `PUT /users/1` should have the same effect on the resource.
Also, a `GET /users/1` is a safe method (or *nullipotent*), because it does not
have any effect on the state of the resource.

**But, is DELETE idempotent?** According to the
[HTTP specification](https://tools.ietf.org/html/rfc7231#section-4.2.2) `DELETE`
_should_ be idempotent.

```
   +---------+------+------------+
   | Method  | Safe | Idempotent |
   +---------+------+------------+
   | CONNECT | no   | no         |
   | DELETE  | no   | yes        |
   | GET     | yes  | yes        |
   | HEAD    | yes  | yes        |
   | OPTIONS | yes  | yes        |
   | POST    | no   | no         |
   | PUT     | no   | yes        |
   | TRACE   | yes  | yes        |
   +---------+------+------------+
```
This means that whenever a `DELETE` request is sent, the state of the system
after fulfilling the request will always be the same, although the server might
implement non-idempotent effects under the hood.

So, knowing that a `DELETE` request will always be idempotent, what should a
response of that request be? Let's start from the basics, the status code.

## HTTP status codes

We usually take HTTP status codes as an ever-present thing slapped onto a HTTP
request. Especially if you are a beginner, understanding how useful status codes
are and how much context they can provide for free, can be hard.

After executing any HTTP request, the HTTP code that will be returned is always
contextual, which means, it should be based on the effect (and hence the result)
of the request. Keeping in mind that a `DELETE` request is idempotent, what
would be an appropriate status code?

Let's look at some status codes that make sense as a response for a `DELETE`
request:

[HTTP 204](https://httpstatuses.com/204) - No content:

> The server has successfully fulfilled the request and that there is no
> additional content to send in the response payload body.

HTTP 204 would work - the server has accepted your `DELETE` request and it has
fulfilled it. The server will not return a payload in the response, hence
`204 No content`.

[HTTP 202](https://httpstatuses.com/202) - Accepted:

> The request has been accepted for processing, but the processing has not been
> completed. The request might or might not eventually be acted upon, as it
> might be disallowed when processing actually takes place.

You tell the server to destroy the resource that the URI points to, but since
it's a long procedure (maybe a background job) to delete the resource, it
returns a `HTTP 202 Accepted`. This means that the server basically says "Cool,
I understand, I am on it!". But, it does not mean that the resource is deleted
as part of the request/response cycle.

[HTTP 200](https://httpstatuses.com/200) OK:

> The request has succeeded.

The server received your request and responds with `200 OK` - which means that
it understood your request and deletes the resource described by the URI.

You can notice that all of the status codes are closely tied to the response
that the API call will return. So, let's talk about the HTTP status codes in
combination with some response payloads.

## Sending a meaningful response

You send a `DELETE` and the server complies. It executes the logic related to
the endpoint and the HTTP verb and deletes the record that you wanted gone. So,
how does it respond back?

First scenario would be - no response body with HTTP 204. Makes sense. But what
would this actually mean? Well let's say you have a Rails API that returns
hypermedia controls in the JSON API spec format. When it returns a resource it
will also return the URIs of the associated resources. When you delete a record,
it will perish from the database - but if you have a `dependent: :destroy` in
your model, it would remove associated records as well. In this occasion, your
client would not know if anything else got removed in the same request-response
lifecycle.

Another option would be a HTTP 200 OK - but as we saw earlier, 200 OK states
that you must return a body in the response. In the HTTP spec, we can see the
following:

> The 200 (OK) status code indicates that the request has succeeded. The payload
> sent in a 200 response depends on the request method. For the methods defined
> by this specification, the intended meaning of the payload can be summarized as:
>
>  - PUT, DELETE  a representation of the status of the action

This means that, according to the HTTP spec the response body has to be a
representation of the status of the action. In other words, it needs to send
back the representation of the resource that has been deleted as a result of the
request. But, if we do that, we will add an additional load on the endpoint,
because it will need to serialize the deleted resource, and as we all know
serialization is expensive.

A third option would be HTTP 204 - Accepted. But as we saw earlier, `Accepted`
as a status code is very contextual on the action under the hood. It can only be
returned if the request has been accepted, but the action has not been put into
effect yet.

Whoa, okay, so where now?

## Are we overthinking it?

Just like in most of the cases where computer science and engineering is
involved - **it depends**. You could say that it's easy to keep all of this
simple, but often we have to be careful. After all, we don't want to be lazy and
oversimplify our APIs.

Let's take a tiny example - an API for a blog CMS. We will look at a resource,
namely the `/authors` resource. This means that, we will have the following
endpoints available:

{% highlight text %}
GET /authors        # returns a collection of authors
GET /authors/:id    # returns an author with the ID of :id
POST /authors       # create an author
PUT /authors/:id    # create/update an author
DELETE /authors/:id # destroy an author
{% endhighlight %}

So, we are interested in the behaviour of the last endpoint, `DELETE /authors/:id`.
When a request is succesfully fulfilled on this endpoint, the record which was
identified by the `:id` param is destroyed. Since the request is idempotent, it
means that you can freely request to delete a resource multiple times, but the
effect on the system will be the same - the resource will not be present.

In most of the cases, the REST APIs that we use are
[level 3](http://ieftimov.com/sprinkle-some-hateoas-on-rails-apis#level-3).
This means that usually we have couple of options on how to respond to this
request, HTTP 204, or HTTP 200, which we already discussed at the begining of
this article. But, what happens if you are building a
[level 4](http://ieftimov.com/sprinkle-some-hateoas-on-rails-apis#level-4) REST
API?

## Handling the state in the client

If you have a hypermedia API, things can get a little bit more complex. For those
of you that don't know what hypermedia APIs, I recommend reading my previous
article about REST APIs -
"[Sprinkle some HATEOAS on your Rails APIs](http://ieftimov.com/sprinkle-some-hateoas-on-rails-apis)".
It covers all of Fielding's REST levels and explains them thoroughly. After you
get a grasp of hypermedia APIs, you can come back to this to more easily
understand the issue at hand.

Hypermedia clients usually have to be a little bit smarter, due to the fact that
the APIs they use are smarter. This means that when we update the state of the
hypermedia API's system, the client should be aware of the state change. In
more concrete wording - when we delete a resource, the client must know that the
resource does not exist anymore on the server side.

What does that mean for our API? Well, remember the `DELETE /authors/:id`
endpoint? If a request to that endpoint returns an `HTTP 204` (which explicitly
states that the reponse body is empty), then our clients need to know that the
nested resources _might_ be deleted because there might bea cascading delete.

For example, if the `DELETE /authors/:id` request also removes all of the posts
that were written by an author, the client might still think that the posts
are available. This means that the client will allow a request to
`GET /authors/:id/posts/`, and the API needs to know to respond with an
appropriate response and HTTP status. But, will any of the aforementioned
statuses cut it in this case? You might think that even `HTTP 404` would do it,
but is it semantically correct?

Sure, `HTTP 404` states that the resource is not found, but in cases like this
we actually want to inform the consumer that there was something there before,
but not anymore. This means that the best HTTP status code in these cases is
`HTTP 410`:

> HTTP 410 GONE
> The target resource is no longer available at the origin server and that this
> condition is likely to be permanent.

HTTP 410 will inform the consumer and the client that there was something there
before, but not it's gone (hence the status code). This will be semantically much
more correct, but it will add implementation implications to the API itself. It
means that the API will have to know that a resource has existed in a certain
place. This means that records might have to be flagged as deleted, but not
actually deleted and so on.

As you can see, the complexity that hypermedia APIs add to the client can be big,
and both the client and the API have to be very smartly built to allow proper
usage and leverage of hypermedia.

## In closing

As we some in our tiny examples, although things can most often look trivial, if
we want to have semantically correct APIs, we need to be very careful of every
aspect of our API. HTTP statuses, although oftern perceived as trivial, if
applied correctly add a lot of power to our APIs and the clients.
very powerful addition.
