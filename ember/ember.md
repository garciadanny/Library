# Ember.js

Notes taken while going through the [ember.js tutorial](http://emberjs.com/guides/getting-started/)

## Routing

When a router isn't specified to map a url to a template, ember will create a default `IndexRoute` with a RouteName of `index` that maps to `/`.

#### Custom Routing

When you create a router that maps a url to a template, but you don't create a route for it, ember will create a default route for it using the ember [naming convention](http://emberjs.com/guides/concepts/naming-conventions/). 

Example: 

If you create a router that maps `/` to a `todos` template, ember will create a `TodosRoute` and name the route `todos`. (This isn't important now but it'll also create a default `TodosController`) 

	Todos.Router.map(function() {
	  	this.resource('todos', {path: '/'});
	});

You can use this `TodosRoute` to add custom behavior. 

	Todos.TodosRoute = Ember.Route.extend({
	  model: function() {
	    return this.store.find('todo');
	  }
	});
	
Route Name | Route | Controller | Template | URL
-----------|-------|------------|----------|----
todos  | TodosRoute | TodosController | todos | #/


## Models

	Todos.TodosRoute = Ember.Route.extend({
	  model: function() {
	    return this.store.find('todo');
	  }
	});
	
In the custom route above, we're specifying that the model associated with this `route` is `todo`. Based on the ember naming conventions, ember will look for a model named `Todo`.

	// Model
	Todos.Todo = DS.Model.extend({
	  title: DS.attr('string'),
	  isCompleted: DS.attr('boolean')
	});
	
`this` is the route object and the `store` is whatever we specified to be the data source. In this case we specified the data source to be fixtures. 

app.js

	Todos.ApplicationAdapter = DS.FixtureAdapter.extend();

Ember will automatically look in the `Todo` model for a FIXTURES constant. 

	Todos.Todo.FIXTURES = [
	  {
	    id: 1,
	    title: 'Learn Ember.js',
	    isCompleted: true
	  },
	  ...
	];
	
## Dynamically adding classes to the DOM (Helpers)

	<li {{bind-attr class="todo.isCompleted:completed"}}>
	  <input type="checkbox" class="toggle">
	  <label>{{todo.title}}</label><button class="destroy"></button>
	</li>
	
`{{ bind-attr ... }}` allows us to write a CSS class of "completed" if `todo.isComplted` returns `true`.

[bind-attr](http://emberjs.com/api/classes/Ember.Handlebars.helpers.html#method_bind-attr)

[bind and bind-attr blog post](http://www.emberist.com/2012/04/06/bind-and-bindattr.html)

## 2 Way Data Binding

Continuing with the example above, we can use 2 way data binding to create a new model instance (a new todo list item). We'll use ember's `{{input}}` helper.

index.html

	<h1>todos</h1>
	{{input
	  type="text"
	  id="new-todo"
	  placeholder="What needs to be done?"
	  value=newTitle
	  action="createTodo"
	}}
	
This `{{input}}` helper will connect the `value` attribute of the `<input>` with the `newTitle` property on the template's controller (TodosController). Adding 2 way data binding functionality. 
Note: `newTitle` should not be a string in the input field. 

It will also connect a user interaction (pressing the `<enter>` key) to a method in the TodosController called `createTodo` via `action="createTodo"`.

i.e.

	Todos.TodosController = Ember.ArrayController.extend({
	  actions: {
	    createTodo: function() {
	      // Get the todo title set by the "New Todo" text field
	      var title = this.get('newTitle');
	      // If the title is an empty string, simply return.
	      if (!title.trim()) { return; }
	
	      var todo = this.store.createRecord('todo', {
	        title: title,
	        isCompleted: false
	      });
	
	      // Clear the new todo text field
	      this.set('newTitle', '');
	
	      // Save the model
	      todo.save();
	    }
	  }
	});
	
## Giving Each Model It's Own Controller To Add Behavior

Here we're going to add functionality to be able to mark a todo item as complete/incomplete by wrapping each todo item in it's own controller (we'll call it `TodoController`). All we need to do is add an `itemController` argument in the existing `{{each}}` handlebars helper.

	{{#each todo in model itemController="todo"}}
	  <li {{bind-attr class="todo.isCompleted:completed"}}>
	    {{input type="checkbox" checked=todo.isCompleted class="toggle"}}
	    <label>{{todo.title}}</label><button class="destroy"></button>
	  </li>
	{{/each}}
	
This will use 2 way data binding and update the `isCompleted` property when it is changed on either end. 

todo_controller.js
	
	// Notice this is a ObjectController, where our TodosController is a ArrayController 
	
	Todos.TodoController = Ember.ObjectController.extend({
	  isCompleted: function(key, value) {
	    var model = this.get('model');
	
	    if (value === undefined) {
	      // if the checkbox hasn't been clicked
	      // i.e. if there is no value
	      return model.get('isCompleted');
	    }
	    else {
	      model.set('isCompleted', value);
	      model.save();
	      return value;
	    }
	  }.property('model.isCompleted')
	});
	
Note: The `isCompleted` function is marked a computed property whose value is **dependent** on the value of `model.isCompleted`.

##### [Computed Properties](http://emberjs.com/guides/object-model/computed-properties/)
Computed properties let you declare functions as properties. You create one by defining a computed property as a function, which Ember will automatically call when you ask for the property. You can then use it the same way you would any normal, static property. It's super handy for taking one or more normal properties and transforming or manipulating their data to create a new value.

	
[ObjectController](http://emberjs.com/api/classes/Ember.ObjectController.html)

## Adding behavior to an Array Controller
We currently have the count of the remaining todos left to be completed hard coded. We will now dynamically load the remaining count. 

index.html
	
	<span id="todo-count">
	  <strong>{{remaining}}</strong> {{inflection}} left
	</span>

todos_controller.js

	actions: {
	  // ... additional lines truncated for brevity ...
	},
	
	remaining: function() {
	  return this.filterBy('isCompleted', false).get('length');
	}.property('@each.isCompleted'),
	
	inflection: function() {
	  var remaining = this.get('remaining');
	  return remaining === 1 ? 'item' : 'items';
	}.property('remaining')

We have created 2 computed properties `remaining` and `inflection`. `remaining` is dependent on the `isCompleted` property of each model. This enables 2 way data binding. If the isCompleted value of any todo changes, this property will be recomputed. If the value has changed, the section of the template displaying the count will be automatically updated to reflect the new value.

`inflection` is dependent on the `remainging` computed property. The section of the template displaying the count will be automatically updated to reflect the new value.

## Toggling between states on the app (changing the DOM)

We'll add some functionality that will display an `<input>` element when a user double clicks on a todo item to edit it, otherwise, it'll just display the todo item. 

index.html

	{{#each todo in model itemController="todo"}}
	  <li {{bind-attr class="todo.isCompleted:completed todo.isEditing:editing"}}>
	    {{#if todo.isEditing}}
	      <input class="edit">
	    {{else}}
	      {{input type="checkbox" checked=todo.isCompleted class="toggle"}}
	      <label {{action "editTodo" on="doubleClick"}}>{{todo.title}}</label><button class="destroy"></button>
	    {{/if}}
	  </li>
	{{/each}}

1) Here we've added the CSS helper that adds the `editing` class to the `<li>` if `isEditing` is set to true.

	<li {{bind-attr class="todo.isCompleted:completed todo.isEditing:editing"}}>
	
2) We're adding an `if/else` statement that displays the `<input>` field is `isEditing` is set to true, otherwise, it displays the todo `<li>`.

3) We're adding a custom `action` and calling it `editTodo` that gets triggered when a user doubles clicks on a todo `<li>` via `on="doubleClick"`. See [Template Action Helpers](http://emberjs.com/guides/templates/actions/)

todo_controller.js

	Todos.TodoController = Ember.ObjectController.extend({
	  actions: {
	    editTodo: function() {
	      this.set('isEditing', true);
	    }
	  },
	
	  isEditing: false,
	
	  isCompleted: function(key, value) {
		  // ... additional lines truncated for brevity ...
	  }.property('model.isCompleted')
	});

1) We add the `action` helper we added in step 3 above via the `actions` property. This simply sets the `isEditing` property to true when a user double clicks on a todo item. 

2) We add an `isEditing` property that defaults to `false`.

## Ember views, custom view helpers, hooks on view elements

##### Saving changes and toggling between states

When a user double-clicks on a todo item to edit it, we want to: 

1) Focus in on the `<input>` element

2) Allows a user to edit the item.

3) When the user hits `<enter`, save the todo, and redisplay the todo item. 

4) When the user focuses out by clicking somewhere else, save the todo, and redisplay the todo item.  


###### 1) Create a custom **component** and register it with handlebars to make it available in the template

edit_todo_view.js

	Todos.EditTodoView = Ember.TextField.extend({
	  didInsertElement: function() {
	    this.$().focus();
	  }
	});
	
	Ember.Handlebars.helper('edit-todo', Todos.EditTodoView);
	
This creates an `<input>` textfield, and adds an event handler to it that focuses in on the element when it appears on the DOM (when the user double-clicks to edit). It also registers it as a helper so you can use it in your templates. 

###### 2) Use `edit-todo` helper in the view. 

index.html

	{{#if todo.isEditing}}
	  {{edit-todo class="edit" value=todo.title focus-out="acceptChanges"
	                           insert-newline="acceptChanges"}}
	{{else}}
	...
	
1) `edit-todo` helper creates an `<input>` element on the DOM with an event handler to focus in on it.

2) We set the `value`of the `<input>` field to the title of the current todo item.

3) We use the `focus-out` and `insert-newline` hooks to call the `acceptChanges` **action** when a user mouses away or presses `<enter>`. See [Ember.TextSupport](http://emberjs.com/api/classes/Ember.TextSupport.html#property_action)

todo_controller.js
	
	actions: {
	  acceptChanges: function() {
	    this.set('isEditing', false);
	
	    if (Ember.isEmpty(this.get('model.title'))) {
	      this.send('removeTodo');
	    } else {
	      this.get('model').save();
	    }
	  },
	  removeTodo: function () {
	    var todo = this.get('model');
	    todo.deleteRecord();
	    todo.save();
	  }
	},
	
1) Set `isEditing` to false to change the view to show the todo item.
2) If the todo is empty, menaing the user deleted all of the text, assume they want to delete the todo, and delete it for them.
3) Otherwise, simply save the updated todo item.

