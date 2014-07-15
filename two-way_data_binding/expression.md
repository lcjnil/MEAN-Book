# Expression

ng has it's own expression, you could use it with **two-curly**
```
{{ your expression}}
```
You could use **ng-bind** directive also
```
<span ng-bind="your expression"></span>
```

Pro is that when angularjs is loading, you won't see awkward {{you expression}} but none.

And expression follows some rules:

- you can't use if/for in expression
- it's less strict than js, won't throw exception
