# Backbone.js + Express.js SPA boilerplate

To build single pages application is seconds, not hours.

NOTE: it still in **progress**. If you would like to contribute, please join the discussion/implementation of currently opened [issues](https://github.com/alexanderbeletsky/backbone-express-spa/issues).

## Contents

* [Description](#description)
* [Application example](#example)
* [Installation](#installation)
* [Express.js](#expressjs)
	* [Serving master page](#serving-master-page)
	* [API end-points](#api-endpoints)
	* [Authorization and CORS](#authorization-cors)
* [Backbone.js](#backbonejs)
	* [RequireJS and CommonJS](#requirejs-and-commonjs)
	* [Routing](#routing)
	* [View Manager](#view-manager)
	* [Applications](#applications)
	* [Main view and subviews](#main-view-and-subviews)
	* [Transitions](#transitions)
* [Testing](#testing)
	* [Backbone.js (front-end) tests](#front-end-tests)
	* [Express.js (back-end) tests](#back-end-tests)
	* [Functional (web driver) tests](#functional-tests)
* [SEO](#seo)
* [Build for production](#build-for-production)
	* [Concatenate and minify](#concatenate-and-minify)
	* [Gzip content](#gzip-content)
	* [Development and production](#gzip-content)
	* [Cache busting](#cache-busting)
	* [Optimization results](#optimization-results)
* [Deployment](#deployment)

<a name="description"/>
## Description

SPA infrastructure setup could be time consuming. It's typical problem, to configure `requirejs`, initial routing and view manager, to prevent memory leaks. This project could be used as good start to build own single page application.

This project is complete and minimal setup for building single page applications running on ``Express.js`` framework as back-end and ``Backbone.js`` as front-end.

SPA itself is rather simple concept, but it requires some infrastructure to have in place, before build up new application. This project already includes this infrastructure.

<a name="example"/>
## Application example

'TheMailer' - simple app for managing emails, contacts, tasks.

<a name="installation"/>
## Installation

Clone github repository,

```
$ git clone git@github.com:alexanderbeletsky/backbone-express-spa.git
```

Install npm dependencies,

```
$ npm install
```

Install bower dependencies,

```
$ bower install
```

Run app (development mode),

```
$ node app.js
```

<a name="expressjs"/>
## Express.js

[Express.js](http://expressjs.com/) is used as back-end development framework. It's simple and easy to configure for SPA.

In API-oriented architecture back-end is responsible for 2 main purposes:

* Serving master page html
* Providing API end-points for web client

### Master page

[Master page](views/master.ejs) is main (and typically one) html page returned from server. It includes all styles and javascript, provides very basic layout and placeholder for application.

```html
<div class="container">
	<div id="app" class="container"></div>
</div>
```

After master page is served back to client the rest of UI and logic is build by ``Backbone.js``.

<a name="serving-master-page"/>
### Serving master page

To serve master pages application includes middleware component [serveMaster.js](source/middleware/serveMaster.js). It would respond with master page html for any request, expect the requests for `/api`, `/components`, `/css/` or `/js`.

<a name="api-endpoints"/>
### API end-points

API is HTTP, JSON based end-points. Sources are located at ``source/api``. Each API module returns a function that takes ``app`` instance and setup HTTP verb handler for a route.

```js
module.exports = function (app) {
	app.get('/api/emails', function (req, res) {
		res.json({status: 'GET /api/users'});
	});

	app.post('/api/emails', function (req, res) {
		res.json({status: 'POST /api/users'});
	});

	app.put('/api/emails/:id', function (req, res) {
		res.json({status: 'PUT /api/users/' + req.params.id});
	});

	app.del('/api/emails/:id', function (req, res) {
		res.json({status: 'DELETE /api/users/' + req.params.id});
	});
};
```

To enable API end-point, you should modify ``app.js`` file, like

```js
// api endpoints
require('./source/api/emails')(app);
require('./source/api/contacts')(app);
require('./source/api/tasks')(app);
```

<a name="authorization-cors"/>
## Authorization and CORS

TODO.

<a name="backbonejs"/>
## Backbone.js

[Backbone.js](http://backbonejs.org/) is the one of most popular front-end development framework (library). It provides abstractions for models, views, collections and able to handle client-side routing.

Front-end architecture is build on modular structure and relying on [AMD](https://github.com/amdjs/amdjs-api/wiki/AMD) to allow build scalable applications.

<a name="requirejs-and-commonjs"/>
### RequireJS and CommonJS

[RequireJS](http://requirejs.org/) picked up as asynchronous javascript module loading. ``RequireJS`` uses it's own style for defining modules, specifying the dependency as array of strings.

```js
define([
	'/some/dep',
	'another/dep',
	'yet/another/dep',
	'text!./templates/template.html,
	jQuery,
	Backbone'], function(SomeDep, AnotherDep, YetAnotherDep, template, $, Backbone) {
		// module implementation...
	});
```

With some time spent on Node.js programming, CommonJS style becomes more convenient to use. Fortunately ``RequireJS`` has [CommonJS](http://requirejs.org/docs/commonjs.html) style implementation.

```js
define(function (require) {
	// dependencies
	var SomeDep = require('/some/dep');
	var AnotherDep = require('another/dep');

	// export
	return {};
});
```

<a name="routing"/>
### Routing

All routing logic is placed in [/core/router.js](public/js/core/router.js). There are 3 routes defined in boilerplate.

Each route handler is responsible for *starting up* new application. Application `run` function takes ``ViewManager`` instance.

<a name="view-manager"/>
### View manager

SPA application typical threat is *memory leaks*. Memory leaks might appear for a few reasons, one of the most famous reason for Backbone applications are, so called, [zombie views](http://lostechies.com/derickbailey/2011/09/15/zombies-run-managing-page-transitions-in-backbone-apps/).

[/core/viewManager.js](public/js/core/viewManager.js) is responsible for disposing views during switch from one router to another.

Besides of that, it handles *transitions* during application switch.

<a name="applications"/>
### Applications

Application is concept of grouping `models`, `collections`, `views` of unit in one place. The rule is, "one route - one application". Router matches the route, loading the application entry point and passes `viewManager` (or any other parameters, like id's or query strings) into application.

All applications are [apps](public/js/apps) folder.

[app.js](public/js/apps/home/app.js) is entry point of application and it's responsible for several things:

* Fetching initial application data
* Instantiating Main View of application

```js
define(function(require) {
	var MainView = require('./views/MainView');

	return {
		run: function (viewManager) {
			var view = new MainView();
			viewManager.show(view);
		}
	};
});
```

<a name="mainview-and-subviews"/>
### Main view and subviews

*Main view* responsible for UI of application. It's quite typically that main view is only instantiating subviews and passing the models/collections further down.

[MainView.js](public/js/apps/home/views/MainView.js) keeps track of subviews in ``this.subviews`` arrays. Each subview will be closed by `ViewManager` [dispose](public/js/core/viewManager.js#L22) function.

```js
var MainView = Backbone.View.extend({
	initialize: function () {
		this.subviews = [];
	},

	render: function () {
		var headerView = new HeaderView();
		this.subviews.push(headerView);
		this.$el.append(headerView.render().el);

		var footerView = new FooterView();
		this.subviews.push(footerView);
		this.$el.append(footerView.render().el);

		return this;
	}
});
```

<a name="templates"/>
### Templates

[Handlebars](http://handlebarsjs.com/) is picked up as templating engine, powered by [require-handlebars-plugin](https://github.com/SlexAxton/require-handlebars-plugin). Templates are stored on application level in `template` folder. Handlebars plugin is configured to keep templates in `.html` files.

View is loading template through `!hbs` plugin and uses that in `render()` function.

```js
var HeaderView = Backbone.View.extend({
	template: require('hbs!./../templates/HeaderView'),

	render: function () {
		this.$el.html(this.template({title: 'Backbone SPA boilerplate'}));
		return this;
	}
});
```

<a name="transitions"/>
## Transitions

Transitions is a very nice feature for single pages applications. It adds the visual effects of switching from one application to another.

Boilerplate is relying on wonderful [animate.css](https://github.com/daneden/animate.css) library. [core/transition.js](public/js/core/transition.js) is responsible for applying transition style. It's being called from [/core/viewManager.js](public/js/core/viewManager.js).

Once you decide to have transitions in your app, simply modify [master.ejs](views/master.ejs) and add ``data-transition`` attribute to application ``div``.

```html
<div class="container">
	<div id="app" class="container" data-transition="fadeOutLeft"></div>
</div>
```

Checkout the list of available transitions on [animate.css](https://github.com/daneden/animate.css) page. You can apply anything you want, please note "Out" transition type is suited the best.

<a name="testing"/>
## Testing

TODO.

<a name="front-end-tests"/>
### Backbone.js (front-end) tests

TODO.

<a name="back-end-tests"/>
### Express.js (back-end) tests

TODO.

<a name="functional-tests"/>
### Functional (web driver) tests

TODO.

<a name="seo"/>
## SEO

TODO.

<a name="build-for-production"/>
## Build for production

Modern web applications contain a lot of JavaScript/CSS files. While application is *loading* all that recourses have to be in-place, so browser issuing HTTP requests to fetch them. As more application grow, as more requests need to be done.. as slower initial loading is. There are two ways of *optimization* of initial application loading:

* concatenate and minify (decrease HTTP request)
* gzip content (decrease payload size)

Application could operate in several modes - development, production. In development mode, we don't care about optimizations at all. Even more, we are interested to get not processed source code, to be able to debug easily. In production mode, we have to apply as much effort as possible to decrease initial load time.

<a name="concatenate-and-minify"/>
### Concatenate and minify

[RequireJS](http://requirejs.org/) comes together with [optimization](http://requirejs.org/docs/optimization.html) tool, called `r.js`. It's able to concatenate and minify both JavaScript and CSS code.

To simplify the process, we'll use [GruntJS](http://gruntjs.com/) tasks runner. GruntJS is very handy tool, with great community around and very rich contrib library. There is a special task to handle `RequireJS` optimizations, called [grunt-contrib-requirejs](https://github.com/gruntjs/grunt-contrib-requirejs).

[Gruntfile.js](Gruntfile.js) contains all required configuration. To run grunt,

```
$ grunt
```

The result of the grunt run is new folder [/public/build](/public/build/) that contains 2 files: `main.css`, `main.js` - concatenated and minified JavaScript and CSS code.

<a name="gzip-content"/>
### Gzip content

Besides concatenation, it's important to compress resources. Express.js includes this functionality out of the box, as [compress()](http://expressjs.com/api.html#compress) middleware function.

<a name="development-and-production"/>
### Development and production

The configuration distinction goes in [app.js](app.js) file.

```js
app.configure('development', function(){
	app.use(express.errorHandler());							// apply error handler
	app.use(express.static(path.join(__dirname, 'public')));
	app.use(middleware.serveMaster.development());				// apply development mode master page
});

app.configure('production', function(){
	app.use(express.compress());								// apply gzip content
	app.use(express.static(path.join(__dirname, 'public'), { maxAge: oneMonth }));
	app.use(middleware.serveMaster.production());				// apply production mode master page
});
```

[serveMaster.js](source/middleware/serveMaster.js) middleware component is would serve different version of master page, for different mode. In development mode, it would use uncompressed JavaScript and CSS, in production mode, ones that placed in [/public/build](/public/build/) folder.

<a name="cache-busting" />
### Cache busting

Caching is in general good since it helps to application to be loaded faster, but it could hurt while you re-deploy application. Browsers do not track the actual content of file, so if the content has changed, but URL still the same, browser will ignore that.

Besides, different browsers have different caching strategies. IE for instance, is 'famous' with is aggressive caching.

Cache busting is widely adopted technique. There are some different implementations for that, but one of the most effective is: name your resources in the way, so if content has changed the name of resource would change as well. Basic implementation is to prefix file names with **hash** computed on file contents.

Boilerplate uses [grunt-hashres](https://github.com/Luismahou/grunt-hashres) task for that (currently I'm using my own [fork](https://github.com/alexanderbeletsky/grunt-hashres), hope that changes are promoted to main repo soon). That task transforms the [grunt-contrib-requirejs](https://github.com/gruntjs/grunt-contrib-requirejs) output files `main.js`, `main.css` into something like `main-23cbb34ffaabd22d887abdd67bfe5b2c.js`, `main-5a09ac388df506a82647f47e3ffd5187.css`.

It also produces [/source/client/index.js](/source/client/index.js) file [serveMaster.js](source/middleware/serveMaster.js) uses to render production master page correctly.

Now, everything that either `.js` or `.css` content is changed, build would produce new files and they are guaranteed to be loaded by browser again.

<a name="optimization-results"/>
### Optimization results

On a left side you see application running in development mode, on a right side in  production mode.

![optimization results](/public/img/optimizations.png?raw=true)

Even for such small application as 'TheMailer', the benefits are obvious:

* Requests: 55 / 4 ~ 14 times fewer.
* Payload: 756Kb / 43.4Kb ~ 17 times smaller.
* Load time: 898ms / 153ms ~ 6 times faster.

<a name="deployment"/>
## Deployment

TODO.


<a name="generator"/>
## Yeoman generator

TODO.

# Legal Info (MIT License)

Copyright (c) 2013 Alexander Beletsky

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
