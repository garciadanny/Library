# Angular Tutorial

Notes taken while going through the [angular.js tutorial](https://docs.angularjs.org/tutorial)

DISCLAIMER: There's probably a ton of typos.

## View and Templates

In Angular, the view *is a **projection** of the model* through the HTML template. This means that whenever the model changes, Angular refreshes the appropriate bindings, which updates the view. 

## How Uses The MVC Pattern

*Example:*

index.html

	<!doctype html>
	<html lang="en" ng-app="phonecatApp">
	<head>
	  ...
	</head>
	<body ng-controller="PhoneListCtrl">
	
	  <ul>
	    <li ng-repeat="phone in phones">
	      {{phone.name}}
	      <p> {{phone.snippet}} </p>
	    </li>
	  </ul>
	
	</body>
	</html>

The expressions in the curly braces `{{phone.name}}` denotes bindings which are referring to our `application model`, which is set up in our `PhoneListCtrl`. 

We have specified an `angular module` to load using `ng-app="phonecatApp"`. This module will contain the `PhoneListCtrl`. 


controllers.js

	var phonecatApp = angular.module('phonecatApp', []);

	phonecatApp.controller('PhoneListCtrl', function ($scope) {
	    $scope.phones = [
	        {
	            'name': 'Nexus S',
	            'snippet': 'Fast just got faster with Nexus S.'
	        },
	        {
	            'name': 'Motorola XOOM™ with Wi-Fi',
	            'snippet': 'The Next, Next Generation tablet.'
	        },
	        {
	            'name': 'MOTOROLA XOOM™',
	            'snippet': 'The Next, Next Generation tablet.'
	        }
	    ]
	});
	
The data `model` (a simple array of phones in object literal notation) is now instantiated within the `PhoneListCtrl` controller. The `controller` is simply a constructor function that takes a `$scope` parameter.

The `PhoneListCtrl` is now registered with the `phonecatApp` module. 

Altough the `PhoneListCtrl` isn't doing very much, it plays an important role. It provides **context** for our data model. The `PhoneListCtrl` allows us to establish data-binding between the `model` and the `view`.

The `PhoneListCtrl` controller attaches the phone data to the `$scope` that was injected into our controller function. This `scope` is a prototypical descendant of the root scope that was created when the application was defined. This controller scope is available to all bindings located within the `<body ng-controller="PhoneListCtrl">` tag.

The concept of a `scope` in Angular is crucial. A `scope` can be seen as the glue which allows the `template`, `model` and `controller` to work together. Angular uses scopes, along with the information contained in the template, data model, and controller, to keep models and views separate, but in sync. Any changes made to the model are reflected in the view; any changes that occur in the view are reflected in the model.

![angular](images/template_model_view.png "Templat Model View")


## A Simple Testing Example

test/unit/controllersSpec.js

	describe('Phonecat controllers', function () {
	  describe('PhoneListCtrl', function() {
	    beforeEach( module('phonecatApp'));
	
	    it("should create 'phones' model with 3 phones", inject(function ($controller) {
	      var scope = {},
	          ctrl = $controller('PhoneListCtrl', {$scope: scope});
	
	      expect(scope.phones.length).toBe(3);
	    }));
	  });
	});
	
1) Before each test, we're telling angular to load the `phonecatApp` module.

2) We tell angular to [`inject`](https://docs.angularjs.org/api/auto/service/$injector) the `$controller` **service** into our test function. 

3) We create a **mock** `$scope` object. 

4) We use the `$controller` service to instantiate the `PhoneListCtrl` with the mock `$scope` object.


## Adding A Model Property And Binding To It From The Template

controllers.js

	var phonecatApp = angular.module('phonecatApp', []);

	phonecatApp.controller('PhoneListCtrl', function ($scope) {
	    $scope.phones = [
	        ...
	    ]
	    $scope.name = "World";
	});
	
index.html

	<p>Hello, {{name}}!</p> ====> "Hello, World!"
	
## Filtering Repeaters

