---
layout: post
title: Fast Rails app prototyping
category: posts
---

There are often times when you have an idea that will refuse to leave your mind.
It is the most beautiful sensation you can ever have.
As an any other idea, you must first see it shape before you can throw it away.
At this point it is the most important to establish a very effective and noise free channel between your idea and its implementation.
I will describe what I use and what I do to quickly make small Rails apps, and what I use for it.

    git clone git@github.com:alisnic/naked_rail.git myapp
    cd myapp
    rm -rf .git
    git init && git add . && git commit -a -m "initial commit"
  

# Prototyping
I use an [app template][1] for all my new apps. As many things that I created, it was done to write less, and includes the following libs:

- [HAML][11] + [SASS][12] + [CoffeeScript][13]
- [Twitter Bootstrap][14]
- PagesController which renders its action. To add a view, all I need to do is to create a `foo.haml` file in `app/views/pages`, and it will be accessible at `/pages/foo`. No new routes, no new controllers. You first idea is static.
- [guard-livereload][6]. I have a vim window alongside the browser on my FullHD monitor. Every time I change the code, the page reloads automatically. This is **very** convenient, especially when writing the markup.
- [flash][9] and [error][10] partials. To show a model validation errors in a form, all I have to do is add `render 'validation_errors', :resource => @entity`
- [better_errors][8]. Really cool gem, makes breaking things pleasant. It will open a ruby shell in the context of the error in the browser.

# Growing
If my idea proved strong and stable, I have the tools to grow on the code I made in the prototyping step, and by that I mean I have all the testing tools I need:

- [Rspec][4]
- [Capybara-webkit][3] (can execute JS) + [navigation steps][2]
- [Shoulda][5]. Testing models is plain easy and straightforward. The main idea - don't test ActiveRecord, it has been tested for you. (ex: `it { should belong_to(:user) }`)
- [FactoryGirl][7]


# Final note
I have presented a set of tools and practices to create a Rails app fast. As many other things out there, it is highly opinionated. Also, the repo which I created is not licensed in any way, so do whatever the hell to want to do with it.


[1]: https://github.com/alisnic/naked_rail
[2]: https://github.com/alisnic/naked_rail/blob/master/features/step_definitions/navigation_steps.rb
[3]: https://github.com/thoughtbot/capybara-webkit
[4]: http://rspec.info/
[5]: https://github.com/thoughtbot/shoulda
[6]: https://github.com/guard/guard-livereload
[7]: https://github.com/thoughtbot/factory_girl
[8]: https://github.com/charliesome/better_errors
[9]: https://github.com/alisnic/naked_rail/blob/master/app/views/application/_flash_messages.html.haml
[10]: https://github.com/alisnic/naked_rail/blob/master/app/views/application/_validation_errors.html.haml
[11]: http://haml.info/
[12]: http://sass-lang.com/
[13]: http://coffeescript.org/
[14]: http://twitter.github.com/bootstrap/
