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
Create an empty Clojure Compojure project for this exercise. After doing this once it should be easy to integrate REPL integration to an existing project as well:

`lein new compojure vimcljsrepl`

Upgrade the used Clojure, Compojure and Leiningen versions immediatelly to something newer. We are going to need these versions when we start to work on ClojureScript. Modify the dependencies in `project.clj` to look like this:
{% highlight clojure %}
:min-lein-version "2.5.0"
:dependencies [[org.clojure/clojure "1.7.0-RC1"]
               [compojure "1.3.4"]
               [ring/ring-defaults "0.1.2"]]
{% endhighlight %}

Also create `clj` directory for Clojure code to separate it from ClojureScript code and move `handler.clj`there:

    mkdir -p src/clj/vimcljsrepl
    mv src/vimcljsrepl/handler.clj src/clj/vimcljsrepl/

Add the `clj` path as a source path to `project.clj`:
{% highlight clojure %}
:source-paths ["src/clj"]
{% endhighlight %}

Enable Vim + REPL integration on Clojure code
---------------------------------------------
Add function to `handler.clj` that can be called from REPL to start the server:

{% highlight clojure %}
(defn run []
  (run-jetty (wrap-reload #'app '(vimcljsrepl.handler)) {:port 8080 :join? false}))
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
:dependencies [[org.clojure/clojure "1.7.0-RC1"]
               [compojure "1.3.4"]
               [ring/ring-defaults "0.1.2"]
               [ring/ring-jetty-adapter "1.3.2"]
               [ring/ring-devel "1.3.2"]]
{% endhighlight %}

You can now try the REPL and the service by running

`lein repl`

and executing the following in the REPL:

    user=> (use 'vimcljsrepl.handler)
    user=> (run)

This will start the server in port 8080. Directing your browser to [http://localhost:8080/][localhost] should show friendly  *Hello World*.

REPL integration to Vim through [fireplace.vim][fireplace] should now work. To give it a try open `handler.clj` in Vim and edit the the string `"Hello World"` to `"Hello REPL"`. Evaluate both functions `approutes` and `app`. You can do this for instance by `cq%` and enter at either opening or closing parantheses of the functions. For more instructions on how to use fireplace see their [github page][fireplace]. Now refresh the browser and it should say *Hello REPL* instead of *Hello World*. No need to restart the server or save the file. Nice. 

Enable Vim + REPL integration on ClojureScript
----------------------------------------------
Next we are going to get the live browser REPL evaluation work from Vim on ClojureScript code. First we need to create some ClojureScript code that we can later modify and evaluate.

Create a new ClojureScript file `core.cljs` with following contents in `src/cljs/vimcljsrepl`:

{% highlight clojure %}
(ns vimcljsrepl.core
  (:require [weasel.repl :as weasel]))

(weasel/connect "ws://localhost:9001" :verbose true)
{% endhighlight %}

When ran in browser this peace of code will connect browser to ClojureScript REPL that we will have running shortly. Connection will be created using [weasel][weasel].

In order to make the above ClojureScript code work we will have to compile it. Add ClojureScript compilation step to application startup by modifying the `run` function in `handler.clj` to look like this:

{% highlight clojure %}
(defn run []
  (cljs.build.api/build "src/cljs" {:optimizations :none
                                    :pretty-print true
                                    :source-map true
                                    :main "vimcljsrepl.core"
                                    :asset-path "js/out"
                                    :output-to "resources/public/js/main.js"
                                    :output-dir "resources/public/js/out"})
  (run-jetty (wrap-reload #'app '(vimcljsrepl.handler)) {:port 8080 :join? false}))
{% endhighlight %}
*DISCLAIMER:* You might want to use [lein-cljsbuild][lein-cljsbuild] and/or [lein-figwheel][lein-figwheel] for ClojureScript compilation but to keep it simple we will go with the approach above.

In order to make the compilation work, add dependency to `handler.clj` so that namespace declaration looks something like this:

{% highlight clojure %}
(ns vimcljsrepl.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            [cljs.build.api :refer [build]]
            [ring.middleware.defaults :refer [wrap-defaults site-defaults]]
            [ring.adapter.jetty :refer [run-jetty]]
            [ring.middleware.reload :refer [wrap-reload]]))
{% endhighlight %}

Also add ClojureScript dependency to `project.clj`:

{% highlight clojure %}
:dependencies [[org.clojure/clojure "1.7.0-RC1"]
               [org.clojure/clojurescript "0.0-3291" :scope "provided"]
               [compojure "1.3.4"]
               [ring/ring-defaults "0.1.2"]
               [ring/ring-jetty-adapter "1.3.2"]
               [ring/ring-devel "1.3.2"]]
{% endhighlight %}

In order for [weasel][weasel] to work add weasel as dependency into the development profile in `project.clj`:

{% highlight clojure %}
{:dev {:dependencies [[weasel "0.7.0"]]}}
{% endhighlight %}

Notice also that you can remove the dependencies `javax.servlet/servlet-api` and `ring-mock` since we are not going to need them for this.

Add ClojureScript source path to source paths in `project.clj`. `source-paths` should now look like:

{% highlight clojure %}
:source-paths ["src/clj" "src/cljs"]
{% endhighlight %}

Finally, to load the newly compiled ClojureScript code create `index.html` in `resources/public` with contents like this:
{% highlight html %}
<html>
<head>
</head>
<body>
  <script type="application/javascript" src="js/main.js"></script>
</body>
</html>
{% endhighlight %}

Now if you restart the `lein repl` and run the commands to start the server and direct your browser to [http://localhost:8080/index.html][localhost-index] you should see from browser development tools that the browser tries to establish connection with the REPL.

Finally we are going to put in building blocks so that [fireplace.vim][fireplace] can use [piggieback][piggieback] to setup [weasel][weasel] nREPL session to which the browser can connect.

We are going to setup the `project.clj` first. Modify the development environment section so that it looks like following:
{% highlight clojure %}
{:dev {:repl-options {:init-ns vimcljsrepl.handler
                      :nrepl-middleware [cemerick.piggieback/wrap-cljs-repl]}
       :dependencies [[weasel "0.7.0"]
                      [org.clojure/tools.nrepl "0.2.10"]
                      [com.cemerick/piggieback "0.2.1"]]}}
{% endhighlight %}
As you can see we will use [piggieback][piggieback] as the nREPL middleware. This allows Vim to "piggieback" to [weasel][weasel] REPL. [piggieback][piggieback] requires `org.clojure/tools.nrepl` to be included explicitly. `init-ns` is set to `vimcljsrepl.handler`. This is just to remove the need to explicitly run `(use 'vimcljsrepl.handler)` when REPL is launched and can be left out.

We need a function that will launch REPL session in [piggieback][piggieback]. Create one in `handler.clj`:
{% highlight clojure %}
(defn repl-env []
  (weasel.repl.server/stop)
  (weasel.repl.websocket/repl-env :ip "0.0.0.0" :port 9001))
{% endhighlight %}

And update requires in `handler.clj` namespace accordingly:
{% highlight clojure %}
(ns vimcljsrepl.handler
  (:require [compojure.core :refer :all]
            [compojure.route :as route]
            weasel.repl.websocket
            weasel.repl.server
            [cljs.build.api :refer [build]]
            [ring.middleware.defaults :refer [wrap-defaults site-defaults]]
            [ring.adapter.jetty :refer [run-jetty]]
            [ring.middleware.reload :refer [wrap-reload]]))
{% endhighlight %}

Everything is now set for the ClojureScript Vim REPL integration to work.

To try it out run the REPL with `lein repl`, start the server by running `(run)` (notice that since `init-ns` was modified you don't need to run `(use 'vimcljsrepl.handler)` anymore). When the server is running go to Vim and run `:Piggieback (vimcljsrepl.handler/repl-env)`. This will halt the Vim until you connect to REPL with your browser. Direct your browser to [http://localhost:8080/index.html][localhost-index]. Connection from browser to REPL should now be established and the Vim should respond normally again. Go to `core.cljs` - file and add following line (no need to save the file):

{% highlight clojure %}
(js/alert "Hello Browser!")
{% endhighlight %}

Evaluate the alert for instance by entering `cq%` and enter at either opening or closing parantheses and observe the alert box on the browser.

Your REPL integration to both Clojure and ClojureScript are now working.

[fireplace]: https://github.com/tpope/vim-fireplace
[localhost]: http://localhost:8080/
[localhost-index]: http://localhost:8080/index.html
[weasel]:    https://github.com/tomjakubowski/weasel
[piggieback]: https://github.com/cemerick/piggieback
[lein-cljsbuild]: https://github.com/emezeske/lein-cljsbuild
[lein-figwheel]: https://github.com/bhauman/lein-figwheel
