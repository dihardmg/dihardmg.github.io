---
layout: post
title: "Ruby, Rails and HTTP Caching"
tags: [rails, ruby, http, caching]
header-img: img/posts/sprinkle-some-hateoas-on-rails-apis.jpeg
card-img: img/cards/sprinkle-some-hateoas-on-rails-apis-1.png
---

We have all heard about HTTP. Right? No? What do you mean?

Hypertext Transfer Protocol (or HTTP) is the protocol that powers the web.
Although HTTP might look simple to the untrained eye, it actually has a lot of
benefits and semantic rules of it's intended behaviour and design choices. All
of this knowledge is stored into documents called RFC's (Request For Comments),
which have been well organzied and archived by IETF.

One of the tools in HTTP's toolbelt is caching. The good thing about this caching
mechanism is that since every browser "speaks HTTP", they can all utilise it out
of the box. All a browser needs is for the server to provide the correct HTTP
header directives, so it can understand the specifics of the cache that it's
supposed to keep.
