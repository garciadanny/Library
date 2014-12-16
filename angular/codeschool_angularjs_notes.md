# Angular.JS

A client-side JS framework for adding interactivity to HTML.

Notes taken while going through [codeschool's angular.js course](https://www.codeschool.com/courses/shaping-up-with-angular-js)

DISCLAIMER: There's probably a ton of typos.


How do we tell our HTML when to trigger our JS?

## Directives

A directive is a marker on an HTML tag that tells Angular to run or reference some JS code. It's how we bind our HTML or behavior.

*Example:* 

index.html
	
	<!DOCTYPE html>
	<html>
		<body ng-controller="StoreController">
			.
			.
		</body>
	</html>
	
app.js

	function StoreController () {
		alert("Welcome to the controller");
	}
	
	
## Modules

Where we keep pieces of our angular app (where encapsulate our code). We also define the dependencies for this module in the module. 

*Example: Creating a store application*

Creating a module: 

	var app = angular.module('store', [DEPENDENCIES GO IN THIS ARRAY])

If no dependencies are defined, you must pass in an empty array. 

You'll need to add a `directive` to the `<html>` tag to tell angular to create an angular app by running the above `module` when the HTML document loads.

*Example:*

index.html

	
	<!DOCTYPE html>
	<html ng-app="store">
		<body ng-controller="StoreController">
			.
			.
			<script type="text/javascript" src="angular.min.js"></script>
			<script type="text/javascript" src="app.js"></script>
		</body>
	</html>
	
Now that our angular app is created, we can start writing `Expressions`.

## Expressions

`expressions` is how we insert dynamic values into our HTML.

*Example:*

	<p>                       			<p>
		I am {{ 4 + 6 }}  		--->		I am 10
	</p>								</p>
	
	<p>                       			<p>
		{{ "hello" + "you" }}  --->			hello you
	</p>								</p>
	
	
## Controllers

Controllers in angular is where we define our app's behavior by defining functions and values. 

*Example:*

*We're going to create a controller to add data to a page.*

We're first going to wrap our app in a closure, it's good practice to do so.
We're then going to create our controller, controller name should be CamelCase.

app.js

	(function() {
		var app = angular.module('store', [])
		
		app.controller('StoreController, function() {
			this.product = gem;
		});
		
		var gem = {
			name: 'Cool gem',
			price: 2.95,
			description: 'some description'
		}
	})();
	
	
In our HTML, we want to bind a `directive` to the element on the page where we want to append the data. The name of the `directive` is the name of the controler, `StoreController` and we want to alias is with something like `store` with `as` to have access to the controller in our `expressions`.

index.html

	<!DOCTYPE html>
	<html ng-app="store">
		<body ng-controller="StoreController as store">
			
			<h1> {{ store.product.name }} </h1>
			<h2>$ {{ store.product.price}} </h2>
			<p> {{ store.product.description}} </p>
			
			<script type="text/javascript" src="angular.min.js"></script>
			<script type="text/javascript" src="app.js"></script>
		</body>
	</html>
	
## A few of many useful Directives

### `ng-show`

Only display certain HTML or data if a value of expression is `true`.

*Example:*

If our gem had a value `available` that was set to `true` if it was available for purchase and set to `false` if soldout, we could add a `directive` to only display a "Buy" button if the gem was available for purchase.

	<button ng-show="store.product.available">Buy</button> 

gem object

	var gem = {
			name: 'Cool gem',
			price: 2.95,
			description: 'some description',
			available: true
	}

index.html

	<!DOCTYPE html>
	<html ng-app="store">
		<body ng-controller="StoreController as store">
			
			<h1> {{ store.product.name }} </h1>
			<h2>$ {{ store.product.price}} </h2>
			<p> {{ store.product.description}} </p>
			<button ng-show="store.product.available">Buy</button>
			<script type="text/javascript" src="angular.min.js"></script>
			<script type="text/javascript" src="app.js"></script>
		</body>
	</html>
	

### `ng-hide`

`ng-hide` is just like `ng-show` except it'll hide something on a page of the value in the expression is set to `true`. This would make sense if instead of having a `available` property in our `gem`, we had a `soldOut` property. 

