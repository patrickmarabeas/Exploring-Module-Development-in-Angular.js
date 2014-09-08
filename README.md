# Exploring Module Development in Angular.js

This started off as an article about building a simple ScrollSpy module. Simplicity got away from me, however, so I'll focus on some of the more interesting bits and pieces that make this module tick! You may wish to have the [completed code](https://github.com/patrickmarabeas/ng-ScrollSpy.js/tree/v3.0.0) with you as you read this to see how it fits together as a whole - as well as the missing code and logic.

Modular applications are those which are "composed of a set of highly decoupled, distinct pieces of functionality stored in modules" (Addy Osmani). By having loose coupling between modules, the application becomes easier to maintain and functionality can be easily swapped in and out.

As such, functionality of our module will be strictly limited to the activation of one element when another is deemed to be viewable by the user. Linking, smooth scrolling and other features navigation elements might have, should be another modules concern.