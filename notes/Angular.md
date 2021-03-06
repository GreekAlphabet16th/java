# AngularJS

## AngularJS中的MVC体系

- model层


```
数据层，为当前视图中可用的数据。即$scope，是一个带有属性、方法的对象。
```
- controller层

```
通常指angular.module('app').contrller('Ctrl',Ctrl);
对$scope进行一些属性、方法的处理并绑定到视图层。
```
- view层

```
通常指前端html页面，controller将model层$scope的数据绑定到html中。
```

### 1.$scope的作用范围

- $scope

```
scope 是一个 JavaScript对象，带有属性和方法，这些属性和方法可以在视图和控制器中使用。作用范围在其对应的controller中
```
- $rootscope

```
$rootScope 可作用于整个应用中。是各个 controller 中 scope 的桥梁。用 rootscope 定义的值，可以在各个 controller 中使用。
```


### 2.ctronller之间的通信

```
Angularjs为在scope中为我们提供了冒泡和隧道机制，$broadcast会把事件广播给所有子controller，而$emit则会将事件冒泡传递给父controller，$on则是angularjs的事件注册函数，有了这一些我们就能很快的以angularjs的方式去解决angularjs controller之间的通信
```

- view

```
<div ng-app="app" ng-controller="parentCtr">
    <div ng-controller="childCtr1">name :
         <input ng-model="name" type="text" ng-change="change(name);" />
     </div>
     <div ng-controller="childCtr2">Ctr1 name:
     <input ng-model="ctr1Name" />
    </div>
</div>
```

- ctroller

> 注意父controller在广播时候一定要改变事件name。

```
angular.module("app", []).controller("parentCtr",
function ($scope) {
    $scope.$on("Ctr1NameChange",
    function (event, msg) {
        console.log("parent", msg);
        $scope.$broadcast("Ctr1NameChangeFromParrent", msg);//递归
    });
}).controller("childCtr1", function ($scope) {
    $scope.change = function (name) {
        console.log("childCtr1", name);
        $scope.$emit("Ctr1NameChange", name);//冒泡
    };
}).controller("childCtr2", function ($scope) {
    $scope.$on("Ctr1NameChangeFromParrent",
 
    function (event, msg) {
        console.log("childCtr2", msg);
        $scope.ctr1Name = msg;
    });
});
```

### 3.service、factory、provider的区别

- factory

> factory也就是设计模式中的工厂方法，即提供一个方法，该方法返回一个对象实例。

```
var app = angular.module('app', []);
app.factory('MyFactory', function() {
    var result = {};
    result.greeting = 'Hello from factory';
    return result;
//contrller拿到的相当于result对象，即：
var factoryResult = MyFactory();
```
- service

> service相当于构造函数，通过new实例化。

```
app.service('MyService', function() {
    this.greeting = 'Hello from service';
});
//contrller拿到的为this指向的对象，即：
var serviceObj = new MyService();
```
- provider

> provider必须提供一个$get方法，返回一个对象。

```
app.provider('MyProvider', function() {
    this.$get = function() {
        var result = {};
        result.greeting = 'Hello from provider';
        return result;
    }
});
//controller拿到的为$get方法返回的对象
var provider = new MyProvider().$get();
```
- factory、service、provider使用

```
app.controller('TestController',TestController);
app.$inject = ['$scope', 'MyFactory', 'MyService', 'MyProvider'];
function TestController($scope, myFactory, myService, myProvider){
    $scope.greetingFromFactory = myFactory.greeting;
    $scope.greetingFromService = myService.greeting;
    $scope.greetingFromProvider = myProvider.greeting;
}
```
- provider可以在应用启动时进行配置

> provider可以在module启动时进行配置，完成一些初始化工作。


```
app.provider('MyProvider', function() {
    var defaultName = 'anonymous';
    this.setName = function(newName) {
        defaultName = newName;
    }
    this.$get = function() {
        var result = {};
        result.defaultName = defaultName;
        return result;
    }
});

app.config(function(MyProvider) {
    MyProvider.setName('Angularjs Provider');
});
```
### 4.异步执行$q

> Promise是一种模式，以同步操作的流程形式来操作异步事件，避免了层层嵌套，可以链式操作异步事件。[异步执行$q的解释](https://www.cnblogs.com/ZengYunChun/p/6438330.html)