*Example:*

		<button ng-hide="store.product.soldOut">Buy</button> 
		

### `ng-repeat`

Allows you to iterate over an `array`.

*Example:*

Let's say we had a list of gems instead of just one:

	var gem = [
		{
			name: 'Cool gem',
			price: 2.95,
			description: 'some description',
			available: true
		},
		{
		.
		.
		}
	]
	
index.html

	<!DOCTYPE html>
	<html ng-app="store">
		<body ng-controller="StoreController as store">
		
			<div ng-repeat="product in store.products">
				<h1> {{ store.product.name }} </h1>
				<h2>$ {{ store.product.price}} </h2>
				<p> {{ store.product.description}} </p>
				<button ng-show="store.product.available">Buy</button>
			<div>
			
			<script type="text/javascript" src="angular.min.js"></script>
			<script type="text/javascript" src="app.js"></script>
		</body>
	</html>


`product` is the single element within the `products` array.

	<div ng-repeat="product in store.products"></div>
	
### `ng-src`

`ng-src` allows us to load the contents in `src=""` before the HTML is rendered.

*Example:*

Let's say our gem object contained an `images` property that contained an `array` of images.

	var gem = {
			name: 'Cool gem',
			price: 2.95,
			description: 'some description',
			available: true,
			images: [
				{
					full: 'path_to_image.png'
				}
			]
	}
	
Simply using an `img` tag with `src` will not work because the broswer will try to load the image before the expression evaluates. 
	
	<img src="{{ product.images[0].full }}" />
	
The `ng-src` `directive` will ensure the `expression` is evaluated before the browser tries to load the image. 

		<img ng-src="{{ product.images[0].full }}" />
		
If we would have simply written `src={{product.images[0].full}}` the browswer would have attempted to make an http request to an invalid location i.e. `http://localhost:3000/app/{{product.images[0].full}}`. Which is before Angular has a chance to evaluate the expression and inject the valid address. 
	

## Formatting with Filters

Filters allow you to `pipe |` data into them and returns a "filtered" output.

`{{ data* | filter:options* }}`

If `product.price` returns `2`, we can pipe it to a currency filter that'll prepend the `currency` symbol (localized) with the proper decimal places.

*Example:*

	{{ product.price | currency }}
	=> $2.00
	
#### Other filters:

**Turning timestamps into dates**

	{{ '2233444555' | date:'MM/dd/yyyy @ h:mma' }}
	=> 12/27/2014 @ 12:50am
	
**Uppercase and Lowercase**

	{{ 'octagon gem' | uppercase }}
	=> OCTAGON GEM
	
**limitTo**

	{{ "My Description" | limitTo:8 }}
	=> My Descr
	
	<li ng-repeat="produc in store.products | limitTo:3">
	=> Only display the first 3 elements in an array
	
**orderBy**

	<li ng-repeat="product in store.products | orderBy:'-price'">
	=> descending order

With a minus sign, products will be displayed in descending order base on 	price. Highest pricest products first. Without a minus sign it'll display in ascending order. 

## 2-way data binding

An example using the `ng-click` `directive`.

Let's say we had tabs on our page and each time we click on a tab, it displays new content. 

*Example:*

	<section>
		<ul class="nav nav-pill">
			<li> <a href ng-click="tab = 1">Description</a> </li>
			<li> <a href ng-click="tab = 2">Specifications</a> </li>
			<li> <a href ng-click="tab = 3">Reviews</a> </li>
		</ul>
		{{ tab }}
	</section>
	
When you click on a different tab, the HTML will change to display the tab number. 

Here, we're utilizing 2-way data binding. Our `expression` `{{ tab }}` is automatically updated when the value of `tab` changes. So in our case, when the tab that's clicked changes, the value in the `expression` get's automatically changed.

**Displaying Content**

Simply showing the value of the current `tab` when a tab gets clicked isn't useful. To show specific content when a tab is clicked, we need to bind that content to the `tab`.