In this example we'll use angular's `filter` function along with the `ng-repeat` directive to allow a user to enter search criteria and immediately see the effects of their search on the phone list. This demonstrates how cool data binding is. 

indext.html

	<!doctype html>
	<html lang="en" ng-app="phonecatApp">
	<head>
	  ...
	</head>
	
	<body ng-controller="PhoneListCtrl">
	    
	      <div class="col-md-2">
	        <!--Sidebar-->
	        Search: <input ng-model="query">
	      </div>
	    
	      <div class="col-md-10">
	        <!--Body content-->
	        <ul class="phones">
	          <li ng-repeat="phone in phones | filter:query">
	            {{phone.name}}
	            <p> {{phone.snippet}} </p>
	          </li>
	        </ul>
	      </div>
	    
	</body>
	</html>

When the page loads, angular binds the name of the input box to a variable with the same name in the data model and keeps the two in sync. 

The data that a user types into the input box named `query` is immediately available as a `filter` input in the list repeater. `phone in phones | filter: query`. 

When changes to the data model cause the repeater's input to change, the repeater updates the DOM to reflect the current state of the model. 

The `filter` function uses the `query` value to create a new array that contains only those records that match the query. 

`ng-repeat` automatically updates the view in respnose to the changing number of phones returned by the `filter`.

![angular](images/filter.png "Filter")

## Writing End To End Tests
In this example we'll right an end to end test for our search functionality using protractor. 

test/e2e/scenarios.js

	describe('PhoneCat App', function() {
	  describe('Phone list view', function() {
	    beforeEach(function() {
	      browser.get('app/index.html');
	    });
	
	    it('should filter the phone list as user types', function() {
	      var phoneList = element.all(by.repeater('phone in phones')),
	          query = element(by.model('query'));
	
	      expect( phoneList.count() ).toBe(3);
	
	      query.sendKeys('nexus');
	      expect( phoneList.count()).toBe(1);
	
	      query.clear();
	      query.sendKeys('motorola');
	      expect( phoneList.count()).toBe(2);
	    });
	  });
	});
	
Here `element` wraps a raw DOM element or HTML string as a jQuery element and we can specify that we'd like to search for an element `by` a particular `directive`.

## A Quck Note On Binding

Sometimes when using `{{ }}`, the curly braces are displayed on the page while the page is loading. A better solution is to use the `ng-bind` or `ng-bind-template` directives which ar invisible to the user while the page loads. 

*From:*

	<title>Google Phone Gallery: {{ query }}</title>

*To:*

	<title ng-bind-template="Google Phone Gallery: {{query}}">Google Phone Gallery</title>
	
## Another Filtering Example
In our previous filtering example we saw how we could filter our data based on a query. We can append additional filters such as an `orderBy` filter that'll allow a user to select from a drop-down menu and filter the data alphabetically or from newest to oldest. 

If you wanted to sort from oldest to newest, just add a `-`.

	<option value="-age">Oldest</option>

index.html

	<div class="col-md-2">
        <!--Sidebar-->
        Search: <input ng-model="query">

        Sort by:
        <select ng-model="orderProp">
          <option value="name">Alphabetical</option>
          <option value="age">Newest</option>
        </select>
    </div>
      
    <div class="col-md-10">
        <!--Body content-->
        <ul class="phones">
          <li ng-repeat="phone in phones | filter:query | orderBy:orderProp">
            {{phone.name}}
            <p> {{phone.snippet}} </p>
          </li>
        </ul>
	</div>
	
Here, we're chaining the `orderBy` filter to the filter to further process the input into the repeater. This means the user can first search for "motorola", then add another filter to sort them alphabetically or in descending order.

Angular creates a two way data-binding between the `select` element and the `orderProp` model. `orderProp` is then used as the input for the `orderBy` filter.

