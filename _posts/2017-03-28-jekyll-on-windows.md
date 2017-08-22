---
title: Using Jekyll on Windows
teaser: Plenty of bugs when you launch a Jekyll template on Windows? Read this post!
category: problems
tags: [windows, jekyll]
featured_comments:
  - url: 
---

When I set up Jekyll on my Windows laptop, the process didn't go that smoothly.

I first downloaded Ruby 2.4.1 and when I entered the command <code>bundle update</code> it gave me error:
<pre><code>This is a code block.
</code></pre>

It actually didn't mean you need to install make on Windows. In the Ruby's [github installation guide](https://github.com/oneclick/rubyinstaller2) it mentions you must install the corresponding DevKit.

Just remember to add Ruby, MSYS2 and the DevKit's bin folder into PATH enviroment variable under **System variables** section. For version 2.4.1, you also need to install <code>ridk</code> using <code>ridk install</code> command.
There is another sentence mentioning that some gem packages need <code>pacman</code> and unfortunately mine needs it. <code>pacman</code> can be installed following [this guide](http://www.msys2.org/).
Everything was ready by this step, well, I thought. I start launching my website using <code>bundle exec jekyll serve</code>, but it poped out error saying that my template needs Ruby version earlier than 2.4.1... 
I re-installed Ruby 2.3.4 and everything worked great.

**I suggest try installing older Ruby versions (ex. 2.3.4) since many templates are not compatible with 2.4.1.**


---
