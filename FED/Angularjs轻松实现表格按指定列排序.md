---
title: Angularjs轻松实现表格按指定列排序
tags: [javascript,angularjs]
date: 2016/05/18
---

使用Angularjs的过滤器，可以很容易的实现在表格中，点击某一列标题进行排序，实现过程如下：

html代码：

```html
<table class="table table-border" ng-app="myapp" ng-controller="orderByCtrl">
    <thead>
        <tr>
            <th>inx</th>
            <th ng-click="col='name';desc=!desc">name</th>
            <!-- 当点击列标题时，执行click事件，将排序条件反转，即，如果原来是升序则将按降序，降序亦如此 -->
            <th ng-click="col='gender';desc=!desc">gender</th>
            <th ng-click="col='age';desc=!desc">age</th>
            <th ng-click="col='score';desc=!desc">score</th>
        </tr>
    </thead>
    <tbody>
        <tr ng-repeat="d in data|orderBy:col:desc">
            <td ng-bind="$index+1"></td>
            <td ng-bind="d.name"></td>
            <td ng-bind="d.gender"></td>
            <td ng-bind="d.age"></td>
            <td ng-bind="d.score"></td>
        </tr>
    </tbody>
</table>
```

js代码：

```javascript
var app = angular.module('myapp', []);

app.controller('orderByCtrl', function($scope) {
    $scope.col = 'name';//默认按name列排序
    $scope.desc = 0;//默认排序条件升序
    $scope.data = [{
        name: 'name 1',
        gender: 'male',
        age: 26,
        score: 70
    }, {
        name: 'name 2',
        gender: 'female',
        age: 24,
        score: 84
    }, {
        name: 'name 3',
        gender: 'male',
        age: 20,
        score: 76
    }, {
        name: 'name 4',
        gender: 'female',
        age: 22,
        score: 64
    }];


})
```

让运行界面好看些，使用了bootstrap.min.css样式库。为了交互性考虑，在表头增加了手指样式

```css
th {
    cursor: pointer;
}
```

运行结果[点击这里](https://jsfiddle.net/Lionney/xowyoaxj/)查看
