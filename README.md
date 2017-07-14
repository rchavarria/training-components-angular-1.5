# training-components-angular-1.5

Notes taken during course [Building components with Angular 1.5](https://app.pluralsight.com/library/courses/building-components-angular-1-5/table-of-contents)

# Table of contents

1. Programming components
2. Routing with components
3. Composition with components

## 1. Programming components

### Introduction

They mix angular.controller() and angular.directive() into a single concept, angular.component()

### Our first component

They look a lot like directives

```
module.component('movieList', {
  templateUrl: '...',
  controller: function () {
    this.message = '..';
  }
}
```

### Component controllers

Instead of create controllers and directives, it's recommended to create components. They look very similar to directives. They use the *controller as* syntax.

New lifecycle hooks: $onInit, $onDestroy, $onChanges, $postLink

### Using $http in $onInit

You can use $onInit hook to initialize your controller, without caring of having problems
when instanciating the controller. Much easy to test.

### Components versus Directives

Components are not as powerful as directives.

But they join controllers and directives.

Components are used for elements (they can't be used for attribues) and their controllers
are always isolated.

### Building a rating component

This component will be used inside of the `movie-list` component, that has a list of movies.
So it needs to know about a `movie`.

With directives, it can be done with an isolated scope and `@` or `=` or `&`...

With components, it's done with **bindings**

Defining the component:

```
angular.module('movieRating', {
  templateUrl: ...
  bindings: {
    // '<' tells that an outside component will inject this value into `rating` in this
    // component's controller
    rating: '<',
  }
});
```

Then, in the HTML:

```
movie comes from them movie-list component
<movie-rating rating="movie.rating"></movie-rating>
```

`$onChanges` is called when a value from outside changes. In this example, when `movie.rating`
changes, `$onChanges` in `movieRating` component will be called.

### Testing a component

Quite similar to controllers, but you use the service `$componentController` instead of
`$controller`:

```
var scope = $rootScope.$new();

// before
$controller('my-controller', { $scope: scope });

// with componentas
$componentController('my-component', { $scope: scope });
```

## 2. Routing with components

*Routing* is the ability to change the UI in response to a change in the URL

### Routing with the original router

`ng-view` loads the right component, depending on the router configuration.

A simple configuration:

```
angular.module('...').config(function ($routeProvider) {
  $routeProvider
    .when('/list', { template: '<movie-list></movie-list>' })
    .when('/about', { template: '<app-about></app-about>' })
    .otherwise({ redirectTo: '/list' });
});
```

### Introducing the Router Component

You need to install the router component. It's a `npm` dependency (in those days it was
`angular-component-router`).

There is no `$routeProvider` configuration.

There is no more `ng-view` directives.

We need to create a component, that will handle all pages, all routes.

We need to register a service with a specific name to tell the router what component is
our router component, the one that will handle routes. This component MUST be the root
HTML element of our app.

`movie-app` will be our router:

```
angular.module('...').value('$routerRootComponent', 'movieApp');
```

In the template of our router component, the *views* (the other components) will be
placed where `ng-outlet` directive is. I mean, `ng-outlet` is similar to `ng-view`
with the old router.

How to configure the router component:

```
angular.module('...').component('movieApp', {
  templateUrl: '...',
  $routeConfig: [
    { path: '/list', component: 'movieList', name: 'List' },
    { path: '/about', component: 'appAbout', name: 'About' },
    { path: '/**', redirectTo: [ 'List' ] },
  ]
});
```

Note: quite similar to the configuration of the old one.

### Create links

Instead of hardcode links to pages in our html, we can use `ng-link` directive, to point
to a route (you provide a `name` in the configuration):

```
// before
<a href="#/list">List</a>

// now
<a ng-link="['List']">List</a>
```

### Using route parameters

Used for example, to see a detail of an item. So, you'll need the item id or some parameter.

Router configuration for this:

```
{ path: '/detail/:id', component: 'movieDetails', name: 'Details' },
```

In HTML, we'll use the `ng-link` directive to generate the link to that detail, passing a
parameter:

```
<a ng-link="['Details', {id: movie.id}]"></a>
```

It's done in a component that has access to the list of movies, and it has a `movie` object.

### Component router lifecycle hooks

Hooks:

- `$canActivate`
- `$routerOnActivate`
- `$routerCanDeactivate`
- `$routerOnDeactivate`
- `$routerCanReuse`

How are they used in my component:

```
angular.module('...').component('movieDetails', {
  templateUrl: '...',
  $canActivate: function () {
    // this hook must be used in the component configuration
  },
  controller: function () {
    var model = this;
    
    // this and the rest of hooks are methods in the controller
    model.$routerOnActivate = function (nextRoute, previousRouter) {
      // this hook is called with two paramesters, next and previous routes, that contain
      // the routes parameters and other properties
      assert nextRoute.params === { id: XXX }; 
    };
  }
});
```

### Navigating programmatically

You hava a button, with a `ng-click` method. When you do a click, you'll be directed to a
route.

For your component, you'll need to create a new binding:

```
angular.module('...').component('movieDetails', {
  templateUrl: '...',
  controller: function () {},
  bindings: {
    '$router': '<',  // something external to be a property of my controller
  }
});
```

In the controller:

```
model.goTo = function (movieId) {
  // basically the same as `ng-link`
  model.$router.navigateTo([ 'Details', {id: movieId} ]);
};
```

## 3. Composition with components

Last module