Note: The current action may be invoked twice (via acceptChanges) leading to an exception. For details on how to handle this situation, please see [the latest version of the todo demo source](https://github.com/tastejs/todomvc/blob/gh-pages/examples/emberjs/js/controllers/todo_controller.js).

## Reusing custom actions

We can use the `removeTodo` action we created above to delete todos when a user clicks on the "x" delete button. 

index.html

	<button {{action "removeTodo"}} class="destroy"></button>
	
## Child Routes (Nested Templates)

Remember we created a router that maps "/" to a `todos` template and automatically created a `TodosRoute` for us. 

Technicaly we created a `TodosRoute` and ember automatically looks for a template named `todos`.

	Todos.Router.map(function() {
	  	this.resource('todos', {path: '/'});
	});

	
Route Name | Route | Controller | Template | URL
-----------|-------|------------|----------|----
todos  | TodosRoute | TodosController | todos | #/

We can nest routes under the `TodosRoute` by adding a function to the `todos` resource:

router.js

	Todos.Router.map(function() {
	  	this.resource('todos', {path: '/'}, function(){
	  		// nested routes go here
	  	});
	});
	
Adding this empty function automatically creates 3 nested routes by default, one of which is `todos.index`:



Route Name |Nested Route Name| Route | Controller | Template | URL
-----------|---------|------------|----------|----
todos  |   |TodosRoute|TodosController| todos | 
------------->|todos.index|TodosIndexRoute|TodosIndexController|todos/index|#/

With this knowledge, we can extract our `<ul>` of todo items into it's own template. Looking at the table above, we see that we need to name this template `todos/index` (if we want this `<ul>` to be associated with our `index` route).

index.html

	<body>
	<script type="text/x-handlebars" data-template-name="todos/index">
	  <ul id="todo-list">
	    .
	    .
	    .
	  </ul>
	</script>
	<!--- ... additional lines truncated for brevity ... --> 
	
Now in our `todos` template, we can replace the `<ul>` with a Handlebars `{{outlet}}` helper. 

index.html

	<section id="main">
	  {{outlet}}
	
	  <input type="checkbox" id="toggle-all">
	</section>

The `{{outlet}}` helper will dynamically update as the routes change. As you can see in the table above, the `/` path is now associated with the `todos/index` template so in the browser, `/` will display our `<ul>` injected into the `todos` template. 

Next we can explicitly specify that the model for this nested route `TodosIndexRoute` is the same as the model for our `TodosRoute`; though this happens by default. 

router.js

	Todos.TodosIndexRoute = Ember.Route.extend({
	  model: function() {
	    return this.modelFor('todos');
	  }
	});
	
Notice this is the plural version of todo; Indicating this uses the same model as the TodosRoute.

# Another example of nested routes

Here we'll look into anothe example of nested routes by allowing a user to click on a link to show only the incomplete (Active) todos.

Ember provides us with a `link-to` helper that specifies a route as well as any additional behavior. 

index.html

	 {{#link-to "todos.active" activeClass="selected"}}Active{{/link-to}}

In this link-to we're specifying the `active` route which is nested within `todos`.
We're also using the built-in `activeClass` property which adds a class of `selected` if the link is clicked (active).

Next we create the `active` child route. 

router.js

	Todos.Router.map(function() {
	  this.resource('todos', { path: '/' }, function() {
	    // additional child routes
	    this.route('active');
	  });
	});

**Within** the `todos` parent route, this creates the following child route: 

Route Name | Route | Controller | Template | URL
-----------|-------|------------|----------|----
todos.active| TodosActiveRoute | TodosActiveController| todos/active | #/active

Next, we'll want to set the model data for this route (`TodosActiveRoute`) to be only the incomplete todos. Also, as you notice above, the template for the `active` route is `todos/active` which would change the template within `{{outlet}}`. But we'd like to continue to use the `todos/index` template which contains the markup for the todo list. We can continue to use the `todos/index` template by using the `renderTemplate` method within our `TodosActiveRoute`.


	Todos.TodosActiveRoute = Ember.Route.extend({
	  model: function(){
	    return this.store.filter('todo', function(todo) {
	      return !todo.get('isCompleted');
	    });
	  },
	  renderTemplate: function(controller) {
	    this.render('todos/index', {controller: controller});
	  }
	});