*Example:*

	<div class="panel" ng-show="tab === 1">
		<h4>Description</h4>
		<p> {{ product.description }} </p>
	</div>
	
	<div class="panel" ng-show="tab === 2">
		<h4>Specifications</h4>
		<blockquote>Some specifications</blockquote>
	</div>
	
	<div class="panel" ng-show="tab === 3">
		<h4>Reviews</h4>
		<blockquote>Some reviews</blockquote>
	</div>
	
Now, when one of the tabs in the first example gets clicked, it'll display the proper `panel`. 

**Displaying Default Content**

In our current example, if the page is refreshed, none of the `panel`s will show. If we want to default one of our `panel`s to display upon page refresh. We need to use the `ng-init` `directive`.

	<section ng-init="tab =1">
		<ul class="nav nav-pill">
			<li> <a href ng-click="tab = 1">Description</a> </li>
			<li> <a href ng-click="tab = 2">Specifications</a> </li>
			<li> <a href ng-click="tab = 3">Reviews</a> </li>
		</ul>
		{{ tab }}
	</section>
	
**Adding Styling Based On An Event**

An example using the `ng-class` `directive`.

Let's say that when we click on one of the tabs above, we'd like to add the Bootstrap `active` class to highlight the active tab. We can use the `ng-class` `directive` to add a class if a certain condition is `true`. 

	<li ng-class="{active:tab === 1 }"> 
		<a href ng-click="tab = 1">Description</a> 
	</li>
	
	<li ng-class="{active:tab === 2 }"> 
		<a href ng-click="tab = 2">Specifications</a> 
	</li>
	
	<li ng-class="{active:tab === 3 }"> 
		<a href ng-click="tab = 3">Reviews</a> 
	</li>
	
Here we're passing in an `Object` to the `ng-class` `directive` and saying: "Only add the `active` class if the tab that was clicked is equal to the tab we're specifying".

## Organizing Logic

In the *2-way data binding* example, we had too much logic in our HTML. As you can see, it got "dirty" pretty quickly. One way to pull this logic out and organize it is by creating a `controller` for it. 

**Creating A Controller For Our `Panel` Logic**

Here's our `panel` from our previous example. We've added a `PanelController` and aliased it as `panel`.

index.html

	<section ng-init="tab =1" ng-controller="PanelController as panel">
		<ul class="nav nav-pill">
			<li ng-class="{active:tab === 1 }"> 
				<a href ng-click="tab = 1">Description</a> 
			</li>
	
			<li ng-class="{active:tab === 2 }"> 
				<a href ng-click="tab = 2">Specifications</a> 
			</li>
	
			<li ng-class="{active:tab === 3 }"> 
				<a href ng-click="tab = 3">Reviews</a> 
			</li>
		</ul>
	</section>
	
app.js

	app.controller('PanelController', function() {
	
	});

**Step 1**

The first step should be to move our `ng-init` `directive` into our `controller` because things dealing with initialization and configuration make more sense to be handled by our controller.

index.js

		<section ng-controller="PanelController as panel">
			.
			.
			.
		</section>

app.js

	app.controller('PanelController', function() {
		this.tab = 1;
	});

**Step 2**

The next step would be to move the assignment logic in our `<a>` tag outside of our HTML and into our controller. Then, create a function in our controller to do the assignment. 

index.html

	<section ng-controller="PanelController as panel">
			.
			<a href ng-click="panel.selectTab(1)">Description</a> 
			<a href ng-click="panel.selectTab(2)">Description</a> 
			<a href ng-click="panel.selectTab(3)">Description</a> 
			.
		</section>
app.js

	app.controller('PanelController', function() {
		this.tab = 1;
		
		this.selectTab = function(setTab) {
			this.tab = setTab;
		};
	});
	
**Step3**

The last thing we'll do is create a function for our comparison in the `<li>` tags and move that logic into the controller. This function will be used within the `<li>` tags for each tab and the `<li>` tags for each `panel`.

