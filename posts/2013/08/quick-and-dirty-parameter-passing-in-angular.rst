.. link: 
.. description: 
.. tags: code, javascript, angularjs
.. date: 2013-08-09 21:27:00
.. slug: quick-and-dirty-parameter-passing-in-angular
.. title: Quick and dirty parameter passing in Angular
.. description: In which I discover a nice hack for passing information to controllers instatiated by repeaters in AngularJS.
.. comments: true

.. role:: javascript(code)
    :language: javascript

**Don't use this code, instead, read this post and then read** |link|_

.. _link: ../09/quick-and-dirty-parameter-passing-in-angular-revisited.html

.. |link| replace:: **the follow-up.**

Since my last post I've switched teams at work after completing a massive redesign of all of my old team's webapps. The overhaul saw the unification of the look and feel of the product's core webapps, with a significant amount of instrumentation and modularity to enable the less aesthetically-inclined people within the team to contribute without reducing the overall coherence of the application.

It just so happens that I joined my new team just as they were planning a massive redesign of their frontend! After assessing our users' needs and desires, we set out to redesign our stack. I'm not going to go into the full details of the transition in this post, but we settled on a Python stack running AngularJS, Flask, uWSGI and nginx (a huge improvement, development-wise, from our current JQueryUI, TurboGears, mod_wsgi and Apache system).

Prior to this redesign effort, my only experiences with client side MVC frameworks were with KnockoutJS, a great JavaScript library for manging databinding. However, when you scaled it up to include things like validation, complex objects, RESTful behavior against remote resources, and DOM manipulation, it really started to show its seams.  Part of what attracted me to Angular was its similar approach to data binding, but with a stricter separation of concerns between the controller and view layers.

I could continue to wax poetic about our decision, but I'll save that for another post. I just wanted to provide some backstory for this post.  

Just the other day I found myself writing a controller that instantiates new child controllers via a repeater...

.. code:: html
  
    <div data-ng-controller="ParentController">
        <div data-ng-repeat="childGuid in childrenGuids" data-ng-controller="ChildController" />
    </div>
    
This worked fine. Pushing a new item to ParentController's :javascript:`$scope.children` array would generate a new ChildController.  However, I soon discovered that what I was attempting to engineer would require each ChildController to have a unique ID, `generated by the ParentController prior to the child's instantiation.`  I attempted to use templating to my advantage in various ways, for example:

.. code:: html
  
    <div data-ng-controller="ParentController">
        <div data-ng-repeat="childGuid in childrenGuids" data-ng-controller="ChildController">
            <input type="hidden" value="{{childGuid}}" />
        </div>
    </div>

No dice.  Why not try broadcasting from the parent scope after I push something to the stack? I feared race conditions.  Why not a directive?  It felt like too much code for something that should be simple.  After reading the relevant portions of the Angular docs several times over, I threw together a nice little hack that got the job done.

.. code:: html
  
    <div data-ng-controller="ParentController">
        <div data-ng-repeat="childGuid in childrenGuids" data-ng-controller="ChildController" data-id="{{childGuid}}"/>
    </div>

.. code:: javascript

    var ParentController = function() {
        $scope.childrenGuids = [];
        
        $scope.addChild = function() {
            var guid = newGuid();
            $scope.childrenGuids.push(guid);
        };
    };
    
    var ChildController = function() {
        $scope.id = '';
        
        var init = function() {
            //finish initialization
        };
        
        $attrs.$observe('id', function(value) {
            if (!$scope.id) { //defensive sanity check
                $scope.id = value;
                init();
            }
        });
    };
    
Essentially, what I'm doing is passing the ID in via the controller declaration. However, because of the order in which Angular digests the markup, interpolation of templated attributes happens after the construction of the controller. Because of that, I had to instruct the framework to do some extra processing when the interpolation event fired and the :code:`data-id` attribute was updated.  After that the controller is free to finish its initialization phase.

Happy with the outcome, I pushed my changes out to my team and made a note to revisit this block when I get around to refactoring this section (it'll happen, don't worry).

EDIT: `It happened <../09/quick-and-dirty-parameter-passing-in-angular-revisited.html>`_.