controllers.js

	var phonecatApp = angular.module('phonecatApp', []);

	phonecatApp.controller('PhoneListCtrl', function ($scope) {
	  $scope.phones = [
	    {
	      'name': 'Nexus S',
	      'snippet': 'Fast just got faster with Nexus S.',
	      'age': 2
	    },
	    ...
	  ]
	  $scope.orderProp = 'age';
	});
	
An important thing to note about the code above is that we're setting a default value for `orderProp`. This is technically not necessary but a nice to have. In this example, the binding works in the direction from our model to the UI. Now if you select "Alphabetically" in the drop down menu, the model will be updated as well and the phones will be reordered. That is the data-binding doing its job in the opposite direction — from the UI to the model.

#### Unit Test

controllerSpec.js

	describe('Phonecat controllers', function () {
	  describe('PhoneListCtrl', function() {
	    var scope, ctrl;
	
	    beforeEach( module('phonecatApp'));
	
	    beforeEach( inject(function ($controller) {
	      scope = {};
	      ctrl = $controller('PhoneListCtrl', {$scope: scope});
	    }));
	
	    it("should create 'phones' model with 3 phones", inject(function () {
	      expect(scope.phones.length).toBe(3);
	    }));
	
	    it("should set the default value of orderProp model", inject(function () {
	      expect(scope.orderProp).toBe('age');
	    }));
	  });
	});
	
Here we're writing a unit test to ensure a default value is properly set for our `orderProp` model. We've also extracted the controller construction into a `beforeEach` block.

#### End to End Test

scenarios.js 

	it('should be able to control phone order via drop down box', function() {
      var phoneNameColumn = element.all(by.repeater('phone in phones').column('{{phone.name}}'));

      function getNames() {
        return phoneNameColumn.map( function(elm) {
          return elm.getText();
        });
      }

      query.sendKeys('tablet');
      expect(getNames()).toEqual([
        "Motorola XOOM\u2122 with Wi-Fi",
        "MOTOROLA XOOM\u2122"
      ]);

      element(by.model('orderProp')).element(by.css('option[value="name"]')).click();
      expect(getNames()).toEqual([
        "MOTOROLA XOOM\u2122",
        "Motorola XOOM\u2122 with Wi-Fi"

      ]);
    });
    
## Services
In this example we're going to use Angular's `$http` service to make a `GET` request to our web server to fetch the data in `app/phones/phones.json`. To the browser and our app, making a request to fetch the data in this json file looks exactly like if we were hitting an actual API endpoint. 

controllers.js

	var phonecatApp = angular.module('phonecatApp', []);

	phonecatApp.controller('PhoneListCtrl', function ($scope, $http) {
	  $http.get('phones/phones.json').success( function(data) {
	    $scope.phones = data;
	  });
	
	  $scope.orderProp = 'age';
	});
	
