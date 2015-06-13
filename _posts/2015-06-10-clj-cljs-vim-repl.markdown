---
layout: post
title:  "Clojure + ClojureScript + Vim + REPL"
date:   2015-06-10 20:50:19
categories: 
---
Although there are multiple blog posts about getting Clojure / ClojureScript toolchain to work on Vim I decided that there is room for one more. Motivation for this post is to make it easy for anybody to set up toolchain for Clojure + ClojureScript project where Vim REPL integration works seamlessly for both Clojure and ClojureScript code. In this setup Clojure REPL runs in host Clojure runtime and ClojureScript REPL executes code on browser.

First thing to do is to make sure you [fireplace.vim][fireplace] installed in your Vim bundles. For the setup described here I am using fireplace version from commit hash `89aee9c...` (for lack of better versioning) and Vim 7.4.52. Just follow installation instructions at fireplace github repo. This should be straight forward.

What follows is not that straight forward though. We are going to get Vim REPL integration work on ClojureScript so that the ClojureScript code is executed in the browser. On the side we will also get REPL integration from Vim to Clojure.

Create test project
-------------------
Create an empty Clojure Compojure project for this exercise. After doing this once it should be easy to integrate REPL integration to an existing project:

`lein new compojure clj-cljs-vim-repl`

Enable Vim + REPL integration on Clojure code
---------------------------------------------
Add function to `handler.clj` that can be called from REPL to start the server:

{% highlight clojure %}
(defn boot []
  (run-jetty (wrap-reload #'app '(clj-cljs-vim-repl.handler)) {:port 8080}))
{% endhighlight %}

`run-jetty` will run the service in Jetty standalone server. `wrap-reload` will reload the namespace before each request is handled.

The above relies on some Jetty dependencies. Add them to the `:require` - clause of the `handler.clj`:

{% highlight clojure %}
(ns clj-cljs-vim-repl.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [ring.middleware.defaults :refer [wrap-defaults site-defaults]]
            [ring.adapter.jetty :refer [run-jetty]]
            [ring.middleware.reload :refer [wrap-reload]]))
{% endhighlight %}

And to `project.clj`:

{% highlight clojure %}
:dependencies [[org.clojure/clojure "1.6.0"]
               [compojure "1.3.1"]
               [ring/ring-defaults "0.1.2"]
               [ring/ring-jetty-adapter "1.3.2"]
               [ring/ring-devel "1.3.2"]]
{% endhighlight %}

You can now try the REPL and the service by running

`lein repl`

and executing the following in the REPL:

    user=> (use 'clj-cljs-vim-repl.handler)
    user=> (boot)

This will start the server in port 8080. Directing your browser to [http://localhost:8080/][localhost] should print *Hello World*.

REPL integration to Vim through [fireplace.vim][fireplace] should now work. To give it a try open `handler.clj` in Vim and edit the the string `"Hello World"` to `"Hello REPL"`. Evaluate both functions `approutes` and `app`. You can do this for instance by `cq%` and enter at either opening or closing parantheses of the functions. For more instructions on how to use fireplace see their github page. Now refresh the browser and it should say *Hello REPL* instead of *Hello World*. No need to restart the server or save the file. Nice. 

[fireplace]: https://github.com/tpope/vim-fireplace
[localhost]: http://localhost:8080/

