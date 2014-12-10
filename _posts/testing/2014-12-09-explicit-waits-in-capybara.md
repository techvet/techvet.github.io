---
layout: post
title: "Explicit Waits in Capybara"
modified:
categories: testing
excerpt:
tags: [capybara, ruby]
image:
  feature:
date: 2014-12-09T21:55:21-06:00
---

Capybara has a way to use explicit waits but when I did a simple google search for 'explicit waits in capybara' I couldn't seem to find any direct answers. After working on a large test suite I was informed by a team mate how we can achieve this.

~~~ ruby
seconds_to_wait = 10
Capybara.using_wait_time seconds_to_wait do
 #your page interactions
 page.has_css?('table tr.foo')
end
~~~

Basically you can wrap any capybara actions with <code>Capybara.using\_wait_time</code> and put any capybara code you want to execute with an explicit wait inside the block like the example above.

Here is the link to the capybara documentation on that method: 
[RubyDoc Link](http://www.rubydoc.info/github/jnicklas/capybara/Capybara.using_wait_time)