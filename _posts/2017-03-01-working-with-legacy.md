---
layout: post
title: "Should working with legacy be awful?"
tags: [rest, api, design]
header-img: img/posts/working-with-legacy.jpg
card-img: img/cards/working-with-legacy.png
---
# Should working with legacy be awful? 

I am sure most of you feel some kind of a hype in any part of your life. Some must buy a Tesla because it’s awesome (which in fact, it is!), others must buy Kanye’s newest Yeezys, others the latest iPhone or Apple Watch. We people like the newest and coolest, we like to be cool, we like being fresh and original and the whole excitement that surrounds being original. 

Couple of weeks ago I was listening to a very interesting episode of the great Freakonomics podcast, called [In Praise of Maintenance](http://freakonomics.com/podcast/in-praise-of-maintenance/). If you haven’t already heard it, or you don’t know Freakonomics at all, I highly recommend checking it out. In the episode it was mentioned exactly that - people like building new and exciting stuff but we frown upon maintenance work. The host and his guest, which is Larry Summers (ex-U.S. Treasury Secretary) discussed the state of USA’s infrastructure. It was interesting noting that the American Society of Civil Engineers annually publishes a report on the state of the physical infrastructure in the United States. What was very unexpected is that  the reports show that the infrastructure in the USA is getting worse and worse with every year. It essentially means that although the USA is an economic superpower with lots of smart people and great educational institutions, it's infrastructure was in a very bad shape. Simply put, it is not maintained (enough).

This reminded me very much of my job, creating and maintaining software. We developers like working on a new thing, with a new technology or on a greenfield project. But, when it comes to “maintaining the old infrastructure” we always feel the pain. More specifically, in our case it’s called “legacy”. Let’s talk a little bit about legacy and maintenance in software development. Or, in other words, can working on legacy be as interesting and as engaging as working on a new thing?

## Legacy hurts
Let’s first define legacy code. According to Wikipedia the definition of legacy is:

> Legacy code is source code that relates to a no-longer supported or manufactured operating system or other computer technology.

But over time, the meaning on legacy has changed a bit. Michael Feathers coined an informal definition of legacy as follows:

> Source code inherited from someone else, or source code inherited from an older version of the software.

Basically, whichever definition you like, legacy means old(er) source code which we can not (or we find it very hard to) understand. So, to get it out of the way - maintenance is always hard and hurts. Or, better said, it’s just **not that fun** to do.

So, why does it actually hurt to work with legacy? Let’s take a step back and think about how this code that is hurtful came to be. A company that had a problem that it was worth automating (or streamlining) hired engineers to come up with a solution. The engineers, with their teams, **analysed** the problem at hand, **learned** about how the company works, learned about the domain that the company works in and applied all of this context to build this solution.

Usually, as the business grows, it outgrows the tools that it uses. Which is a very good problem to have. But, maintaining this old software becomes harder and harder due to the same points from above - context and domain knowledge. 

If we are dealing with a extremely old or unmaintained project, we might not even know how to boot the server of the application. The README of the project can be in a very bad shape (or even nonexistent), so we don’t even know what we are dealing with.

Other times, the information provided in the documentation can be scarce - because that team that has been working on it simply has all of the detail in their minds. Sure, you might say that that is a sign of poor culture or practice, but keep in mind that the team actually built a product that works. Because if it didn’t work - it wouldn’t become legacy.

So yeah, as we already saw, legacy can hurt quite a bit for many reasons. Whether it’s technical, or organisational, legacy can cause a lot of pain to people that will deal with it every day.

But, does it have to?

## Stability
Any big company has some form of legacy in it - it can be a process, or software or anything else. Maybe even a slow printer that creates long queues. But, if we think about any company with legacy software, we almost always thing of a huge company. Usually companies that have grown enough can afford (intentionally or unintentionally) to have legacy software as it’s engine. But, for us as engineers that maintain this legacy is this a bad thing? Does it have to be “damning”? What’s the alternative?

Startups? Sure, startups are cool, but they are very unstable. Companies that have legacy software have more stability, because legacy means business, not a shiny startup that’s burning through their investor’s money.  Agency? No assurance that you won’t be working with legacy. You just might charge better for it.

So, whether or not we like to admit it - legacy brings stability. You might ask how. Well, think about this - if a company does not have the resources can burn itself to the ground if it tries to maintain legacy. On the other hand, profitable companies (a.k.a. businesses) can maintain it because they actually make money. Their size and profitability allows them to hire manpower and throw more people at the problem. That’s why although this setting might not sound “cool” like a fresh startup using the latest and greatest, but it sure brings a lot of stability to the table.

## Product
As a software engineer I spent the first years of my career as a consultant - I worked for multiple clients, different projects both short and long term. Although financially (more) rewarding, consulting work often gives the “gun for hire” feeling. On the flip side, working full-time in a product company gives you exposure to the very tiny bits and bolts of the domain that the product resides in. For example, if you work for Paypal you will get such deep knowledge of payment system which, given enough exposure to it, will actually render you an expert in the domain.
 
But putting the prospects of working in a product company aside, how would legacy software impact your work on the product? On one hand, you might think in the direction of complexity of the codebase - the more iterations and testing done, more use-cases and scenarios are added to the product. Which is often true - the more niche the product gets and the bigger it becomes the codebase will suffer more and more. These side-effects of a grown-up company are inevitable, every company suffers from them on some scale.

On the other hand, having a nice product on your hands to work every day is rewarding because of the users. The bigger the product (usually) more users. This allows interesting A/B testing, learning the behaviour of the users and the impact of your changes although however small they might seem. So, although legacy can add a lot of complexity to your everyday work, a single UI change can impact the business in a way that you couldn’t image before.

This often sounds scary but it's also very exciting. It brings a quick feedback loops but also a big impact.

## Money
Very often, when you as a technical person, work on an improvement however simple it might be you can always A/B the test the impact of the improvement. This way you can always know how your impact affected the company's KPIs (and hence, it's business). For example, if you improve the page load time of a very flow-critical page, you can test this in an A/B test and see the impact. Who knows, maybe that speed-bump can significantly improve the conversion of that page.

Money doesn’t always imply legacy, but legacy always implies money. This means that via various experiments you, as engineer or as a team, can add some good numbers to the company's goals. Tackling company's OKRs although might sound as a very management kind of a thing, it's also a very good basis for salary negotiation. So always measure your improvements, however small they might be!

## In closing
Often we think about legacy software as a hard-to-maintain code. And we are very often right. But also we always have to think about what legacy also brings to the table. It often is stability (as in stable salary), product to expriment with, nice user base to measure on.

We just have to look a little bit more widely and try to improve the legacy code and the business in one go!
