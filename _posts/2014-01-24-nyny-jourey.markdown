---
layout: post
title: NYNY 3.2 - a micro web framework meets the Rails' router
category: posts
---

_Summary_: [New York, New York][nyny] 3.2 uses the Rails' router.

As a web developer that writes in Rails, I got a lot of interests in the
internals of web frameworks. Rails, however, is pretty big, and a person can
get lost in its source code.

So I decided that I should make a small web framework myself, and half a year
ago I released "New York, New York", which I positioned as a small Sinatra
clone at that time.

Time has passed, and I spend all my free coding resources to hack on NYNY and
to experiment with it. During this period I also started using NYNY to prototype
any ideas I had for web apps, slowly improving it. It did well as a framework, and I never
stopped using it.

However, I was missing something. I was missing Rails' routing. But I was not
ready to implement something similar myself, because that would violate one
of the primary goals of NYNY - being simple and obvious. And then I met
[Journey][journey] - the Rails' router.

What seemed a crazy idea at the time, now has become a reality. NYNY 3.2 uses
Journey for routing. Defaults, constraints, splats and all the syntax you are
used to is available in NYNY (in a much simple form):

You can use any construct or convention [supported in Rails][bound-params]
for the path string:

```ruby
class App < NYNY::App
  get '/:foo(/:bar)' do
    "Hello #{params[:foo]} #{params[:bar]}!"
  end
end
```

You can use [the same constraints][constraints] you use in Rails:

```ruby
class App < NYNY::App
  get '/', :constraints => {:format => :html} do
    'html'
  end
end
```

And same defaults:

```ruby
class App < NYNY::App
  get '/', :defaults => {:format => 'html'} do
    'html'
  end
end
```

NYNY does that and much more in 281 lines of code (including empty lines).

So, if you are a guy that:

- wants to understand how to integrate Journey in your web framework
- wants to hack on something small
- wants to use a web framework with very simple semantics and powerful routing.
- has an idea or a suggestion for NYNY

Check out [NYNY][nyny], I'll try te be as helpful as I can.

[nyny]: https://github.com/alisnic/nyny
[journey]: https://github.com/rails/journey
[constraints]: http://guides.rubyonrails.org/routing.html#request-based-constraints
[bound-params]: http://guides.rubyonrails.org/routing.html#bound-parameters


