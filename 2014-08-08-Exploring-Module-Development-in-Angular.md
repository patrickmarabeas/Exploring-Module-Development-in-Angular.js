A guest post by Patrick Marabeas, a freelance frontend developer who loves learning and working with cutting edge web technologies. He spends much of his free time developing Angular Modules, such as: [ng-FitText](https://github.com/patrickmarabeas/ng-FitText.js), [ng-Slider](https://github.com/patrickmarabeas/ng-Slider.js), [ng-YouTubeAPI](https://github.com/patrickmarabeas/ng-YouTubeAPI.js). You can follow him on Twitter: [@patrickmarabeas](https://twitter.com/PatrickMarabeas)

# Exploring module development in Angular.js

This started off as an article about building a simple ScrollSpy module. Simplicity got away from me, however, so I'll focus on some of the more interesting bits and pieces that make this module tick! You may wish to have the [completed code](https://github.com/patrickmarabeas/ng-ScrollSpy.js/tree/v3.0.0) with you as you read this to see how it fits together as a whole - as well as the missing code and logic.

Modular applications are those which are "composed of a set of highly decoupled, distinct pieces of functionality stored in modules" (Addy Osmani). By having loose coupling between modules, the application becomes easier to maintain and functionality can be easily swapped in and out.

As such, functionality of our module will be strictly limited to the activation of one element when another is deemed to be viewable by the user. Linking, smooth scrolling and other features navigation elements might have, should be another modules concern.

### Lets build a ScrollSpy module!

Let's start by defining a new module. Using chained sequence rather than declaring a variable for the module is preferable as you don't pollute the global scope, and saves you when other modules have used the same var.

```javascript
'use strict';
angular.module('ngScrollSpy', []);
```

### Structuring the module

I'm all about making modules which are dead simple to implement for the developer. Begone with superfluous parents, attributes and controller requirements! All we need is:

1. a directive (`scrollspyBroadcast`) that sits on each content section and determines if it's been scrolled to (active and added to stack) or not
2. a directive (`scrollspyListen`) that sits on each navigation (or whatever) element and listens for changes to the stack - triggering a class if it is the current active element

We'll use a factory (`SpyFactory`) to deal with the stack (adding to, removing from, and broadcasting change).

The major issue with a ScrollSpy module (particularly in Angular) is dynamic content. We could use MutationObservers - but they aren't [widely supported](http://caniuse.com/mutationobserver) and polling is just bad form. Let's just leverage scrolling itself to update element positions. We could also take advantage of `$rootScope.$watch` to watch for any digest calls received by `$rootScope` but it hasn't been included in the version this article will link to.

To save every single `scrollspyBroadcast` directive from calculating `documentHeight` and `window` positions/heights another factory (`PositionFactory`) will deal with these changes. This will be done via a scroll event in a `run` block.

This is a basic visualisation of how our module is going to interact:

![module structure](/images/scrollspy-structure.png)

### Adding module wide configuration

By using `value`, `provider` and `config` blocks, module wide configuration can be implemented without littering our view with data attributes, having a superfluous parent wrapper or the developer needing to alter the module file.

The `value` block acts as the default configuration for the module.

```javascript
.value('config', {
    'offset': 200,
    'throttle': true,
    'delay': 100
  })
```

The `provider` block allows us to expose API for application-wide configuration. Here we are exposing config, which the developer will be able to set in the `config` block.

```javascript
.provider('scrollspyConfig', function() {
  var self = this;
  this.config = {};
  this.$get = function() {
    var extend = {};
    extend.config = self.config;
    return extend;
  };
  return this;
});
```

The user of the ScrollSpy module can now implement a `config` block in their application. The scrollspyConfig `provider` is injected into it (NOTE: the injected name requires "Provider" on the end) - giving the user access to manipulate the modules configuration from their own codebase.

```javascript
theDevelopersFancyApp.config(['scrollspyConfigProvider', function(scrollspyConfigProvider) {
    scrollspyConfigProvider.config = {
        offset: 500,
        throttle: false,
        delay: 100
    };
}]);
```

The `value` and `provider` blocks are injected into the necessary directive - `config` being extended upon by the application settings (`scrollspyConfig.config`).

```javascript
.directive('scrollspyBroadcast', ['config', 'scrollspyConfig', function(config, scrollspyConfig) {
    return {
      link: function() {
        angular.extend(config, scrollspyConfig.config);

        console.log(config.offset) //500

        ...
```

### Updating module wide properties

It wouldn't be efficient for all directives to calculate generic values such as the document height and position of the window. We can put this functionality into a service, inject it into a `run` block and have it call for updates upon scroll.

```javascript
  .run(['PositionFactory', function(PositionFactory) {
    PositionFactory.refreshPositions();
    angular.element(window).bind('scroll', function() {
      PositionFactory.refreshPositions();
    });
  }])

  .factory('PositionFactory', [ function(){
    return {
      'position': [],
      'refreshPositions': function() {
        this.position.documentHeight = //logic
        this.position.windowTop = //logic
        this.position.windowBottom = //logic
      }
    }
  }])
```

`PositionFactory` can now be injected into the required directive.

```javascript
.directive('scrollspyBroadcast', ['config', 'scrollspyConfig', 'PositionFactory', function(config, scrollspyConfig, PositionFactory) {
  return {
    link: function() {

      console.log(PositionFactory.documentHeight); //1337

    ...
```

### Using original element types

```html
<a data-scrollspyListen>Some text!</a>
<span data-scrollspyListen>Some text!</span>
<li data-scrollspyListen>Some text!</li>
<h1 data-scrollspyListen>Some text!</h1>
```

These should all be valid. The developer shouldn't be forced a specific element when using the `scrollspyListen` directive. Nor should the view fill with superfluous wrappers to allow the developer to retain their original elements. Fortunately, the `template` property can take a function (which takes two arguments `tElement` and `tAttrs`). This gives access to the element prior to replacement. In this example, transclusion could also be replaced by using `element[0].innerText` instead. This would remove the added child span that gets created.

```javascript
.directive('scrollspyListen', ['$timeout', 'SpyFactory', function($timeout, SpyFactory) {
  return {
    replace: true,
    transclude: true,
    template: function(element) {
      var tag = element[0].nodeName;
      return '<' + tag + ' data-ng-transclude></' + tag + '>';
    },

    ...
```

### Show me all of it!

This is just a taste of the inter-module functionality that you can leverage when building your components.

The completed codebase can be found [over on GitHub](https://github.com/patrickmarabeas/ng-ScrollSpy.js). Version  at time or writing [v3.0.0](https://github.com/patrickmarabeas/ng-ScrollSpy.js/tree/v3.0.0)