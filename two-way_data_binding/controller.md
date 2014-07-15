# Controller

The **controller** in **MVC** means a part to prepare data for model.

And in ng, it mark out the **scope** of variable(function is also variable in JS too).

```
<body ng-app>
<div ng-controller="Ctrl">
  <input type="text" ng-model="name"/>
  <p>Hello, {{name}}</p>
</div>
<script>
  Ctrl = function ($scope) {
    $scope.name = 'World!';
  }
</script>
</body>
```

**ng-controller** set the controller for the DOM.
And run it you will see, the name has an initial value.

Image that the initial value is not a constant but the reponse you get from a server. Maybe you could understand how data binding powerful is.
