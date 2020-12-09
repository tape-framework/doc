### Tape Framework pre.alpha

STATUS: Pre-alpha, in design and prototyping phase. Feedback appreciated.  
This is an early preview. You will have to use clojure.deps with git coordinates for now.

Inspired by [Duct Framework](https://github.com/duct-framework), [Tape Framework](https://github.com/tape-framework) 
tries to be a Frontend Doppelgänger. Useful for folks wanting ready-made DI/IOC-glue for their new Re-Frame + Reagent 
apps. It also gives slightly more infrastructure, so you don't start at zero.

There is a small [Tutorial](../docs/Tutorial.md) and [Internals](../docs/Internals.md) description.
To see how code based on it looks, check out the [Tape 7GUis](https://github.com/tape-framework/7guis) implementation.
To generate an app, check out the [Code Generator](https://github.com/tape-framework/clj-template).

It does:
1. Organize the frontend app around an Integrant system.
2. Build that system out of a module system (`tape.module` ported to cljc from Duct Framework).
3. Explore an alternative interface to Re-Frame (`tape.mvc`) that fits with Integrant.
4. Use conventions to thin the code down and make obvious what's where and what gets handled by what.
5. Build a body of recipes for dev workflow, routing, forms, validation etc.
   (`tape.dev`, `tape.router`, `tape.tools`, `tape.toasts`).

Future plans:
1. Right now `tape.mvc` is mostly just a proxy to Re-Frame. The **longer term stake** here is for things from system map
   to be injected in handlers and views (via fn params metadata) and allow to **move away from globals** (event-reactor,
   app-db & other fxs state: to live in the system map). Expect breaking changes for that to happen.
2. Extract the "current view" mechanism in a module.

Notes:
1. It's all optional, you can use just the pieces you want. Don't like `tape.mvc`? Use plain `ig/init-key`. Don't
   like a certain module? Leave it out / bring your own.
2. The default experience is "prefer easy". If you prefer simple, just don't use the parts that seem magic (like, say,
   modules discovery) and be a bit more verbose in configs.

Individual merits of some of the subprojects:
1. [Tape Module](https://github.com/tape-framework/module) the module system ported from Duct, now available in 
   ClojureScript - with a certain level of generality.
2. [Tape RefMap](https://github.com/tape-framework/refmap) like Integrant's RefSet but brings the key in the output and
   returns a hash-map.
3. [Tape Tools](https://github.com/tape-framework/tools) lens and form bits are useful outside Tape too.
4. [Tape Dev](https://github.com/tape-framework/dev) can be used standalone.
