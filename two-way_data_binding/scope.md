# Scope

**Scope** is where an variable could access. And ng's controller set the scope.
We will see a more complex example.

```
<body ng-app>
<div ng-controller="Ctrl">
  <ul>
    <li ng-repeat="person in persons">{{person.name}} {{person.age}}</li>
  </ul>
</div>

<script>
  Ctrl = function($scope) {
    $scope.persons = [
      {name: 'NiL', age: 19},
      {name: 'ZD', age: 18},
      {name: 'Dian', age: 21}
    ];
  }
</script>
</body>
```
You could see, controller Ctal's one argument is **$scope**, the name is special in ng for prefix dollar, that means service in ng.

So, $scope really means the scope of Ctrl.

More, li's **ng-repeat** directive, create 3 element in DOM, and create 3 sub-scope too. You could easily find these with a chrome extension Batarang.

In fact, ng's scope is more like a DOM tree. It started with the $rootScope created by **ng-app**. And all the controller's scope is the child of $rootScope. It's an **inheritance structure** in JS. When it reads an variable, it will try to find it from bottom to top.