index.html

	
	<section ng-controller="PanelController as panel">
		<ul class="nav nav-pill">
			<li ng-class="{active: panel.isSelected(1) }"> 
				<a href ng-click="panel.selectTab(1)">Description</a> 
			</li>
	
			<li ng-class="{active:panel.isSelected(2) }"> 
				<a href ng-click="panel.selectTab(2)">Description</a> 
			</li>
	
			<li ng-class="{active:panel.isSelected(3) }"> 
				<a href ng-click="panel.selectTab(3)">Description</a> 
			</li>
		</ul>
		
		<ul>
		.
		.
			<div class="panel" ng-show="panel.isSelected(3)">
				<h4>Reviews</h4>
				<blockquote>Some reviews</blockquote>
			</div>
		</ul>
	</section>
	
app.js

	app.controller('PanelController', function() {
		this.tab = 1;
		
		this.selectTab = function(setTab) {
			this.tab = setTab;
		};
		
		this.isSelected = function(checkTab) {
			return this.tab === checkTab;
		};
	});
	
## Forms and Models

This next example will show how to establish a 2-way data binding between content a user inputs on a form and an HTML element, creating a "live preview" of the submission. This requires a new `directive` called `ng-model`.

index.html

	<!-- The form -->
	
	<h4>Reviews</h4>

	<!-- Reviews AFTER they get submitted -->
	<blockquote ng-repeat="review in product.reviews">
		<b>Stars: {{ review.stars }}</b>
		{{ review.body }}
		<cite>by: {{ review.author }}</cite>
	</blockquote>
	
	<form name="reviewForm">
		<!-- Live preview of review before submission -->
		<blockquote>
			<b>Stars: {{ review.stars }}</b>
			{{ review.body }}
			<cite>- {{ review.author }}</cite>
		</blockquote>
	
		<select ng-model="review.stars">
			<option value="1">1 star</option>
			<option value="2">2 stars</option>
			...
		</select>
		<textarea ng-model="review.body"></textarea>
		<label>by:</label>
		<input type="email" ng-model="review.author" />
		<input type="submit" value="Submit" />
	</form>


As you may have noticed, we're not defining `review` anywhere in our form. We could use the `ng-init` `directive` in our HTML to define it but it'll make more sense to move this initialization inside of a controller. 

**Creating A Controller For Our Review Form**

app.js

	app.controller('ReviewController, function() {
		this.review = {};
	});


index.html

	<form name="reviewForm" ng-controller="ReviewController as reviewCTRL">
		<!-- Live preview of review before submission -->
		<blockquote>
			<b>Stars: {{ reviewCTRL.review.stars }}</b>
			{{ reviewCTRL.review.body }}
			<cite>- {{ reviewCTRL.review.author }}</cite>
		</blockquote>
	
		<select ng-model="reviewCTRL.review.stars">
			<option value="1">1 star</option>
			<option value="2">2 stars</option>
			...
		</select>
		<textarea ng-model="reviewCTRL.review.body"></textarea>
		<label>by:</label>
		<input type="email" ng-model="reviewCTRL.review.author" />
		<input type="submit" value="Submit" />
	</form>



**Adding Functionality To Our Form Submit**

Using the `ng-submit` `directive` which allows us to call a function when the form is submitted. 

In our `controller` we're going to create a `addReview` function that takes in a `product` and adds a review to it. Then, in our form we're going to use the `ng-submit` `directive` to add the function and pass in the product. 

app.js

	app.controller('ReviewController, function() {
		this.review = {};
		
		this.addReview = function(product) {
			product.reviews.push(this.review);
			// set the review to an empty object to clear
			// out the form and the live preview in our HTML.
			this.review = {};
		};
	});
	
	
index.html

	<form name="reviewForm" ng-controller="ReviewController as reviewCTRL" ng-submit="reviewCTRL.addReview(product)">
		<!-- Live preview of review before submission -->
		<blockquote>
			<b>Stars: {{ reviewCTRL.review.stars }}</b>
			{{ reviewCTRL.review.body }}
			<cite>- {{ reviewCTRL.review.author }}</cite>
		</blockquote>
	
		<select ng-model="reviewCTRL.review.stars">
			<option value="1">1 star</option>
			<option value="2">2 stars</option>
			...
		</select>
		<textarea ng-model="reviewCTRL.review.body"></textarea>
		<label>by:</label>
		<input type="email" ng-model="reviewCTRL.review.author" />
		<input type="submit" value="Submit" />
	</form>


