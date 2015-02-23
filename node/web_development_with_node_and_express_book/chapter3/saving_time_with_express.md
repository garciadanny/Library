# Chapter 3

## `package.json`

`npm` manages project dependencies as well as meta data about the project in a
file called `package.json`. The easiest way to create this file is to run
`npm init`: It will ask you a series of questions and generate a `package.json`
file to get you started. For the "entry point" question use the name of your
project. i.e. `meadowlark.js`.

Every time you run npm you'll get warnings unless you provide a repository URL
in `package.json` and a nonempty README file.

Unless your hosting service or deployment system requires your main application
file to have a specific name (app.js, server.js, index.js), there's no reason
why you can't name if after the application (meadowlark.js).

## Routing

### `app.VERB`

`app.VERB` takes two parameters: a path and a function.

The path is what defines the routes and it doesn't care about trailing slahses
or cases, it also doesn't consider query strings when performing the match.

Express defaults to a status code of `200`; you don't have to specify it
explicitly.

### A minimal example

  app.get('/', function(req, res) {
    res.type('text/plain');
    res.send('Medowlark Travel');
  });

  app.get('/about', function(req, res) {
    res.type('text/plain');
    res.send('About Meadowlark Travel');
  });

  // custom 404 page
  app.use(function(req, res) {
    res.type('text/plain');
    res.status(404);
    res.send('404 - Not Found');
  });

  // custom 500 page
  app.use(function(err, req, res, next) {
    console.error(err.stack);
    res.type('text/plain');
    res.status(500);
    res.send('500 - Server Error');
  });

The custom `404` and `500` pages must be handled slightly differently. Instead of
using `app.get`, they're using `app.use` which is the method by which Express
adds middleware. In this case, think of this middleware as a catch all handler
for anything that didn't get matched by a route.

**In express, the order in which routes and middleware are added is significant.**

If we put the `404` handler above the routes, the home and about pages would stop
working and instead result in a `404`.

Though our routes are pretty simple, they currently support wildcards
( `/about/*` ) which can lead to problems with ordering. For example, if we
wanted to add sub pages to About, the following would not work:

  app.get('/about/*', function(req, res) {

  })

  app.get('/about/contact', function(req, res) {

  })

  app.get('/about/directions', function(req, res) {

  })

In this example, the contact and directions pages would never get matched
because the first handler uses a wildcard in it's path.

Express can distinguish between `404` and `500` handlers by the number of args
their callback functions take.

## Setting a view tempalte engine

  // set up handlebars view engine
  var handlebars = require('express-handlebars')
                  .create({ defaultLayout: 'main' });

  app.engine('handlebars', handlebars.engine);
  app.set('view engine', 'handlebars');

[`app.engine`](http://expressjs.com/4x/api.html#app.engine) registers the
template engine (`handlebars.engine`) as `'handlebars'`.

[`app.set`](http://expressjs.com/4x/api.html#app.set) sets the name
`'view engine'` to the registered template engine named `'handlebars'`.

## Static Files & Views

Express relies on middleware to handle static files and views. The `static`
middleware allows us to designate one or more directories as containing static
resources that will be delivered to the client without any special handling.
This is where you'd put things like images, CSS files, client-side js.

To designate a directory for static files, we add the following line before
declaring any routes:

  app.use(express.static(__dirname + '/public'));