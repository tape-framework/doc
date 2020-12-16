## Tutorial

This tutorial is meant to make you familiar with Tape Framework. It's not a comprehensive reference.

#### Prerequisites

Before learning how to use Tape Framework you need to know the following:
- [Reagent](https://github.com/reagent-project/reagent)
- [Re-Frame](https://github.com/day8/re-frame)
- [Integrant](https://github.com/weavejester/integrant)
- [Integrant REPL](https://github.com/weavejester/integrant-repl)
- [clj-new](https://github.com/seancorfield/clj-new)
- [Figwheel](https://figwheel.org/)
- [Reitit](https://github.com/metosin/reitit)

#### Generate an app

We generate an app by means of `tape.clj-template`; we assume [clj-new](https://github.com/seancorfield/clj-new) is
installed and configured.  

Until the first release you will have to first clone each subproject:

```bash
mkdir tape
cd tape
git clone git@github.com:tape-framework/versions.git
git clone git@github.com:tape-framework/refmap.git
git clone git@github.com:tape-framework/module.git
git clone git@github.com:tape-framework/mvc.git
git clone git@github.com:tape-framework/tools.git
git clone git@github.com:tape-framework/toasts.git
git clone git@github.com:tape-framework/dev.git
git clone git@github.com:tape-framework/clj-template.git
```

Now generate an app:

```bash
CLJ_CONFIG=./versions/ clj -A:versions \
  -Sdeps '{:deps {tape/clj-template {:local/root "./clj-template"}}}' \
  -X:new clj-new/create :template tape :name myname/myapp
```

This will generate the following files tree:

```text
myapp
├── deps.edn
├── dev
│   └── src
│       └── cljs
│       │   └── user.cljs
│       └── myname
│           └── myapp
│               └── dev.cljs
├── resources
│   └── public
│   │   └── index.html
│   │   └── favicon.ico
│   │   └── css
│   │       └── style.css
│   └── myname
│       └── myapp
│           └── config.edn
└── src
    └── myname
        └── myapp
            ├── core.cljs
            └── app
                ├── layouts
                │   └── app.cljs
                └── hello
                    ├── controller.cljs
                    └── view.cljs
```

#### Open the REPL

First, start a Clojure REPL:

```bash
cd myapp
CLJ_CONFIG=../versions/ clj -A:versions:test # development tools are in the test alias
```

Then, start a ClojureScript REPL via Figwheel:

```clojure
(build/fig) ;; starts Figwheel and opens a browser at http://localhost:9500/
```

Development tooling based on [Integrant REPL](https://github.com/weavejester/integrant-repl) is in `myname.myapp.dev` 
namespace:

```clojure
(in-ns 'myname.myapp.dev)
;; (repl/halt), (go), (repl/reset) etc. 
```

Note we assume the following namespaces below:

```clojure
(:require
 [reagent.core :as r]
 [re-frame.core :as rf]
 [tape.mvc.controller :as c :include-macros true]
 [tape.mvc.view :as v :include-macros true]
 [tape.router :as router]
 [tape.toasts.controller :as toasts.c]
 [tape.toasts.view :as toasts.v]
 [myname.myapp.app.hello.controller :as hello.c]
 [myname.myapp.app.hello.view :as hello.v])
```

#### Layouts and current view

The generated app has in `app/layouts/app.cljs` a reagent function called `app`. This is mounted in the DOM and is the
entrypoint to our app. In it, we define an HTML page layout and render the "current view".  

```clojure
(defn app []
  (let [current-view-fn @(rf/subscribe [::v/current-fn])]
    [:<>
     [:section.section
      [:div.container
       (when (some? current-view-fn)
         [current-view-fn])]]
     [toasts.v/index]]))
```

The "current page" that changes alongside the URL (whether via hash-change or History API) is so established on the web
that we decided to make it available by default in Tape. You don't have to necessarily use it, but if you want to, it's
available.

The "current view" is a reagent view function that can be changed by manipulating `::v/current` entry in app-db. It's 
value must be a namespaced keyword mapping to a controller handler (more on that later):

```clojure
{::v/current ::hello.c/index} ;; points to hello.v/index
```

Note that we use the controller in the value. There are two existing subscriptions:

```clojure
(rf/subscribe [::v/current]) ;; yields the keyword ::hello.c/index
(rf/subscribe [::v/current-fn]) ;; yields the function hello.v/index
```

#### Controllers

Controllers are namespaces with Re-Frame handlers, but these are not registered directly to Re-Frame. We use a special 
DSL. The generated app has a sample controller in `myname/myapp/app/hello/controller.cljs`. In it we have an event 
handler:

```clojure
(defn ^::c/event-db index [_db [_ev-id _params]]
  {::say "Hello Tape Framework!"})
```

At startup, we dispatch the `[::home.c/index]` event that's handled by this handler (events are always namespaced and 
there is a correspondence between event names and handler names). Let's add another one below it that changes the 
greeting:

```clojure
(defn ^::c/event-db change [db [_ev-id _params]]
  (assoc db ::say "Hello World!"))
```

We also have a subscription in our `hello` controller for our greeting:

```clojure
(defn ^::c/sub say [db _query]
  (::say db))
```

At the end we call the `(c/defmodule)` macro that inspects the namespace and defines a `tape.module`. This is added in
the modules config map by "modules discovery".

When a controller namespace is required in another namespace, the naming convention is as follows:

```clojure
(:require [myname.myapp.app.hello.controller :as hello.c])
```

#### Views

Views are namespaces with Reagent functions. Functions that can become the "current view" must be annotated with 
`^::v/view` and we call them view functions. Reagent functions that are not annotated we call partials. The generated 
app has a sample view in `myname/myapp/app/hello/view.cljs`. In it, we have Reagent function:

```clojure
(defn ^::v/view index []
  (let [say @(rf/subscribe [::hello.c/say])]
    [:p.hello-tape say]))
```

There can be a name correspondence between a controller event handler and a view function, in our case here:
`hello.c/index` <-> `hello.v/index`. If there is such a correspondence, after the handler runs, if there is no 
`::v/current` in set app-db, our view function is automatically set as current (by an interceptor).  

At the end we call the `(v/defmodule myname.myapp.app.hello.controller)` macro that inspects the namespace and defines a
`tape.module`. This is added in the modules config map by "modules discovery". Note that the argument here is the 
corresponding controller namespace, as a symbol, **fully qualified**.

Let's add a button that calls our `hello.c/change` handler:

```clojure
(defn ^::v/view index []
  (let [say @(rf/subscribe [::hello.c/say])]
    [:p.hello-tape say]
    [:button.button {:on-click #(rf/dispatch [::hello.c/change])}]))
```

When a view namespace is required in another namespace, the naming convention is as follows:

```clojure
(:require [myname.myapp.app.hello.view :as hello.v])
```

#### Routing

The `tape.router` module adds routing capabilities based on [Reitit](https://github.com/metosin/reitit). Routes are 
defined in controllers at the beginning in a var named `routes`. Our sample `hello` controller has the following routes:

```clojure
(def routes
  ["" {:coercion rcs/coercion}
   ["/say" ::index]])
```

When the URL of the page changes (via hash-change or History API - depending on how the router is configured in the
config map - see `myname/myapp/core.clj`) and event corresponding to the matching route is dispatched. In our case above
When the page navigates to "http://localhost:9500/#/say" the event `[::hello.c/index params]` is dispatched. The 
`params` contains path and query params matched by the route. Let's add a new route to our `change` handler: 

```clojure
(def routes
  ["" {:coercion rcs/coercion}
   ["/say" ::index]
   ["/change/:to" ::change]])
```

And let's make our `change` handler aware of the `:to` param:

```clojure
(defn ^::c/event-db change [db [_ev-id params]]
  (assoc db ::say (-> params :path :to)))
```

We can now change our button in the view to a link:

```clojure
(defn ^::v/view index []
  (let [say @(rf/subscribe [::hello.c/say])]
    [:p.hello-tape say]
    [:a.button {:href (router/href* [::hello.c/change {:to "Wazaaa"}])}]))
```

We used the name of the route and params, and Reitit assembled the route path for us (a sort of reverse routing).
When we click the link the browse address changes to "localhost:9500/#/change/Wazaaa". This is matched by the router
dispatching `[::hello.c/change {:path {:to "Wazaaa"}}]`. This is handled by `hello.c/change` and the greeting is 
changed.

#### Timeouts and Intervals

To be idiomatic when setting timeouts and intervals `tape.tools` offers the following modules for your `config.edn`:

```clojure
{:tape.tools.timeouts.controller/module nil
 :tape.tools.intervals.controller/module nil}
```

To set a timeout dispatch the following event:

```clojure
(rf/dispatch [::timeouts.c/set {:ms 3000                  ;; miliseconds
                                :set [::was-set]          ;; optional, dispatched as [::was-set timeout-id]
                                :timeout [::timed-out]}]) ;; dispatched on timeout
``` 

To clear a timeout: `(rf/dispatch [::timeouts.c/clear timeout-id])`.  
Similar for intervals.

#### Working with forms

A form with a number of fields is mapped to a hash-map in app-db. Let's say we have a login form with an email and 
password.

##### Form controller

We start by creating a controller with:
1. An event handler that sets a pair in the map.
2. A subscription that reads the map.

```clojure
(ns myname.myapp.app.guis.login.controller
  (:require [tape.mvc.controller :as c :include-macros true]))

(defn ^::c/event-db field
  "Assoc in app-db login map value v at key k."
  [db [_ k v]] (assoc-in db [::login k] v))

(defn ^::c/sub login
  "Subscription to the login map"
  [db _] (::login db))

(c/defmodule)
```

##### Form view

In our corresponding view, we make a `form-fields` partial that will render the form fields.  
Using `tape.tools` lens we make a function that "gets" by reading the subscription and "sets" by dispatching an event.
Using `tape.tools`'s `form/field` we make inputs that control our hash-map via the above lens function.
Finally, we attempt login by dispatching an event if the HTML5 Validation API doesn't complain, via `form/when-valid`.

```clojure
(ns myname.myapp.app.guis.login.view
  (:require [re-frame.core :as rf]
            [tape.mvc.view :as v :include-macros true]
            [tape.tools :as tools]
            [tape.tools.ui.form :as form]
            [myname.myapp.app.guis.login.controller :as login.c]))

(defn- form-fields []
  (let [lens (tools/lens* ::login.c/login  ;; the subscription
                          ::crud.c/field)] ;; the handler
    [:<>
     [:div.field
      [:label.label "Email"]
      [:div.control
       [form/field {:type :text, :class "input", :source lens, :field :email, :required true}]]]
     [:div.field
      [:label.label "Password"]
      [:div.control
       [form/field {:type :password, :class "input", :source lens, :field :password, :required true}]]]]))

(defn ^::v/view new []
  [:form
   [:h2 "Login"]
   [form-fields]
   [:div.field
    [:label.label "Password"]
    [:div.control
     [:button.button.is-primary {:on-click (form/when-valid #(rf/dispatch [::login.c/create]))} "Log in"]]]])

(c/defmodule myname.myapp.app.guis.login.controller)
```

#### Toasts notifications

Generally you want to notify the user of success/failure of various actions. The `tape.toasts` module allows you to 
flash such messages. If you used the Tape Framework app generator the module is already configured. To show a message
dispatch `(rf/dispatch [::toasts.c/create :info "Some message"])`. Toast kind can be one of: `:success`, `:danger`, 
`:warning`, `:info`. Example in controller handler:

```clojure
(defn ^::c/event-fx create [{:keys [db]} _]
  {:db         (update db ::people conj (::person db))
   :dispatch-n [[::router/navigate [::index]]
                [::toasts.c/create :success "Person added!"]]})
```

#### Ergonomic API

When dispatching and subscribing, we have a number of macros called the
"Ergonomic API". These are equivalent to the ones in Re-Frame (or Tape - 
ending in *), except they take a symbol instead of a keyword in the first
position of the vector. This symbol is IDE navigable ("jump to definition").
The macros macroexpand to the standard API, thus have no runtime cost.

```clojure
(v/dispatch [posts.c/index])                ;; => (rf/dispatch [::posts.c/index])
(v/subscribe [posts.c/posts])               ;; => (rf/subscribe [::posts.c/posts])
(router/href [greet.c/hi {:say "Hi!"}])     ;; => (router/href* [::greet.c/hi {:say "Hi!"}])
(router/navigate [greet.c/hi {:say "Hi!"}]) ;; => (router/navigate* [::greet.c/hi {:say "Hi!"}])
(tools/lens posts.c/post posts.c/field)     ;; => (tools/lens* ::posts.c/post ::posts.c/field)
```

#### See also

See the READMEs of each subproject.

#### License

Copyright © 2020 clyfe

Distributed under the MIT license.