## Form Validations

Using `required` to specify which form fields are required. 


Before using Angular's built-in form validations we want to turn off the browser's default validations.

	<form novalidate>
		<select required></select>
		...
		<input required />
	</form>
	
The way the form is currently set up, we're still able to submit the form even if one or all of the form fields is invalid. 

Angular provides a `$valid` property on the form's `name` that returns whether the form is valid or not. With this knowledge we can ensure that our `ng-submit` `directive` only submits the form if `$valid` returns `true`.
	
	
	<form name="reviewForm" ng-submit="reviewForm.$valid && reviewCTRL.addReview(product)">

With `reviewForm.$valid &&` in our `ng-submit`, a new review will only get added if the `form` is valid. In other words, the second part of the expression: `reviewCTRL.addReview(product)` will only get evaluated if the form is valid.

**Informing The User Of An Invalid Form Field**

Angular, by default, automatically adds CSS classes to form fields for validations. 

*Example:*

In the input field below, 2 CSS classes are automatically added. They are `ng-pristine` and `ng-invalid`.

	<input name="author" type="email" class="ng-pristine ng-invalid" ng-model="reviewCTRL.review.author" required />
	
`ng-pristine` means the form hasn't been edited yet.

`ng-invalid` means the form is invalid. This makes since because if it hasn't been edited then the form fields don't yet contain valid data. 

Once a user starts typing, Angular automatically adds a `ng-dirty` CSS class. Once the field contains valid data, Angular automatically adds a `ng-valid` CSS class. 

We can use these classes to display a message on the page to inform the user of invalid data or to highlight a form field with invalid data. 

styles.css

	.ng-invalid.ng-dirty {
		border-color: red;
	}
	
	.ng-valid.ng-dirty {
		border-color: green;
	}
	

**Other Validations**

Angular can validate againse emails, urls, and numbers. With numbers, you can even specify a min and max value. 


## Custom Directives

There are different types of custom directives one could build with angular. These include directives that can: 

* Expand templates
	* These are great when you have duplicate HTML code. You can pull out this code into a separate HTML file and include it wherever you need it throughout your application. Think of them as partials in Rails. These directives can also include controller logic. They way you include them in your project is by defining custom HTML tags. `<product-info></product-info>`.
	
* Help with expressing complex UIs.

* Call events and registering event handlers.

* Reusing common components.

Now for the case of DRYing up your code by moving duplicate HTML code into a separate file, one could use the `ng-include` directive:

	<h3 ng-include=" 'product-title.hmtl' "></h3>
	
The problem with this is that because the browser is only able to pull one HTML file at a time `index.html`, angular then has to make a separate `ajax` request to get the contents of `product-title.html`. This is where custom directive become useful. 

**Creating A Custom Directive**

We're going to make a template-expanding directive to pull out duplicate HTML code. 

This is what our directive will look like: 

	<product-title></product-title>

First, we'll create a directive that returns a directive definition object which is a configuration that defines how our directive will work. 

app.js

	app.directive('productTitle', function() {
		return {
		
		}
	});
	
We need to give our directive two important configuration options. 

1) `restrict: 'E'`

2) `templateUrl: 'product-title.html'`

`restrict: 'E'` specifies the type of directive. In this case it is an HTML `E`lement. `templateURL` simply specifies the HTML document. 
	
app.js

	app.directive('productTitle', function() {
		return {
			restrict: 'E',
			templateUrl: 'product-title.html'
		}
	});
	
We can also create what are called Attribute Directives which look like this: 

index.html

	<h3 product-title></h3>

app.js	

	app.directive('productTitle', function() {
		return {
			restrict: 'A', // A for attribute directive
			templateUrl: 'product-title.html'
		}
	});
	
However, Element directives are typically used for UI widgets like the first example and Attribute directives are used for mixin behaviors such as adding a tooltip.


## Directive Controllers

Creating a `directive` that uses a controller. 

In this example, we're going to encapsulate our tabs functionality from one of our previous examples into a `directive`. Here's the HTML as a refresher. 

