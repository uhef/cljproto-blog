---
layout: post
title:  "Clojure + ClojureScript + Vim + REPL"
date:   2015-06-10 20:50:19
categories: 
---
Although there are multiple blog posts about getting Clojure / ClojureScript toolchain to work on Vim I decided that there is room for one more. Motivation for this post is to make it easy for anybody to set up toolchain for Clojure + ClojureScript project where Vim REPL integration works seamleslly for both Clojure and ClojureScript code. In this setup Clojure REPL runs in host Clojure runtime and ClojureScript REPL executes code on browser.

First thing to do is to make sure you [fireplace.vim][fireplace] installed in your Vim bundles. For the setup described here I am using fireplace version from commit hash `89aee9c...` (for lack of better versioning) and Vim 7.4.52. Just follow installation instructions at fireplace github repo. This should be straight forward.

What follows is not that straight forward though. We are going to get Vim REPL integration work on ClojureScript so that the ClojureScript code is executed in the browser. On the side we will also get REPL integration from Vim to Clojure.

Create an empty Clojure Ring project for this exercise (once following this true it will be easy for you to integrate the ideas to another project as well):

`lein new compojure clj-cljs-vim-repl`

You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve --watch`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll’s dedicated Help repository][jekyll-help].

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
[fireplace]: https://github.com/tpope/vim-fireplace

