# Dependency Injection
You have already see DI before.
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

You might be puzzled that, you use **$scope** service in Ctrl controller, and you use **ng-controller** directive to set div's controller. But how could you use name without any other opertion.

The answer is **DI**, DI help $scope to be injected without any other opertion when you use it. That is to say, the $scope is injected to div.

This is the first look of DI, I will discussed it more.
