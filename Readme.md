
 ![page router logo](http://f.cl.ly/items/3i3n001d0s1Q031r2q1P/page.png)

  The ~1200 byte Express inspired client-side router that gets out of your way.

```js
page('/', index)
page('/user/:user', show)
page('/user/:user/edit', edit)
page('/user/:user/album', album)
page('/user/:user/album/sort', sort)
page('*', notfound)
page()
```

## Running examples

  To run examples do the following:

    $ npm install -g serve
    $ serve examples/profile
    $ open http://localhost:3000

 Currently we have examples for:
 
   - `basic` minimal application showing basic routing
   - `notfound` similar to `basic` with single-page 404 support
   - `album` showing pagination and external links
   - `profile` simple user profiles
   - `query-string` shows how you can integrate plugins using the router
   - `selective-binding` shows how you can bind to specific links only
   - `state` illustrates how the history state may be used to cache data

  __NOTE__: keep in mind these examples do not use jQuery or similar, so
  portions of the examples may be relatively verbose, though they're not
  directly related to page.js in any way.

## API

### page(path, callback[, callback ...])

  Defines a route mapping `path` to the given `callback(s)`.

```js
page('/', user.list)
page('/user/:id', user.load, user.show)
page('/user/:id/edit', user.load, user.edit)
page('*', notfound)
```

  Links that are not of the same origin are disregarded
  and will not be dispatched.

### page(path)

  Navigate to the given `path`.

```js
$('.view').click(function(e){
  page('/user/12')
  e.preventDefault()
})
```

### page.show(path)

  Identical to `page(path)` above.

### page([options])

  Register page's `popstate` / `click` bindings. If you're
  doing selective binding you'll like want to pass `{ click: false }`
  to specify this yourself. The following options are available:

    - `click` bind to click events [__true__]
    - `popstate` bind to popstate [__true__]

### page.start([options])

  Identical to `page([options])` above.

### page.stop()

  Unbind both the `popstate` and `click` handlers.

### Context#save()

  Saves the context using `replaceState()`. For example
  this is useful for caching HTML or other resources
  that were loaded for when a user presses "back".
  
  Routes are passed `Context` objects, these may
  be used to share state, for example `ctx.user =`,
  as well as the history "state" `ctx.state` that
  the `pushState` API provides.

## Routing

  The router uses the same string-to-regexp conversion
  that Express does, so things like ":id", ":id?", and "*" work
  as you might expect.

  Another aspect that is much like Express is the ability to
  pass multiple callbacks. You can use this to your advantage
  to flatten nested callbacks, or simply to abstract components.

### Separating concerns

  For example suppose you had a route to _edit_ users, and a
  route to _view_ users. In both cases you need to load the user.
  One way to achieve this is with several callbacks as shown here:

```js
page('/user/:user', load, show)
page('/user/:user/edit', load, edit)
```

  Using the `*` character we could alter this to match all
  routes prefixed with "/user" to achieve the same result:

```js
page('/user/*', load)
page('/user/:user', show)
page('/user/:user/edit', edit)
```

  Likewise `*` may be used as catch-alls after all routes
  acting as a 404 handler, before all routes, in-between and
  so on. For example:

```js
page('/user/:user', load, show)
page('*', function(){
  $('body').text('Not found!')
})
```

### Default 404 behaviour

  By default when a route is not matched,
  page.js will invoke `page.stop()` to unbind
  itself, and proceed with redirecting to the
  location requested. This means you may use
  page.js with a multi-page application _without_
  explicitly binding to certain links.

### Working with parameters and contexts

  Much like `request` and `response` objects are
  passed around in Express, page.js has a single
  "Context" object. Using the previous examples
  of `load` and `show` for a user, we can assign
  arbitrary properties to `ctx` to maintain state
  between callbacks.

  First to build a `load` function that will load
  the user for subsequent routes you'll need to
  access the ":id" passed. You can do this with
  `ctx.params.NAME` much like Express:

```js
function load(ctx, next){
  var id = ctx.params.id
}
```

  Then perform some kind of action against the server,
  assigning the user to `ctx.user` for other routes to
  utilize. `next()` is then invoked to pass control to
  the following matching route in sequence, if any.

```js
function load(ctx, next){
  var id = ctx.params.id
  $.getJSON('/user/' + id + '.json', function(user){
    ctx.user = user
    next()
  })
}
```

  The "show" function might look something like this,
  however you may render templates or do anything you
  want. Note that here `next()` is _not_ invoked, because
  this is considered the "end point", and no routes
  will be matched until another link is clicked or
  `page(path)` is called.

```js
function show(ctx){
  $('body')
    .empty()
    .append('<h1>' + ctx.user.name + '<h1>');
}
```

  Finally using them like so:

```js
page('/user/:id', load, show)
```
 