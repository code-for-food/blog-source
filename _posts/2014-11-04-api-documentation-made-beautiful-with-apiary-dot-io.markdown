---
layout: post
title: "API documentation made beautiful with Apiary.io"
date: 2014-11-04 22:19:17 +0700
comments: true
author: codeforfoods
author_site: http://codeforfoods.net/about-me
source_name: itworld
source_site: http://www.itworld.com/article/2700666/development/api-documentation-made-beautiful-with-apiary-io.html
categories: [api, apiary.io]
---

##### Apiary is a free, hosted API documentation repository

The least exciting part of any software project I work on is the documentation. With APIs in particular, documentation can be time consuming because it needs to provide enough detail for any consumer to understand and interface with it. For that reason, proper documentation of an API is even more crucial.

The most recent two projects I've been working on both required that full APIs be developed in addition to the core of the platform. On one of them I was building both the API and the client to consume it so the need to describe the API was nil. On the other I'm working in tandem with a colleague and need to communicate the API calls and requirements as they're completed. My first attempt at doing so was pretty lame. I typed out the API calls for a particular controller and attached sample requests/responses in text files and emailed them across the room. I'm happy to say that was the last time I did that.

As soon as I hit send on that email I knew I was doing something stupid. I asked another colleague about what he had used in the past and he pointed me toward [Apiary.io](http://apiary.io). Apiary is a web based repository for API documentation that also includes server mocking, code samples, automated testing, and GitHub sync. It's free for most needs including both public and private API documentation.

I created a new account in a single click since I was already logged into GitHub and I was off and running creating my API documentation. Once I got the hang of the Markdown related syntax called [API Blueprint](http://apiary.io/blueprint), I was able to quickly document each API resource in far greater detail than I had been previously. I then invited my colleague to the Apiary API project and things became much better for both of us. Now he can simply refresh the page to pick up any new documentation and get full detail and examples of each API resource at will.

What does the resulting documentation look like you might ask? Here's a screenshot but you'll want to head over to [one of the featured APIs](http://docs.themoviedb.apiary.io/) to check it out for yourself.

{% img http://images.techhive.com/images/idge/imported/article/itw/2014/02/28/image1-100480198-orig.png %}

So far we've been pretty happy using Apiary. There are some helpful features to the system that can aid your efforts such as reusable request and response models, but it's no silver bullet. Good documentation takes time and it's probably not possible to completely avoid that.

The only real complaint I have with the system so far is that your entire API document is constructed in a single, loooong page. This works OK for reading because things collapse and there is a table of contents, but for editing, it can be a mess. Aside from that it's been a great collaboration and organization tool. It's unlikely that my API would be as well documented as it is in Apiary if I had chosen to write it in some other way.

