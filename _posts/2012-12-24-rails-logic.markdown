---
layout: post
title: On logic in a Rails app
category: posts
---

Recently, Grant Ammons wrote [a nice article](http://gammons.github.com/architecture/2012/12/22/where-the-logic-hides/) about where logic should lie in a Rails app. I would like to complement his opinion, and probably add some real life arguments to it.

Grant extracted logic from the controller, and put it in a separate object, which I also do most of the time. However, a user named DHH (which probably is DHH) said:
> So all you've done here is extract the work of the controller action into a separate object. This adds needless indirection, makes the code base harder to follow, and is completely unnecessary.

I strongly disagree, and I have good reasons for it.

You know what makes the code base hard to follow? Fat controllers with logic in them. I worked on a Rails app with Controllers of 800 lines of code, **it. is. a. freaking. nightmare.**

## My approach
I look at Rails as web presentation layer, it makes my app work in the browser. This has some consequences to my codebase:

- data-only models (no logic should be in models)
- logicless controllers (no logic should be there either)
- anything else is in single-responsibility objects in `app/domain/`

You might think that this is a refactor pattern, and you are right. And you might say that this unnecessary to do from the start, and you may also be right. But the cost of creating a file and moving behaviour there is exactly 0, but the benefits are there from day 1.

## Why I do it
### Fast tests

The least I am coupled to Rails, the fastest my tests are. Currently, on a medium Rails project my tests run 2-3 seconds, where 90% of the time is wasted to boot rails and test models.

I don't test my controllers, because they have no logic in them. My methods in controllers don't have more than 4-5 lines of code.

```ruby
class RatingsController < ApplicationController
  def create
    success = RatesBusiness.add_rating(params[:business_id],
                                       params[:score],
                                       current_user)
    render json: {success: success}
  end
end
```

`RatesBusiness` has some foursquare-related logic besides just creating a entry in the database. Using this approach, I have a *very* fast TDD flow, because most of the time I don't need Rails to test my logic. (any dependency to models is stubbed, as any other external dependency).

Your controller only connects your logic to the web interface.

### Code and forget

If you have a well tested functionality that works, you don't need see it in front of you when you code in your application. This is where single-responsibility objects kick in, I TDD them one time, I know about their existence and what their interface is, I will not ever see their code again.

### Framework agnostic

When your app logic is not coupled with Rails, you do not depend on it, its version, or interface. Isolating some functionality to a service is a matter of moving some files around.

### Reduced stress and debug times

Most of my classes fit in one screen, and have descriptive names as `CategorizesCampagins` or `FiltersBusinesses`, which makes debugging them very, very easy and pleasant. I don't need to scroll the hell out of my screen to understand how an isolated piece of functionality works.

## Conclusion

Rails is just a framework to make your app work on the web, don't mix your logic with rails-y stuff. Your app domain should be defined anywhere besides Rails views, controllers, and models.

## Inspiration

First, and most influential person for me is [Gary Bernhardt](https://twitter.com/garybernhardt) and his screencast series [destroyallsoftware](https://www.destroyallsoftware.com/screencasts). If you want to learn isolated testing - watch this guy, he is amazing. (He even test-drives bash scripts :D)

[Your objects the UNIX way](http://blog.codeclimate.com/blog/2012/11/28/your-objects-the-unix-way/).

Also, my approach was influenced by [Hexagonal Rails](http://www.youtube.com/watch?v=CGN4RFkhH2M) concept.
