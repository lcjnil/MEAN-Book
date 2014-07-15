# Two-way Data Binding

The most facinating in **ng** must be Two-way data binding.

Two way means data binding can happens from **view** to **model** as well as **model** to **view** (You might need to know what is **MVC** first).

And what surprises me is that, this data binding happen without any manual operation. The ng's **Dependency Injection** property will help you complete all the things. Surely, I will interpret **DI** sooner.

Maybe we should see an example first:

```
<body ng-app>
<div>
  <input type="text" ng-model="name"/>
  <p>Hello, {{name}}</p>
</div>
</body>
```

I omit the procedure of inport the ng js. And you can see whenever the input changes, the p tag will change immediately.

**ng-app** **directive** set the scope of DOM, where ng should take over(this scope is different from $scope).

**ng-model** directive set the model identity.

**two-curly** represents that it's an **ng expression**, which should be calculated and replaced, it's no longer string at all.
