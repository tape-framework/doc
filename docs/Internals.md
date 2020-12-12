## Internals

#### Prerequisites

Before learning how to use Tape Framework you need to know the following:
- [Reagent](https://github.com/reagent-project/reagent)
- [Re-Frame](https://github.com/day8/re-frame)
- [Integrant](https://github.com/weavejester/integrant)
- [Integrant REPL](https://github.com/weavejester/integrant-repl)

Having knowledge of how [Duct Framework](https://github.com/duct-framework) works helps.

#### Building blocks

A Tape Framework application based on the following building blocks:
1. `tape.module` a modules systems that yields an Integrant system.
2. `tape.mvc` an interface to Re-Frame that integrates with the modules system.
3. Various (optional) plug-in modules:  
    a) `tape.router` for routing hash-change or HTML5 History.  
    b) `tape.tools` timers and intervals modules & ui helpers.  
    c) `tape.toasts` toasts notifications module.
4. `tape.dev` development workflows.

Before proceeding, you should make yourself familiar with each of these by going over their READMEs.

#### Application startup

An application starts with a `config.edn` configuration map, typically stored in `resources/app-name/config.edn`.
This map has an entry for each module used along with its configuration. Example:

```clojure
{:tape.mvc.controller/module nil
 :tape.mvc.view/module nil
 :tape.mvc/module nil
 :tape.router/module nil
 :tape.tools.timeouts.controller/module nil
 :tape.tools.intervals.controller/module nil
 :tape.toasts.controller/module nil}
```

The following namespace is assumed in further descriptions:

```clojure
(ns myapp.core
  (:require [integrant.core :as ig]
            [tape.module :as module :include-macros true] 
            [tape.mvc :as mvc :include-macros true]))
```

At compile time this map is read by a macro `(module/read-config "myname/myapp/config.edn")`. In it we merge the 
controller and views modules map. This latter map is obtained also at compile time by a macro called "modules 
discovery":

```clojure
(let [{:keys [modules routes]} (mvc/modules-discovery "src/myapp/app")])
```

If that's not to your liking you can manually list each module. Once a modules map is fully assembled with all the 
required configuration it's `ig/init` -ialized resulting in a **modules system**. This modules system is folded in an
**application system config map** that's `ig/init` -ialized into our final **application system**.

#### License

Copyright © 2020 clyfe

Distributed under the MIT license.