`$http` returns a [`promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) object with a `success` method. We call this method to handle the async response. 

the `$` prefix is there to namespace angular provided services. It's best to avoid naming your own custom services or models with a `$` prefix. Properties in angular that are prefixed with `$$` are private. 

### Side Note

If you're planning on minifying your js code, it's important to note that angular's injector will not be able to read the dependencies of the controller unless you annotate the function with the names of the dependencies as strings which will not get minified. 

*Example:*

controllers.js

	phonecatApp.controller('PhoneListCtrl', ['$scope', '$http', function ($scope, $http) {
	  $http.get('phones/phones.json').success( function(data) {
	    $scope.phones = data;
	  });
	
	  $scope.orderProp = 'age';
	}]);
	
### Updating our tests

Angular provides a mock `$http` service that we can use in our unit tests. We configure fake responses to server requests by calling methods on a service called `$httpBackend`.

#### Unit Test

controllersSpec.js

	describe('Phonecat controllers', function () {
	  describe('PhoneListCtrl', function() {
	    var scope, ctrl, $httpBackend;
	
	    beforeEach( module('phonecatApp'));
	
	    beforeEach( inject(function (_$httpBackend_, $rootScope, $controller) {
	      $httpBackend = _$httpBackend_;
	      $httpBackend.expectGET('phones/phones.json').respond([
	        {name: 'Nexus S'},
	        {name: 'Motorola DROID'}
	      ]);
	      scope = $rootScope.$new();
	      ctrl = $controller('PhoneListCtrl', {$scope: scope});
	    }));
	
	    it("should create 'phones' model with 2 phones fetched from xhr", inject(function () {
	      expect(scope.phones).toBeUndefined();
	      $httpBackend.flush();
	
	      expect(scope.phones).toEqual([
	        {name: 'Nexus S'},
	        {name: 'Motorola DROID'}
	      ]);
	    }));
	
	    it("should set the default value of orderProp model", inject(function () {
	      expect(scope.orderProp).toBe('age');
	    }));
	  });
	});
	
1. In the second `beforeEach` block, we inject the [`$httpBackend`](https://docs.angularjs.org/api/ngMock/service/$httpBackend) mock service provided by `ng-mock` in our annonymous function. This service is to write unit tests for apps that use the `$http` service. 

	* The `injector` ignores the leading/trailing underscores. 
	* The reason we added leading/trailing underscores to `$httpBackend` is so that we could attach it to a variable with the same name while avoiding name conflicts.
	

2. We tell the testing harness to expect an incoming request from the controller with `expectGET` and tell it what to respond with by using `respond()`.
	* The responses aren't returned until we call `$httpBackend.flush()`. What this is doing is flushing the request queue in the browser. This causes the promise returned by the `$http` service to be resolved with the "trained" response. 
	
3. We created a new `scope` for our controller by calling `$rootScope.$new()`.

## Routing, Templates, Multiple Views

Up to this point we only have one view that lists all of the phones. Now we will add another view to show detailed information about each phone. We could expand `index.html` to contain template code for both the current view and the detailed phone view but that will get messy quickly. Instead we'll use `index.html` into a **layout template** that is common for all the views in our app. Other **partial templates** will be included in this layout depending on the current **route** (the current view). 

The `$route` service is used in conjunction with the `ng-view` directive. The `ng-view` directive includes the view template for the current route into the layout template. 

*Example:*

index.html

	<html>
		<head>
		...
		<script src="bower_components/angular-route/angular-route.js"></script>
		<script src="js/app.js"></script>
		</head>
		<body>
			<div ng-view></div>
		</body>
	</html>

We've included the `ng-route` module and `app.js` which will hold the root module of our app. We've also removed the template code for our phone list and replaced it with a div containing the `ng-view` directive. We'll create a partial for a phone list template. 


### Defining our routes

app.js

	var phonecatApp = angular.module('phonecatApp', ['ngRoute', 'phonecatControllers']);

	phonecatApp.config(['$routeProvider', function($routeProvider) {
	 $routeProvider.
	     when('/phones', {
	       templateUrl: 'partials/phone-list.html',
	       controller: 'PhoneListCtrl'
	     }).
	     when('/phones/:phoneId', {
	       templateUrl: 'partials/phone-detail.html',
	       controller: 'PhoneDetailCtrl'
	     }).
	     otherwise({
	       redirectTo: '/phones'
	     });
	}]);

1) We're defining our `phonecatApp` by creating a module and defining our dependencies `ngRoute` and `phonecatControllers`

2) Then we're configuring our routes for the app. 

Application routes are declared via the `$routeProvider` which is the provider (creator) of the `$route` service.

Providers are objects that provide (create) instances of services and expose configuration APIs that can be used to control the creation and runtime behavior of a service. In case of the `$route` service, the `$routeProvider` exposes APIs that allow you to define routes for your application such as, `when()` and `otherwise()`, as seen above. 

In our routes, all variables defined with the `:` notation are extracted into the `$routeParams` object which can be used in controllers. 

### Defining our controllers

controllers.js

	var phonecatControllers = angular.module('phonecatControllers', []);

	phonecatControllers.controller('PhoneListCtrl', ['$scope', '$http',
	  function ($scope, $http) {
	    $http.get('phones/phones.json').success( function(data) {
	      $scope.phones = data;
	    });
	
	    $scope.orderProp = 'age';
	  }
	]);
	
	phonecatControllers.controller('PhoneDetailCtrl', ['$scope', '$routeParams',
	  function($scope, $routeParams) {
	    $scope.phoneId = $routeParams.phoneId;
	  }
	]);

1) We've defined our `phonecatControllers` by creating a module. 

2) We've created a `PhoneDetailCtrl` and injected the `$routeParams` object defined by our routes. 


## Custom Filters

1) Create filter

app/js/filter.js

	angular.module('phonecatFilters', []).filter('checkmark',
	    function() {
	      return function(bool) {
	        return bool ? '\u2713' : '\u2718'
	      };
	    }
	); 

2) Register filter with app

app/js/app.js

	var phonecatApp = angular.module('phonecatApp', 
	  [
	    'ngRoute', 
	    'phonecatControllers',
	    'phonecatFilters'
	  ]
	);
	
3) Include filter in index.html

	<head>
	 ...
	 <script src="js/filters.js"></script>
	</head>

	
4) Use filter in phone-detail template

partials/phone-detail.html

	<dt>Infrared</dt>
    <dd>{{phone.connectivity.infrared | checkmark }}</dd>
    <dt>GPS</dt>
    <dd>{{phone.connectivity.gps | checkmark }}</dd>

### Testing custom filters

test/unit/filtersSpec.js

	describe('filter', function() {
	  beforeEach(module('phonecatFilters'));
	
	  describe('checkmark', function() {
	    it('should convert bool values to unicode', inject(
	        function(checkmarkFilter) {
	          expect(checkmarkFilter(true)).toBe('\u2713');
	          expect(checkmarkFilter(false)).toBe('\u2718');
	        }
	    ));
	  });
	});
	
1) Load `phonecatFilters` module into the injector before running tests. 

2) Use `inject` helper function to access filter we want to test. 

3) You need to suffix the filter name with `Filter`. i.e. `checkmarkFilter`. If not, you'll get an error message in your tests. For example you if name it `checkmark` instead, the error messsage will read: 

	Unknown provider: checkmarkProvider <- checkmark
	
This is because internally, every custom filter uses the `filterProvider`.

#### Side Note: 
You can use filters with user input by simply createing a model for the user data. 

*Example:*: 

	<input ng-model="userInput"> Uppercased: {{ userInput | uppercase }}
	
	
## Event Handlers

We'll now add functionality that'll allow a user to click on a thumbnail image that'll replace the larger main image.

controllers.js

	phonecatControllers.controller('PhoneDetailCtrl', ['$scope', '$http', '$routeParams',
	  function($scope, $http, $routeParams) {
	    $http.get('phones/' + $routeParams.phoneId + '.json').success(
	        function(data) {
	          $scope.phone = data;
	          $scope.mainImageUrl = data.images[0];
	        }
	    );
	    $scope.setImage = function(imageUrl) {
	      $scope.mainImageUrl = imageurl;
	    }
	  }
	]);
	
1) We created the `mainImageUrl` *model property* that sets the default image to the first one in the array. 

2) We then created the `setImage` event handler that'll change the value of `mainImageUrl`.

app/partials/phone-detail.html

	<img ng-src="{{mainImageUrl}}" class="phone">

	...
	
	<ul class="phone-thumbs">
	  <li ng-repeat="img in phone.images">
	    <img ng-src="{{img}}" ng-click="setImage(img)">
	  </li>
	</ul>
	...
	
1) We bound the `ng-src` directive of the larger image to `mainImageUrl`. 

2) We use the `ng-click` directive for each thumbnail image so that when a user clicks on a thumbnail, we use the `setImage` event handler to change the value of `mainImageUrl`.


## REST & Custom Services

We'll now define a custom service that represents a RESTful client. This will allow us to make http server requests in an easier way without having to use the low lever `$http` API and methods. 

1) We first need to install the `ng-resource` module by including the dependency in our bower.json file and running `npm install` or `bower install`. 

2) Our custom resource service will be defined `/js/services.js` so we need to include this file as well as the `angular-resource.js` file in our layout template. 

index.html

	<head>
		<script src="bower_components/angular-resource/angular-resource.js"></script>
		<script src="js/services.js"></script>
	</head>
	
3) Create custom service

app/js/services.js

	var phonecatServices = angular.module('phonecatServices', ['ngResource']);

	phonecatServices.factory('Phone', ['$resource', 
	  function($resource){
	    return $resource('phones/:phoneId.json', {}, {
	      query: {
	        method: 'GET', 
	        params: {phoneId: 'phones'},
	        isArray: true
	      }
	    });
	  }
	]);
	
Here, we used the `module` API to register a custom service using a `factory` function. Then we pass in the name of the service `Phone` in the factory function. The `factory` function is similar to a controller's `controller` contructor in that both declare dependencies to be injected via function arguments. 

4) Include `phonecatServices` dependency in the `phonecatApp` module. 

app.js
	
	var phonecatApp = angular.module('phonecatApp',
	  [
	    'ngRoute',
	    'phonecatControllers',
	    'phonecatFilters',
	    'phonecatServices'
	  ]
	);
	
5) Replace usage of `$http` in our controllers with our `Phone` service. Angular's `$resource` service is easier to use than `$http` for interacting with data sources exposed as RESTful resources. 

controllers.js

	var phonecatControllers = angular.module('phonecatControllers', []);

	phonecatControllers.controller('PhoneListCtrl', ['$scope', '$http',
	  function ($scope, $http) {
	    $scope.phones = Phone.query();
	    $scope.orderProp = 'age';
	  }
	]);
	
	phonecatControllers.controller('PhoneDetailCtrl', ['$scope', '$http', '$routeParams',
	  function($scope, $http, $routeParams) {
	
	    $scope.phone = Phone.get({phoneId: $routeParams.phoneId},
	        function(phone) {
	          $scope.mainImageUrl = phone.images[0];
	        }
	    );
	    $scope.setImage = function(imageUrl) {
	      $scope.mainImageUrl = imageUrl;
	    }
	  }
	]);
	
If you notice at the code we added in the `PhoneListCtrl`, 
`$scope.phones = Phone.query();`, we don't have to pass any callback functions when *invoking methods of our Phone service.* This is not a synchronous call however. But what **is** returned synchronously is a **"future"** which is an object that gets filled with data when the XHR response returns. Because of data binding in angular, we can use this future and bind it to our template. Then, when the data arrives, the view will update. 

Sometimes we need to add a callback to do everything we require. As you can see in the `PhoneDetailCtrl`, instead of invoking a method of our Phone service, we're invoking `get` which is provided by `$resource` and supplying a callback to set the `mainImageURL`.

### Testing

1) Update our Karma.config file to include `ngResource`.

##### Updating existing tests

The `$resource` service augments the response object with methods for updating/deleting the resource. Using the standard `toEqual` matcher will cause our tests to fail because the test values would not match exactly. To solve this, we define a `toEqualData` **Jasmine** matcher. It only takes object properties into account and ignores methods. Then we replace every `toEqual` with `toEqualData`.

controllerSpec.js

	describe('Phonecat controllers', function () {

	  beforeEach(function() {
	    this.addMatchers({
	      toEqualData: function(expected) {
	       return angular.equals(this.actual, expected);
	      }
	    });
	  });
	  ...
	  
##### Wrting a simple test for our service

servicesSpec.js

	describe('service', function() {

	  beforeEach(module('phonecatServices'));
	
	  it('check the existence of Phone factory', inject(function(Phone) {
	    expect(Phone).toBeDefined();
	  }));
	});
	
## Animations
###### Setup

1) First we'll need to includ jQuery and `ngAnimate` client side dependencies through bower. Then we need to include these files in the template layout. 
 
index.html

	<head>
	  <!-- for CSS Transitions and/or Keyframe Animations -->
	  <link rel="stylesheet" href="css/animations.css">
	
	  ...
	
	  <!-- jQuery is used for JavaScript animations (include this before angular.js) -->
	  <script src="bower_components/jquery/jquery.js"></script>
	
	  ...
	
	  <!-- required module to enable animation support in AngularJS -->
	  <script src="bower_components/angular-animate/angular-animate.js"></script>
	
	  <!-- for JavaScript Animations -->
	  <script src="js/animations.js"></script>
	
	...
	
	</head>
	
2) Now we create a module in `animations.js` that includes `ngAnimate` as a dependency. We also include this new module in our `phonecatApp` module dependencies. 

### Animating `ngRepeat` 

First we'll add css transition animations to our `ng-repeat` directive in our `phone-list.html`. All that needs to be added in this template is a `.phone-listing` class to our phone list items. 

phone-list.html

	<li ng-repeat="phone in phones | filter:query | orderBy:orderProp" class="tumbnail phone-listing">
	...
    </li>
    
Now we can add animations in our animations.css file. 

animations.css

	.phone-listing.ng-enter,
	.phone-listing.ng-leave,
	.phone-listing.ng-move {
	    -webkit-transition: 0.5s linear all;
	    -moz-transition: 0.5s linear all;
	    -o-transition: 0.5 linear all;
	    transition: 0.5s linear all;
	}
	
	.phone-listing.ng-enter,
	.phone-listing.ng-move {
	    opacity: 0;
	    height: 0;
	    overflow: hidden;
	}
	
	.phone-listing.ng-move.ng-move-active,
	.phone-listing.ng-enter.ng-enter-active {
	    opacity: 1;
	    height: 120px;
	}
	
	.phone-listing.ng-leave {
	    opacity: 1;
	    overflow: hidden;
	}
	
	.phone-listing.ng-leave.ng-leave-active {
	    opacity: 0;
	    height: 0;
	    padding-top: 0;
	    padding-bottom: 0;
	}
	
The `phone-listing` css class is combined with animation hooks that occur when items are inserted into and removed from a list. 

**`ng-enter`:** Applied to the element when a new phone is added to the list and rendered on the page. 

**`ng-move`:** Applied when items are moved around in the list.

**`ng-leave`:** Applied when items are removed from the list on the page. 

The phone listing items are added and removed depending on the data passed to the `ng-repeat` directive. i.e filters or querys. 

When an animation occurs, angular adds two sets of CSS classes to an element. 

1) A "starging" class that represents the style at the beginning of the animation. 
`ng-enter`, `ng-move`, `ng-leave`

2) An "active" class that represents the style at the end of the animation.
`ng-enter-active`, `ng-move-active`, `ng-leave-active`

This allows you to craft an animation from beginning to end. 

The code at the very top of the css file adds fade-in and fade-out animations. The css below that expands elements from 0 to 120 pixels when they are added or moved around and collapses the items before removing them. 

For backwards compatibility with older browsers, you can write javascript-based animations which we'll do later.

### Animating `ng-view`

We'll going to add animations for transitions between route changes in ng-view with css keyframe animations. 

index.html

	<body>
	  <div class="view-container">
	    <div ng-view class="view-frame"></div>
	  </div>
	</body>
	
We've nested `ng-view` inside a parent element `view-container` that is positioned relative so that the postioning of the `ng-view` is relative to this parent as it animates transitions. 

animations.css

	.view-container {
    position: relative;
	}
	
	.view-frame.ng-enter,
	.view-frame.ng-leave {
	    background: white;
	    position: absolute;
	    left: 0;
	    right: 0;
	}
	
	.view-frame.ng-enter {
	    -webkit-animation: 0.5s fade-in;
	    -moz-animation: 0.5s fade-in;
	    -o-animation: 0.5s fade-in;
	    animation: 0.5s fade-in;
	    z-index: 100;
	}
	
	.view-frame.ng-leave {
	    -webkit-animation: 0.5s fade-out;
	    -moz-animation: 0.5s fade-out;
	    -o-animation: 0.5s fade-out;
	    animation: 0.5s fade-out;
	    z-index: 99;
	}
	
	@keyframes fade-in {
	    from { opacity: 0; }
	    to { opacity: 1; }
	}
	
	@-moz-keyframes fade-in {
	    from { opacity: 0; }
	    to { opacity: 1; }
	}
	
	@-webkit-keyframes fade-in {
	    from { opacity: 0; }
	    to { opacity: 1; }
	}
	
	@keyframes fade-out {
	    from { opacity: 1; }
	    to { opacity: 0; }
	}
	
	@-moz-keyframes fade-out {
	    from { opacity: 1; }
	    to { opacity: 0; }
	}
	
	@-webkit-keyframes fade-out {
	    from { opacity: 1; }
	    to { opacity: 0; }
	}
	
What the above CSS does is add a simple fade-in fade-out effet between pages. We're using *absolute positioning* to position the *next* page on top of the *previous* page while performing a cross fade animation in between. As the previous page is jsut about to be removed, it fades out while the new pages fades in right on top of it. The next page is identified by `ng-enter` and the previous page is the one with the `ng-leave` class. Once these animations are complete these classes are removed from the elemetns. 

### Animating `ngClass` with Javascript

We'll now add animation when a user clicks on a thumbnail image in `phone-detail.html`. We'll do this by adding the `.active` class in the HTML to whichever thumbnail gets selected. 

phone-detail.html

We replace:

	<img ng-src="{{mainImageUrl}}" class="phone">
With: 
		
	<div class="phone-images">
	  <img ng-src="{{img}}" class="phone"
	       ng-repeat="img in phone.images"
	       ng-class="{active: mainImageUrl == img}"
	   >
	</div>

We're using the `ng-repeat` directive like we did with the thumbnails to display **all** the profile images as a list. But only when the `mainImageUrl` is the same as the current `img` in the list will we apply the `active` class which will make the image visible. If the `img` does not have the `active` class it'll remain hidden. 

When the `active` class is added to an element, the `active-add` and `active-add-active` classes are added to signal angular to fire an animation. When the `active` class is removed, the `active-remove` and `active-remove-active` classes are added to trigger another animation. 

Before we start writing js, we need to add some css to `app.css` to ensure images are displayed correctly when the page is first loaded. 


animate.js

	var phonecatAnimations = angular.module('phonecatAnimations', ['ngAnimate']);

	phonecatAnimations.animation('.phone', function() {
	  var animateUp = function(element, className, done) {
	    if(className != 'active') {
	      return;
	    }
	
	    element.css({
	      position: 'absolute',
	      top: 500,
	      left: 0,
	      display: 'block'
	    });
	
	    jQuery(element).animate({
	      top: 0
	    }, done);
	
	    return function(cancel) {
	      if(cancel) {
	        element.stop();
	      }
	    };
	  };
	
	  var animateDown = function(element, className, done) {
	    if(className != 'active') {
	      return;
	    }
	
	    element.css({
	      position: 'absolute',
	      left: 0,
	      top: 0
	    });
	
	    jQuery(element).animate({
	      top: -500
	    }, done);
	
	    return function(cancel) {
	      if(cancel) {
	        element.stop();
	      }
	    };
	  };
	
	  return {
	    addClass: animateUp,
	    removeClass: animateDown
	  }
	});
	
First thing to note, in our animations module, we've registered the element that has the `.phone` class, which in this case, is the `img` element in our HTML.

The `addClass` and `removeClass` callback functions are called whenever the `active` class gets added or removed on the `img` element. These callbacks will get fired with `element` passed in as a parameter.

Notice that addClass and removeClass each return a function. This is an optional function that's called when the animation is cancelled (when another animation takes place on the same element) as well as when the animation has completed. A boolean parameter is passed into the function which lets the developer know if the animation was cancelled or not. This function can be used to do any cleanup necessary for when the animation finishes.