index.html

	
	<section ng-controller="PanelController as panel">
		<ul class="nav nav-pills">
			<li ng-class="{active: panel.isSelected(1) }"> 
				<a href ng-click="panel.selectTab(1)">Description</a> 
			</li>
	
			<li ng-class="{active:panel.isSelected(2) }"> 
				<a href ng-click="panel.selectTab(2)">Description</a> 
			</li>
	
			<li ng-class="{active:panel.isSelected(3) }"> 
				<a href ng-click="panel.selectTab(3)">Description</a> 
			</li>
		</ul>
		
		<ul>
		.
		.
			<div class="panel" ng-show="panel.isSelected(3)">
				<h4>Reviews</h4>
				<blockquote>Some reviews</blockquote>
			</div>
		</ul>
	</section>
	
First thing we must do is move this HTML from `index.html` to `product-panels.html`. 

app.js

	app.directive('productPanels',function() {
		return {
			restrict: 'E',
			templateUrl: 'product-panels.html',
			controller: function() {
				...
			},
			controllerAs: 'panels'
		};
	});
	
Now we can include this new `element directive` in our HTML.

index.html

	...
	<product-panels>
	</product-panels>
	
## Dependencies

As you might have noticed from all of our examples, our `app.js` is getting pretty cluttered. Directives and controllers that are concerned with a certain piece of functionality our commingling with others. The follwing is an example of how we can break apart this code into logical chunks. 

Let's break out everything that has to do with products into it's own module. Let's call this new module `store-products`.

app.js
	
	(function(){
		var app = angular.module('store', []);
		
		app.directive('productTitle', function() {...});
		app.directive('productGallery', function() {...});
		app.directive('productPanels', function() {...});
	})();

product.js

	(function(){
		var app = angular.module('store-products', []);
		
		app.directive('productTitle', function() {...});
		app.directive('productGallery', function() {...});
		app.directive('productPanels', function() {...});
	})();
	
Now, we need to tell our `store` module that it depends on the `store-products` module. 

app.js

	(function(){
		var app = angular.module('store', ['store-products']);
		.
		.
		.
	})();

Lastly, we need to include this `products.js` file in our HTML.

index.html

	
	<!DOCTYPE html>
	<html ng-app="store">
		<body ng-controller="StoreController">
			.
			.
			<script type="text/javascript" src="angular.min.js"></script>
			<script type="text/javascript" src="app.js"></script>
			<script type="text/javascript" src="products.js"></script>
		</body>
	</html>
	
	
## Services

Angular provides built in services to give your controllers additional functionality. Services begin with a `$`. Here are some examples of services built into angular: 

* `$http` - Allows you to fetch JSON from a web service.
* `$log` - Allows you to log messages to the JS console.
* `$filter` - Allows you to filter arrays. 

*Example:*

	$http.get('/products.json', {apiKey: 'myKey'})
	
This returns a `promise` object with `.success()` and `.error()`

We need our controllers to tell angular which services they need. We do this via dependency injection, by supplying the name of the service we need and passing that service as an argument to our controller's function. 

*Example:*

app.js

	app.controller('SomeController', ['$http', '$log', function($http, $log) {
	
	} ]);

When Angular loads, it builds an `injector` object. When the built in services load, they register with the injector as available libraries. When our app loads, it registers our controller with the injector. When our controller get's used, the injector passes the servies over to the controller as arguments. 

*Example:*

Using the `$http` service. 

app.js

	app.controller('StoreController', ['$http', function($http) {
		var store = this; // save the current state of `this`
		
		$http.get('/products.json).success(function(data) {
			store.products = data;
		});
	} ]);
	
	
Because this is an asyncronous call, our page is going to load even if we havne't yet recieved a response from the api. It's there for a good idea to initialize `products` to an empty array so when the page loads, there won't be any errors. 

*Example:*

app.js

	app.controller('StoreController', ['$http', function($http) {
		var store = this; // save the current state of `this`
		
		store.products = []
		$http.get('/products.json).success(function(data) {
			store.products = data;
		});
	} ]);
	

# Future Learning Resources

* Angular docs
	* 	https://docs.angularjs.org
	
* https://egghead.io
* thinkster.io
* Codeschool: Soup to bits shaping up with angular