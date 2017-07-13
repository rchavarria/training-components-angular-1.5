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
