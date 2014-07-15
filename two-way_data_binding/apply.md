# $digest and dirty check
ng uses **dirty check** for data binding. When you emit some ng defined event, it will check for all the binding-variable, and make it change.

But it you don't emit these event, the check will never happens. Then you should use manual check, that comes to **$digest** service(or method).

However it's better to call $apply, $apply will call $digest after a funcion is worked.

I will supplement this part later($watch is on the way).
